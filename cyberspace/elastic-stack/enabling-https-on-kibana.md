# Enabling HTTPS on Kibana

## Enabling HTTPS on Kibana — Full Step-by-Step Guide

Documentation for enabling HTTPS on the Kibana interface (Elasticsearch was already running on HTTPS, while Kibana was still running on plain HTTP).

***

### 1. Generate the Private Key

```bash
openssl genrsa -out kibana-server.key 2048
```

### 2. Create the Certificate Config File (OpenSSL config)

Create a file named `kibana-openssl.cnf` containing the certificate details and SAN (Subject Alternative Names):

```ini
[req]
distinguished_name = req_distinguished_name
x509_extensions = v3_req
prompt = no

[req_distinguished_name]
C = EG
ST = Cairo
L = Cairo
O = MyCompany
CN = kibana.example.com

[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = kibana.example.com
DNS.2 = localhost
IP.1 = 127.0.0.1
IP.2 = YOUR_SERVER_IP
```

> **Note:** Replace <mark style="color:$danger;">`kibana.example.com`</mark> and <mark style="color:$danger;">`YOUR_SERVER_IP`</mark> with the actual server values. Modern browsers (Chrome/Firefox) rely on the `[`<mark style="color:$danger;">`alt_names`</mark>`]` section rather than the CN field.

### 3. Generate the Certificate (self-signed)

```bash
openssl req -x509 -new -nodes -key kibana-server.key -sha256 -days 365 -out kibana-server.crt -config kibana-openssl.cnf
```

This produces two files:

* `kibana-server.key`
* `kibana-server.crt`

### 4. Create the Certificates Directory

```bash
sudo mkdir -p /etc/kibana/cert
```

### 5. Move the Files into the Directory

```bash
sudo mv kibana-server.crt kibana-server.key /etc/kibana/cert/
```

### 6. Set File Ownership and Permissions

Kibana runs as the `kibana` user, not `root`, so that user needs read access to the files:

```bash
sudo chown kibana:kibana /etc/kibana/cert/kibana-server.key
sudo chown kibana:kibana /etc/kibana/cert/kibana-server.crt

sudo chmod 640 /etc/kibana/cert/kibana-server.key
sudo chmod 644 /etc/kibana/cert/kibana-server.crt

sudo chmod 750 /etc/kibana/cert
sudo chown kibana:kibana /etc/kibana/cert
```

### 7. Update Kibana Configuration

Edit `/etc/kibana/kibana.yml`:

```bash
sudo nano /etc/kibana/kibana.yml
```

Add/update the following lines:

```yaml
server.ssl.enabled: true
server.ssl.certificate: /etc/kibana/cert/kibana-server.crt
server.ssl.key: /etc/kibana/cert/kibana-server.key
```

> **Critical step and the main failure point:** `server.ssl.enabled` must be set to `true`. If it's left as `false` while a certificate is configured, Kibana will keep serving plain HTTP on the same port, causing an `SSL_ERROR_RX_RECORD_TOO_LONG` error in the browser.

### 8. Restart Kibana

```bash
sudo systemctl restart kibana
```

### 9. Confirm Kibana Is Running (allow 20–40 seconds to boot)

```bash
sudo systemctl status kibana
```

You should see `Active: active (running)`.

### 10. Test the Connection from the Server Itself

```bash
curl -vk https://localhost:5601
```

If you get an HTML response back (even with a certificate warning due to `-k`), Kibana is correctly serving HTTPS.

### 11. Access from the Browser

Open:

```
https://<server-ip>:5601
```

You'll get a normal warning because the certificate is self-signed ("Warning: Potential Security Risk" in Firefox, or "Not Secure" in Chrome) — this is expected. Click **Advanced**, then **Accept the Risk and Continue**.

***
