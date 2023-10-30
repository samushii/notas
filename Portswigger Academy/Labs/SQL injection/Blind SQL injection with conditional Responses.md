
this lab contains a blind SQL injection vulnerability in the cookie header
the application includes a "Welcome back" in the response if the query returns any rows

the database contains a table called users with columns called username and password

exploit the blind SQLi to find out the password of the administrator user

## Solution:

Using Burp Intruder with the 'Cluster Bomb' attack type it is possible to configure two payload sets to be interated through, one will be the substr() position parameter and the other will be the character we're trying

the injection query:

	' AND substring((SELECT password FROM users WHERE username='administrator'),$1$,1) ='$2$
[$1$ and $2$ represent the point where the payload current value will be inserted]

## Notes:

if manually exfiltrating data through this type of vulnerability it is useful to use the '>' and '<' operators and narrowing down from that instead of matching every possible character with '=' 


