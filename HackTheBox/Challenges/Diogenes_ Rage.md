in the main page there is a vending machine and a $1 dollar coupon, you can drag the coupon into the machine and buy the listed products (C8 is the only one that costs more than $1)

in the challenge files there is a database file that doesn't seem vulnerable to injection, a middleware used to create and express.js session and a routes file that defines HTTP methods for the app API

**/api/purchase**
checks for a session (cookie), if it doesn't exist it will then create one and initialize an user with 0 balance, it also shows that successfully buying C8 will return the flag

**/api/coupons/apply**
also checks for session, but has the added functionality of applying the coupon and adding $1 to the user account, it's not possible to redeem a coupon twice

the vulnerability here is that it first adds to the account with db.addBalance and then sets that the user has used the coupon with db.setCoupon so it is possible to trigger a race condition by issuing multiple requests to the /api/coupons/apply using a fresh session cookie 

(solved it using turbo intruder extension on burp, couldn't make it work using curl/python)