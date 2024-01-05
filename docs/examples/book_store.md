# Omnigres Bookstore Integration Showcase

## Step 1: Create a Bookstore Database

```sql
-- Create a Books table
CREATE TABLE books (
    book_id SERIAL PRIMARY KEY,
    title TEXT,
    author TEXT,
    price NUMERIC,
    published_date DATE
);

-- Insert some sample data
INSERT INTO books (title, author, price, published_date) VALUES
    ('The Great Gatsby', 'F. Scott Fitzgerald', 10.99, '1925-04-10'),
    ('To Kill a Mockingbird', 'Harper Lee', 8.50, '1960-07-11'),
    ('1984', 'George Orwell', 12.75, '1949-06-08');
```

## Step 3: Create Python Functions for Price Manipulation

Create a Python file, e.g., `price_functions.py`, with the following content:

```python
# price_functions.py
from omni_python import pg

@pg
def calculate_discounted_price(price: float) -> float:
    # Perform price manipulation (e.g., apply a 10% discount)
    discounted_price = price * 0.9
    return discounted_price

@pg
def mask_price(price: float) -> float:
    # Perform additional price masking or manipulation
    masked_price = round(price, 2)
    return masked_price
```

## Step 4: Configure Virtual File System (vfs)

```sql
-- Configure the virtual file system for Python files
CREATE OR REPLACE FUNCTION demo_function()
RETURNS omni_vfs.local_fs LANGUAGE SQL AS $$
SELECT omni_vfs.local_fs('/demo')
$$;
```

```sql
-- Configure omni_python to use local Python packages
INSERT INTO omni_python.config (name, value) VALUES ('pip_find_links', '/python-wheels');

-- Load Python Functions into the Database
SELECT omni_schema.load_from_fs(demo_function());
```

**Explanation:**
- The virtual file system (vfs) is configured to include Python files from the `/demo` directory.
- The `omni_python` extension is set to find Python packages in the `/python-wheels` directory.
- Python functions are loaded into the database from the configured file system.

## Step 5: Run Docker Container with Omnigres

**DOCKER RUN:**
```bash
docker run --name omnigres \
           -e POSTGRES_PASSWORD=omnigres \
           -e POSTGRES_USER=omnigres \
           -e POSTGRES_DB=omnigres \
           --mount source=omnigres,target=/var/lib/postgresql/data \
           -v $(pwd)/demo:/python-files \
           -p 127.0.0.1:5433:5432 --rm ghcr.io/omnigres/omnigres-slim:latest
```

## Configure omni_json for the Books Table

```sql
-- Configure omni_json for the Books table
SELECT omni_json.define_table_mapping(
    books,
    $${
        "columns": {
            "book_id": { "exclude": true },
            "title": { "path": "book_title" },
            "price": {
                "transform": {
                    "input": { "type": "numeric", "function": "calculate_discounted_price" },
                    "output": { "type": "numeric", "function": "mask_price" }
                }
            },
            "published_date": { "exclude": true }
        }
    }$$
);
```

## Step 5: Retrieve and Customize JSON Data

```sql
-- Retrieve JSON data with customization
SELECT to_jsonb(books.*) FROM books
```

## Step 6: Insert and Update JSON Data

```sql
-- Updating from JSON

-- Assume some_id is the book_id you want to update
UPDATE books
SET
    -- Explicitly listed fields to be updated
    (published_date) =
        (SELECT
            '1981-12-12'::DATE AS published_date)
WHERE book_id = some_id;

-- Inserting from JSON

-- Assume the data for the new book is provided as a JSON object
WITH new_book_data AS (
    SELECT
        '{"title": "New Book", "author": "New Author", "price": 15.99, "published_date": "2022-01-01"}'::JSONB AS json_data
)
INSERT INTO books (title, author, price, published_date)
SELECT
    json_data->>'title' AS title,
    json_data->>'author' AS author,
    calculate_discounted_price((json_data->>'price')::NUMERIC) AS price,
    (json_data->>'published_date')::DATE AS published_date
FROM new_book_data;
```

## Step 7: Query Python Functions in SQL
```sql
-- Query using Python functions
SELECT title, mask_price(price) AS masked_price
FROM books;
```

This comprehensive demo showcases the powerful capabilities of Omnigres extensions, including `omni_json` for JSON customization and `omni_python` for seamless Python integration within a bookstore database. The integration enhances JSON retrieval, manipulation, and SQL queries with Python functions, providing a robust solution for flexible and efficient data management.
