# Missing Secure Flag

During the security assessment of the target.com web application, it was discovered that the critical session cookie _identity is issued without the Secure attribute. 
While the HttpOnly flag is correctly implemented (preventing client-side script access via XSS), the absence of the Secure flag allows the browser to transmit this sensitive session identifier over unencrypted HTTP connections. In a standard production environment, session cookies must always be protected with the Secure flag to ensure they are strictly confined to encrypted HTTPS channels.

# Proof of Concept (PoC)
1. Navigate to the target web application at https://target.com and perform a standard user authentication or session initialization.
2. Open the browser's Developer Tools (F12) and navigate to the "Storage" tab (or "Application" depending on the browser), then select "Cookies" -> "https://target.com".
3. Locate the core authentication/session cookie named _identity.
4. Inspect the attributes of the _identity cookie. Observe that under the "Secure" column, the value is explicitly set to false.
5. Alternatively, capture the inbound server response headers (such as Set-Cookie) using an intercepting proxy like Burp Suite or the Network tab, and verify that the Secure directive is omitted from the token declaration.
<img width="1280" height="202" alt="image" src="https://github.com/user-attachments/assets/fcf17e22-2a5b-44c9-b01e-f3f5fb107593" />


# Impact:
The absence of the Secure attribute creates a risk of session hijacking via credential exposure over unencrypted networks. 
If a user is connected to an insecure or untrusted network (such as public Wi-Fi) and accidentally triggers or navigates to an unencrypted HTTP link associated with the domain (e.g., via a hardcoded non-HTTPS resource reference, an image, or a legacy redirect), the browser will automatically include the _identity cookie in that cleartext HTTP request. An adversary capable of sniffing local network traffic (Man-in-the-Middle) can intercept the plain text transmission, capture the session token, and impersonate the victim's authenticated session without needing their password.

# Recommendation:
1. Modify the session management configuration on the application server to append the Secure attribute whenever the Set-Cookie header is generated for the _identity and _csrf-frontend tokens:
Set-Cookie: _identity=[value]; Secure; HttpOnly; SameSite=Lax;
2. Ensure that strict Transport Layer Security is enforced across the entire infrastructure by deploying HTTP Strict Transport Security (HSTS) headers. This forces compliant browsers to interact with the domain exclusively over HTTPS, mitigating accidental cleartext leakages.
