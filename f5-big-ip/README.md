# Vulnerability: F5 BIG-IP Internal IP Address Disclosure via Encoded Cookie

The web resource uses an F5 BIG-IP load balancer to distribute incoming traffic. By default, F5 BIG-IP encodes the private IP addresses and ports of internal backend servers in the values of cookie files (specifically, the "BIGipServerkonsultant.mbank.kg" parameter). 
This encoding mechanism is predictable and decodable because it uses a standard translation of the internal IP address to HEX format and reads it in Little-Endian byte order. An unauthorized external actor can decode these values, which leads to the disclosure of the organization's internal network topology and infrastructure (Information Disclosure / CWE-200).

# Proof of Concept (PoC):
1. Intercept any HTTP request to the application "https://target.com" using a proxy server (e.g., Burp Suite).
2. Locate the F5 BIG-IP load balancer parameter in the "Cookie" headers:
   BIGipServerkonsultant.target.com=2455439882.47873.0000
3. Extract the first part of the value (before the first dot): "2455439882" (encoded internal IP address).
4. Extract the second part of the value: "47873" (backend port).
5. Perform decoding using the standard F5 Little-Endian algorithm:
   - 2455439882 -> HEX: 0x925B0A0A -> Reverse byte order: 0A.0A.5B.92 -> Decimal conversion: 10.10.91.146
   - 47873 -> Target backend port.
<img width="1280" height="694" alt="image" src="https://github.com/user-attachments/assets/38bb0b85-e8c7-4072-854d-7b307dfa1512" />

## Decoded Result:
- Internal IP address: 10.10.91.146 (Private RFC 1918 class network)
- Backend server port: 47873 (or 443 depending on the F5 version byte arrangement).

# Impact:
While this vulnerability is classified as Low severity on the CVSS scale, it introduces significant risks during an attacker's reconnaissance phase;:Perimeter 
1) Reconnaissance: The attacker acquires precise information regarding the bank's internal network addressing, the number of backend servers operating in the pool, and the active ports used
2) SSRF Attack Vector: If a Server-Side Request Forgery (SSRF) flaw is discovered in external systems, the attacker will not need to perform blind internal scanning. They will already possess the exact IP addresses of targeted internal systems to execute precise exploits or establish persistence

# Recommendation:
It is recommended to enable cookie encryption on the F5 BIG-IP load balancer to conceal the real internal IP addresses from external users.
Configuration steps for F5 administrators:
1. Navigate to the management console: Local Traffic -> Profiles -> Services -> HTTP.
2. Select or create the HTTP profile used for this resource.
3. In the "Cookie Transformation" section, find the "Cookie Encryption" setting.
4. Enable encryption (change status to Encrypted) and specify the cookie name: "BIGipServerkonsultant.target.com".
5. In the "Cookie Encryption Passphrase" field, set a strong encryption key.
6. Save and apply the configuration changes.
