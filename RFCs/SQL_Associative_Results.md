# SQL Modules `Row` with associative results


## Abstract

When doing a `SELECT` query with any of the SQL modules (e.g. `db_sqlite`, `db_mysql`, etc), it would be
nice to fetch results using the column names instead of indicies.


## Motivation

This makes working with SQL much more friendly, and quite possibly less prone to bugs for developers.  E.g, if a
dev modified an existing `SELECT` query by adding a column to the result, they (most likely) will have to update
all the incides where that result is fetched.

This features exists in PHP's MySQL drivers, and something similar is possible in Qt's SQL framework.


## Description

What would need to be done:

1. Some sort of SQL parsing engine for all supported databaes right now.  It woudln't haven't to be a full fledged one,
   but would have to process what comes after the `SELECT` portion of a SQL statement.  Would need to worry about
   keywords such `AS` for aliasing.
   - There would need to be some research for any odities between all of the SQL variants
2. `SELECT *` would require fetching data about the desired table from the database.  So this means then that this would
   require doing a second SQL query to get the table information.
   - If people don't want a second query happening, a flag could be added in the `getRow()`/`getAllRows()` procs to not      retrive associative results
3. Adding an overload for `[]`, that takes strings as input, and then maps them to the columns

The main downside is that this does include extra complexity and work for Nim.  Overhead would be added to the querying
functions.  I not sure how terribly this would impact performance.



## Examples

### Before
```nim
let row = db.getRow(sql"SELECT id, name, email FROM users;")
echo "[" & row[0] & "] " & row[1] & " -- " & row[2]
```

### After
```nim
let row = db.getRow(sql"SELECT id, name, email FROM users;")
echo "[" & row["id"] & "] " & row["name"] & " -- " & row["email"]
```


## Backward incompatibility

None that I can think of in the API.
