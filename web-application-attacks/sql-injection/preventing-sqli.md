---
description: Techniques to prevent SQLi
---

# Preventing SQLi

These techniques render SQLi useless and thus should be recognized and understood to avoid wasting time.

Below is an example of an unsafe query in Java:

```java
String query = "SELECT account_balance FROM user_data WHERE user_name = "
             + request.getParameter("customerName");
try {
    Statement statement = connection.createStatement( ... );
    ResultSet results = statement.executeQuery( query );
}
...
```

The unvalidated "customerName" parameter that is simply appended to the query allows an attacker to inject any SQL code they want. This can be prevented in the following ways:

## Prepared Statements (with Parameterized Queries)

The use of prepared statements with variable binding (aka parameterized queries) is how all developers should first be taught how to write database queries. They are simple to write, and easier to understand than dynamic queries. Parameterized queries force the developer to first define all the SQL code, and then pass in each parameter to the query later. This coding style **allows the database to distinguish between code and data, regardless of what user input is supplied**.

### Language-Specific Recommendations

* **Java EE** – use `PreparedStatement()` with bind variables
* **.NET** – use parameterized queries like `SqlCommand()` or `OleDbCommand()` with bind variables
* **PHP** – use PDO with strongly typed parameterized queries (using bindParam())
* **Hibernate** - use `createQuery()` with bind variables (called named parameters in Hibernate)
* **SQLite** - use `sqlite3_prepare()` to create a [statement object](http://www.sqlite.org/c3ref/stmt.html)

#### Example

Below is an example of a safe coded prepared statement in Java:

```java
// This should REALLY be validated too
String custname = request.getParameter("customerName");
// Perform input validation to detect attacks
String query = "SELECT account_balance FROM user_data WHERE user_name = ? ";
PreparedStatement pstmt = connection.prepareStatement( query );
pstmt.setString( 1, custname);
ResultSet results = pstmt.executeQuery( );
```

And the same in C# .NET:

```csharp
String query = "SELECT account_balance FROM user_data WHERE user_name = ?";
try {
  OleDbCommand command = new OleDbCommand(query, connection);
  command.Parameters.Add(new OleDbParameter("customerName", CustomerName Name.Text));
  OleDbDataReader reader = command.ExecuteReader();
  // …
} catch (OleDbException se) {
  // error handling
}
```

## Stored Procedures

Similar in concept to prepared statements, stored procedures require the developer to just build SQL statements with parameters which are automatically parameterized unless the developer does something largely out of the norm. The difference between prepared statements and stored procedures is that **the SQL code for a stored procedure is defined and stored in the database itself, and then called from the application**. Both of these techniques have the same effectiveness in preventing SQL injection.

#### Example

The following code example uses a `CallableStatement`, Java's implementation of the stored procedure interface, to execute the same database query. The `sp_getAccountBalance` stored procedure would have to be predefined in the database and implement the same functionality as the query defined above.

```java
// This should REALLY be validated
String custname = request.getParameter("customerName");
try {
  CallableStatement cs = connection.prepareCall("{call sp_getAccountBalance(?)}");
  cs.setString(1, custname);
  ResultSet results = cs.executeQuery();
  // … result set handling
} catch (SQLException se) {
  // … logging and error handling
}
```

## Allow-List Input Validation

Various parts of SQL queries aren't legal locations for the use of bind variables, such as the names of tables or columns, and the sort order indicator (ASC or DESC). In such situations, input validation or query redesign is the most appropriate defense.&#x20;

For the names of tables or columns, ideally those values come from the code, and not from user parameters. But if user parameter values are used for targeting different table names and column names, then the parameter values should be mapped to the legal/expected table or column names to make sure unvalidated user input doesn't end up in the query. Please note, **this is a symptom of poor design and a full rewrite should be considered if time allows**.

#### Example

Below is an example of input validation in the SQL query itself:

```sql
String tableName;
switch(PARAM):
  case "Value1": tableName = "fooTable";
                 break;
  case "Value2": tableName = "barTable";
                 break;
  ...
  default      : throw new InputValidationException("unexpected value provided for table name");
```
