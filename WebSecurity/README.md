# Web Security

### XSS (Cross-site Scripting)  attacks
XSS stands for Cross Site Scripting and it is injection type of attack. Cross site scripting is the method where the attacker injects malicious script into trusted website.
There are 3 types of such attacks.
1. Stored XSS - Vulnerability coming from unprotected and not sanitized user inputs those are directly stored in database and displayed to other users
2. Reflected XSS - Vulnerability coming from unprotected and not sanitized values from URLs those are directly used in web pages
3. DOM based XSS - Similar as reflected XSS, unprotected and not sanitized values from URLs used directly in web pages, with difference that DOM based XSS doesn't even go to server side

#### Stored XSS
Here is an example of attack. The attacker comes on your website and finds unprotected input field such as comment field or user name field and enters malicious 
script instead of expected value. After that, whenever that value should be displayed to other users it will execute malicious code. 
Malicious script can try to access your account on other websites, can be the involved in DDoS attack or similar.

#### Reflected XSS
Reflected XSS is attack that happens when attacker discovers page with such vulnerability, for example:\
expected URL: `https://mywebpage.com/search?q=javascript`\
malicious URL: `https://mywebpage.com/search?q=<script>alert('fortunately, this will not work!')</script>`\
For this attack, we need a reflected field. For example, when you search something in Google, if reflects your search in the top bar of the search result page. 
Simply, malicious URL (JS code in the reflected field) created by perpetrator is sent to the victim. When the victim clicks the URL, vulnerable website reflects code and malicious code is executed on the user's
browser.

#### DOM based XSS
This kind of attack is same as reflected but with difference that malicious URL part will not be sent to server at all. For above example:\
expected URL: `https://mywebpage.com/search?q=javascript`\
malicious URL(reflected XSS): `https://mywebpage.com/search?q=<script>alert('fortunately, this will not work!')</script>`\
malicious URL(DOM based XSS): `https://mywebpage.com/search#q=<script>alert('fortunately, this will not work!')</script>`\
Difference is in the character # being used instead of ?. The browsers do not send part of URL after # to server so they pass it directly to your client code.

#### Protection
1. String validation
Validation is the method where user defines set of rules, and demand untrusted data to satisfy those rules before moving on. This method is good for number values, username, email, password and similar fields with concrete set of syntax rules.
2. String escape
Escape method is useful for cases where you should enable user to use punctuation marks. This method goes through string and looks for special characters, such as < > and replace them with appropriate HTML character entity name.
3. String sanitization
Sanitizing string is used when user is allowed to enter some HTML tags in their comments, articles or similar. The sanitize method goes through text and looks for HTML tags that you specify and removes them.

### CSRF attack
Cross-site request forgery (also known as CSRF) is a web security vulnerability that allows an attacker to induce users to perform actions that they do not 
intend to perform. It allows an attacker to partly circumvent the same origin policy, which is designed to prevent different websites from interfering with 
each other. In a successful CSRF attack, the attacker causes the victim user to carry out an action unintentionally. For example, this might be to change the 
email address on their account, to change their password, or to make a funds transfer. Depending on the nature of the action, the attacker might be able to gain 
full control over the user's account. If the compromised user has a privileged role within the application, then the attacker might be able to take full control 
of all the application's data and functionality.\
If a victim user visits the attacker's web page, the following will happen:
- The attacker's page will trigger an HTTP request to the vulnerable web site.
- If the user is logged in to the vulnerable web site, their browser will automatically include their session cookie in the request (assuming SameSite cookies are not being used).
- The vulnerable web site will process the request in the normal way, treat it as having been made by the victim user, and change their email address.

#### Protection
The most robust way to defend against CSRF attacks is to include a CSRF token within relevant requests. The token should be:
- Unpredictable with high entropy, as for session tokens in general.
- Tied to the user's session.
- Strictly validated in every case before the relevant action is executed.
