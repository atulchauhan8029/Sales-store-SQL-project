# Online Bookstore SQL Analysis



A SQL analysis of an online bookstore's catalogue, customers and orders, covering sales by genre and author, customer purchasing behaviour, revenue and stock levels. The project uses three related tables and 20 queries, progressing from basic filtering to multi-table joins, aggregation and an inventory calculation.

## Business Problem

An online bookstore needs a reliable view of what sells, who buys, how much revenue is generated and whether recorded stock can cover demand. The raw data sits in three separate tables (books, customers and orders), so answering these questions requires joining and aggregating them correctly. This project translates a set of operational questions into SQL and documents what each query does, what it returns and where its logic has limits.

## Business Objectives

| Objective | Related analysis |
|---|---|
| Understand catalogue composition and pricing | Genre filters, publication year, most expensive books, average Fantasy price |
| Measure sales performance | Units sold by genre and author, total revenue, most frequently ordered book |
| Understand customer behaviour | Repeat customers, highest-spending customer, customers by country and city |
| Assess inventory position | Lowest stock, total stock, estimated remaining stock after orders |

## Technology Stack

| Component | Detail |
|---|---|
| Database | PostgreSQL (the script uses `SERIAL` and `NUMERIC`) |
| Language | SQL |
| Data source | Three CSV files: `Books.csv`, `Customers.csv`, `Orders.csv` |
| Client | Any PostgreSQL client, such as `psql` or pgAdmin |

## Dataset Overview

Each table contains 500 records.

### `Books`

| Column | Type | Description |
|---|---|---|
| `Book_ID` | `SERIAL`, primary key | Unique identifier for each book |
| `Title` | `VARCHAR(100)` | Book title |
| `Author` | `VARCHAR(100)` | Book author |
| `Genre` | `VARCHAR(50)` | Book category (7 genres) |
| `Published_Year` | `INT` | Year of publication |
| `Price` | `NUMERIC(10,2)` | Unit price |
| `Stock` | `INT` | Recorded stock quantity |

### `Customers`

| Column | Type | Description |
|---|---|---|
| `Customer_ID` | `SERIAL`, primary key | Unique identifier for each customer |
| `Name` | `VARCHAR(100)` | Customer name |
| `Email` | `VARCHAR(100)` | Email address |
| `Phone` | `VARCHAR(15)` | Phone number |
| `City` | `VARCHAR(50)` | City |
| `Country` | `VARCHAR(150)` | Country |

### `Orders`

| Column | Type | Description |
|---|---|---|
| `Order_ID` | `SERIAL`, primary key | Unique identifier for each order record |
| `Customer_ID` | `INT`, foreign key | References `Customers(Customer_ID)` |
| `Book_ID` | `INT`, foreign key | References `Books(Book_ID)` |
| `Order_Date` | `DATE` | Date of the order |
| `Quantity` | `INT` | Number of copies ordered |
| `Total_Amount` | `NUMERIC(10,2)` | Recorded order total |

### Relationships

- `Orders.Customer_ID` references `Customers.Customer_ID` (many orders per customer).
- `Orders.Book_ID` references `Books.Book_ID` (many orders per book).
- `Orders` is the transaction table linking the two. Each order record holds a single book, so one row in `Orders` corresponds to one book line, not a multi-item basket.

## Business Questions Answered

The questions below are taken from the project brief (`SQL Project Questions Day 30.pdf`), numbered as in the SQL script and grouped by theme.

### Basic Analysis

| Theme | Questions |
|---|---|
| Catalogue and inventory | 1) Retrieve all books in the "Fiction" genre<br>2) Find books published after the year 1950<br>5) Retrieve the total stock of books available<br>6) Find the details of the most expensive book<br>9) List all genres available in the Books table<br>10) Find the book with the lowest stock |
| Customers | 3) List all customers from the Canada |
| Orders and revenue | 4) Show orders placed in November 2023<br>7) Show all customers who ordered more than 1 quantity of a book<br>8) Retrieve all orders where the total amount exceeds $20<br>11) Calculate the total revenue generated from all orders |

### Advanced Analysis

| Theme | Questions |
|---|---|
| Sales performance | 1) Retrieve the total number of books sold for each genre<br>4) Find the most frequently ordered book<br>6) Retrieve the total quantity of books sold by each author |
| Pricing | 2) Find the average price of books in the "Fantasy" genre<br>5) Show the top 3 most expensive books of 'Fantasy' Genre |
| Customer behaviour | 3) List customers who have placed at least 2 orders<br>7) List the cities where customers who spent over $30 are located<br>8) Find the customer who spent the most on orders |
| Inventory | 9) Calculate the stock remaining after fulfilling all orders |

## SQL Techniques Demonstrated

| Technique | How it is used in this project |
|---|---|
| `SELECT`, `WHERE` | Filter books by genre and year, customers by country, orders by quantity, amount and date |
| `BETWEEN` | Restrict orders to a date range (November 2023) |
| `DISTINCT` | List unique genres and cities |
| `ORDER BY`, `LIMIT` | Return the most expensive book, lowest-stock book, top 3 Fantasy books and top-ranked results |
| `JOIN` | Combine `Orders` with `Books` and `Customers` to analyse sales by genre, author and customer |
| `LEFT JOIN` | Keep every book in the stock calculation, including books with no orders |
| `GROUP BY` | Aggregate by genre, author, customer and book |
| `HAVING` | Keep only customers with at least two orders |
| `SUM()` | Total stock, total revenue, units sold, customer spend |
| `COUNT()` | Number of orders per customer and per book |
| `AVG()` | Average Fantasy book price |
| `COALESCE()` | Treat books with no orders as zero ordered quantity |
| DDL | `CREATE TABLE`, `DROP TABLE IF EXISTS`, primary and foreign key constraints, `SERIAL`, `NUMERIC` |

## Project Structure

```text
.
├── Books.csv
├── Customers.csv
├── Orders.csv
├── sales store.sql
├── SQL Project Questions Day 30.pdf
└── README.md
```

## Setup and Execution

**Prerequisites:** a PostgreSQL server and a client such as `psql` or pgAdmin.

The script creates the database and tables in one file, so the steps below run it in stages. Running the whole file in a single pass would create the tables in whichever database you are connected to, not `OnlineBookstore`.

**1. Create the database**

```sql
CREATE DATABASE OnlineBookstore;
```

**2. Connect to it**

In `psql`: `\c onlinebookstore` (PostgreSQL folds unquoted names to lowercase). In pgAdmin, open a Query Tool on the `OnlineBookstore` database.

**3. Create the tables**

Run the `DROP TABLE IF EXISTS` and `CREATE TABLE` statements from `sales store.sql`. The tables must exist before any data is imported.

**4. Import the CSV files**

Load the files in this order so that foreign keys resolve: `Books`, `Customers`, `Orders`. In `psql`, replace the placeholder with the location of the files on your machine:

```sql
\copy Books     FROM '<path-to>/Books.csv'     WITH (FORMAT csv, HEADER true)
\copy Customers FROM '<path-to>/Customers.csv' WITH (FORMAT csv, HEADER true)
\copy Orders    FROM '<path-to>/Orders.csv'    WITH (FORMAT csv, HEADER true)
```

In pgAdmin, the same import is available through *Import/Export Data* on each table (CSV format, header enabled).

**5. Run the analysis queries**

Execute the `SELECT` statements in `sales store.sql`. The three `SELECT *` statements directly after the table definitions return empty results if run before the import.

**Re-running the script**

The script drops `Books` before `Orders`, but `Orders` has a foreign key to `Books`, so PostgreSQL will reject the drop on a second run while `Orders` exists. Drop the tables in the order `Orders`, `Customers`, `Books` (or use `DROP TABLE ... CASCADE`) before re-creating them.

**Sequences after import**

The CSV files supply explicit ID values, so the `SERIAL` sequences are not advanced by the import. This has no effect on the analysis queries, but would matter if new rows were inserted afterwards.

## Key Findings and Business Insights

**How these results were obtained.** Each query in `sales store.sql` was executed against the supplied CSV files and the outputs were cross-checked independently with Python. This was done in a SQLite environment, not a PostgreSQL instance, so re-running the script in PostgreSQL is recommended to confirm the output on your own setup.

### Basic queries

| # | Query | Result |
|--:|---|---|
| 1 | Books in the Fiction genre | 60 books |
| 2 | Books published after 1950 | 292 books |
| 3 | Customers from Canada | **0 rows as written** (see limitations); 3 customers when the country match is case-insensitive |
| 4 | Orders placed in November 2023 | 25 orders |
| 5 | Total stock | 25,056 units |
| 6 | Most expensive book | *Proactive system-worthy orchestration* by Robert Scott (Mystery), $49.98 |
| 7 | Orders with quantity above 1 | 438 orders |
| 8 | Orders with total amount above $20 | 473 orders |
| 9 | Genres in the Books table | 7 genres |
| 10 | Lowest stock | 0 units; 5 books are tied at zero, so the single row returned is not uniquely determined |
| 11 | Total revenue from all orders | $75,628.66 |

### Advanced queries

| # | Query | Result |
|--:|---|---|
| 1 | Units sold by genre | Mystery 504, Science Fiction 447, Fantasy 446, Romance 439, Non-Fiction 351, Biography 285, Fiction 225 |
| 2 | Average Fantasy price | $25.98 |
| 3 | Customers with at least 2 orders | 139 customers (maximum 6 orders for one customer) |
| 4 | Most frequently ordered book | Highest order count is 4, shared by 7 books; the query returns only one of them |
| 5 | Top 3 Fantasy books by price | *Stand-alone content-based hub* ($49.90), *Innovative 3rdgeneration database* ($49.23), *Optimized even-keeled analyzer* ($48.97) |
| 6 | Units sold by author | 314 authors returned; highest are Patrick Contreras (28), Melissa Taylor (27), Emily James and Thomas Trujillo (24 each) |
| 7 | Cities of customers with an order above $30 | 443 rows as written (city and amount pairs) covering 283 distinct cities |
| 8 | Highest-spending customer | Kim Turner (Customer ID 457), $1,398.90 |
| 9 | Remaining stock after orders | 500 rows; 30 books show negative remaining stock (lowest -18); 183 books have no orders |

### Observations

- The dataset records 500 orders, 2,697 units and $75,628.66 in revenue, an average of about $151 per order.
- Mystery is the highest-selling genre by units, followed closely by Science Fiction, Fantasy and Romance. Fiction is the lowest at 225 units.
- 307 of the 500 customers have placed at least one order, and 139 of them have ordered more than once.
- In 30 cases, the quantity ordered exceeds the recorded stock. In this dataset, that points to either a data inconsistency or unrecorded restocking, and should be investigated before any stock decision is made.

These are descriptive observations of the supplied data. They do not establish causes or forecast future performance.

## Data Quality and Analytical Limitations

**Query-specific limitations**

- **Case-sensitive country filter (Basic Q3).** The script filters on `'canada'`, but the stored value is `'Canada'`. PostgreSQL string comparison is case-sensitive, so the query as written returns no rows. `LOWER(country) = 'canada'` or `ILIKE` would return the 3 matching customers.
- **Order frequency versus quantity sold (Advanced Q4).** The query counts order records per book, not units sold. A book ordered twice in quantities of 10 ranks below a book ordered three times in quantities of 1. The question could equally be answered with `SUM(quantity)`, and the two methods can give different rankings.
- **Ties hidden by `LIMIT 1` (Basic Q10, Advanced Q4).** Both queries return a single row although several rows tie for first place (5 books at zero stock; 7 books at 4 orders). Which tied row appears is not guaranteed. A `RANK()` window function or a subquery on the maximum value would return all of them.
- **City list returns amounts too (Advanced Q7).** The query selects `DISTINCT c.city, total_amount`, so it returns unique city and amount pairs (443 rows) instead of a list of cities (283). Removing `total_amount` from the `SELECT` list would match the stated question.
- **Single-order threshold (Advanced Q7).** The query filters individual orders above $30. It does not identify customers whose total spend exceeds $30, which would need `GROUP BY` with `HAVING SUM(total_amount) > 30`.
- **Author totals cover ordered books only (Advanced Q6).** The inner join excludes authors whose books were never ordered (314 authors returned against 493 in the catalogue).

**Assumptions in the calculations**

- **Revenue** is the sum of the recorded `Total_Amount`. In the supplied data this equals `Quantity × Price` for every order, but the dataset has no information on discounts, tax, shipping, returns or cancellations.
- **Remaining stock** assumes `Books.Stock` is the opening inventory and that `Orders` captures every stock movement. Negative results show that this assumption does not hold for 30 books.
- **Pricing** uses the current price in `Books`. Historical price changes are not tracked.

**Data checks performed**

- No null values in any table.
- Primary keys are unique, and every `Customer_ID` and `Book_ID` in `Orders` exists in its parent table.
- Order dates run from December 2022 to December 2024.

## Conclusion

This project demonstrates the core SQL skills used in day-to-day data analysis: designing a relational schema, joining several tables, aggregating at different levels, and checking whether a query actually answers the business question it was written for. Reviewing the output, including ties, case sensitivity and unmatched records, is as much a part of the work as writing the query. The same approach carries over to real analyst tasks such as sales reporting, customer segmentation and inventory monitoring.

## Author

**Atul Chauhan**
