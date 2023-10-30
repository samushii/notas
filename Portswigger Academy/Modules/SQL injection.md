## Detecting SQLi

SQLi can be manually detected by using a systematic set of tests against every entry point in the application, which typically involves:

- ' character
- submitting SQL-specific syntax that evaluates to the original value of the entry point, and to a different value and looking for systematic changes between the responses
- submitting boolean conditions such as OR 1=1 and OR 1=2 and looking for differences in the app response
- submitting payloads designed to trigger time delays (such as sleep() command) and looking for differences in time taken to respond
- submitting payloads designed to trigger an out-of-band network interaction when executed within a SQL query, and monitoring for those interactions

## Second-order SQL injection

first-order SQLi arises when the application takes user input from an HTTP request and, while processing that request, incorporates the input into a SQL query in an unsafe way

second-order SQLi happens when user input is placed into the database and then retrieved at a later time, no vulnerability arises at the moment data is stored, but when data is retrieved it ends up being incorporated into a SQL query in an unsafe way

generally it arises in situations where developers are aware of SQLi and safely handle the initial placement of the input into the database, but when data is being processed later it is deemed as safe and the vulnerability is triggered

## Preventing SQL injection

most instances of SQLi can be prevented using parametrized queries (also known as prepared statements), it can be used for any situation where untrusted input appears as data within the query, including the **WHERE**, **INSERT** or **UPDATE** statements. They can not be used to handle untrusted input in other parts of the query, such as table/column names, or the **ORDER BY** clause, applications that put untrusted data in those parts of the query will need a different approach, such as using a list of allowed values

## Enumerating the database

it is often necessary to gather information about the database itself, including type and version of db software, as well as contents of the db in terms of which tables and columns are present

### Enumerating type and version

different db types have different ways of checking the version, such as

| DB Type | Query | 
| - | - | 
| Microsoft, MySQL | SELECT @@version |
| Oracle | SELECT * FROM v$version | 
| PostgreSQL | SELECT version() |

example injection:
	' UNION SELECT @@version -- 


## Listing the contents of the database

most database types (with the notable exception of Oracle) have a set of views called the information schema which provide information about the database

a query to *information_schema.tables* to list the tables in the database

	SELECT * FROM information_schema.tables

returns output like the following:

| TABLE_CATALOG | TABLE_SCHEMA | TABLE_NAME | TABLE_TYPE |
| - | - | - | - |
| MyDatabase | dbo | Products | BASE TABLE |
| MyDatabase | dbo | Users | BASE TABLE |
| MyDatabase | dbo | Feedback | BASE TABLE |


it is then possible to query *information_schema.columns*

	SELECT * FROM information_schema.columns WHERE table_name = 'Users'

with an output like:

| TABLE_CATALOG | TABLE_SCHEMA | TABLE_NAME | COLUMN_NAME | DATA_TYPE | 
| - | - | - | - | - | 
| MyDatabase | dbo | Users | UserId | int |
| MyDatabase | dbo | Users | Username | varchar |
| MyDatabase | dbo | Users | Password | varchar | 

### Equivalent to information schema on Oracle:

it is possible to obtain the same information with slightly different queries

list tables by querying *all_tables*

	  SELECT * FROM all_tables

list columns by querying *all_tab_columns*

	SELECT * FROM all_tab_columns WHERE table_name = 'USERS'



## UNION attacks

when an application is vulenrable to SQL injection and the results of the query are returned within the application's responses, the UNION keyword can be used to retrieve data from other tables within the database

the UNION keyword lets one or more additional SELECT queries be executed and appended to the original query

for it to work two key requirements must be met:
- the individual queries must return the same number of columns
- the data types in each column must be compatbile between the individual queries

### Determining the number of columns required in a SQL injection UNION attack

the first method involves inejcting a series of ORDER BY clauses and incrementing the specified column index until an error occurs

	' ORDER BY 1--
	' ORDER BY 2--
	' ORDER BY 3--

when the specified index exceeds the number of columns in the result set the database will return an error, a generic error or simply no return results, provided it is possible to detect some difference in the application's response you can infer how many columns are being returned

the second method involves submitting a series of UNION SELECT payloads specifying different numbers of NULL values

	' UNION SELECT NULL--
	' UNION SELECT NULL,NULL--
	' UNION SELECT NULL,NULL,NULL--

again if the number of nulls does not match the number of columns the database might return an error, a generic error or no results

### Finding columns with useful data types using an UNION attack

generally the interesting data to retrieve with an UNION attack will be in string form, so it is needed to find a column in the original query whose data-type is compatible with string data

it is possible to probe each column after determining the number of columns returned


	' UNION SELECT 'a',NULL,NULL--
	' UNION SELECT NULL,'a',NULL--
	' UNION SELECT NULL,NULL,'a'--

if the data type is not compatbile the injection will cause an error, if it is compatbile the application response might contain some additional content including the injected string ['a' in this case]

### Retrieving multiple values within a single column

if only one compatbile column is being returned it is possible to return multiple values together by concatenating them, ideally inputting a separator for clarity reasons

	' UNION SELECT username || ':' || password FROM users-- 
[consult the database documentation to see which concatenation operators are available]

examples:
- oracle : 'foo'||'bar'
- microsoft:  'foo'+'bar'
- postgresql: 'foo'||'bar' or concat(foo,bar)
- mysql: 'foo' 'bar' [with a space in between] or CONCAT('foo','bar')


## Blind SQL injection

blind SQL injection occurs when the application is vulnerable to SQLi but the response does not include the resulsts of the relevant SQL query or the details of any database errors, so in this case it is not possible to use techniques such as UNION attacks

### Exploiting blind SQL injection by triggering conditional responses

if it is possible to perceive a different behavior on whether or not the query succeeds then it is possible to exfiltrate data by doing a series of conditional comparisons

for example, in a cookie parameter that is being compared to values stored in the database it might be possible to inject and exfiltrate the *password* of a given *administrator* in the *user*s table by doing the following:

	' AND substring((SELECT password FROM users WHERE username='administrator'),1,1) > 'm \\ returns true so we keep testing letters above 'm'

	' AND substring((SELECT password FROM users WHERE username='administrator'),1,1) > 't \\ returns false so it is possible to infer that the first letter of the password is in between m and (including) t

	' AND substring((SELECT password FROM users WHERE username='administrator'),1,1) = 's \\returns true so through a series of comparisons it was possible to determine the first letter of the password

the full password can be exfiltrated by changing the position parameter of the substring() function to check for every character in the password 

