# Transactions and Error Handling

SQL Server supports try and catch syntax for error handling. 

```sql
-- Set up the TRY block
BEGIN TRY
	-- Add the constraint
	ALTER TABLE products
		ADD CONSTRAINT CHK_Stock CHECK (stock >= 0);
END TRY
-- Set up the CATCH block
BEGIN CATCH
	SELECT 'An error occurred!';
END CATCH
```

```sql
-- Set up the first TRY block
BEGIN TRY
	INSERT INTO buyers (first_name, last_name, email, phone)
		VALUES ('Peter', 'Thompson', 'peterthomson@mail.com', '555000100');
END TRY
-- Set up the first CATCH block
BEGIN CATCH
	SELECT 'An error occurred inserting the buyer! You are in the first CATCH block';
    -- Set up the nested TRY block
    BEGIN TRY
    	INSERT INTO errors 
        	VALUES ('Error inserting a buyer');
        SELECT 'Error inserted correctly!';
	END TRY
    -- Set up the nested CATCH block
    BEGIN CATCH
    	SELECT 'An error occurred inserting the error! You are in the nested CATCH block';
    END CATCH 
END CATCH
```

Not all errors can be detected by a try/catch construct. 

Anatomy of an error:

```
Msg 2627, Level 14 State 1, Line 1
```

Msg refers to the error code. SQL has a catalog of errors with codes from 1 to 49999. You can also define your own errors starting from 50001. 

Level is the severity level: 0-10 for informational messages, 11-16 errors that can be corrected by the user, 17-24 software problems and fatal errors.

State is 1 if server displays error, 0-255 for your own errors. 

Line indicates on which line the error occurred. You may also seem the name of the procedure or trigger that caused the error. 

Uncatchable errors: 

* Severity lower than 11 (these are just warnings)
* Severity of 20 or higher (they will stop the connection)
* Compilation errors: objects and columns that don't exist, functions written wrongly

Several functions allow us to access error information. These are `ERROR_NUMBER()`, `ERROR_SEVERITY()`, `ERROR_STATE()`, `ERROR_PROCEDURE()`, `ERROR_LINE()`, and `ERROR_MESSAGE()`. These functions must be placed within the `CATCH` block. If an error occurs within the `TRY` block, they return information about the error.

```sql
-- Set up the TRY block
BEGIN TRY  	
	SELECT 'Total: ' + SUM(price * quantity) AS total
	FROM orders  
END TRY
-- Set up the CATCH block
BEGIN CATCH  
	-- Show error information.
	SELECT  ERROR_NUMBER() AS number,  
        	ERROR_SEVERITY() AS severity_level,  
        	ERROR_STATE() AS state,
        	ERROR_LINE() AS line,  
        	ERROR_MESSAGE() AS message; 	
END CATCH 
```

---

 `RAISERROR` is used to raise an error. `THROW ` is an alternative. You can give them error number, message and state.

```sql
-- Change the value
DECLARE @product_id INT = 50;

IF NOT EXISTS (SELECT * FROM products WHERE product_id = @product_id)
	RAISERROR('No product with id %d.', 11, 1, @product_id);
ELSE 
	SELECT * FROM products WHERE product_id = @product_id;
```

```sql
BEGIN TRY
	-- Change the value
    DECLARE @product_id INT = 5;	
    IF NOT EXISTS (SELECT * FROM products WHERE product_id = @product_id)
        RAISERROR('No product with id %d.', 11, 1, @product_id);
    ELSE 
        SELECT * FROM products WHERE product_id = @product_id;
END TRY
BEGIN CATCH
	SELECT ERROR_MESSAGE();
END CATCH
```

Use `THROW` without parameters to re-throw an error: 

```sql
CREATE PROCEDURE insert_product
  @product_name VARCHAR(50),
  @stock INT,
  @price DECIMAL

AS

BEGIN TRY
	INSERT INTO products (product_name, stock, price)
		VALUES (@product_name, @stock, @price);
END TRY
-- Set up the CATCH block
BEGIN CATCH
	-- Insert the error and end the statement with a semicolon
    INSERT INTO errors VALUES ('Error inserting a product');
    -- Re-throw the error
	THROW; 
END CATCH
```

Throw an error with a customized error message. You can customize it with `CONCAT` and `FORMATMESSAGE`. 

```sql
DECLARE @first_name NVARCHAR(20) = 'Pedro';

-- Concat the message
DECLARE @my_message NVARCHAR(500) =
	CONCAT('There is no staff member with ', @first_name, ' as the first name.');

IF NOT EXISTS (SELECT * FROM staff WHERE first_name = @first_name)
	-- Throw the error
	THROW 50000, @my_message, 1;
```

```sql
DECLARE @product_name AS NVARCHAR(50) = 'Trek CrossRip+ - 2018';
-- Set the number of sold bikes
DECLARE @sold_bikes AS INT = 10;
DECLARE @current_stock INT;

SELECT @current_stock = stock FROM products WHERE product_name = @product_name;

DECLARE @my_message NVARCHAR(500) =
	-- Customize the error message
	FORMATMESSAGE('There are not enough %s bikes. You have %d in stock.' , @product_name, @current_stock);

IF (@current_stock - @sold_bikes < 0)
	-- Throw the error
	THROW 50000, @my_message, 1;
```

---

A **transaction** is the execution of one or more statements, such that either all or none of the statements are executed. It's a series of operations that together must behave as an atomic operation. We execute all of them or none of them. 

```sql
BEGIN TRY  
	BEGIN TRAN;
		UPDATE accounts SET current_balance = current_balance - 100 WHERE account_id = 1;
		INSERT INTO transactions VALUES (1, -100, GETDATE());
        
		UPDATE accounts SET current_balance = current_balance + 100 WHERE account_id = 5;
		INSERT INTO transactions VALUES (5, 100, GETDATE());
	COMMIT TRAN;
END TRY
BEGIN CATCH  
	ROLLBACK TRAN;
END CATCH
```

---

Errors:

1. Dirty reads
2. Non-repeatable reads
3. Phantom reads

Different levels of transactions isolation are required in order to avoid these errors. 













