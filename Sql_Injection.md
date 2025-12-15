
# What is SQL Injections 

## 1. Introduction 

### 1.1 Definition

SQL injection (SQLi) is a **web security vulnerability that allows an attacker to interfere with the queries that an application makes to its database**. This can allow an attacker to view data that they are not normally able to retrieve. This might include data that belongs to other users, or any other data that the application can access. In many cases, an attacker can modify or delete this data, causing persistent changes to the application's content or behavior.[1]
SQL injection attacks allow attackers to spoof identity, tamper with existing data, cause repudiation issues such as voiding transactions or changing balances, allow the complete disclosure of all data on the system, destroy the data or make it otherwise unavailable, and become administrators of the database server.

SQL Injection can be define as **code injection technique where malicious SQL statements are inserted into an entry field (such as a login form or URL parameter) to manipulate the backend database**. 

### 1.2 Why it matters 
SQL injection is one of the most prevalent and dangerous web application vulnerabilities because a successful attack can have severe consequences for individuals and businesses. The **Open Web Application Security Project** (**OWASP**) consistently ranks injection as a top security risk.
Refer 3 : Injection (**A03:2021**), which includes SQL Injection, is ranked as the **third most critical web application security risk**. The **2021** report noted that **94%** of applications were tested for some form of injection, with a high incidence rate.

Key impacts includes 
1. Unauthorized access to sensitive data
2. Data modification or deletion
3. Authentication bypass
4. system compromise
5. Reputational and financial damage result in loss of customer trust

### 1.3 Historical significance and famous breach examples
The first discussions of SQL injection as a vulnerability occurred in **1998**, with the attack method documented in **Phrack magazine** by security researcher **Jeff Forristal**.

Some famous breach examples

#### Heartland Payment Systems (2009)
The Heartland Payment Systems data breach, publicly announced in January 2009, was a monumental cyberattack that compromised approximately 130 million credit and debit cards. It was, at the time, the largest criminal breach of card data ever recorded. 
The data exposed in the breach included payment transaction data, data coded into the card's magnetic strip, social security numbers, and banking information.

#### Sony PlayStation Network (2011)
An SQLi vulnerability was exploited to breach Sony's network, compromising approximately 77 million user accounts and resulting in significant financial losses and service disruptions.


## 2 Core Concepts

### 2.1 What is SQL and how queries are constructed
**SQL** (**Structured Query Language**) is the standard programming language used for storing, managing, and retrieving data from a relational database management system (RDBMS).

### 2.2 What is SQL used for?
SQL is the foundation of data management in many applications across various industries, from banking and e-commerce to data science and healthcare. 

Its primary functions are categorized into several types of commands:
Data Query Language (DQL): Retrieving specific data from the database using the SELECT command.

- Data Manipulation Language (DML): Modifying the data within the tables using commands like INSERT, UPDATE, and DELETE.
- Data Definition Language (DDL): Defining or modifying the database schema and objects (like tables and indexes) using CREATE, ALTER, and DROP commands.
- Data Control Language (DCL): Managing permissions and access control for database users using GRANT and REVOKE.
- Transaction Control Language (TCL): Ensuring data integrity by managing transactions with commands like COMMIT and ROLLBACK. 


### 2.3 Input handling and unsafe string concatenation

SQL Injection vulnerabilities primarily arise from two key mistakes in handling user input:

i. Unsafe handling of user-supplied input.
ii. Dynamic construction of SQL queries using string concatenation.

When this 2 comes together attacker can manipulates whole query

SQL Injection happens when applications directly combine user input with SQL commands. Instead of treating user data as separate, it becomes part of the executable query.

For example 

query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
Here, $id comes directly from user input and is concatenated into the SQL query.

Injection here can be :
An attacker can craft input like ' OR '1'='1 to manipulate the query:

SELECT first_name, last_name FROM users WHERE user_id = '' OR '1'='1';

The WHERE clause evaluates to true for all rows, exposing all user data.

OR 

query = "SELECT * FROM users WHERE username = '" . $username . "' AND password = '" . $password . "'";

Input: username = ' OR '1'='1

Resulting Query:
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '';

( Note : This is only for education purpose only. )

2.4 Difference between Trusted vs Untrusted Input


#### Trusted Input
Trusted input comes from sources that are internal, verified, and controlled. The application can safely assume the data is well-formed and predictable.

**Characteristics**:
i. Generated internally by the application or system logic (e.g., system timestamps, calculated IDs, hard-coded constants).
ii. Data from fully authenticated and authorized users within a secure internal network may be considered trusted in a controlled context.
iii. Conforms to expected data types and constraints.

**Examples**:
i. Configuration files bundled with the application.
ii. Internal service-to-service communication within a secure network.
iii. Constants and values defined in the source code.

**Handling Guidelines**:
i. Minimal validation needed.
ii. Can often be used directly in processing or queries.
iii. Periodically review to ensure sources remain secure.
iv. Security Risk: Low (if processed correctly).

#### Untrusted Input
Untrusted input comes from external or uncontrolled sources. It may contain malicious content and must be treated cautiously.

**Characteristics**:
i. Supplied by users via forms, search bars, or text fields.
ii. Data in URL parameters, HTTP headers, cookies, and file uploads.
iii. Data received from third-party APIs or external systems outside the organization’s control.

**Key Point**:
All untrusted input should be assumed hostile until proven safe. Direct use without validation can lead to SQL Injection, XSS, command injection, and other attacks.

**Handling Guidelines**:
Rigorous validation, sanitization, or encoding required.
Always use parameterized queries or prepared statements for database operations.
Input should never be directly concatenated into SQL queries or command strings.
Security Risk: High (if not handled properly).

## 3 Types of SQL Injections 

SQL Injection attacks are categorized based on how an attacker retrieves data or confirms the success of the attack. The main types are In-band SQLi, Blind SQLi, and Out-of-band SQLi.

### 3.1 In-Band SQLi
In-band SQLi is the simplest and most common type of SQLi where the attacker uses the same communication channel to inject malicious SQL and receive the results.

Subtypes:
#### 3.1.1 Error-Based SQLi
Mechanism: The attacker intentionally causes database errors to gain information about the database structure, tables, or columns.
Example: Injecting input like ' to trigger an SQL error revealing the database type or table names.
Use Case: Useful for reconnaissance and crafting further attacks.

#### 3.1.2 Union-Based SQLi
Mechanism: Uses the UNION SQL operator to combine the results of the original query with the attacker’s query.
Example: SELECT name FROM users WHERE id=1 UNION SELECT username, password FROM admin;
Use Case: Extracts sensitive data from other tables while still using the normal application response.

### 3.2 Blind SQLi
Blind SQLi occurs when the application does not display query results or errors, forcing the attacker to infer information through behavior or timing.

Subtypes:
#### 3.2.1 Boolean-Based Blind SQLi
Mechanism: The attacker injects queries that evaluate to TRUE or FALSE and observes application behavior (e.g., page changes or content differences).
Use Case: Allows attackers to extract information bit by bit based on true/false responses.
#### 3.2.2 Time-Based Blind SQLi
Mechanism: The attacker uses queries that delay the database response (e.g., SLEEP() in MySQL) if a condition is true. The attacker measures the response time to infer the database content.
Use Case: Extracts sensitive data even when there’s no visible output.

### 3.3 Out-of-Band SQLi
Out-of-band SQLi is used when neither in-band nor blind SQLi is feasible. The attacker relies on alternative channels (like DNS or HTTP requests) to retrieve data.
Mechanism: The database is tricked into sending data to an external server controlled by the attacker (e.g., xp_dirtree in SQL Server or LOAD_FILE() in MySQL).
Use Case: Effective when normal channels are restricted or slow, allowing attackers to exfiltrate data externally.


## 4 Internal Working of an SQL Injection Attack

SQL Injection (SQLi) attacks exploit an application's failure to distinguish between code (SQL commands) and data (user input). This allows attackers to manipulate queries, access unauthorized data, or modify the database.

### 4.1 How Queries Get Manipulated

SQLi happens when user input is directly concatenated into SQL queries without proper sanitization.

Example 

$productId = $_GET['id'];
$query = "SELECT name, description FROM products WHERE id = " . $productId;

result = mysqli_query($conn, $query);

**Expected Input id = 123**

SELECT name, description FROM products WHERE id = 123;

**Malicious input: id=123 OR 1=1**

SELECT name, description FROM products WHERE id = 123 OR 1=1;

Here, 1=1 always evaluates to true, returning all records instead of just one.

### 4.2 Misuse of Control Characters and Escape Sequences

Attackers use special characters to alter the SQL query:

Single Quote ('): Ends the intended string literal to start malicious SQL.
Comments (--, #, /* */): Ignore remaining parts of the original query.
Semicolon (;): Separate multiple SQL statements for batch execution.

For example 

select * from users where username = {username_input} and password = {password_input}

Malicious input 
username =XYZ' -- 
password = asfsjkdflaskdf

select * from users where username = 'XYZ' -- ' and password = 'asfsjkdflaskdf'

here, -- will comment out query so it will just match for username.

Why it works: Applications fail to escape these characters, allowing the DBMS to interpret them as commands instead of data.


## 5 Prevention Techniques

### 5.1 Parameterized Queries and Prepared Statements
Parameterized queries, also called prepared statements, are the most effective defense against SQL injection. They work by separating the SQL command structure from user-provided input.

How Prepared Statements Work Internally:
Prepare:
The application sends a SQL template to the Database Management System (DBMS) with placeholders instead of actual values.
SELECT name, description FROM products WHERE id = ?;

At this stage, the DBMS parses the query, checks syntax, and creates an execution plan. No user input has been included yet, so the structure of the query is fixed.

Bind:
User input is sent separately and bound to the placeholders.
Example: The user enters 123 as the product ID. This value is attached to the placeholder without altering the SQL logic.

Execute:
The DBMS executes the precompiled query with the bound values. Any control characters in the input (like ', ;, or --) are treated as literal data, not executable code.
This prevents attackers from modifying the query logic or injecting additional commands.


### 5.2 ORM-Level Protections

Object-Relational Mapping (ORM) frameworks provide an abstraction layer to interact with the database using objects instead of raw SQL. Examples include Hibernate (Java), Entity Framework (C#), and Django ORM (Python).

How ORM Helps Prevent SQLi:
Automatic Parameterization:
Most ORM methods automatically use parameterized queries internally, ensuring user input is never concatenated into SQL strings.
Query Builders:
ORMs provide query builders that escape user input and prevent SQL injection automatically.
Avoids Raw SQL Concatenation:
By using ORM functions for database queries (like User.objects.get(username=user_input) in Django), developers avoid manually writing vulnerable SQL strings.

Caveat:
Using raw SQL queries inside an ORM without proper parameterization still introduces risk.
Developers must always use the ORM’s parameter binding methods when raw queries are necessary.


### 5.3 Stored Procedures

Stored procedures are precompiled SQL queries stored in the database. Applications call these procedures instead of building dynamic SQL strings in the code.

How Stored Procedures Help Prevent SQLi:
The query structure is defined and compiled inside the database.
User input is passed as parameters, which the DBMS treats strictly as data.
Reduces the chance of query manipulation from application-side string concatenation.

Caution:
If a stored procedure internally constructs dynamic SQL using user input, it can still be vulnerable.
Proper parameter binding must always be enforced inside the procedure.


### 5.4 Input Validation and Whitelisting

Input validation is a secondary defense that ensures user input conforms to expected patterns before it reaches the database.

Techniques:
Whitelisting (Recommended):
Only accept known-good values. For example:
Product IDs should only contain digits 0–9.
Email fields must match a standard email format.
Enum or dropdown fields should only accept predefined values.
Length Checks:
Restrict input size to the expected maximum length to reduce buffer and injection risks.
Type Checks:
Ensure numeric fields contain numbers, dates are valid, and strings do not exceed expected characters.

Benefits:
Reduces attack surface by blocking invalid input early.
Works best in combination with parameterized queries.
Note: Blacklisting (“deny-lists” of forbidden characters) is less effective because attackers can bypass filters with alternative encoding or SQL syntax variations.


### 5.5 Escaping and Sanitization

Escaping is a method where special characters in user input are converted into safe forms so that the DBMS treats them as literal data.

Example:
Convert single quotes ' to '' in SQL strings.
Escape backslashes and other special characters depending on the database.

Caution:
Escaping is database-specific (MySQL, SQL Server, PostgreSQL differ in syntax).
Manual escaping is error-prone and difficult to maintain.
Always prefer parameterized queries; escaping should be used only as a last resort.


### 5.6 Web Application Firewalls (WAFs)

A WAF is a security tool that sits in front of the web application and monitors incoming HTTP requests.

How a WAF Helps Prevent SQLi:
Detects common SQLi patterns and blocks suspicious input.
Protects legacy or unpatched applications temporarily.
Can prevent automated SQLi attacks and reduce immediate risk.

Limitations:
Cannot replace secure coding practices.
Sophisticated attackers can bypass WAF rules using obfuscation or encoding.
WAFs should complement, not replace, parameterized queries and input validation.

## 6 Performance and Security Considerations

Security controls must be effective without imposing unnecessary performance overhead. SQL injection prevention techniques—when applied correctly—provide both strong security and high performance.

### 6.1 Cost of checking and validating input
Input validation ensures that user-supplied data meets expected formats, lengths, and types. While crucial for security, validation introduces a computational cost.

Most validation (regex checks, type casting, length limits) is computationally cheap.
Numeric checks (isnumeric)
String length enforcement
Basic pattern checks
Execution time is usually microseconds per input.

Good input validation reduces unnecessary database queries.
- Invalid inputs are rejected early.
- This reduces load on the database, which is far more expensive than CPU validation.


### 6.2 Why prepared statements are efficient
Prepared statements not only protect against SQL injection but also offer major performance benefits, especially in high-traffic applications.

When a prepared statement is created, the DBMS:
Parses the SQL
Validates syntax
Determines the optimal way to execute (query plan)
For repeated executions, the DBMS reuses the same compiled plan, avoiding re-parsing the SQL every time.
This greatly improves performance for queries that run frequently (e.g., logins, product lookups).


### 6.3 Reducing attack surface through schema design
Good database schema design can greatly reduce the impact of SQL injection—even if an attack occurs. Optimized schemas naturally enforce boundaries that limit attacker abilities.

Designing the schema so that the application uses:
read-only accounts for fetch operations
separate accounts for writes
no access to administrative tables
This minimizes the damage an attacker can cause.

Storing sensitive information in isolated tables or schemas adds barriers:
Put user credentials in a dedicated auth schema with restricted permissions
Keep logging, analytics, and business data separate
Restrict cross-schema access
Even if an injection affects one part of the system, attackers cannot automatically access everything.

Avoid Dynamic SQL Inside the Database

Prevent the use of:
EXECUTE('SELECT * FROM ' || table_name);
Refer 4 

### Summary :
SQL Injection remains a critical vulnerability with severe consequences if unaddressed. Effective mitigation includes:
-   Parameterized queries and prepared statements
-   ORM usage    
-   Stored procedures with proper parameterization
-   Rigorous input validation and whitelisting
-   Escaping special characters (when necessary)
-   Complementary WAF deployment
-   Secure database schema design
    

Proper implementation of these techniques not only prevents SQLi but also enhances performance and maintains data integrity.


I’m still learning, so if there’s anything I got wrong or something you’d like to add, please share in the comments. Let’s learn together!

**Thank You.**

## Resources 
[1] https://portswigger.net/web-security/sql-injection

[2] https://en.wikipedia.org/wiki/SQL_injection

[3] https://owasp.org/Top10/2021/A03_2021-Injection/
