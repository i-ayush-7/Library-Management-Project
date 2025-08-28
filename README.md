# Library Management System using SQL Project --P2

## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  
**Database**: `library_db`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

![Library_project](https://github.com/i-ayush-7/Library-Management-Project/blob/main/Library%20Image.jpg)

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup
![ERD](https://github.com/i-ayush-7/Library-Management-Project/blob/main/ERD%20Diagram.png)

- **Database Creation**: Created a database named `library_db`.
- **Table Creation**: Created tables for branches, employees, members, books, issued status, and return status. Each table includes relevant columns and relationships.

```sql
CREATE DATABASE library_db;

DROP TABLE IF EXISTS branch;
CREATE TABLE branch
(
            branch_id VARCHAR(10) PRIMARY KEY,
            manager_id VARCHAR(10),
            branch_address VARCHAR(30),
            contact_no VARCHAR(15)
);


-- Create table "Employee"
DROP TABLE IF EXISTS employees;
CREATE TABLE employees
(
            emp_id VARCHAR(10) PRIMARY KEY,
            emp_name VARCHAR(30),
            position VARCHAR(30),
            salary DECIMAL(10,2),
            branch_id VARCHAR(10),
            FOREIGN KEY (branch_id) REFERENCES  branch(branch_id)
);


-- Create table "Members"
DROP TABLE IF EXISTS members;
CREATE TABLE members
(
            member_id VARCHAR(10) PRIMARY KEY,
            member_name VARCHAR(30),
            member_address VARCHAR(30),
            reg_date DATE
);



-- Create table "Books"
DROP TABLE IF EXISTS books;
CREATE TABLE books
(
            isbn VARCHAR(50) PRIMARY KEY,
            book_title VARCHAR(80),
            category VARCHAR(30),
            rental_price DECIMAL(10,2),
            status VARCHAR(10),
            author VARCHAR(30),
            publisher VARCHAR(30)
);



-- Create table "IssueStatus"
DROP TABLE IF EXISTS issued_status;
CREATE TABLE issued_status
(
            issued_id VARCHAR(10) PRIMARY KEY,
            issued_member_id VARCHAR(30),
            issued_book_name VARCHAR(80),
            issued_date DATE,
            issued_book_isbn VARCHAR(50),
            issued_emp_id VARCHAR(10),
            FOREIGN KEY (issued_member_id) REFERENCES members(member_id),
            FOREIGN KEY (issued_emp_id) REFERENCES employees(emp_id),
            FOREIGN KEY (issued_book_isbn) REFERENCES books(isbn) 
);



-- Create table "ReturnStatus"
DROP TABLE IF EXISTS return_status;
CREATE TABLE return_status
(
            return_id VARCHAR(10) PRIMARY KEY,
            issued_id VARCHAR(30),
            return_book_name VARCHAR(80),
            return_date DATE,
            return_book_isbn VARCHAR(50),
            FOREIGN KEY (return_book_isbn) REFERENCES books(isbn)
);

```

### 2. CRUD Operations

- **Create**: Inserted sample records into the `books` table.
- **Read**: Retrieved and displayed data from various tables.
- **Update**: Updated records in the `employees` table.
- **Delete**: Removed records from the `members` table as needed.

**Task 1: Create a New Book Record**
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

```sql
INSERT INTO BOOKS(ISBN, BOOK_TITLE, CATEGORY, RENTAL_PRICE, STATUS, AUTHOR, PUBLISHER)
VALUES
('978-1-60129-456-2',
'To Kill a Mockingbird',
'Classic',
6.0,
'yes',
'Harper Lee',
'J.B. Lippincott & Co.');
SELECT * FROM BOOKS
```
**Task 2: Update an Existing Member's Address**

```sql
SELECT * FROM MEMBERS

UPDATE MEMBERS
SET MEMBER_ADDRESS = '125 Main St'
WHERE MEMBER_ID = 'C101';
```

**Task 3: Delete a Record from the Issued Status Table**
-- Objective: Delete the record with issued_id = 'IS121' from the issued_status table.

```sql
SELECT * FROM ISSUED_STATUS

DELETE FROM ISSUED_STATUS
WHERE ISSUED_ID = 'IS121'
```

**Task 4: Retrieve All Books Issued by a Specific Employee**
-- Objective: Select all books issued by the employee with emp_id = 'E101'.

```sql
SELECT * FROM ISSUED_STATUS
WHERE ISSUED_EMP_ID = 'E101'
```


**Task 5:** List Members IDs of those who have issued more than one book with total number of books issued by them.

```sql
SELECT ISSUED_MEMBER_ID, 
COUNT(*) AS TOTAL_BOOKS_ISSUED
FROM ISSUED_STATUS
GROUP BY 1
HAVING COUNT(*) > '1'
```

**Task 6:** Select names of those members who have issued more than one book.

```sql
SELECT MEMBERS.MEMBER_NAME, ISSUED_STATUS.ISSUED_MEMBER_ID, 
COUNT(ISSUED_STATUS.ISSUED_MEMBER_ID) AS TOTAL_BOOKS_ISSUED
FROM MEMBERS
JOIN ISSUED_STATUS ON MEMBERS.MEMBER_ID = ISSUED_STATUS.ISSUED_MEMBER_ID
GROUP BY MEMBERS.MEMBER_NAME, ISSUED_STATUS.ISSUED_MEMBER_ID
HAVING COUNT(ISSUED_STATUS.ISSUED_MEMBER_ID) > '1'
```

**Task 7:** Create Summary Tables: Use CTAS to generate new tables based on query results - each book and total book_issued_count

```sql
CREATE TABLE BOOK_COUNTS
AS
SELECT
B.ISBN, B.BOOK_TITLE, COUNT(IST.ISSUED_ID)
FROM BOOKS AS B
JOIN
ISSUED_STATUS AS IST
ON IST.ISSUED_BOOK_ISBN = B.ISBN
GROUP BY 1, 2
```

**Task 8:** Retrieve All Books in a Specific Category**:
```sql 
SELECT * FROM BOOKS
WHERE CATEGORY = 'History'
```

**Task 9:** Find Total Rental Income by Category:

```sql
SELECT
B.CATEGORY,
SUM(B.RENTAL_PRICE), COUNT(*)
FROM BOOKS AS B
JOIN ISSUED_STATUS AS IST
ON IST.ISSUED_BOOK_ISBN = B.ISBN
GROUP BY 1
```

**Task 10:** **List Members Who Registered in the Last 500 Days**:
```sql
SELECT * FROM MEMBERS
WHERE REG_DATE >= CURRENT_DATE - INTERVAL '500 DAYS'
```

**Task 11:** List Employees with Their Branch Manager's Name and their branch details:

```sql
SELECT 
    E1.EMP_ID,
    E1.EMP_NAME,
    E1.POSITION,
    E1.SALARY,
    B.*,
    E2.EMP_NAME AS MANAGER
FROM EMPLOYEES AS E1
JOIN 
BRANCH AS B
ON E1.BRANCH_ID = B.BRANCH_ID    
JOIN
EMPLOYEES AS E2
ON E2.EMP_ID = B.MANAGER_ID
```

**Task 12:** Create a Table of Books with Rental Price Above 7$:
```sql
CREATE TABLE EXPENSIVE_BOOKS AS
SELECT * FROM BOOKS
WHERE RENTAL_PRICE > 7.0
SELECT * FROM EXPENSIVE_BOOKS
```

**Task 13:** Retrieve the List of Books Not Yet Returned
```sql
SELECT
DISTINCT IST.ISSUED_BOOK_NAME
FROM ISSUED_STATUS AS IST
LEFT JOIN
RETURN_STATUS AS RS
ON RS.ISSUED_ID = IST.ISSUED_ID
WHERE RS.ISSUED_ID IS NULL;
```

## Advanced SQL Operations

**Task 14: Identify Members with Overdue Books**  
Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's_id, member's name, book title, issue date, and days overdue.

```sql
SELECT
IST.ISSUED_MEMBER_ID,
M.MEMBER_NAME,
BK.BOOK_TITLE,
IST.ISSUED_DATE,
RS.RETURN_DATE,
CURRENT_DATE - IST.ISSUED_DATE AS OVER_DUES_DAYS
FROM ISSUED_STATUS IST
JOIN MEMBERS AS M
ON M.MEMBER_ID = IST.ISSUED_MEMBER_ID
JOIN BOOKS AS BK
ON BK.ISBN = IST.ISSUED_BOOK_ISBN
LEFT JOIN RETURN_STATUS AS RS
ON RS.ISSUED_ID = IST.ISSUED_ID
WHERE
RS.RETURN_DATE IS NULL
AND
(CURRENT_DATE - IST.ISSUED_DATE) > 30
ORDER BY 1
```


**Task 15: Update Book Status on Return**  
Write a query to update the status of books in the books table to "Yes" when they are returned (based on entries in the return_status table).


```sql

CREATE OR REPLACE PROCEDURE add_return_records(p_return_id VARCHAR(10), p_issued_id VARCHAR(10), p_book_quality VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
    v_isbn VARCHAR(50);
    v_book_name VARCHAR(80);
    
BEGIN
    -- all your logic and code
    -- inserting into returns based on users input
    INSERT INTO return_status(return_id, issued_id, return_date, book_quality)
    VALUES
    (p_return_id, p_issued_id, CURRENT_DATE, p_book_quality);

    SELECT 
        issued_book_isbn,
        issued_book_name
        INTO
        v_isbn,
        v_book_name
    FROM issued_status
    WHERE issued_id = p_issued_id;

    UPDATE books
    SET status = 'yes'
    WHERE isbn = v_isbn;

    RAISE NOTICE 'Thank you for returning the book: %', v_book_name;
    
END;
$$

-- Testing FUNCTION add_return_records

SELECT * FROM books
WHERE isbn = '978-0-307-58837-1';

SELECT * FROM issued_status
WHERE issued_book_isbn = '978-0-307-58837-1';

SELECT * FROM return_status
WHERE issued_id = 'IS135';

-- calling function 
CALL add_return_records('RS138', 'IS135', 'Good');

-- calling function 
CALL add_return_records('RS148', 'IS140', 'Good');

```




**Task 16: Branch Performance Report**  
Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

```sql
CREATE TABLE BRANCH_REPORTS
AS
SELECT
B.BRANCH_ID,
B.MANAGER_ID,
COUNT(IST.ISSUED_ID) AS NUMBER_OF_BOOKS_ISSUED,
COUNT(RS.RETURN_ID) AS NUMBER_OF_BOOKS_RETURNED,
COUNT(BK.RENTAL_PRICE) AS REVENUE_GENERATED
FROM ISSUED_STATUS AS IST
JOIN
EMPLOYEES AS E
ON E.EMP_ID = IST.ISSUED_EMP_ID
JOIN
BRANCH AS B
ON E.BRANCH_ID = B.BRANCH_ID
LEFT JOIN
RETURN_STATUS AS RS
ON RS.ISSUED_ID = IST.ISSUED_ID
JOIN
BOOKS AS BK
ON BK.ISBN = IST.ISSUED_BOOK_ISBN
GROUP BY 1,2;

SELECT* FROM BRANCH_REPORTS
```

**Task 17: CTAS: Create a Table of Active Members**  
Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 2 months.

```sql
CREATE TABLE ACTIVE_MEMBERS
AS
SELECT * FROM MEMBERS
WHERE MEMBER_ID IN
(
SELECT
DISTINCT ISSUED_MEMBER_ID
FROM ISSUED_STATUS
WHERE
ISSUED_DATE <= CURRENT_DATE - INTERVAL '2 Months'
)

SELECT * FROM ACTIVE_MEMBERS
```


**Task 18:** Find Employees with the Most Book Issues Processed. Write a query to find the employees who have processed the book issues.
Also find the total number of books issued by them. Display the employee name, number of books processed, and their branch.
```sql
SELECT
E.EMP_NAME,
B.*,
COUNT(IST.ISSUED_ID)
AS NUMBER_OF_BOOKS_ISSUED
FROM ISSUED_STATUS AS IST
JOIN EMPLOYEES AS E
ON IST.ISSUED_EMP_ID = E.EMP_ID
JOIN BRANCH AS B
ON B.BRANCH_ID = E.BRANCH_ID
GROUP BY 1,2
```

**Task 19: Find Employees with the Most Book Issues Processed.**
Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

```sql
SELECT
E.EMP_NAME as top_3_employees,
B.*,
COUNT(IST.ISSUED_ID)
AS NUMBER_OF_BOOKS_ISSUED
FROM ISSUED_STATUS AS IST
JOIN EMPLOYEES AS E
ON IST.ISSUED_EMP_ID = E.EMP_ID
JOIN BRANCH AS B
ON B.BRANCH_ID = E.BRANCH_ID
GROUP BY 1,2
ORDER BY COUNT(IST.ISSUED_ID) DESC
LIMIT 3
```

**Task 20: Stored Procedure**
Objective: Create a stored procedure to manage the status of books in a library system.
Description: Write a stored procedure that updates the status of a book in the library based on its issuance. The procedure should function as follows:
The stored procedure should take the book_id as an input parameter.
The procedure should first check if the book is available (status = 'yes').
If the book is available, it should be issued, and the status in the books table should be updated to 'no'.
If the book is not available (status = 'no'), the procedure should return an error message indicating that the book is currently not available.

```sql
CREATE OR REPLACE PROCEDURE ISSUE_BOOK
(P_ISSUED_ID VARCHAR(10), P_ISSUED_MEMBER_ID VARCHAR(30), P_ISSUED_BOOK_ISBN VARCHAR(30), P_ISSUED_EMP_ID VARCHAR(10))
LANGUAGE PLPGSQL
AS $$

DECLARE
-- all the variabable
    v_status VARCHAR(10);

BEGIN
-- all the code
    -- checking if book is available 'yes'
    SELECT 
        status 
        INTO
        v_status
    FROM books
    WHERE isbn = p_issued_book_isbn;

    IF v_status = 'yes' THEN

        INSERT INTO issued_status(issued_id, issued_member_id, issued_date, issued_book_isbn, issued_emp_id)
        VALUES
        (p_issued_id, p_issued_member_id, CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);

        UPDATE books
            SET status = 'no'
        WHERE isbn = p_issued_book_isbn;

        RAISE NOTICE 'Book records added successfully for book isbn : %', p_issued_book_isbn;


    ELSE
        RAISE NOTICE 'Sorry to inform you the book you have requested is unavailable book_isbn: %', p_issued_book_isbn;
    END IF;
END;
$$

-- Testing The function

SELECT * FROM BOOKS;

-- "978-0-553-29698-2" -- yes
-- "978-0-375-41398-8" -- no

SELECT * FROM issued_status;

CALL issue_book('IS155', 'C108', '978-0-553-29698-2', 'E104');
CALL issue_book('IS156', 'C108', '978-0-375-41398-8', 'E104');

SELECT * FROM books
WHERE isbn = '978-0-375-41398-8'

```

## Reports

- **Database Schema**: Detailed table structures and relationships.
- **Data Analysis**: Insights into book categories, employee salaries, member registration trends, and issued books.
- **Summary Reports**: Aggregated data on high-demand books and employee performance.

## Conclusion

This project demonstrates the application of SQL skills in creating and managing a library management system. It includes database setup, data manipulation, and advanced querying, providing a solid foundation for data management and analysis.

## Project by: Ayush Shukla

Thank you for your interest in this project!
