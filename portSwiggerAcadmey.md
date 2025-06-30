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
