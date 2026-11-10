# Web Security monitoring

### Components of a Web Service

<figure><img src="../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

### Web Servers

* <mark style="color:$danger;">**apache**</mark> `/var/www/html/`
* <mark style="color:$danger;">**IIS**</mark>
  * **Website Root Directory:** `C:\inetpub\wwwroot`
  * **IIS Configuration Files:** `%windir%\System32\inetsrv\config` (contains `applicationHost.config`)
  * **IIS Application Management DLLs:** `%windir%\System32\inetsrv`
  * **Default Log Files:** `%SystemDrive%\inetpub\logs\LogFiles`
* <mark style="color:$danger;">**NGNIX**</mark> `/usr/share/nginx/html/`

### protected the web

**Protecting the Application**

* Secure Coding: Avoid insecure functions, ensure proper handling of errors, and remove sensitive information.
* Input Validation & Sanitization: Validate and sanitize user input to prevent injection attacks.

**Protecting the Web Server**

* <mark style="color:$danger;">Web Application (WAF):</mark> Filter and block harmful traffic based on defined rules.
* <mark style="color:$danger;">Content Delivery Network (CDN)</mark>: Reduce direct exposure to your server and use integrated WAFs.

**Protecting the Host Machine**

* Least Privilege: Use low-privilege users for services.
* System Hardening: Disable unnecessary services and close unused ports.
* Antivirus: Add endpoint-level protection that blocks known malware.

### Logging

<figure><img src="../.gitbook/assets/image (45).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

### Web Application Firewall (WAF)

1. Signature-Based Detection:  <mark style="color:$danger;">User-Agent that matches a known tool</mark>  <mark style="color:$danger;"></mark><mark style="color:$danger;">`sqlmap/1.8.1`</mark>
2. Heuristic-Based Detection: <mark style="color:$danger;">A long query string with special characters</mark> <mark style="color:$danger;"></mark><mark style="color:$danger;">`search?q=%3Cscript%20(1)`</mark>
3. Anomaly & Behavioral Analysis: <mark style="color:$danger;">A single IP address makes repeated login attempts in a short period of time</mark>
4. Location & IP Reputation Filtering: <mark style="color:$danger;">A request from an IP address that is outside of your normal business area</mark>

<figure><img src="../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

### Content Delivery Network (CDN)

<figure><img src="../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

### Web Shell

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### Web & Auditd Correlation

* A suspicious `POST` request in web logs can be linked to an audit event that includes a `creat` or `execve` syscall
