# Authentication Bypass:
- Simple brute force attack for username and password.
- Check for the Ip blocking and then use the:
  
  ```
  X-Forwarded-For: <IP>
  ```
  
  to bypass the IP blocking.
  
- Check for the response time to when bruteforcing the authentication. 
- if there is longer response for any any of the username or password, It might be the right credential.
- Look for the Logic Flaws refere to the portswigger Lab:
  
  ```
		![Logic Auth Flaw](https://portswigger.net/web-security/learning-paths/authentication-vulnerabilities/password-based-vulnerabilities/authentication/password-based/lab-broken-bruteforce-protection-ip-block)
  ```
- The logic flaws could be in how user is being blocked, if we login with actual authentications whether that bloackage is eradicated or not, the above lab is example of this, similar we can check whether if we have username but don't have the password we can use the above mentioned strategy we can track the login, and check at which point we are authenticated and accounts blockage is lifted, then bruteforce the account.

- In the 2FA after the user has logged In there and is redirected to the 2FA code, behind the scenes if logic flawed user might gets logged in on via username:password, a simple change of the route from 
/2FA to /my-acount can help bypass the 2FA.

- In most of the website the check on the user is made by assigning a code or cookie as the identfier on the initial login, so that backend can manage the state of user and identity of the user, but it could be miss used, if user logins with his account and verify using the 2FA code which I sent for his own account and then at the last moment changes the cookie identifier with the victims identifier it could make the user login to victims account.
- The following labs explain the matter in detail:
	
	  ```
	https://portswigger.net/web-security/learning-paths/authentication-vulnerabilities/vulnerabilities-in-multi-factor-authentication/authentication/multi-factor/flawed-two-factor-verification-logic	
	  
	  ```
- the auth vuln can also occur in the remember functionality of the website, in order check the login status of the user most of the web apps use the coookie hash which if it is losely bounded could and the weak predicatable hash is used it could be exploited by generating a payload, which returns a true session token which is present in the DB of the web app, with that session token attacker can log in easily.
 
- The reset password functionality using the reset link is also vulnerable, what is the user changes the reset link to its on name or desired user and on going to the reset page it can change the password to desired password.


```
https://reset-password.com/reset?=your-name

```

- User can change the your-name with user which they want to exploit.


- if the reset functionality is implemented using the hash we can check how the password is reset wheter the token is reused to validate the user or not or is it using the username to validate, if that is the case we can change the username with the with implied useranme and login into the account using the changed password. 


- If the auth-token is implemented to reset the password we can try leak the that token to our on web-server using the **X-Forwarded-Host** refer to this lab:


```

https://portswigger.net/web-security/learning-paths/authentication-vulnerabilities/vulnerabilities-in-other-authentication-mechanisms/authentication/other-mechanisms/lab-password-reset-poisoning-via-middleware

```

# Access Control vulns:

- These vulnerabilities occur when user with lesser privilages, some how manage to have access to the privilage of the root user or the one with more privilges, there are different types of it but we don't have to go into that details. 
