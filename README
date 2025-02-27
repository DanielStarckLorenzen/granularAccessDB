# PostgreSQL Bookstore Database Documentation

## Database Overview

This document provides all necessary information to connect to and interact with the Bookstore database. The database manages a bookstore's inventory, customer information, orders, and order items with granular access controls.

## Local Setup Instructions

### Prerequisites
- Docker installed on your system
- PostgreSQL client (optional, but useful for testing)
- DBeaver or another database management tool (optional)

### Step 1: Pull and Run PostgreSQL Docker Image
```bash
# Pull the PostgreSQL image
docker pull postgres:latest

# Create a directory for persistent data
mkdir -p ~/postgres-data

# Run PostgreSQL container
docker run --name bookstore-db -e POSTGRES_PASSWORD=secure_password -e POSTGRES_USER=admin -p 5432:5432 -v ~/postgres-data:/var/lib/postgresql/data -d postgres:latest
```

### Step 2: Create Database and Schema
Connect to the PostgreSQL instance:
```bash
docker exec -it bookstore-db psql -U admin
```

Create the database and tables:
```sql
CREATE DATABASE bookstore;
\c bookstore

-- Books table
CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    title VARCHAR(255) NOT NULL,
    author VARCHAR(255) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    cost_price DECIMAL(10, 2) NOT NULL,
    stock INTEGER NOT NULL
);

-- Customers table
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL,
    phone VARCHAR(20),
    credit_card VARCHAR(255)
);

-- Orders table
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(id),
    date DATE NOT NULL,
    status VARCHAR(50) NOT NULL
);

-- Order items table
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(id),
    book_id INTEGER REFERENCES books(id),
    quantity INTEGER NOT NULL,
    price_at_order DECIMAL(10, 2) NOT NULL
);
```

### Step 3: Add Sample Data
```sql
-- Insert sample books
INSERT INTO books (title, author, price, cost_price, stock) VALUES 
('The Great Gatsby', 'F. Scott Fitzgerald', 12.99, 5.50, 100),
('To Kill a Mockingbird', 'Harper Lee', 14.99, 6.25, 85),
('1984', 'George Orwell', 11.99, 4.75, 120),
('Pride and Prejudice', 'Jane Austen', 9.99, 3.80, 75),
('The Hobbit', 'J.R.R. Tolkien', 19.99, 8.25, 60);

-- Insert sample customers
INSERT INTO customers (name, email, phone, credit_card) VALUES 
('John Smith', 'john@example.com', '555-123-4567', 'XXXX-XXXX-XXXX-1234'),
('Jane Doe', 'jane@example.com', '555-987-6543', 'XXXX-XXXX-XXXX-5678'),
('Bob Johnson', 'bob@example.com', '555-456-7890', 'XXXX-XXXX-XXXX-9012'),
('Alice Brown', 'alice@example.com', '555-789-0123', 'XXXX-XXXX-XXXX-3456');

-- Insert sample orders
INSERT INTO orders (customer_id, date, status) VALUES 
(1, '2023-10-01', 'Completed'),
(2, '2023-10-02', 'Processing'),
(3, '2023-10-03', 'Pending'),
(4, '2023-10-04', 'Cancelled'),
(1, '2023-10-05', 'Completed');

-- Insert sample order items
INSERT INTO order_items (order_id, book_id, quantity, price_at_order) VALUES 
(1, 1, 2, 12.99),
(1, 3, 1, 11.99),
(2, 2, 1, 14.99),
(3, 5, 1, 19.99),
(4, 4, 3, 9.99),
(5, 3, 2, 11.99);
```

### Step 4: Set Up Users and Roles with Granular Permissions
```sql
-- Enable row-level security on tables
ALTER TABLE books ENABLE ROW LEVEL SECURITY;
ALTER TABLE customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE order_items ENABLE ROW LEVEL SECURITY;

-- Create roles
CREATE ROLE sales_rep WITH LOGIN PASSWORD 'sales123';
CREATE ROLE customer_service WITH LOGIN PASSWORD 'service123';
CREATE ROLE inventory_manager WITH LOGIN PASSWORD 'inventory123';

-- Grant basic connection permissions
GRANT CONNECT ON DATABASE bookstore TO sales_rep, customer_service, inventory_manager;
GRANT USAGE ON SCHEMA public TO sales_rep, customer_service, inventory_manager;
GRANT USAGE ON ALL SEQUENCES IN SCHEMA public TO sales_rep, customer_service, inventory_manager;

-- SALES_REP permissions
-- Can see all book details except cost_price
-- Can see all customer data except credit_card
-- Can update book stock
CREATE VIEW books_sales_view AS 
SELECT id, title, author, price, stock FROM books;

GRANT SELECT ON books_sales_view TO sales_rep;
GRANT UPDATE (stock) ON books TO sales_rep;

CREATE VIEW customers_sales_view AS
SELECT id, name, email, phone FROM customers;

GRANT SELECT ON customers_sales_view TO sales_rep;
GRANT SELECT ON orders TO sales_rep;
GRANT SELECT ON order_items TO sales_rep;

-- CUSTOMER_SERVICE permissions
-- Can read all book details except cost_price
-- Can read customer name and email only
-- Can read and update order status
CREATE VIEW customers_service_view AS
SELECT id, name, email FROM customers;

GRANT SELECT ON books_sales_view TO customer_service;
GRANT SELECT ON customers_service_view TO customer_service;
GRANT SELECT ON orders TO customer_service;
GRANT UPDATE (status) ON orders TO customer_service;
GRANT SELECT ON order_items TO customer_service;

-- INVENTORY_MANAGER permissions
-- Can read and write all book data including cost_price
-- Cannot access customer or order data
GRANT SELECT, INSERT, UPDATE ON books TO inventory_manager;

-- Create and apply row-level security policies
-- For sales_rep
CREATE POLICY sales_books_policy ON books 
    FOR ALL
    TO sales_rep
    USING (true);

CREATE POLICY sales_customers_policy ON customers 
    FOR SELECT
    TO sales_rep
    USING (true);

CREATE POLICY sales_orders_policy ON orders 
    FOR SELECT
    TO sales_rep
    USING (true);

CREATE POLICY sales_order_items_policy ON order_items 
    FOR SELECT
    TO sales_rep
    USING (true);

-- For customer_service
CREATE POLICY service_books_policy ON books 
    FOR SELECT
    TO customer_service
    USING (true);

CREATE POLICY service_customers_policy ON customers 
    FOR SELECT
    TO customer_service
    USING (true);

CREATE POLICY service_orders_policy ON orders 
    FOR ALL
    TO customer_service
    USING (true);

CREATE POLICY service_order_items_policy ON order_items 
    FOR SELECT
    TO customer_service
    USING (true);

-- For inventory_manager
CREATE POLICY inventory_books_policy ON books 
    FOR ALL
    TO inventory_manager
    USING (true);
```

## Connection Details

- **Host**: localhost (or your host IP if deployed remotely)
- **Port**: 5432
- **Database Name**: bookstore
- **Admin User**: admin
- **Admin Password**: secure_password

## User Roles and Credentials

Three user roles have been created with different access permissions:

1. **Sales Representative**
   - Username: sales_rep
   - Password: sales123

2. **Customer Service**
   - Username: customer_service
   - Password: service123

3. **Inventory Manager**
   - Username: inventory_manager
   - Password: inventory123

## Database Schema

### Tables

1. **books**
   - id (SERIAL, PRIMARY KEY)
   - title (VARCHAR)
   - author (VARCHAR)
   - price (DECIMAL)
   - cost_price (DECIMAL)
   - stock (INTEGER)

2. **customers**
   - id (SERIAL, PRIMARY KEY)
   - name (VARCHAR)
   - email (VARCHAR)
   - phone (VARCHAR)
   - credit_card (VARCHAR)

3. **orders**
   - id (SERIAL, PRIMARY KEY)
   - customer_id (INTEGER, FOREIGN KEY to customers.id)
   - date (DATE)
   - status (VARCHAR)

4. **order_items**
   - id (SERIAL, PRIMARY KEY)
   - order_id (INTEGER, FOREIGN KEY to orders.id)
   - book_id (INTEGER, FOREIGN KEY to books.id)
   - quantity (INTEGER)
   - price_at_order (DECIMAL)

### Views

1. **books_sales_view**
   - Contains all book details except cost_price

2. **customers_sales_view**
   - Contains customer details except credit_card

3. **customers_service_view**
   - Contains only customer id, name, and email

## Access Permissions Matrix

| Resource | Sales Rep | Customer Service | Inventory Manager |
|----------|-----------|------------------|-------------------|
| books.title | READ | READ | READ/WRITE |
| books.author | READ | READ | READ/WRITE |
| books.price | READ | READ | READ/WRITE |
| books.cost_price | NO ACCESS | NO ACCESS | READ/WRITE |
| books.stock | READ/WRITE | READ | READ/WRITE |
| customers.name | READ | READ | NO ACCESS |
| customers.email | READ | READ | NO ACCESS |
| customers.phone | READ | NO ACCESS | NO ACCESS |
| customers.credit_card | NO ACCESS | NO ACCESS | NO ACCESS |
| orders.* | READ | READ/WRITE | NO ACCESS |
| order_items.* | READ | READ | NO ACCESS |

## Connection Instructions

### Using psql (PostgreSQL CLI)

To connect with any of the roles, use the following command format:

```bash
psql -h localhost -p 5432 -d bookstore -U <username>
```

Replace `<username>` with the appropriate role username (sales_rep, customer_service, or inventory_manager).

You will be prompted for the password. Enter the corresponding password from the credentials section.

### Using DBeaver

1. Download and install DBeaver from [https://dbeaver.io/download/](https://dbeaver.io/download/)

2. Open DBeaver and click on "New Database Connection" (plug icon with a plus sign)

3. Select PostgreSQL from the list and click "Next"

4. Enter connection details:
   - Host: localhost
   - Port: 5432
   - Database: bookstore
   - Username: (use one of the role usernames)
   - Password: (enter the corresponding password)

5. Click "Test Connection" to verify it works, then click "Finish"

6. You can create separate connections for each role to easily test different permissions

### Examples

#### Connecting as Sales Representative

```bash
psql -h localhost -p 5432 -d bookstore -U sales_rep
```

Password: sales123

#### Connecting as Customer Service

```bash
psql -h localhost -p 5432 -d bookstore -U customer_service
```

Password: service123

#### Connecting as Inventory Manager

```bash
psql -h localhost -p 5432 -d bookstore -U inventory_manager
```

Password: inventory123

## Testing Access Permissions

### Sales Representative

1. View books (should show all columns except cost_price):
   ```sql
   SELECT * FROM books_sales_view;
   ```

2. Try to view full books table (should fail on cost_price column):
   ```sql
   SELECT * FROM books;
   ```

3. Update book stock (should succeed):
   ```sql
   UPDATE books SET stock = 95 WHERE id = 1;
   ```

4. Try to update book price (should fail):
   ```sql
   UPDATE books SET price = 13.99 WHERE id = 1;
   ```

5. View customer data (should show all except credit_card):
   ```sql
   SELECT * FROM customers_sales_view;
   ```

6. Try to view credit cards (should fail):
   ```sql
   SELECT credit_card FROM customers;
   ```

### Customer Service

1. View customer data (limited to name and email):
   ```sql
   SELECT * FROM customers_service_view;
   ```

2. Try to view customer phone (should fail):
   ```sql
   SELECT phone FROM customers;
   ```

3. Update order status (should succeed):
   ```sql
   UPDATE orders SET status = 'Shipped' WHERE id = 2;
   ```

4. Try to modify book data (should fail):
   ```sql
   UPDATE books SET stock = 90 WHERE id = 1;
   ```

### Inventory Manager

1. View and update book data including cost_price (should succeed):
   ```sql
   SELECT * FROM books;
   UPDATE books SET cost_price = 5.75 WHERE id = 1;
   ```

2. Try to access customer data (should fail):
   ```sql
   SELECT * FROM customers;
   ```

3. Try to access order data (should fail):
   ```sql
   SELECT * FROM orders;
   ```

## Troubleshooting

If you encounter connection issues:
1. Ensure the database is running and accessible from your network
2. Verify you're using the correct username and password
3. Check that your IP is allowed to connect to the database

For any permission-related issues, please refer to the Access Permissions Matrix to verify what operations should be allowed for your role.
