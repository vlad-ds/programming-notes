# MongoDB

Database --> Collecctions --> Documents -> Fields --> Subdocuments.

```python
from pymongo import MongoClient
#client is a dictionary of databases
db = client['nobel']
#db is a dictionary of collections
prizes_collection = db["prizes"]
```

```python
# Save a list of names of the databases managed by client
db_names = client.list_database_names()
print(db_names)

# Save a list of names of the collections managed by the "nobel" database
nobel_coll_names = client.nobel.list_collection_names()
print(nobel_coll_names)
```

```python
# Connect to the "nobel" database
db = client.nobel

# Retrieve sample prize and laureate documents
prize = db.prizes.find_one()
laureate = db.laureates.find_one()

# Print the sample prize and laureate documents
print(prize)
print(laureate)
print(type(laureate))

# Get the fields present in each type of document
prize_fields = list(prize.keys())
laureate_fields = list(laureate.keys())

print(prize_fields)
print(laureate_fields)
```

Queries are executed by constructing filter documents with query operators. 

```python
# Save a filter for laureates born in the USA, Canada, or Mexico
criteria = { "bornCountry": 
                { "$in": ["USA", "Canada", "Mexico"]}
             }

# Count them and save the count
count = db.laureates.count_documents(criteria)
print(count)
```

You can use dot notation to access deep fields.

```python
# Filter for laureates born in Austria with non-Austria prize affiliation
criteria = {"bornCountry": "Austria", 
              "prizes.affiliations.country": {"$ne": "Austria"}}

# Count the number of such laureates
count = db.laureates.count_documents(criteria)
print(count)
```

```python
# Filter for documents without a "born" field
criteria = {"born": {"$exists": False}}

# Save count
count = db.laureates.count_documents(criteria)
print(count)
```

You can use dot notation to access array indexes. 

```python
# Filter for laureates with at least three prizes
criteria = {"prizes.2": {"$exists": True}}

# Find one laureate with at least three prizes
doc = db.laureates.find_one(criteria)

# Print the document
print(doc)
```

---

Identifying unique values:

```python
db.laureates.distinct("gender")
```

This is a convenience method for an aggregation. We will learn how to create custom aggregations. 

The `distinct` aggregation is efficient if there is a collection index on the field. 

```python
# Countries recorded as countries of death but not as countries of birth
countries = set(db.laureates.distinct('diedCountry')) - set(db.laureates.distinct('bornCountry'))
print(countries)
```

Nobel laureates born in USA. To which countries were they affiliated? 

```python
db.laureates.distinct('prizes.affiliations.country', {'bornCountry': 'USA'})
```

```python
# Save a filter for prize documents with three or more laureates
criteria = {"laureates.2": {"$exists": True}}

# Save the set of distinct prize categories in documents satisfying the criteria
triple_play_categories = set(db.prizes.distinct("category", criteria))
assert set(db.prizes.distinct("category")) - triple_play_categories == {"literature"}
```

The `$elemMatch` operator matches documents that contain an array field with at least one element that matches all the specified query criteria.

Queries support Regex.

```python
from bson.regex import Regex

# Save a filter for laureates with prize motivation values containing "transistor" as a substring
criteria = {"prizes.motivation": Regex("transistor")}

# Save the field names corresponding to a laureate's first name and last name
print([(laureate["firstname"], laureate["surname"]) 
       for laureate in db.laureates.find(criteria)])
```

---

**Projection** is about reducing multidimensional data. It specifies which fields we want to return from our query. Fields with value 1 are included (you can also pass a list of desired fields). `"_id"` is included by default. The query returns a `Cursor`, which works like a generator. 

```python
docs = db.laureates.find(
	filter={"gender": "org"},
	projection=["bornCountry", "firstname"])
```

If a projected field does not exist, it is not considered (no error).

Sorting. Example of sorting first by ascending year, and then by descending category: 

```python
db.prizes.find(
    {"year": {"$gt": "1966"},
    		 "$lt": "1970"},
    sort = [("year", 1), ("category", -1)]
)
```

In JavaScript (and in the MongoDB shell) you would simply pass a dictionary to `sort`. This works because it retains the order of the keys. In Python there is no such guarantee, so we pass a list. 

---

It is possible to create a custom index over one or more values in order to help performance. It is common to define indexes that "cover" common queries. 

When to use indexes?

* Queries with high specificity
* Large documents
* Large collections

```python
#see which indexes exist for a collection
db.laureates.index_information()
```

```python
# Specify an index model for compound sorting
index_model = [("category", 1), ("year", -1)]
db.prizes.create_index(index_model)

# Collect the last single-laureate year for each category
report = ""
for category in sorted(db.prizes.distinct("category")):
    doc = db.prizes.find_one(
        {"category": category, "laureates.share": "1"},
        sort=[("year", -1)]
    )
    report += "{category}: {year}\n".format(**doc)

print(report)
```

Limits and skips. 

```python
#with cursor methods
db.prizes.find({"laureates.share": "3"})
		 .sort([("year", 1)])
    	 .skip(3)
         .limit(3)
#with arguments (find_done does not support cursor methods)
db.prizes.find_one({"laureates.share": "3"},
                  skip=3, sort=[("year", 1)])   
```

Using sort, skip and limit to retrieve "pages":

```python
from pprint import pprint

# Write a function to retrieve a page of data
def get_particle_laureates(page_number=1, page_size=3):
    if page_number < 1 or not isinstance(page_number, int):
        raise ValueError("Pages are natural numbers (starting from 1).")
    particle_laureates = list(
        db.laureates.find(
            {"prizes.motivation": {"$regex": "particle"}},
            ["firstname", "surname", "prizes"])
        .sort([("prizes.year", 1), ("surname", 1)])
        .skip(page_size * (page_number - 1))
        .limit(page_size))
    return particle_laureates

# Collect and save the first nine pages
pages = [get_particle_laureates(page_number=page) for page in range(1,9)]
pprint(pages[0])
```

----

An aggregation pipeline is a sequence of stages. 

```python
cursor = db.laureates.aggregate([
    {"$match": {"bornCountry": "USA"}},
    {"$project": {"prizes.year": 1},
    {"$sort": OrderedDict([("prizes.year", 1)])},
    {"$limit": 3},
    {"skip": 1}
	])
```

```python
# Translate cursor to aggregation pipeline
pipeline = [
    {"$match": {"gender": {"$ne": "org"}}},
    {"$project": {"bornCountry": 1, "prizes.affiliations.country": 1}},
    {"$limit": 3}
]

for doc in db.laureates.aggregate(pipeline):
    print("{bornCountry}: {prizes}".format(**doc))
```

Aggregation pipelines can use field paths. 

```python
#project the n_prizes field by taking the size of prizes
db.laureates.aggregate([
    {"$project": {'n_prizes': {"$size": "$prizes"}}}
]).next()
```

`$prizes` is a field path: it takes the value of the prizes field for each document processed at that stage by the pipeline. You can create new fields, or overwrite old ones, during aggregation.

`{"$size": "$prizes"}` is an operator expression. size works like a function that takes prizes as parameter. Operators can have multiple parameters, like here: 

```python
db.laurates.aggregate([
	{"$project": {"solo_winner": {"$in": ["1", "$prizes.share"]}}}
])
```

To group values by `"bornCountry"`, we map the values to `"_id"`. 

```python
db.laureates.aggregate([{"group": {"_id": "$bornCountry"}}])
```

To get just one value and aggregate over all: 

```python
db.laureates.aggregate([
	{"$project": {"n_prizes": {"$size": "$prizes"}}},
	{"$group": {"_id": None, "n_prizes_total": {"$sum": "$n_prizes"}}}
])
```

```python
# Count prizes awarded (at least partly) to organizations as a sum over sizes of "prizes" arrays.
pipeline = [
    {"$match": {"gender": "org"}},
    {"$project": {"n_prizes": {"$size": "$prizes"}}},
    {"$group": {"_id": None, "n_prizes_total": {"$sum": "$n_prizes"}}}
]

print(list(db.laureates.aggregate(pipeline)))
```

A more complex pipeline:

```python
from collections import OrderedDict

original_categories = sorted(set(db.prizes.distinct("category", {"year": "1901"})))
pipeline = [
    {"$match": {"category": {"$in": original_categories}}},
    {"$project": {"category": 1, "year": 1}},
    
    # Collect the set of category values for each prize year.
    {"$group": {"_id": "$year", "categories": {"$addToSet": "$category"}}},
    
    # Project categories *not* awarded (i.e., that are missing this year).
    {"$project": {"missing": {"$setDifference": [original_categories, "$categories"]}}},
    
    # Only include years with at least one missing category
    {"$match": {"missing.0": {"$exists": True}}},
    
    # Sort in reverse chronological order. Note that "_id" is a distinct year at this stage.
    {"$sort": OrderedDict([("_id", -1)])},
]
for doc in db.prizes.aggregate(pipeline):
    print("{year}: {missing}".format(year=doc["_id"],missing=", ".join(sorted(doc["missing"]))))
```

You can unroll a field with `"$unwind"`. 

Counting born countries for each Nobel category:

```python
pipeline = [
    # Unwind the laureates array
    {"$unwind": "$laureates"},
    {"$lookup": {
        "from": "laureates", "foreignField": "id",
        "localField": "laureates.id", "as": "laureate_bios"}},

    # Unwind the new laureate_bios array
    {"$unwind": "$laureate_bios"},
    {"$project": {"category": 1,
                  "bornCountry": "$laureate_bios.bornCountry"}},

    # Collect bornCountry values associated with each prize category
    {"$group": {"_id": "$category",
                "bornCountries": {"$addToSet": "$bornCountry"}}},

    # Project out the size of each category's (set of) bornCountries
    {"$project": {"category": 1,
                  "nBornCountries": {"$size": "$bornCountries"}}},
    {"$sort": {"nBornCountries": -1}},
]
for doc in db.prizes.aggregate(pipeline): print(doc)
```

We can add fields to a pipeline without having to project existing fields. 



