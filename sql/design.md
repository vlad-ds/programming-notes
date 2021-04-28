# Database Design

Database design questions:

* Schemas. How should my data be logically organized?
* Normalization. Should my data have minimal dependency and redundancy?
* Views. What joins will be done most often?
* Access control. Should all users of the data have the same level of access?
* DBMS. How do I pick between all the SQL and noSQL options?

"Before implementing anything, figure out your business requirements because there are many design decisions you'll have to make. "

Two approaches to processing data. They define how data is going to flow, be structured and stored. 

* OLTP. Online transaction processing.

  * Stored in an operational database.

* OLAP. Online analytical processing. 

  * OLTP database pulled and cleaned to create an OLAP data warehouse.

   ![Screenshot from 2021-04-03 12-59-36](/home/vlud/Pictures/Screenshot from 2021-04-03 12-59-36.png)

Types of data: 

1. Structured data. E.g. SQL and relational dbs. 
   1. Follows a schema
   2. Defined data types & relationships
2. Semi-structured data. NoSQL, XML, JSON. 
   1. Does not follow a larger schema.
   2. Self-describing structure.
3. Unstructured data. E.g. media files and raw text. 
   1. Schemaless. 
   2. Most of the data in the world. 

Tradeoffs to consider: ease of analysis (structured, semi-structured, unstructured) on one hand, flexibility and scalability on the other (the opposite order applies).

Data storage solutions:

1. Traditional databases. OLTP, storing real-time relational structured data.
2. Data warehouses. OLAP, for analyzing archived structured data.
3. Data lakes. For storing data of all structures = flexibility and scalability. For analysing Big Data. 

**Data warehouses**: 

* OLAP, optimized for analytics. 
* Organized for reading/aggregating data. 
* Usually read-only. 
* Contain data from multiple sources. 
* Make use of Massively Parallel Processing (MPP). 
* Typically use a denormalized schema and dimensional modeling. 
* Amazon Redshift, Azure SQL Data Warehouse, Google Big Query. 
* A "data mart" is a subset of a DWH dedicated to a specific topic.

**Data lakes**:

* Store all types of data at lower cost. 
* Retains all data and can take up petabytes. 
* Schema-on-read as opposed to schema-on-write.
* Needs to catalog data otherwise becomes a data swamp. 
* It's becoming popular to run big data analytics directly on data lakes with Apache Spark and Hadoop. 

Approaches to data flows: 

* ETL. Data is transformed, adapted to the schema and put in storage (in a DWH).
* ELT. Data is stored in its native form in a data lake. Portions of data are transformed for different purposes (e.g. warehousing or deep learning).

----

**Database Design**. Determines how data is logically stored. Aspects:

* Database models. High-level specifications for database structure. 
  * The most popular is the relational model. 
  * Other options include NoSQL models, object-oriented model, network model. 
* Schema. The blueprint of the DB. 
  * Defines tables, fields, relationships, indexes and views. 
  * When inserting data, schemas must be respected.

Steps of DB design: 

The first step of DB design is **data modeling**. 

1. Conceptual data model. Describes entities, relationships and attributes. Tools: data structure diagrams, entity-relational diagrams and UML diagrams. 
2. Logical data model. Defines tables, columns, relationships. Tools: db models and schemas. 
3. Physical data model. Describes physical storage. Tools: partitions, CPUs, indexes, backup systems and tablespaces. 

**Dimensional modeling**. Adaptation of the relational model for data warehouse design. Optimized for OLAP queries: aggregate data, not updating. Uses the **star schema**. Two types of tables:

* Fact tables. Holds records of a key metric which changes often. 
* Dimension tables. Hold descriptions of attributes, don't change as often. 

E.g. a song would be a fact table. It would hold foreign keys to dimension tables which describe the album, artist, label. 

---

**Star schema**. Simplest form of the dimensional model. It's made up of **fact** and **dimension** tables. 

* Fact tables holds records of a metric. It's connected to dimensions via foreign keys. Changes regularly.
* Dimension tables hold descriptions of attributes. Do not change as often. 

![Screenshot from 2021-04-03 13-49-02](/home/vlud/Pictures/Screenshot from 2021-04-03 13-49-02.png)

The fact table holds a transaction. The dimensions table hold information on the book, the store and the time. 

**Snowflake schema**. Extension of the star schema. The fact table is the same. Dimension tables extend more, over more than one dimension. This is because the dimension tables are **normalized**. 

![Screenshot from 2021-04-03 13-54-05](/home/vlud/Pictures/Screenshot from 2021-04-03 13-54-05.png)

**Normalization**. A technique that divides tables into smaller tables and connects them via relationships. The goal is to **reduce redundancy** and **increase data integrity**. The basic idea is to identify repeating groups of data and create new tables for them. 

Example of creating a snowflake schema, by creating a new table for authors:

```sql
-- Create a new table for dim_author with an author column
CREATE TABLE dim_author (
    author varchar(256)  NOT NULL
);

-- Insert authors 
INSERT INTO dim_author
SELECT DISTINCT author FROM dim_book_star;

-- Add a primary key 
ALTER TABLE dim_author ADD COLUMN author_id SERIAL PRIMARY KEY;

-- Output the new table
SELECT * FROM dim_author;
```

The normalized snowflake schemas has more tables. Which means more joins and slower queries. So why would we want to normalize a database? 

* Normalization saves space as it reduces redundancy.
* Normalization ensures better data integrity. It enforces data consistency, enables safer updating, removing and inserting. And it makes it easier to redesign by extending. 
* E.g. you can enforce naming conventions through referential integrity (instead of writing California in different ways, you will have to enter the ID for California.)
* E.g. to change the spelling of California, you will only need to change one record. And you can be confident that the new spelling will be enacted for all stores. 

Ultimately it comes down to how read- or write-intensive your database is going to be:

* OLTP is write intensive. It is typically highly normalized. It prioritizes quicker and safer insertion of data. 
* OLAP is read intensive. It is typically less normalized. 

The goals of normalization are to:

* Be able to characterize the level of redundancy in a relational schema.
* Provide mechanisms for transforming schemas in order to remove redundancy.

*Database Design*, Adrienne Watt.

There are many normal forms (NF) of different orders. 

* **1NF**. Each record must be unique - no duplicate rows. Each cell must hold one value.
* **2NF**. Must satisfy 1NF. Primary key is one column. If there's a composite primary key, each non-key column must be dependent on all keys. 
* **3NF**. Satisfies 2NF. No transitive dependencies: non-key columns can't depend on other non-key columns. 

A database that isn't normalized enough is prone to 3 types of anomaly errors:

* Update anomaly. Data inconsistency caused by data redundancy when updating.
* Insertion anomaly. Being unable to add a new record due to missing attributes. 
* Deletion anomaly. When deleting a record causes unintentional loss of data. 

---

### Database views

A view is the result set of a stored query on the data, which the db users can query just as they would in a persistent database collection set.

Views are virtual tables that are not part of the physical schema. The query, not the data is stored in memory. Data is aggregated from data in tables. It can be queried like a regular table. 

The benefit of the view is that you don't need to retype common queries or alter schemas. 

To get all the views in Postgres:

```sql
-- This query excludes system views
SELECT * FROM INFORMATION_SCHEMA.views
WHERE table_schema NOT IN ('pg_catalog', 'information_schema')
```

Advantages of views:

1. Don't take up any storage.
2. Allow access control (hide sensitive columns, restrict what user can see).
3. Mask the complexity of queries (useful for highly normalized schemas).

Creating and using a view:

```sql
-- Create a view for reviews with a score above 9
CREATE VIEW high_scores AS
SELECT * FROM REVIEWS
WHERE score > 9;

-- Count the number of self-released works in high_scores
SELECT COUNT(*) FROM high_scores
INNER JOIN labels ON high_scores.reviewid = labels.reviewid
WHERE label = 'self-released';
```

We can grant and revoke privileges for a view. 

```sql
-- granting update rights to everyone
GRANT UPDATE ON ratings TO PUBLIC;
-- revoking insert rights from a user
REVOKE INSERT ON films FROM db_user;
```

Users with the necessary privilege can update a view or insert new date:

```sql
UPDATE films SET kind = 'Dramatic' WHERE kind = 'Drama'

INSERT INTO films (code, title, did, date_prod, kind)
	VALUES ('T_601', 'Yojimbo', 106, '1961-06-16', 'Drama')
```

Not all views however are updateable. Because the update affects the underlying tables. It's good practice to avoid modifying data through views. 

To drop a view:

```sql
DROP VIEW view_name [CASCADE | RESTRICT]
```

RESTRICT is default and returns an error if there are objects that depend on the view. 

CASCADE will drop the objects that depend on the view. 

To redefine a view:

```sql
CREATE OR REPLACE VIEW view_name AS new_query
```

The new query must generate the same column names, order and data types, although it might also add new columns at the end. 

It is also possible to alter views:

```sql
ALTER VIEW [ IF EXISTS ] name ALTER [ COLUMN ] column_name SET DEFAULT expression
```

**Materialized views** are physically materialized. They store the query results, not the query. They are refreshed or rematerialized when prompted. 

They are useful for queries with long execution time. The data is only as updated as the last time it was refreshed. So they're not good for use cases where the data needs to updated often. 

Materialized views are typically used in DWH (OLAP). 

```sql
CREATE MATERIALIZED VIEW my_mv AS SELECT * FROM existing_Table;

REFRESH MATERIALIZED VIEW my_mv
```

Typically we use DAGs to manage view dependencies and schedule refresh times. Two views can't depend on each other. 

---

### Roles and access control

A database role is an entity that:

* Defines the role's privileges
* Interacts with the client auth system
* Can be assigned to one or more users
* Are global across a database cluster installation

```sql
CREATE ROLE data_analyst;
CREATE ROLE intern WITH PASSWORD 'password' VALID UNTIL '2020-01-01';
CREATE ROLE admin CREATEDB;
```

```sql
GRANT UPDATE ON ratings TO data_analyst;
REVOKE UPDATE ON ratings FROM data_analyst;
```

Create a user role and assign it to a group role:

```sql
CREATE ROLE data_analyst;
CREATE ROLE alex WITH PASSWORD 'password';
GRANT data_analyst TO alex;
REVOKE data_analyst FROM alex;
```

Postgres has a set of default roles. 

```sql
-- Grant data_scientist update and insert privileges
GRANT UPDATE, INSERT ON long_reviews TO data_scientist;

-- Give Marta's role a password
ALTER ROLE marta WITH PASSWORD 's3cur3p@ssw0rd';
```

---

### Table partitioning

As tables grow queries become slower. That's when we want to split tables into multiple smaller parts (partitioning). Partitioning is part of the physical data model. 

Types:

* Vertical partitioning. Split a table by its columns. Store rarely retrieved fields on a slower medium.
* Horizontal partitioning. Split over the rows. E.g. according to a timestamp. 

When horizontal partitioning is applied to spread a table over several machines, it's called **sharding**. Sharding opens the way to parallelization.

Vertical partitioning:

```sql
-- Create a new table called film_descriptions
CREATE TABLE film_descriptions (
    film_id INT,
    long_description TEXT
);

-- Copy the descriptions from the film table
INSERT INTO film_descriptions
SELECT film_id, long_description FROM film;
    
-- Drop the descriptions from the original table
ALTER TABLE film
DROP COLUMN long_description;

-- Join to view the original table
SELECT * FROM film 
JOIN film_descriptions
ON film_descriptions.film_id = film.film_id
```

Horizontal partition: 

```sql
-- Create a new table called film_partitioned
CREATE TABLE film_partitioned (
  film_id INT,
  title TEXT NOT NULL,
  release_year TEXT
)
PARTITION BY LIST (release_year);

-- Create the partitions for 2019, 2018, and 2017
CREATE TABLE film_2019
	PARTITION OF film_partitioned FOR VALUES IN ('2019');

CREATE TABLE film_2018
	PARTITION OF film_partitioned FOR VALUES IN ('2018');

CREATE TABLE film_2017
	PARTITION OF film_partitioned FOR VALUES IN ('2017');

-- Insert the data into film_partitioned
INSERT INTO film_partitioned
SELECT film_id, title, release_year FROM film;

-- View film_partitioned
SELECT * FROM film_partitioned;
```

---

### Data integration

Combines data from different sources, formats, technologies to provide users with a translated and unified view of that data. 

Unified data model needs to be decided based on the use case. 

Transformations. Program that extracts data from the sources and transforms it in accordance with the unified data model. Your integration tool should be flexible, reliable and scalable. Automated testing and proactive alerts. 

Security is also an important concern. Anonymize data during ETL. 

---

### DBMS

Database Management System. Manages:

1. Data
2. Database schema
3. Database engine

Business case and type of data --> Type of Database --> DBMS choice.

Two kinds: SQL and NoSQL. 

SQL. Based on the relational model. Best option when data is structured and unchanging. Data must be consistent. 

NoSQL. Less structured. Document centered. No well defined rows and columns. Good for rapid growth and flexibility. Four types:

1. Key-value. Session information in web apps (shopping carts). Redis.
2. Document store. Values have some structure. Content management apps (blogs and video platforms). MongoDB.
3. Columnar database. Each column is a separate file in the system storage. Scalable, faster at scale. Big data analytics. Example is Cassandra. 
4. Graph database. Social media data, recommendations. Neo4J. 















