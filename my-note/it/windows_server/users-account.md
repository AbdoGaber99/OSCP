---
description: 'Step-by-Step: Create a User in an Active Directory Domain'
icon: users
---

# Users Account

## <mark style="color:red;">**Method 1: Using Active Directory Users and Computers (GUI)**</mark>



1. **Open Active Directory Users and Computers (ADUC)**

* Press `Win + R`, type **`dsa.msc`**, and press **Ente**

2. **Navigate to the Organizational Unit (OU**

* Expand your domain (e.g., `cairo.local`).
* Locate the **OU** where you want to create the user (e.g., `Users` )

4. **Create a New User**

* Right-click on the desired **OU** > Select **New** > **User**.
* **Enter User Information**
  * Fill in the following details:
    * **First Name**: (Abdo)
    * **Last Name**: (Gaber)
    * **User Logon Name**: (abdo.gaber)

5. **Set the Password**

* Enter and confirm the password.
* Choose password options:
  * **User must change password at next logon** (Recommended).
  * **Password never expires** (Not recommended for security).

6. **Finish User Creation**

* Click **Next** > **Finish**

<figure><img src="../../../.gitbook/assets/2 (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/3 (5).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:red;">**Method 2: Using PowerShell (CLI)**</mark>

1. **Open PowerShell as Administrator**
   * Press `Win + X` > Click **Windows PowerShell (Admin)**.
2. **Run the Command to Create a User**

```powershell
New-ADUser -Name "Abdo Gaber" -GivenName "Abdo" -Surname "Gaber" -SamAccountName "abdo.g-aber" -UserPrincipalName "abdo.gaber@cairo.local" -Path "OU=Users,DC=cairo,DC=local" -AccountPassword (ConvertTo-SecureString "P@ssw0rd" -AsPlainText -Force) -Enabled $true

```

3. **Verify User Creation**

```powershell
Get-ADUser -Identity "abdo.gaber"

```

<figure><img src="../../../.gitbook/assets/4 (3).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:red;">**Method 2: Using Command Prompt (CMD)**</mark>

1. **Open CMD as Administrator**
   * Press `Win + R`, type **`cmd`**, and press `Ctrl + Shift + Enter`.
2. **Create a User Account**

```powershell
dsadd user "cn=abdo,cn=users,dc=cairo,dc=local" -pwd p@ssw0rd -fn Abdo -ln Gaber mustchpwd yes
```

if to add OU

```
dsadd user "cn=abdo,ou=name-of-OU,dc=cairo,dc=local" -pwd p@ssw0rd -fn Abdo -ln Gaber mustchpwd yes
```



3. **Verify User Exists in Domain**

```
net user abdo.gaber /domain
```

## <mark style="color:red;">**Step-by-Step: Configure Logon Hours for a Domain User**</mark>

Logon hours restrict when a user can log in to the domain. You can configure logon hours using **Active Directory Users and Computers (GUI)**

#### **Using Active Directory Users and Computers (GUI)**

1. **Open Active Directory Users and Computers (ADUC)**
   * Press `Win + R`, type **`dsa.msc`**, and press `Enter`.
2. **Find the User**
   * Navigate to **Users** (or the appropriate container where the user is stored).
   * Right-click the user and select **Properties**.
3. **Set Logon Hours**
   * Go to the **Account** tab.
   * Click **Logon Hours**.
   * Select the allowed or denied hours by clicking on the grid.
   * Click **OK** and then **Apply**.
4. **Verify Settings**
   * Try logging in during restricted hours to confirm the settings are applied.

<figure><img src="../../../.gitbook/assets/5 (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/6 (3).png" alt=""><figcaption><p>restrict when a user can log in to the domain From 8:00AM to 4:00PM from Sun to Thur</p></figcaption></figure>

## <mark style="color:red;">**Step-by-Step: Restrict a User to Log in from Only One Computer in a Domain**</mark>

Restricting a user to log in from a single computer enhances security by preventing unauthorized access from other devices. You can configure this restriction using **Active Directory Users and Computers (GUI)**

#### **Method 1: Using Active Directory Users and Computers (GUI)**

1. **Open Active Directory Users and Computers (ADUC)**
   * Press `Win + R`, type **`dsa.msc`**, and press `Enter`.
2. **Locate the User Account**
   * Navigate to **Users** (or the relevant container).
   * Right-click on the user and select **Properties**.
3. **Configure Logon Workstations**
   * Go to the **Account** tab.
   * Click **Log On To...**
   * Select **The following computers** and enter the allowed computer name(s).
   * Click **OK**, then **Apply**, and **OK** again.
4. **Verify the Restriction**
   * Try logging in from an unauthorized computer to confirm the restriction is enforced.

<figure><img src="../../../.gitbook/assets/7 (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/8 (2).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:red;">**Step-by-Step: Set an Account Expiration Date in Active Directory**</mark>

Setting an expiration date for a user account is useful for temporary users, contractors, or security policies requiring automatic deactivation.

#### **Method 1: Using Active Directory Users and Computers (GUI)**

1. **Open Active Directory Users and Computers (ADUC)**

* Press `Win + R`, type **`dsa.msc`**, and press `Enter`.

2. **Find the User Account**

* Navigate to **Users** (or the appropriate Organizational Unit).
* Right-click on the user and select **Properties**.

3. **Set the Account Expiration Date**

* Go to the **Account** tab.
* Under **Account expires**, select **End of** and choose the expiration date.
* Click **Apply**, then **OK**.

4. **Verify the Expiration Date**

* Open the **user properties** again to ensure the correct date is set



<figure><img src="../../../.gitbook/assets/9 (1).png" alt=""><figcaption></figcaption></figure>

## <mark style="color:red;">**Step-by-Step: Configure Roaming Profiles in Active Directory**</mark>

Roaming Profiles allow users to access their personal desktop, files, and settings from any domain-joined computer. This is useful in enterprise environments where users switch devices frequently.

#### **Step 1: Create a Shared Folder for Roaming Profiles**

1. **Log in to the File Server**
   * This will be the server that stores user profiles.
2. **Create a Shared Folder**
   * Open **File Explorer** and create a folder (e.g., `C:\Profiles`).
   * Right-click the folder and select **Properties**.
3. **Set Sharing Permissions**
   * Go to the **Sharing** tab → Click **Advanced Sharing**.
   * Check **Share this folder** and click **Permissions**.
   * Remove `Everyone` and add **Authenticated Users**.
   * Assign **Full Control, Change, and Read** permissions.
   * Click **OK** and close.
4. **Set NTFS Security Permissions**
   * Go to the **Security** tab → Click **Advanced**.
   * Disable **inheritance** and remove existing users except for `SYSTEM` and `Administrators`.
   * Click **Add**, type **Authenticated Users**, and select:
     * **Traverse folder / execute file**
     * **List folder / read data**
     * **Create folders / append data**
   * Click **OK** to save.



#### **Step 2: Configure Roaming Profiles in Active Directory**

1. **Open Active Directory Users and Computers (ADUC)**
   * Press `Win + R`, type `dsa.msc`, and press **Enter**.
2. **Edit a User Account**
   * Locate the user, right-click, and select **Properties**.
3. **Set the Profile Path**
   * Go to the **Profile** tab.
   * In the **Profile Path** field, enter:

```
\\ServerName\Profiles\%username%
```

* Replace `ServerName` with your file server’s name IN MY CASE server1.
* `%username%` is a variable that automatically creates a folder for each user

4. **Apply and Save Changes**
   * Click **Apply** → **OK**

<figure><img src="../../../.gitbook/assets/10 (3).png" alt=""><figcaption></figcaption></figure>
