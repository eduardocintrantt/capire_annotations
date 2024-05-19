A linguagem CDS (Core Data Services) fornece diversos tipos internos que são mapeados para tipos SQL correspondentes. A seguir, uma tabela com os tipos, seus argumentos, exemplos de valores e os tipos SQL correspondentes:

| CDS Type      | Arguments / Remarks    | Example Value                | SQL (6)        |
| ------------- | ---------------------- | ---------------------------- | -------------- |
| `UUID`        | an opaque string (1)   | `'be071623-8699-4106-...'`   | _NVARCHAR(36)_ |
| `Boolean`     |                        | `true`                       | _BOOLEAN_      |
| `UInt8`       | (2)                    | `133`                        | _TINYINT_      |
| `Int16`       |                        | `1337`                       | _SMALLINT_     |
| `Int32`       |                        | `1337`                       | _INTEGER_      |
| `Integer`     |                        | `1337`                       | _INTEGER_      |
| `Int64`       |                        | `1337`                       | _BIGINT_       |
| `Integer64`   |                        | `1337`                       | _BIGINT_       |
| `Decimal`     | (precision, scale) (3) | `15.2`                       | _DECIMAL_      |
| `Double`      |                        | `15.2`                       | _DOUBLE_       |
| `Date`        |                        | `'2021-06-27'`               | _DATE_         |
| `Time`        |                        | `'07:59:59'`                 | _TIME_         |
| `DateTime`    | _sec_ precision        | `'2021-06-27T14:52:23Z'`     | _TIMESTAMP_    |
| `Timestamp`   | _µs_ precision (4)     | `'2021-06-27T14:52:23.123Z'` | _TIMESTAMP_    |
| `String`      | (length ) (5)          | `'hello world'`              | _NVARCHAR_     |
| `Binary`      | (length) (5)           |                              | _VARBINARY_    |
| `LargeBinary` |                        |                              | _BLOB_         |
| `LargeString` |                        | `'hello world'`              | _NCLOB_        |
| `Vector`      | (dimensionality) (7)   |                              | _REAL_VECTOR_  |
