# Online Bookstore SQL Analysis

## Project Overview

This project uses MySQL to analyze an online bookstore's sales, customers and inventory. It is built on three related tables (`Books`, `Customers` and `Orders`, 500 records each) and contains 20 queries, moving from basic filtering to multi-table joins, aggregation and a stock calculation.

## Business Objectives

- **Sales performance:** units sold by genre and author, most frequently ordered book
- **Customer behavior:** repeat customers, highest spender, customer locations
- **Pricing and revenue:** most expensive books, average Fantasy price, total revenue
- **Inventory:** lowest stock and estimated remaining stock after all orders

## Dataset and Schema

| Table | Purpose | Key |
|---|---|---|
| `Books` | Catalogue: title, author, genre, year, price, stock | PK `Book_ID` |
| `Customers` | Customer name, contact details, city, country | PK `Customer_ID` |
| `Orders` | One book per order: date, quantity, total amount | PK `Order_ID`; FK `Customer_ID`, FK `Book_ID` |

`Orders` links the other two tables through `Customer_ID` and `Book_ID`. The script declares these references inline, but MySQL ignores that syntax, so the relationships are logical only (see Setup). The source brief is `SQL Project Questions Day 30.pdf`.

## SQL Skills Demonstrated

- **Filtering and sorting:** `WHERE`, `BETWEEN`, `DISTINCT`, `ORDER BY`, `LIMIT`
- **Joins:** `JOIN` across all three tables; `LEFT JOIN` to retain unordered books
- **Aggregation:** `GROUP BY`, `HAVING`, `SUM`, `COUNT`, `AVG`
- **Calculated fields:** `COALESCE` for the remaining-stock calculation
- **Schema design:** `CREATE TABLE` with primary keys, `SERIAL` and `NUMERIC`

## Key Findings and Business Insights

The 20 queries in `sales store.sql` were executed against the supplied CSVs on MySQL 8.0 (default collation, Windows-style table-name handling) and all ran without errors. I have not tested other MySQL versions or MySQL Workbench itself. Column 2 holds calculated facts; column 3 is my interpretation, not a proven cause.

| Business question | Calculated result | Interpretation |
|---|---|---|
| How large is the business? | 500 orders, 2,697 units, $75,628.66 revenue, about $151 per order | A small dataset; figures describe this sample only. |
| Which genres drive demand? | Mystery 504, Science Fiction 447, Fantasy 446, Romance 439 units (68% of all units). Fiction is lowest at 225 | Demand is concentrated in four genres; Fiction underperforms despite 60 titles. |
| How loyal are customers? | 307 of 500 customers have ordered; 139 of them (45%) ordered more than once | Repeat purchasing is substantial among buyers, but 193 customers have no orders. |
| Is revenue dependent on a few customers? | The top customer spent $1,398.90, 1.8% of revenue | Revenue is spread widely; no single-customer dependence. |
| Does stock match demand? | 30 books have more units ordered than recorded stock (worst: -18). 183 books have no orders. Ordered units equal 11% of total recorded stock | Stock is uneven: some titles are oversold while many are untouched. Negative values may also reflect unrecorded restocking. |

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

Requires a MySQL server (tested on 8.0) and a client such as MySQL Workbench or the `mysql` command line.

1. **Create and select the database.** The script has `CREATE DATABASE` but no `USE` statement, so add it:
   ```sql
   CREATE DATABASE OnlineBookstore;
   USE OnlineBookstore;
   ```
2. **Create the tables** by running the `DROP TABLE` / `CREATE TABLE` statements from `sales store.sql`.
3. **Import the CSVs** into `Books`, `Customers` and `Orders`. With the Workbench *Table Data Import Wizard*, or with `LOAD DATA` (replace the placeholder path; shown for `Customers`, repeat for the other two):
   ```sql
   LOAD DATA LOCAL INFILE '<path-to>/Customers.csv' INTO TABLE Customers
   FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
   LINES TERMINATED BY '\r\n' IGNORE 1 LINES;
   ```
4. **Run the analysis queries** in `sales store.sql`.

**Setup issues to be aware of**

- **No database selected:** running the script without `USE OnlineBookstore;` fails with error 1046 (verified).
- **Foreign keys are not created:** MySQL parses inline `col INT REFERENCES table(col)` but does not create the constraint. I confirmed the script produces 0 foreign keys. All `Orders` IDs do match existing books and customers, but the database does not enforce it. Adding explicit `FOREIGN KEY` clauses would also require matching column types, because `SERIAL` is `BIGINT UNSIGNED` in MySQL while the `Orders` key columns are `INT` (not tested here).
- **Line endings in `Customers.csv`:** the file uses Windows line endings. `LOAD DATA` with the default `'\n'` terminator stores a hidden carriage return at the end of every `Country` value, and the Canada filter then returns no rows (verified). Use `'\r\n'` as shown above.
- **`LOAD DATA LOCAL`:** requires `local_infile` to be enabled on the server and in the client; it is off by default on MySQL 8.0 servers.

## Data Quality and Limitations

No null values were found, and `Total_Amount` equals `Quantity × Price` in every order. The issues below concern how the original queries behave. Corrected figures are separate from the original outputs and are **not** produced by the script.

| Query | Issue | Original output (MySQL 8.0) | Corrected analysis |
|---|---|---|---|
| Basic Q3 (customers from Canada) | Filters on `'canada'`, but the data stores `'Canada'`. It works only because the default MySQL collation is case-insensitive | 3 rows | Matching the stored value `'Canada'` makes the query portable across collations |
| Advanced Q4 (most frequently ordered book) | Ranks by order count, not units. `LIMIT 1` hides a 7-way tie at 4 orders, so the book returned is not guaranteed (my MySQL run and my earlier SQLite check returned different books) | 1 book | The tied books sold between 15 and 28 units; ranking by `SUM(quantity)` gives a single top book (28 units) |
| Basic Q10 (lowest stock) | `LIMIT 1` hides a tie | 1 book | 5 books have 0 stock |
| Advanced Q7 (cities of customers who spent over $30) | Selects `DISTINCT city, total_amount` and filters single orders, not cumulative spend | 443 city-amount rows | 283 distinct cities (single order over $30); 284 if cumulative spend over $30 is intended |
| Advanced Q9 (remaining stock) | Assumes `Stock` is opening inventory and `Orders` captures all movements | 30 negative balances | Needs restocking data to resolve |

Revenue is the recorded `Total_Amount`; the data contains no discounts, returns or cancellations.

## Conclusion and Author

This project covers relational schema design, multi-table joins, aggregation and an inventory calculation. Checking query output for ties, case handling and mismatches with the question asked is as important as writing the query, and the limitations section reflects that. The analysis is descriptive and based on a sample dataset.

**Author:** Atul Chauhan
