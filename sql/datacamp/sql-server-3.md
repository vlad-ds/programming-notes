# Triggers

A trigger is a special type of stored procedure that is automatically executed when an event occurs in the database server. 

* Data Manipulation Language (DML) triggers are executed on `INSERT`, `UPDATE` or `DELETE` statements. 
* Data Definition Language (DDL) triggers fire in response to statements executed at the database or server level, like CREATE, ALTER, or DROP. 
* Logon triggers respond to `LOGON` events, when a user session is established. 

Based on behavior:

* `AFTER` triggers execute after an action. 
* `INSTEAD OF` trigger. Will not perform the initial operation, will execute replacement code. E.g. prevent updates or deletions.

```sql
-- Create a new trigger that fires when deleting data
CREATE TRIGGER PreventDiscountsDelete
ON Discounts
-- The trigger should fire instead of DELETE
INSTEAD OF DELETE
AS
	PRINT 'You are not allowed to delete data from the Discounts table.';
```

```sql
-- Set up a new trigger
CREATE TRIGGER OrdersUpdatedRows
ON Orders
-- The trigger should fire after UPDATE statements
AFTER UPDATE
-- Add the AS keyword before the trigger body
AS
	-- Insert details about the changes to a dedicated table
	INSERT INTO OrdersUpdate(OrderID, OrderDate, ModifyDate)
	SELECT OrderID, OrderDate, GETDATE()
	FROM inserted;
```

```sql
-- Set up a new trigger
CREATE TRIGGER OrdersUpdatedRows
ON Orders
-- The trigger should fire after UPDATE statements
INSTEAD OF UPDATE
-- Add the AS keyword before the trigger body
AS
	-- Insert details about the changes to a dedicated table
	INSERT INTO OrdersUpdate(OrderID, OrderDate, ModifyDate)
	SELECT OrderID, OrderDate, GETDATE()
	FROM inserted;
```

Why use DML triggers:

1. Initiating actions when manipulating data
2. Preventing data manipulation
3. Tracking data or database object changes
4. User auditing and database security 

AFTER trigger usage example: large data is inserted into a sales table. After insertion, start a data cleansing procedure. Generate a table report with the procedure results. Notify the database administrator.

INSTEAD OF trigger usage example. Prevent data deletion. 

---

DML AFTER trigger. Applies to events `INSERT`, `UPDATE` and `DELETE`. 

Example: when I delete a product from the `Products` table, archive it to a `retiredProducts` table. 

`inserted` and `ðeleted` are tables automatically created by SQL server. You can use them in your trigger actions. `inserted` stores new rows from `INSERT` and `UPDATE` statements. `deleted` stores value from `UPDATE` and `DELETE`. 

```sql
-- Create the trigger
CREATE TRIGGER TrackRetiredProducts
ON Products
AFTER DELETE
AS
	INSERT INTO RetiredProducts (Product, Measure)
	SELECT Product, Measure
	FROM deleted;
```

`INSTEAD OF` triggers can only be used with DML statements. `INSERT`, `UPDATE` and `DELETE`. Example: prevent orders data from being updated. 

```sql
-- Create the trigger
CREATE TRIGGER PreventOrdersUpdate
ON Orders
INSTEAD OF UPDATE
AS
	RAISERROR ('Updates on "Orders" table are not permitted.
                Place a new order to add new products.', 16, 1);
```

DDL triggers (Data Definition Language) respond to `CREATE`, `AŁTER` and `DROP`. They support only the `AFTER` keyword. They are attached to database or servers. In the body of the trigger we often insert the information into a logs table. You can also prevent the action with a `ROLLBACK` command. 

```sql
-- Create the trigger to log table info
CREATE TRIGGER TrackTableChanges
ON DATABASE
FOR CREATE_TABLE,
	ALTER_TABLE,
	DROP_TABLE
AS
	INSERT INTO TablesChangeLog (EventData, ChangedBy)
    VALUES (EVENTDATA(), USER);
```

```sql
-- Add a trigger to disable the removal of tables
CREATE TRIGGER PreventTableDeletion
ON DATABASE
FOR DROP_TABLE
AS
	RAISERROR ('You are not allowed to remove tables from this database.', 16, 1);
    -- Revert the statement that removes the table
    ROLLBACK;
```

`LOGON` triggers are fired when a user logs on and creates a connection. They fire after authentication phase but before the session establishment. It can only be attached at the server level. 

```sql
-- Create a trigger firing when users log on to the server
CREATE TRIGGER LogonAudit
-- Use ALL SERVER to create a server-level trigger
-- Execute with 'sa' admin account
ON ALL SERVER WITH EXECUTE AS 'sa'
-- The trigger should fire after a logon
AFTER LOGON
AS
	-- Save user details in the audit table
	INSERT INTO ServerLogonLog (LoginName, LoginDate, SessionID, SourceIPAddress)
	SELECT ORIGINAL_LOGIN(), GETDATE(), @@SPID, client_net_address
	FROM SYS.DM_EXEC_CONNECTIONS WHERE session_id = @@SPID;
```

---

Advantages of triggers: 

* Used for database integrity
* Enforce business rules directly in the database
* Control on which statements are allowed in a database 
* Implementation of complex business logic triggered by a single event
* Simple way to audit databases and user actions

Disadvantages:

* Difficult to view and detect. Not easy to manage centrally
* Invisible to client applications or when debugging code
* Can affect server performance

You can view triggers in the UI. Or you can do some special queries. 

Best practices:

* Well-documented database design.
* Simple logic in trigger design.
* Avoid overusing triggers. 

```sql
-- Gather information about database triggers
SELECT name AS TriggerName,
	   parent_class_desc AS TriggerType,
	   create_date AS CreateDate,
	   modify_date AS LastModifiedDate,
	   is_disabled AS Disabled,
	   is_instead_of_trigger AS InsteadOfTrigger,
       -- Get the trigger definition by using a function
	   OBJECT_DEFINITION (object_id)
FROM sys.triggers
UNION ALL
-- Gather information about server triggers
SELECT name AS TriggerName,
	   parent_class_desc AS TriggerType,
	   create_date AS CreateDate,
	   modify_date AS LastModifiedDate,
	   is_disabled AS Disabled,
	   0 AS InsteadOfTrigger,
       -- Get the trigger definition by using a function
	   OBJECT_DEFINITION (object_id)
FROM sys.server_triggers
ORDER BY TriggerName;
```

AFTER triggers are usually used to keep a history of row changes. 

```sql
-- Create a trigger to keep row history
CREATE TRIGGER CopyCustomersToHistory
ON Customers
-- Fire the trigger for new and updated rows
AFTER INSERT, UPDATE
AS
	INSERT INTO CustomersHistory (CustomerID, Customer, ContractID, ContractDate, Address, PhoneNo, Email, ChangeDate)
	SELECT CustomerID, Customer, ContractID, ContractDate, Address, PhoneNo, Email, GETDATE()
    -- Get info from the special table that keeps new rows
    FROM inserted;
```

INSTEAD OF triggers are used to prevent operations. They can be used to enforce conditional logic. Example: an attempt to insert a new order is made. The trigger fires and checks whether there is enough product stock. If yes, the action continues. If not, an error is thrown. 

```sql
-- Prevent any product changes
CREATE TRIGGER PreventProductChanges
ON Products
INSTEAD OF UPDATE
AS
	RAISERROR ('Updates of products are not permitted. Contact the database administrator if a change is needed.', 16, 1);
```

```sql
-- Create a new trigger to confirm stock before ordering
CREATE TRIGGER ConfirmStock
ON Orders
INSTEAD OF INSERT
AS
	IF EXISTS (SELECT *
			   FROM Products AS p
			   INNER JOIN inserted AS i ON i.Product = p.Product
			   WHERE p.Quantity < i.Quantity)
	BEGIN
		RAISERROR ('You cannot place orders when there is no stock for the order''s product.', 16, 1);
	END
	ELSE
	BEGIN
		INSERT INTO Orders (OrderID, Customer, Product, Price, Currency, Quantity, WithDiscount, Discount, OrderDate, TotalAmount, Dispatched)
		SELECT OrderID, Customer, Product, Price, Currency, Quantity, WithDiscount, Discount, OrderDate, TotalAmount, Dispatched FROM Orders;
	END;
```

DDL triggers can be created at the DB level or at the server level. Useful for database auditing. 

```sql
-- Create a trigger to prevent database deletion
CREATE TRIGGER PreventDatabaseDelete
-- Attach the trigger at the server level
ON ALL SERVER
FOR DROP_DATABASE
AS
   PRINT 'You are not allowed to remove existing databases.';
   ROLLBACK;
```

----

Use `DROP TRIGGER` to delete a trigger. 

Use `DISABLE TRIGGER` to disable a trigger. 

Use `ALTER TRIGGER` to modify a trigger. 

```sql
-- Fix the typo in the trigger message
ALTER TRIGGER PreventDiscountsDelete
ON Discounts
INSTEAD OF DELETE
AS
	PRINT 'You are not allowed to remove data from the Discounts table.';
```

The view `sys.triggers` contains information about triggers. 



