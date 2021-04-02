# Intro to SQL

Postgres provides the `information_schema` db by default. It is a meta-database that holds information about your current database. For example to see the columns of the table `pg_config`:

```sql
SELECT table_name, column_name, data_type
FROM information_schema.columns
WHERE table_name = 'pg_config';
```

You are interested in the tables with  `public` schema, since the other tables hold system information. 

```sql
-- Query the right table in information_schema
SELECT table_name 
FROM information_schema.tables
-- Specify the correct table_schema value
WHERE table_schema = 'public';
```

```sql
-- Query the right table in information_schema to get columns
SELECT column_name, data_type 
FROM information_schema.columns 
WHERE table_name = 'university_professors' AND table_schema = 'public';
```

Rename a column: 

```sql
-- Rename the organisation column
ALTER TABLE affiliations
RENAME COLUMN organisation TO organization;

-- Delete the university_shortname column
ALTER TABLE affiliations
DROP COLUMN university_shortname;

-- Drop table
DROP TABLE university_professors;
```

---

Integrity constraints are rules that enforce data quality and consistency. Types:

1. Attribute constraints, e.g. data types on columns.
2. Key constraints, e.g. primary keys. 
3. Referential integrity constraints, enforced through foreign keys. 

The most common types in Postgres are `text`, `varchar [x]`, `char  [x]`, and `boolean`. 

To change a column type:

```sql
-- Specify the correct fixed-length character type
ALTER TABLE professors
ALTER COLUMN university_shortname
TYPE CHAR(3);
```

How truncate a column before converting its type, so it doesn't overflow: 

```sql
-- Convert the values in firstname to a max. of 16 characters
ALTER TABLE professors 
ALTER COLUMN firstname
TYPE VARCHAR(16)
USING SUBSTRING(firstname FROM 1 FOR 16);
```

Other important constraints are `UNIQUE` and `NOT NULL`. 

How to add a NOT NULL constraint:

```sql
-- Add not null constraint
ALTER TABLE students
ALTER COLUMN home_phone
SET NOT NULL;

-- Remove constraint
ALTER TABLE students
ALTER COLUMN ssn
DROP NOT NULL;
```

```sql
-- Add unique constraint
ALTER TABLE universities
ADD CONSTRAINT university_short_name_unq UNIQUE(university_shortname);
```

---

A key is an attribute that identifies a record uniquely. A superkey is a key which maintains this property if an attribute is removed; as opposed to a minimal key. 

There can only be one primary key per table, chosen among the candidate (minimal) keys. Unique and not null constraints apply. They must also be time invariant. 

Create a primary key through the combination of two columns: 

```sql
CREATE TABLE example (
    a integer,
    b integer,
    c integer,
    PRIMARY KEY (a, c)
)
```

Add the primary key constraint:

```sql
ALTER TABLE table_name
ADD CONSTRAINT some_name PRIMARY KEY (column_name)
```

Renaming a column and making it primary key: 

```sql
-- Rename the organization column to id
ALTER TABLE organizations
RENAME COLUMN organization TO id;

-- Make id a primary key
ALTER TABLE organizations
ADD CONSTRAINT organization_pk PRIMARY KEY (id);
```

Surrogate keys are primary keys based on a column that exists just for this purpose. 

In Postgres this is done using the `serial` type. 

Create a new column and make it primary key:

```sql
-- Add the new column to the table
ALTER TABLE professors 
ADD COLUMN id serial;

-- Make id a primary key
ALTER TABLE professors 
ADD CONSTRAINT professors_pkey PRIMARY KEY (id);
```

Create surrogate key from two columns:

```sql
-- Count the number of distinct rows with columns make, model
SELECT COUNT(DISTINCT(make, model)) 
FROM cars;

-- Add the id column
ALTER TABLE cars
ADD COLUMN id varchar(128);

-- Update id with make + model
UPDATE cars
SET id = CONCAT(make, model);
```

---

A foreign key is a designated column that points to the primary key of another table. The foreign key constraint (aka referential integrity) is that each value of FK must exist in PK of the other table. Duplicates and null values are allowed for FK. 

Add a foreign key:

```sql
-- Rename the university_shortname column
ALTER TABLE professors
RENAME COLUMN university_shortname TO university_id;

-- Add a foreign key on professors referencing universities
ALTER TABLE PROFESSORS 
ADD CONSTRAINT professors_fkey FOREIGN KEY (university_id) REFERENCES universities (id);
```

N:M relationships (many to many) must be modeled with a table containing foreign keys for every connected tables.

Add a column to a table based on values from another table:

```sql
-- Update professor_id to professors.id where firstname, lastname correspond to rows in professors
UPDATE affiliations
SET professor_id = professors.id
FROM professors
WHERE affiliations.firstname = professors.firstname AND affiliations.lastname = professors.lastname;

-- Have a look at the 10 first rows of affiliations again
SELECT * FROM affiliations LIMIT 10;
```

Referential integrity from table A to table B is violated if: 

1. A record in table B that is referenced from a record in table A is deleted
2. A record in table A referencing a non-existing record from table B is inserted

You can decide what happens in case of violations. `ON DELETE NO ACTION` is the default option, and it blocks an action that violates referential integrity. `ON DELETE CASCADE` deletes referencing records until referential integrity is restored (e.g. if a university is deleted, all affiliated professors are deleted too).`SET NULL` will set the referencing column to NULL. 

How to change the ON DELETE behavior on a column: 

```sql
-- Identify the correct constraint name
SELECT constraint_name, table_name, constraint_type
FROM information_schema.table_constraints
WHERE constraint_type = 'FOREIGN KEY';

-- Drop the right foreign key constraint
ALTER TABLE affiliations
DROP CONSTRAINT affiliations_organization_id_fkey;

-- Add a new foreign key constraint from affiliations to organizations which cascades deletion
ALTER TABLE affiliations
ADD CONSTRAINT affiliations_organization_id_fkey FOREIGN KEY (organization_id) REFERENCES organizations (id) ON DELETE CASCADE;
```

