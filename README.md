# Library-Management-System-DBMS
“A DBMS based Library Management System using SQL | Includes ER Diagram, Data Requirements, Queries &amp; Sample Dataset.”
-- Library Management System SQL Script
-- Database: libproject

DROP DATABASE IF EXISTS libproject;
CREATE DATABASE libproject;
USE libproject;

-- Employees table
CREATE TABLE employee (
  employ_id INT AUTO_INCREMENT PRIMARY KEY,
  employ_name VARCHAR(100) NOT NULL,
  role VARCHAR(50),
  salary DECIMAL(10,2)
) ENGINE=InnoDB;

-- Branch table
CREATE TABLE branch (
  branch_no INT AUTO_INCREMENT PRIMARY KEY,
  manager_id INT NOT NULL,
  branch_address VARCHAR(255) NOT NULL,
  contact_no VARCHAR(20) NOT NULL,
  FOREIGN KEY (manager_id) REFERENCES employee(employ_id)
    ON DELETE RESTRICT ON UPDATE CASCADE
) ENGINE=InnoDB;

-- Books table
CREATE TABLE books (
  isbn INT NOT NULL PRIMARY KEY,
  book_title VARCHAR(200) NOT NULL,
  category VARCHAR(100) NOT NULL,
  rental_price DECIMAL(8,2) NOT NULL,
  status VARCHAR(50) DEFAULT 'available',
  author VARCHAR(150),
  publisher VARCHAR(150)
) ENGINE=InnoDB;

-- Customer table
CREATE TABLE customer (
  customer_id INT AUTO_INCREMENT PRIMARY KEY,
  customer_name VARCHAR(150) NOT NULL,
  customer_address VARCHAR(255),
  registration_date DATE NOT NULL
) ENGINE=InnoDB;

-- Issue status table (which book is issued to which customer)
CREATE TABLE issue_status (
  issue_id INT AUTO_INCREMENT PRIMARY KEY,
  issued_cust INT NOT NULL,
  issued_book_title VARCHAR(200) NOT NULL,
  issue_date DATE NOT NULL,
  isbn_book INT NOT NULL,
  FOREIGN KEY (issued_cust) REFERENCES customer(customer_id)
    ON DELETE RESTRICT ON UPDATE CASCADE,
  FOREIGN KEY (isbn_book) REFERENCES books(isbn)
    ON DELETE RESTRICT ON UPDATE CASCADE
) ENGINE=InnoDB;

-- Return status table
CREATE TABLE return_status (
  return_id INT AUTO_INCREMENT PRIMARY KEY,
  return_cust INT NOT NULL,
  returned_book_title VARCHAR(200) NOT NULL,
  return_date DATE NOT NULL,
  isbn_book2 INT NOT NULL,
  -- isbn_book2 references books.isbn
  FOREIGN KEY (isbn_book2) REFERENCES books(isbn)
    ON DELETE RESTRICT ON UPDATE CASCADE,
  -- return_cust should reference a customer who issued a book;
  -- referencing issue_status(issued_cust) directly is not ideal because issue_status can have multiple rows per customer.
  -- We will reference customer instead (ensure consistency in application logic).
  FOREIGN KEY (return_cust) REFERENCES customer(customer_id)
    ON DELETE RESTRICT ON UPDATE CASCADE
) ENGINE=InnoDB;

-- Sample data inserts (based on PPT but cleaned & corrected)
INSERT INTO employee (employ_name, role, salary) VALUES
  ('emp1','manager',30000.00),
  ('emp2','worker',10000.00),
  ('emp3','worker',10000.00),
  ('emp4','reader',20000.00),
  ('emp5','assist',20000.00);

INSERT INTO branch (manager_id, branch_address, contact_no) VALUES
  (1, 'branch_addr1', '9876543210'),
  (3, 'branch_addr3', '9876543323');

INSERT INTO books (isbn, book_title, category, rental_price, status, author, publisher) VALUES
  (1000, 'book1', 'comedy', 5.00, 'available', 'author1', 'pub1'),
  (1001, 'book2', 'scifi', 3.00, 'available', 'author2', 'pub2'),
  (1003, 'book3', 'romance', 1.00, 'unavailable', 'author3', 'pub3'),
  (1004, 'book4', 'thriller', 7.00, 'available', 'author4', 'pub4');

INSERT INTO customer (customer_name, customer_address, registration_date) VALUES
  ('cus1', 'hom1', '2008-10-10'),
  ('cus2', 'hom2', '2008-03-03'),
  ('cus3', 'hom3', '2009-03-03'),
  ('cus4', 'hom4', '2009-04-04');

-- Issue entries
INSERT INTO issue_status (issued_cust, issued_book_title, issue_date, isbn_book) VALUES
  (2, 'book1', '2010-01-01', 1001),
  (4, 'book4', '2010-01-01', 1004);

-- Return entries
INSERT INTO return_status (return_cust, returned_book_title, return_date, isbn_book2) VALUES
  (2, 'book1', '2010-10-01', 1001),
  (3, 'book1', '2010-10-01', 1001);

-- Example useful queries

-- 1) List all books with status
SELECT isbn, book_title, category, status, author, publisher FROM books;

-- 2) Get all customers who currently have issued books (join issue_status)
SELECT c.customer_id, c.customer_name, i.issue_id, i.issued_book_title, i.issue_date, b.isbn
FROM customer c
JOIN issue_status i ON c.customer_id = i.issued_cust
JOIN books b ON i.isbn_book = b.isbn;

-- 3) Check returns
SELECT r.return_id, r.return_cust, c.customer_name, r.returned_book_title, r.return_date
FROM return_status r
JOIN customer c ON r.return_cust = c.customer_id;

-- 4) Find books currently issued but not yet returned (requires application logic or tracking column).
-- A simple approximation: books issued but count of returns for same isbn by same customer is less than issued count.
-- (This is a conceptual query; for strict tracking add an issue_status.is_returned boolean or link issue_id in return_status.)

