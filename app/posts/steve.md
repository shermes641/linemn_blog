

**BT-1600**
=======
----------

![schema](http://xverter.com/b2b_account_relations1.png)
![schema](http://xverter.com/b2b_account_relations.png)

There are two endpoints to GET accounts. In both cases we want to filter the accounts so that we only return accounts a user has access to. This may involve a fairly large query because users can be mapped to accounts **directly** with the 
**users_to_accounts TABLE.** 

But they can also be mapped indirectly though 
sales orgs, sales groups, and customer groups. 

For each one of those we'll need to join to **TABLES:**
**user_access_to_sales_orgs**, 
**user_access_to_customer_groups**, 
**user_access_to_sales_groups** 

**accounts_to_sales_groups**
**accounts_to_customer_groups**

which we'll then join to the accounts table to get the total list of accounts. 

The endpoints we'll need this for are:
**GET /IDENTITY/organizations/:organization_id/accounts**
**GET /IDENTITY/accounts**

get 

if user not in org **NotFound** else
   return  (org accs) right join  (user accs) 

**Reset Token Functionality**
========
----------

**Blocked user**   
----------------

**NOTE:** Rest docs do not match this API 2014-10-27

 1. First get a password reset token with user email base64 encoded  (schoolhead@demo.ua.com)

		DELETE /users/passwords?ee=c2Nob29saGVhZEBkZW1vLnVhLmNvbQ
		 
 
 
 2. Email will be sent to "schoolhead@demo.ua.com" with link to enter new password
 
 
		
		Hello George Jones, This is a 'Reset Password' email. 
		Needs a link to page to change your password! 
		Here's the token 635ltu

 3.User goes to link, enters new password, UI posts to backend to change password 
	 
		 POST localhost:9020/users/passwords
			 {"email_lower": "schoolhead@demo.ua.com",
			 "password": "test",
			 "reset_token": "e2pguv"} 			 


----------

**Requests / Responses**
-------		

400 error  ---  Request is missing required query parameter 'ee'

	DELETE localhost:9020/users/passwords
	
500 error --- There was an internal server error.

	DELETE /users/passwords?ee
	DELETE /users/passwords?ee=12cdhjasdkh

400 error  --- Illegal request-target, unexpected character '=' at position 50 

	DELETE /users/passwords?ee=c2Nob29saGVhZEBkZW1vLnVhLmNvbQ==

204 --- NoContent
schoolhead@demo.ua.com  isActive = False
reset token created ("d255o0") with proper user_id (2) and expires date (now + 1 day)

	DELETE /users/passwords?ee=c2Nob29saGVhZEBkZW1vLnVhLmNvbQ

----------

Bad email
404 error  --- The requested resource could not be found but may be available again in the future.

	POST localhost:9020/users/passwords
		{"email_lower": "dealerhead@demo.ua.com",
		 "password": "newpassword",
		 "reset_token": "d255o0"}

Bad token		 
404 error  --- The requested resource could not be found but may be available again in the future.

	POST localhost:9020/users/passwords
		{"email_lower": "schoolhead@demo.ua.com",
		 "password": "newpassword",
		 "reset_token": "d255o0xxx"}
		 
Good email and token
200 --- password set and user marked as active (isActive = true) 

	POST localhost:9020/users/passwords
		{"email_lower": "schoolhead@demo.ua.com",
		 "password": "newpassword",
		 "reset_token": "d255o0"}

----------

**Logged in user**
----------------

Auth header must be included

No Auth header
403 error  ---  "Auth Failed"
	
	POST localhost:9020/users/reset_passwords
		{"password": "test",
		 "old_password": "newpassword"}
	
Wrong current password		 
400 error ---  The request contains bad syntax or cannot be fulfilled.
	
	POST localhost:9020/users/reset_passwords
		{"password": "newpassword",
		 "old_password": "test"}
	
Correct current password		 
200  ---  OK
	
	POST localhost:9020/users/reset_passwords
		{"password": "test",
		 "old_password": "newpassword"}