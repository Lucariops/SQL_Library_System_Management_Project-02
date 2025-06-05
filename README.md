# SQL_Library_System_Management_Project-02

## Project Overview

**Project Title**: Library Management System  
**Level**: Intermediate  
**Database**: `SQL_Project_02`

This project demonstrates the implementation of a Library Management System using SQL. It includes creating and managing tables, performing CRUD operations, and executing advanced SQL queries. The goal is to showcase skills in database design, manipulation, and querying.

![Library_project](https://github.com/Lucariops/SQL_Library_System_Management_Project-02/blob/main/library.jpg)

## Objectives

1. **Set up the Library Management System Database**: Create and populate the database with tables for branches, employees, members, books, issued status, and return status.
2. **CRUD Operations**: Perform Create, Read, Update, and Delete operations on the data.
3. **CTAS (Create Table As Select)**: Utilize CTAS to create new tables based on query results.
4. **Advanced SQL Queries**: Develop complex queries to analyze and retrieve specific data.

## Project Structure

### 1. Database Setup
![ERD](https://github.com/Lucariops/SQL_Library_System_Management_Project-02/blob/main/ERD%20Model.png)

-- Database Creation
CREATE DATABASE SQL_Project_02;

-- Drop Tables

``` sql
DROP TABLE IF EXISTS Return_Status;
DROP TABLE IF EXISTS Issued_Status;
DROP TABLE IF EXISTS Members;
DROP TABLE IF EXISTS Employees;
DROP TABLE IF EXISTS Branch;
DROP TABLE IF EXISTS Books;

-- Create Books Table

CREATE TABLE Books (
    isbn VARCHAR(50) PRIMARY KEY,
    book_title VARCHAR(100),
    category VARCHAR(50),
    rental_price DECIMAL(10,2),
    status VARCHAR(10),
    author VARCHAR(50),
    publisher VARCHAR(50)
);

-- Create Branch Table

CREATE TABLE Branch (
    branch_id VARCHAR(10) PRIMARY KEY,
    manager_id VARCHAR(10),
    branch_address VARCHAR(30),
    contact_no VARCHAR(20)
);

-- Create Employees Table

CREATE TABLE Employees (
    emp_id VARCHAR(10) PRIMARY KEY,
    emp_name VARCHAR(30),
    position VARCHAR(30),
    salary DECIMAL(10,2),
    branch_id VARCHAR(10),
	FOREIGN KEY (branch_id) REFERENCES  Branch(branch_id)
);

-- Create Members Table

CREATE TABLE Members (
    member_id VARCHAR(10) PRIMARY KEY,
    member_name VARCHAR(30),
    member_address VARCHAR(30),
    reg_date DATE
);

-- Create Issued_Status Table

CREATE TABLE Issued_Status (
    issued_id VARCHAR(10) PRIMARY KEY,
    issued_member_id VARCHAR(10),
    issued_book_name VARCHAR(100),
    issued_date DATE,
    issued_book_isbn VARCHAR(50),
    issued_emp_id VARCHAR(10),
	FOREIGN KEY (issued_member_id) REFERENCES Members(member_id),
    FOREIGN KEY (issued_emp_id) REFERENCES Employees(emp_id),
    FOREIGN KEY (issued_book_isbn) REFERENCES Books(isbn)
);

-- Create Return_Status Table

CREATE TABLE Return_Status (
    return_id VARCHAR(10) PRIMARY KEY,
    issued_id VARCHAR(10),
    return_book_name VARCHAR(100),
    return_date DATE,
    return_book_isbn VARCHAR(50),
    FOREIGN KEY (issued_id) REFERENCES Issued_Status(issued_id)
);
```

-- Project TASK

-- ### 2. CRUD Operations
-- Task 1. Create a New Book Record
-- "978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.')"

``` sql
INSERT INTO Books (isbn, book_title, category, rental_price, status, author, publisher)
VALUES ('978-1-60129-456-2', 'To Kill a Mockingbird', 'Classic', 6.00, 'yes', 'Harper Lee', 'J.B. Lippincott & Co.');
```

-- Task 2: Update an Existing Member's Address

``` sql
UPDATE Members
SET member_address = '69 Main Street'
WHERE member_id = 'C101';
```

-- Task 3: Delete a Record from the Issued Status Table
-- Objective: Delete the record with issued_id = 'IS140' from the issued_status table.

``` sql
DELETE FROM Issued_Status
WHERE issued_id = 'IS140';
```

-- Task 4: Retrieve All Books Issued by a Specific Employee
-- Objective: Select all books issued by the employee with emp_id = 'E101'.

``` sql
SELECT issued_book_name, issued_book_isbn, issued_date, issued_member_id
FROM Issued_Status
WHERE issued_emp_id = 'E101';
```

-- Task 5: List Members Who Have Issued More Than One Book
-- Objective: Use GROUP BY to find members who have issued more than one book.

``` sql
SELECT issued_member_id, COUNT(*) AS total_books_issued
FROM Issued_Status
GROUP BY issued_member_id
HAVING COUNT(*) > 1;
```

-- ### 3. CTAS (Create Table As Select)

-- Task 6: Create Summary Tables**: Used CTAS to generate new tables based on query results - each book and total book_issued_cnt

``` sql
CREATE TABLE Book_Issue_Count AS
SELECT issued_book_isbn, issued_book_name, COUNT(*) AS total_issued_cnt
FROM Issued_Status
GROUP BY 1, 2;

SELECT * FROM Book_Issue_Count;
```

-- ### 4. Data Analysis & Findings

-- Task 7. **Retrieve All Books in a Specific Category:

``` sql
SELECT * FROM Books
WHERE category = 'History';
```

-- Task 8: Find Total Rental Income by Category:

```sql
SELECT category, SUM(rental_price) AS total_rental_income, Count(*)
FROM Books
GROUP BY category;
```

-- Task 9. **List Members Who Registered in the Last 720 Days**:

```sql
SELECT * FROM Members
WHERE reg_date >= CURRENT_DATE - INTERVAL '720 Days';
```

-- Task 10: List Employees with Their Branch Manager's Name and their branch details**:

``` sql
SELECT e.emp_id, e.emp_name, e.position, e.salary, 
       b.branch_id, b.branch_address, b.contact_no, 
       m.emp_name AS manager_name
FROM Employees e
JOIN Branch b ON e.branch_id = b.branch_id
LEFT JOIN Employees m ON b.manager_id = m.emp_id;
```

-- Task 11. Create a Table of Books with Rental Price Above a Certain Threshold

```sql
CREATE TABLE Expensive_Books AS
SELECT *
FROM Books
WHERE rental_price > 5.50;

SELECT * FROM Expensive_Books;
```

-- Task 12: Retrieve the List of Books Not Yet Returned

``` sql
SELECT i.issued_book_name, i.issued_book_isbn, i.issued_member_id, i.issued_date
FROM Issued_Status i
LEFT JOIN Return_Status r ON i.issued_id = r.issued_id
WHERE r.return_id IS NULL;
```

-- ### Advanced SQL Operations

-- Task 13: Identify Members with Overdue Books
-- Write a query to identify members who have overdue books (assume a 30-day return period). Display the member's name, book title, issue date, and days overdue.

```sql
SELECT m.member_name, i.issued_book_name, i.issued_date, 
       (CURRENT_DATE - i.issued_date) - 30 AS days_overdue
FROM Issued_Status i
JOIN Members m ON i.issued_member_id = m.member_id
LEFT JOIN Return_Status r ON i.issued_id = r.issued_id
WHERE r.return_id IS NULL AND (CURRENT_DATE - i.issued_date) > 30;
```

-- Task 14: Update Book Status on Return
-- Write a query to update the status of books in the books table to "available" when they are returned (based on entries in the return_status table).

``` sql
CREATE OR REPLACE PROCEDURE add_return_records(p_return_id VARCHAR(10), p_issued_id VARCHAR(10))
LANGUAGE plpgsql
AS $$

DECLARE
    v_isbn VARCHAR(50);
    v_book_name VARCHAR(80);
    
BEGIN
    -- all your logic and code
    -- inserting into returns based on users input
    INSERT INTO return_status(return_id, issued_id, return_date)
    VALUES
    (p_return_id, p_issued_id, CURRENT_DATE);

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

issued_id = IS136
ISBN = WHERE isbn = '978-0-7432-7357-1'

SELECT * FROM books
WHERE isbn = '978-0-7432-7357-1';

SELECT * FROM issued_status
WHERE issued_book_isbn = '978-0-7432-7357-1';

SELECT * FROM return_status
WHERE issued_id = 'IS136';

-- calling function 
CALL add_return_records('RS138', 'IS135');

-- calling function 
CALL add_return_records('RS148', 'IS136');
```

-- Task 15: Branch Performance Report
-- Create a query that generates a performance report for each branch, showing the number of books issued, the number of books returned, and the total revenue generated from book rentals.

``` sql
CREATE TABLE Branch_Reports
AS
SELECT 
    b.branch_id,
    COUNT(DISTINCT i.issued_id) AS total_books_issued,
    COUNT(DISTINCT r.return_id) AS total_books_returned,
    SUM(bo.rental_price) AS total_revenue_generated
FROM Branch b
LEFT JOIN Employees e ON b.branch_id = e.branch_id
LEFT JOIN Issued_Status i ON e.emp_id = i.issued_emp_id
LEFT JOIN Return_Status r ON i.issued_id = r.issued_id
LEFT JOIN Books bo ON i.issued_book_isbn = bo.isbn
GROUP BY 1
ORDER BY total_revenue_generated DESC;

SELECT * FROM Branch_Reports;
```

-- Task 16: CTAS: Create a Table of Active Members
-- Use the CREATE TABLE AS (CTAS) statement to create a new table active_members containing members who have issued at least one book in the last 16 months.

``` sql
DROP TABLE IF EXISTS Active_Members;

CREATE TABLE Active_Members AS
SELECT DISTINCT m.member_id, m.member_name, m.member_address, m.reg_date
FROM Members m
JOIN Issued_Status i ON m.member_id = i.issued_member_id
WHERE i.issued_date >= CURRENT_DATE - INTERVAL '16 MONTH'
ORDER BY member_id ASC;

SELECT * FROM Active_Members;
```

-- Task 17: Find Employees with the Most Book Issues Processed
-- Write a query to find the top 3 employees who have processed the most book issues. Display the employee name, number of books processed, and their branch.

``` sql
SELECT 
    e.emp_name,
    e.branch_id,
    COUNT(i.issued_id) AS books_processed
FROM Employees e
JOIN Issued_Status i ON e.emp_id = i.issued_emp_id
GROUP BY 1, 2
ORDER BY books_processed DESC
LIMIT 3;
```

-- Task 18: Stored Procedure
-- Objective: Create a stored procedure to manage the status of books in a library system.
--     Description: Write a stored procedure that updates the status of a book based on its issuance or return. Specifically:
--     If a book is issued, the status should change to 'no'.
--     If a book is returned, the status should change to 'yes'.

``` sql
CREATE OR REPLACE PROCEDURE Issue_Book(p_issued_id VARCHAR(10), p_issued_member_id VARCHAR(30), p_issued_book_isbn VARCHAR(30), p_issued_emp_id VARCHAR(10))
LANGUAGE plpgsql
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

        INSERT INTO issued_status(issued_id, issued_member_id, issued_book_name, issued_date, issued_book_isbn, issued_emp_id)
        VALUES
        (p_issued_id, p_issued_member_id, 'DEMON SLAYER', CURRENT_DATE, p_issued_book_isbn, p_issued_emp_id);

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
SELECT * FROM books;
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

## How to Use

1. **Clone the Repository**: Clone this repository to your local machine.
   ```sh
   git clone https://github.com/najirh/Library-System-Management---P2.git
   ```

2. **Set Up the Database**: Execute the SQL scripts in the `Database Setup Section` file to create and populate the database.
3. **Run the Queries**: Use the SQL queries in the `CRUD Section, CTAS Section and Data Analysis & Findings Section ` file to perform the analysis.
4. **Explore and Modify**: Customize the queries as needed to explore different aspects of the data or answer additional questions.

## Author - Pranav Shah
