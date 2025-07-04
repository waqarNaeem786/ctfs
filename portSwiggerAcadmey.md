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
