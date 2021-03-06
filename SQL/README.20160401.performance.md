1. Is the following SQL good or bad practice from a performance perspective?
  - Searching for all rows with the year 2012:
    ```
    CREATE INDEX tbl_idx ON tbl (date_column);

    SELECT text, date_column
    FROM tbl
    WHERE YEAR(date_column) = '2012';
    ```
  - **Answer: Bad practice**
  - **Explanation:**
    - Wrapping the table column in a function renders the index useless for this query.
  - Improvement
    - Write queries for continuous periods as explicit range condition:
    
      ```
      SELECT text, date_column
      FROM tbl
      WHERE date_column >= STR_TO_DATE('2012-01-01', '%Y-%m-%d')
        AND date_column <  STR_TO_DATE('2013-01-01', '%Y-%m-%d')
      ```

2. Is the following SQL good or bad practice from a performance perspective?
  - Two queries, searching by a common column:
    ```
    CREATE INDEX tbl_idx ON tbl (a, b);

    SELECT id, a, b
    FROM tbl
    WHERE a = 123 AND b = 42;

    SELECT id, a, b
    FROM tbl
    WHERE b = 42;
    ```
  - **Answer: Bad practice**
  - **Explanation:**
    - The index covers the first query only, the second query cannot use the index to the best extent possible.
  - Improvement
    - Changing the column order makes the index suitable for both queries—without additional overhead. The index should therefore look like this (columns exchanged):
    
      ```
      CREATE INDEX tbl_idx ON tbl (b, a);
      ```

3. Is the following SQL troublesome or bulletproof from a performance perspective?
  - Searching within a string:
    ```
    CREATE INDEX tbl_idx ON tbl (text);

    SELECT id, text
    FROM tbl
    WHERE text LIKE '%TERM%';
    ```
  - **Answer: Troublesome. There is high risk for performance problems.**
  - **Explanation:**
    - 'LIKE' expressions starting with a wildcard cannot use an index to locate the matching entries. There is no simple way to tune such a query. Use another access path if possible (e.g.: additional where conditions). Otherwise consider using a full-text index.

4. How will the change affect query performance?
  - Current situation, selecting about hundred rows out of a million
    ```
    CREATE INDEX tab_idx ON tbl (a, date_column);

    SELECT date_column, count(*)
    FROM tbl
    WHERE a = 123
    GROUP BY date_column;
    ```
  - Changed query, selecting about ten rows out of a million
  
    ```
    SELECT date_column, count(*)
    FROM tbl
    WHERE a = 123 AND b = 42
    GROUP BY date_column;
    ```
  - **Answer: After getting changed, the query will be much slower**
  - Explanation
    - The query will be much slower—regardless of the data. The original query is executed as an index-only scan. It doesn't need to access the table because the index covers the entire query—all referenced columns are covered in the index. Although the additional where clause reduces the number of returned rows, it requires a table access to fetch the column B, which is not included in the index. That means that the new query cannot run as an index-only scan; it must access the table as well. This access is additional work that slows the query down—regardless whether the final result is smaller due to this filter.


