# Security Misconfiguration / Missing SPF and DMARC Records

During the infrastructure configuration review, it was discovered that the domain target.com lacks both valid SPF (Sender Policy Framework) and DMARC (Domain-based Message Authentication, Reporting, and Conformance) records in its DNS configuration. 
The complete absence of an SPF record means there is no authorized list of IP addresses allowed to send emails on behalf of the domain. Coupled with the missing DMARC policy, receiving mail servers have no security instructions on how to handle unauthorized emails, making the domain highly vulnerable to identity spoofing.

# Proof of Concept (PoC)
1. Use a tool like dig or nslookup to verify the absence of the records: 
  dig txt target.com (Observe that no SPF record starting with v=spf1 is returned).
  dig txt _dmarc.target.com (Observe that no DMARC record is found).
2. Use an online email spoofing service or a custom mail script to forge an email.
3. Set the sender address to an unauthorized corporate identity (e.g., support@target.com) and the recipient to a personal testing email address.
4. Send the email and observe that it successfully delivers to the destination inbox without being rejected by the receiving server's security filters.
<img width="1280" height="437" alt="image" src="https://github.com/user-attachments/assets/e83e56ee-0356-4603-a4f3-7ce594d1df15" />
<img width="1280" height="341" alt="image" src="https://github.com/user-attachments/assets/2ed07c16-7d2c-4834-887d-362a7701670e" />



# Impact:
1. Unrestricted Email Spoofing: Attackers can send high-credibility phishing emails under the guise of official company addresses to employees, partners, or customers.
2. Bypassing Mail Filters: Since there are no strict rules (v=spf1 -all or p=reject), major email providers are forced to accept the forged emails, increasing the overall success rate of social engineering and spear-phishing attacks.

# Recommendation:
1. Implement an SPF Record: Define a TXT record for target.com specifying all legitimate outbound mail servers, ending with a hard fail mechanism, for example: v=spf1 ip4:188.244.115.56 -all (adjust based on actual infrastructure).
2. Implement a DMARC Record: Publish a DMARC TXT record under _dmarc.target.com to monitor and enforce email authentication policy: v=DMARC1; p=quarantine; rua=mailto:dmarc-reports@target.com
