# Database Design

Database design questions:

* Schemas. How should my data be logically organized?
* Normalization. Should my date have minimal dependency and redundancy?
* Views. What joins will be don most often?
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

