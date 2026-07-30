# MySQL Data Types and Allowed Attributes

## Legend

| Attribute      | Meaning                                        |
| -------------- | ---------------------------------------------- |
| PRIMARY KEY    | Uniquely identifies each row                   |
| FOREIGN KEY    | References another table                       |
| NOT NULL       | Value is required                              |
| NULL           | Value may be empty                             |
| UNIQUE         | No duplicate values allowed                    |
| AUTO_INCREMENT | Automatically generates sequential numbers     |
| DEFAULT        | Uses a default value when none is supplied     |
| UNSIGNED       | Allows only positive numbers                   |
| ZEROFILL\*     | Pads numbers with leading zeros _(deprecated)_ |
| COMMENT        | Adds documentation to the column               |

---

# Numeric Data Types

| Data Type      | Purpose                        | Common Attributes                                                |
| -------------- | ------------------------------ | ---------------------------------------------------------------- |
| TINYINT        | Very small integers            | NOT NULL, NULL, DEFAULT, UNSIGNED                                |
| SMALLINT       | Small integers                 | NOT NULL, NULL, DEFAULT, UNSIGNED                                |
| MEDIUMINT      | Medium integers                | NOT NULL, NULL, DEFAULT, UNSIGNED                                |
| INT / INTEGER  | Standard whole numbers         | PRIMARY KEY, AUTO_INCREMENT, NOT NULL, UNIQUE, DEFAULT, UNSIGNED |
| BIGINT         | Very large integers            | PRIMARY KEY, AUTO_INCREMENT, NOT NULL, DEFAULT, UNSIGNED         |
| DECIMAL(p,s)   | Exact decimal values (money)   | NOT NULL, DEFAULT, UNSIGNED                                      |
| NUMERIC(p,s)   | Same as DECIMAL                | NOT NULL, DEFAULT, UNSIGNED                                      |
| FLOAT          | Approximate decimal            | NOT NULL, DEFAULT                                                |
| DOUBLE         | Larger floating-point          | NOT NULL, DEFAULT                                                |
| BIT            | Bit values                     | NOT NULL, DEFAULT                                                |
| BOOLEAN / BOOL | True/False (stored as TINYINT) | NOT NULL, DEFAULT                                                |

---

# Character Data Types

| Data Type  | Purpose                     | Common Attributes               |
| ---------- | --------------------------- | ------------------------------- |
| CHAR(n)    | Fixed-length text           | NOT NULL, NULL, DEFAULT         |
| VARCHAR(n) | Variable-length text        | NOT NULL, NULL, DEFAULT, UNIQUE |
| TINYTEXT   | Small text                  | NULL, NOT NULL                  |
| TEXT       | Large text                  | NULL, NOT NULL                  |
| MEDIUMTEXT | Larger text                 | NULL, NOT NULL                  |
| LONGTEXT   | Very large text             | NULL, NOT NULL                  |
| ENUM(...)  | One value from a list       | NOT NULL, DEFAULT               |
| SET(...)   | Multiple values from a list | NOT NULL, DEFAULT               |

---

# Binary Data Types

| Data Type    | Purpose                  | Common Attributes |
| ------------ | ------------------------ | ----------------- |
| BINARY(n)    | Fixed binary data        | NOT NULL, DEFAULT |
| VARBINARY(n) | Variable binary data     | NOT NULL, DEFAULT |
| TINYBLOB     | Small binary object      | NULL              |
| BLOB         | Binary large object      | NULL              |
| MEDIUMBLOB   | Larger binary object     | NULL              |
| LONGBLOB     | Very large binary object | NULL              |

---

# Date and Time Data Types

| Data Type | Purpose                          | Common Attributes                   |
| --------- | -------------------------------- | ----------------------------------- |
| DATE      | Calendar date                    | NOT NULL, DEFAULT                   |
| TIME      | Time only                        | NOT NULL, DEFAULT                   |
| DATETIME  | Date and time                    | NOT NULL, DEFAULT                   |
| TIMESTAMP | Date/time with automatic updates | NOT NULL, DEFAULT CURRENT_TIMESTAMP |
| YEAR      | Year value                       | NOT NULL, DEFAULT                   |

---

# JSON and Spatial Types

| Data Type  | Purpose       | Common Attributes |
| ---------- | ------------- | ----------------- |
| JSON       | JSON document | NOT NULL, NULL    |
| GEOMETRY   | Spatial data  | NULL              |
| POINT      | Coordinates   | NULL              |
| LINESTRING | Lines         | NULL              |
| POLYGON    | Areas         | NULL              |

---

# Common Column Attributes

| Attribute      | Can Be Used With                                       | Description                               |
| -------------- | ------------------------------------------------------ | ----------------------------------------- |
| PRIMARY KEY    | Any suitable type (usually INT, BIGINT, CHAR, VARCHAR) | Unique identifier for each row            |
| FOREIGN KEY    | Same type as referenced column                         | Creates relationship between tables       |
| AUTO_INCREMENT | Integer types only                                     | Generates sequential values automatically |
| NOT NULL       | Almost every type                                      | Requires a value                          |
| NULL           | Almost every type                                      | Allows empty values                       |
| UNIQUE         | Most types                                             | Prevents duplicates                       |
| DEFAULT        | Most types                                             | Supplies a default value                  |
| UNSIGNED       | Numeric types                                          | Removes negative numbers                  |
| COMMENT        | All types                                              | Documents the column                      |
| CHECK          | Most types                                             | Validates values (MySQL 8+)               |

---

# Typical Analytics Engineering Examples

| Column       | Data Type     | Attributes                                            |
| ------------ | ------------- | ----------------------------------------------------- |
| EmpKey       | INT           | PRIMARY KEY AUTO_INCREMENT                            |
| JobID        | VARCHAR(25)   | NOT NULL                                              |
| EmployeeName | VARCHAR(100)  | NOT NULL                                              |
| HourlyRate   | DECIMAL(10,2) | NOT NULL                                              |
| HoursWorked  | DECIMAL(5,2)  | NOT NULL DEFAULT 0                                    |
| Revenue      | DECIMAL(12,2) | DEFAULT 0                                             |
| Cost         | DECIMAL(12,2) | DEFAULT 0                                             |
| DateWorked   | DATE          | NOT NULL                                              |
| CreatedAt    | TIMESTAMP     | DEFAULT CURRENT_TIMESTAMP                             |
| UpdatedAt    | TIMESTAMP     | DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP |

---

# Rules of Thumb

| If storing...        | Use                |
| -------------------- | ------------------ |
| IDs                  | INT AUTO_INCREMENT |
| Money                | DECIMAL            |
| Whole numbers        | INT                |
| Fractions            | DECIMAL            |
| Short names          | VARCHAR            |
| Large notes          | TEXT               |
| Dates                | DATE               |
| Date & time          | DATETIME           |
| Automatic timestamps | TIMESTAMP          |
| Yes/No               | BOOLEAN            |
| JSON documents       | JSON               |

---

## Things to Remember

- `AUTO_INCREMENT` only works on integer types.
- `UNSIGNED` only works on numeric types.
- `PRIMARY KEY` automatically implies `NOT NULL` and `UNIQUE`.
- A table can have only **one PRIMARY KEY**, but it may contain multiple columns (composite key).
- A table may have many `UNIQUE` constraints.
- `VARCHAR` is preferred over `CHAR` unless every value has the same length.
- Use `DECIMAL` for financial data—avoid `FLOAT` and `DOUBLE` for currency because they can introduce rounding errors.
