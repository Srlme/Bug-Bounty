# Sensitive Data Exposure / Infrastructure Information Disclosure

A critical Information Disclosure vulnerability was discovered due to an enabled debug/development mode on the production server. The application leaks highly sensitive system and database infrastructure details when errors or unhandled exceptions occur. 
Two distinct leakage vectors were identified: 
1. Full Path Disclosure: Unhandled framework exceptions return a complete stack trace in JSON format, exposing absolute server paths (e.g., /var/www/astepltr/data/www/target.com/...).
2. Database Schema Disclosure: Input validation failures (such as a database column truncation error SQLSTATE[22001]) cause the backend to expose the exact raw SQL query being executed. This leaks the internal structure of the database, including table names (user) and sensitive column names (password_hash, auth_key, verification_token, iin, etc.).

# Proof of Concept (PoC)
## Vector 1 (Path Disclosure):
1. Navigate to the password reset endpoint.
2. Submit a request to trigger an unhandled server exception.
3. Observe the full file system path layout in the response stack trace.
<img width="1280" height="641" alt="image" src="https://github.com/user-attachments/assets/1b0410c9-50f2-4fd1-abda-c950ee448dd9" />


## Vector 2 (Database Schema Disclosure): 
1. Go to the registration/signup page.
2. Fill in the form parameters, but supply an excessively long string inside the SignupForm[full_name] parameter (e.g., containing long strings or HTML tags).
3. Intercept the request and submit it.
4. Analyze the HTTP 500 response, which contains the raw INSERT INTO 'user' SQL query, fully exposing the backend database schema.
<img width="1170" height="695" alt="image" src="https://github.com/user-attachments/assets/b916c9af-44fa-4864-8723-b8e74de093f1" />


# Impact:
1) Infrastructure Map: Attackers obtain an accurate blueprint of the server's file system structure and the database architecture.
2) Targeted Exploitation: Leaking database schemas dramatically lowers the bar for discovering and exploiting hidden SQL injection points, as table and column names are already known.
3) Sensitive Assets Exposure: Knowing where files reside on the disk (/var/www/...) helps attackers construct paths for potential Local File Inclusion (LFI) or arbitrary file read vulnerabilities.

# Recommendation:
1. Disable Production Debugging: Set YII_DEBUG to false and ensure YII_ENV is set to 'prod' within the global environment configuration to block developer error reporting.
2. Enforce Custom Error Handling: Implement a global catch-all error handling configuration that logs detailed exception traces locally (for admins only) and displays a uniform, neutral error response to end-users.
3. Audit Input Constraints: Align the front-end/back-end application validation rules with the structural column length constraints defined in the database schema to prevent raw database level errors from being triggered.
