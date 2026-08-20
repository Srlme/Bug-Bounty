# Session Fixation / Improper Session Invalidation

A critical session management vulnerability was identified within the authentication logic of target.com. The application fails to invalidate concurrent active sessions when a user changes their account password.
The lack of session invalidation (termination) after a password change is a vulnerability that allows an attacker to maintain access to the service. Most users expect that resetting a password will prevent anyone else from accessing their account

# Proof of Concept (PoC)
1) Create an account on target.com
2) Log into the account using the same credentials in two different browsers (Browser 1 and Browser 2).
3) Open the profile/settings page in both browsers.
4) Navigate to the "Change Password" section and change the password in Browser 1.
5) Switch to Browser 2, modify any profile details (name, phone number, or profile picture), and click "Save".
6) Refresh the page in Browser 2. If the session remains active, the vulnerability is present; if the session is terminated, the vulnerability does not exist.
7) If you were logged out in browser 2, try clicking the 'Login' button. You will be logged in immediately.
<img width="1280" height="715" alt="image" src="https://github.com/user-attachments/assets/7cfc5c4c-5e3d-4f77-90d3-0196343bc109" />


# Impact:
The business and security impact of this vulnerability is assessed as Medium-to-High. If an attacker gains unauthorized access to a victim's account (via session hijacking, shoulder surfing, or an un-invalidated shared device), the legitimate owner cannot revoke the attacker's access by simply changing their password.Even though the password is changed, the attacker's active session cookie (_identity) remains fully operational. The attacker retains permanent, unauthorized access to the victim's personal data, can manipulate profile information, and continue to use the service maliciously under the victim's identity until the cookie naturally expires.

# Recommendation:
1. Implement a session revocation mechanism tied to credential modification events. When a user successfully updates or resets their password, the backend server must immediately invalidate all active session records/tokens associated with that specific User ID in the session store (e.g., Redis, database, or JWT blacklisting).
2. If the application utilizes stateful database-driven sessions, destroy or delete all session rows matching the target user_id except for the currently active session ID that initiated the change request.
3. If using stateless tokens, implement a tracking column such as a password_version counter or a session_salt within the database user table. Include this value inside the token validation logic so that changing the password alters the version, causing all older tokens to instantly fail authentication checks.
