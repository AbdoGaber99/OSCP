# Web Application Attacks

## Web Application Enumeration

* **Certificate Parsing (Subject Alternative Name (SAN))**

```bash
curl https://crt.sh/?q=domain.com
```

* **Subdomain Enumeration**

```bash
gobuster dns -d domain.com -w wordlist
```

* **Service Enumeration**

```bash
nmap domain.com
```

* **Directory Brute-Forcing**

```bash
dirsearch -u domain.com -e php
```

```bash
gobuster dir -u target_url -w wordlist
```

* Debugging Page Content
* Inspecting HTTP Response Headers and Sitemaps

```bash
curl https://www.google.com/robots.txt
```

```bash
curl -s https://target.com/sitemap.xml
```

* Enumerating and Abusing APIs
