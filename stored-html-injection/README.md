# Stored HTML Injection (HTML Email Injection)

A Stored HTML Injection vulnerability was discovered in the contact feedback form. The application fails to properly sanitize and sanitize user-supplied input in the ContactForm[name] parameter before embedding it into the email body sent to the administrator. 
An attacker can exploit this vulnerability by injecting malicious HTML tags into the form fields. When an administrator opens the incoming notification email or views the message in the backend panel, the injected HTML code executes within the context of their email client or browser session. This can lead to phishing attacks, defacement of the email content, or session hijacking if upgraded to XSS.

# Proof of Concept (PoC)
1. Navigate to the target website: https://target.com
2. Locate the contact/feedback form.
3. Fill out the form, and intercept the outgoing POST request using a proxy tool (e.g., Burp Suite).
4. Inject the following HTML payload into the ContactForm[name] parameter (properly URL-encoded as shown in the PoC request):
Click here to confirm
5. Send the request. Note that the server responds with a "200 OK" and {"success": true, "message": "..."} confirming the successful processing of the payload without any sanitization.
6. When the administrator opens the generated email notification, the injected HTML tags will render and execute.
<img width="1280" height="583" alt="image" src="https://github.com/user-attachments/assets/9e955b9c-318b-458a-a4c5-4a306af01509" />


# Impact:
1. Phishing & Social Engineering: Attackers can alter the visual structure of the email to spoof official administrative alerts, forcing the reader to click on malicious links or input credentials on a phishing page.
2. Information Disclosure: By embedding external assets (e.g., <img src="...">), an attacker can track when the email is opened, leaking the administrator's IP address, User-Agent, and geographic location.
3. Potential Session Hijacking: If the target environment allows script execution or lacks a strong Content Security Policy (CSP), this flaw can easily be escalated to a Stored Cross-Site Scripting (XSS) vulnerability.

# Recommendation:
1. Implement Context-Aware Output Encoding: Ensure that all user-supplied data is strictly encoded (HTML Entity Encoding) before rendering it inside email templates or web pages. Transform characters like < and > into &lt; and &gt;.
2. Utilize Strict Sanitization Libraries: If HTML formatting is required by design, use robust, industry-standard sanitization libraries (e.g., DOMPurify or built-in Yii framework security helpers tools like HtmlPurifier) to strip out dangerous tags and attributes before processing.
3. Enforce Plain Text Emails: Configure the mailer component to send notifications in plain text format rather than rich HTML if advanced styling is not strictly necessary.
