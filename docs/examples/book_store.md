Demo that illustrates the capabilities of Omnigres extensions (omni_json and omni_python) for JSON customization and Python integration in a bookstore database.

### Step 1: Create a Bookstore Database

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

### Step 2: Configure omni_json for the Books Table

```sql
-- Configure omni_json for the Books table
SELECT omni_json.define_table_mapping('books', '
{
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
}
');
```

### Step 3: Create Python Functions for Price Manipulation

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

### Step 4: Load Python Functions into the Database

```sql
-- Load the Python functions into the database
CREATE OR REPLACE FUNCTION load_python_functions()
RETURNS omni_python.inline LANGUAGE SQL AS $$
from omni_python import load_module

load_module('price_functions')
$$;

SELECT omni_schema.load_from_inline(load_python_functions());
```

### Step 5: Retrieve and Customize JSON Data

```sql
-- Retrieve JSON data with customization
SELECT omni_json.retrieve('books', 'book_id = 1') AS customized_json;
```

### Step 6: Insert and Update JSON Data

```sql
-- Insert JSON data
INSERT INTO books (title, author, price, published_date)
VALUES ('New Book', 'New Author', 15.99, '2022-01-01');

-- Update JSON data
UPDATE books
SET price = calculate_discounted_price(price)
WHERE book_id = 4;
```

### Step 7: Query Python Functions in SQL

```sql
-- Query using Python functions
SELECT title, mask_price(price) AS masked_price
FROM books;
```

This demo showcases the configuration of `omni_json` for JSON customization, the development and integration of Python functions for price manipulation, the retrieval and manipulation of JSON data, and the execution of Python functions within SQL queries.
