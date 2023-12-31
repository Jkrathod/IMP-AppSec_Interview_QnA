<h4>Cross-Site Request Forgery (CSRF)</h4>

CSRF is also known as session riding attack is a web security vulnerability that tricks a web browser into executing an unwanted action.
When a website sends a data request to another website on behalf of a user along with the user session cookie attacker launch a CSRF Attack.

--- ---
**`MITIGATION`**
```
    1. Using Anti-CSRF tokens
    2. Double-Submitting Cookies
    3. Same-Site Cookies
    4. Enabling User Interaction
    5. Custom Headers for Requests
```
--- ---
**`Generate CSRF PoC`**

In Burp Suite Pro, you can use the tool to demonstrate a CSRF vulnerability by creating a proof of concept (PoC) using a request that does not include a CSRF token. This request can be used to update client information, and will show how an attacker can exploit the lack of CSRF protection to make unauthorized changes.

**Right Click --> Engagement Tools --> Generate CSRF PoC**


- POST REQUEST --> HTML (What the evil server would have)
```html
<html>
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://domain.com/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="example-change&#64;gmail&#46;com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

- Go in options and use auto-submit to include this line `...>document.forms[0].submit();<...`
- Take note that in the following example we have added the following from the original PoC of BurpSuite Pro

GET REQUEST --> HTML (What the evil server would have)
```html
<html>
  <body>
  <script>history.pushState('', '', '/')</script>
    <form action="https://domain.com/my-account/change-email" method="GET">
<!-- Place Options -->
      <input type="hidden" name="email" value="example-change&#64;gmail&#46;com" />
      <input type="submit" value="Submit request" />
      
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>
```

---
**`METHOD OF TEST CSRF ATTACK`**
##### `No CSRF Protection`

Another way to test for vulnerabilities related to CSRF is to observe if the application does not have any CSRF token implemented. This can be done by checking the requests made by the application and verifying if there are any tokens being used to protect against CSRF attacks. If there are no tokens present, it may indicate a vulnerability in the application's security.

Steps:
	- Generate a template
	- Append automation

---

##### `Remove CSRF Token`

Another way to test for vulnerabilities related to CSRF is to observe if the application's security can be bypassed by removing the CSRF token from a request. This can be done by intercepting a request that includes a CSRF token, removing the token, and forwarding the modified request to the server to see if it is still accepted. If the request is accepted without the token, it may indicate a vulnerability in the application's CSRF protection.

Steps:
	- Remove the token
	- Test if the request work
	- Generate a template
	- Append automation

---

##### `Change Request Type`

Another way to test for vulnerabilities related to CSRF is to observe if the application's security can be bypassed by changing the request type from POST to GET. This can be done by intercepting a POST request that includes a CSRF token, changing the request method from POST to GET, and forwarding the modified request to the server to see if it is still accepted. If the request is accepted without the token, it may indicate a vulnerability in the application's CSRF protection.

Steps:
	- Select in Burp Suite (repeater) the "Change request method" --> Change POST to GET
	- Generate a template
	- Append automation

---

##### `Client-Side Redirect`

The process of changing a user's email address may be hindered by security measures such as the refer header, which requires the user to first visit a page on the website before making changes. However, if a client-side redirect vulnerability is discovered, an attacker may be able to exploit it for a CSRF attack. This is done by crafting a GET request in the redirect, which can then be used to perform malicious actions, such as changing various informations.

Steps:
- Find a redirection vulnerability (Example redirect to post ID, but input ".."" take off the id and perform redirection)
- Create a request for changing variable (Ex: email)
- Change the request for a GET request ---> Dont forget, you might need to URL encode elements
- Try appending the request (redirection) with the GET request
- Generate a template from the redirection and change the value of the variable for the GET request
- Append Automation

---

##### `Change Request Type (Overwrite)`

It may not be possible to change a POST request to a GET request, but it is sometimes possible to override this by modifying the PoC provided by BurpSuite and forcing the request to be made via a GET request.

Steps:
- Select in Burp Suite (reapeater) the "Change request method" --> Change POST to GET
- Generate a template
- Append automation
- Hide the POST request with following input `<form action="...` ---> `<input type="hidden" name="_method" value="POST">` 

##### `CSRF token from another user`

Another way to test for vulnerabilities related to CSRF is to observe if the application's security can be bypassed by using a CSRF token from another user. This can be done by intercepting a request that includes a CSRF token, extracting the token from the request and using it in another user request, and forwarding the modified request to the server to see if it is still accepted. If the request is accepted using a token of another user, it may indicate that the tokens are not properly linked to the user session and the application may be vulnerable to CSRF attacks.

Steps:
- Connect to your account (Will generate a CSRF that will be transmited to another user)
- Intercept the request or copy the **CSRF token** in the source code of the form (Your account)
- Use the CSRF token in the generated template
- Append this after the input Email (to include a CSRF token in the request) or change the token
	- `<input type="hidden" name="csrf" value="vre8wybewuo_CSRF_TOKEN_uybfweruuef" />`
- Append automation

---

##### `CSRF token &  key from another user (Work/Dont Work)`

Another way to test for vulnerabilities related to CSRF is to observe if the application's security can be bypassed by using a CSRF token and key (cookie) from another user. This can be done by intercepting a request that includes a CSRF token and key, extracting the token and key from the request and using them in another user request, and forwarding the modified request to the server to see if it is still accepted. If the request is accepted using a token and key of another user, it may indicate that the tokens and keys are not properly linked to the user session and the application may be vulnerable to CSRF attacks.

It is also possible that the application is using a anti-CSRF token and a anti-CSRF cookie. It is important in this case to check if both are being checked on the server, if one of them is missing the request should be rejected.

==Did not work when i try it==

Steps:
- Connect to your account (Will generate CSRF (Token + Key) that will be transmited to another user)
- Intercept the request or copy the **CSRF token & key (cookie)** (Your account)
- Try injecting cookie and token in an other user account 
- Craft a payload (template) + ---> (Allow injection of the CSRF key cookie in the header)
	- Find a place in the website were you can make a request with parameters and include cookie
	- Change the request for `?search=XYZ%0d%0aSet-Cookie:%20csrfKey=CSRF_KEY` and check if you can inject a cookie via the request (only working if search does not has CSRF...)
	- If you can, use the default template (remove the automation)
	- Insert the following line
		- `<input type="hidden" name="csrf" value="vre8wybewuo_CSRF_TOKEN_uybfweruuef" />`
	 - Insert the CSRF cookie with img source and redirect on error (will provoque error)
		 - 1.  `<img src="https://DOMAIN/?search=test%0d%0aSet-Cookie:%20csrfKey=YOUR-KEY" onerror="document.forms[0].submit()">`

More information check the following video ---> https://www.youtube.com/watch?v=p8DZL0xTgyE&t

---

## Extra Method

**The Seven Sins**

- Anti-CSRF Tokens allow the server to uniquely distinguish who actually requests the resource/action, thereby preventing CSRF attacks. However, the following seven common implementation weaknesses enable attackers to bypass security:

- Removing the Anti-CSRF Token: Completely removing the Anti-CSRF token parameter from the original request may allow to bypass CSRF protection due to lack of validation at server-side.

- Altering the Anti-CSRF Token: When token checks are loosely handled, spoofing a same length token by changing some part of token may allow bypassing CSRF protection.

- Using the Attacker’s Anti-CSRF Token: When the server only checks if a token is valid but does not check which user the token is associated with, an attacker can simply provide their own CSRF token to satisfy server’s check and bypass the CSRF protection.

- Spoofing the Anti-CSRF Token: When tokens are generated using weak or no cryptography, an attacker may potentially generate spoofed tokens that impersonates other user in order to bypass CSRF protection.

- Using guessable Anti-CSRF Tokens: When an application is using a fairy small or guessable token, an attacker can potentially try to guess or brute force a valid token for the victim user to bypass CSRF protection.

- Stealing Anti-CSRF Tokens: When CSRF tokens are passed as cookie parameters without Secure and HTTPOnly flags, an attacker can potentially steal the CSRF token via XSS or other attacks.

- Swapping HTTP Verbs: When applications do not implement the same levels of security in similar HTTP methods such as GET and POST, an attacker may be able to swap verbs to bypass security. This method is covered in the article The Dark Side of Request Routing.

---
