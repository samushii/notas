
## Description: 

this lab contains a blind SQL injection vulnerability, the application uses a tracking cookie for analytics and performs a query containing the value of the cookie

the results of the squery are not returned and the application does not respond any differently based on whether the query returns any rows or causes an error, but the query is executed synchronously so it is possible to trigger conditional delays to infer information

exploit the SQLi to trigger a 10 second delay to solve the lab

## Solution:

injecting the pg_sleep(10) function:

	[cookie-value]'||pg_sleep(10)--



