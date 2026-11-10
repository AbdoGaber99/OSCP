---
icon: sitemap
---

# DHCP Server

<div data-full-width="true"><figure><img src="../../../.gitbook/assets/1111111111111111111.png" alt=""><figcaption><p>DHCP Server Configuration</p></figcaption></figure></div>

## Introduction to DHCP Protocol

**Dynamic Host Configuration Protocol (DHCP)** is a network protocol used to **automatically assign IP addresses** and other network configurations (such as subnet mask, gateway, and DNS) to devices on a network. This eliminates the need for manual IP configuration, making network management more efficient.**work at UDP And port 67(Server), 68(client)**

#### **🔹 How DHCP Works (DORA Process)**

DHCP operates using a **four-step process** called **DORA**:

1️⃣ **Discover:**

* The client **broadcasts** a DHCP **Discover** message to find a DHCP server.

2️⃣ **Offer:**

* The DHCP server responds with an **Offer**, suggesting an available IP address.

3️⃣ **Request:**

* The client sends a **Request** to confirm the offered IP address.

4️⃣ **Acknowledge:**

* The DHCP server sends an **Acknowledgment (ACK)**, confirming the lease of the IP address

#### **🔹 Key DHCP Components**

✅ **DHCP Server** – Assigns IP addresses and manages leases.\
✅ **DHCP Client** – The device requesting an IP address.\
✅ **DHCP Scope** – A range of IP addresses assigned to clients.\
✅ **Lease Time** – The duration for which an IP is assigned

## **Step-by-Step: Installing and Configuring DHCP on Windows Server**

#### **🛠 Step 1: Install the DHCP Server Role**

1️⃣ Open **Server Manager**.\
2️⃣ Click **Manage** → Select **Add Roles and Features**.\
3️⃣ Click **Next** on the **Before You Begin** page.\
4️⃣ Select **Role-based or Feature-based installation**, then click **Next**.\
5️⃣ Choose the **server** from the list and click **Next**.\
6️⃣ Scroll down and check **DHCP Server**, then click **Next**.\
7️⃣ Click **Next** on the **Features** page.\
8️⃣ Click **Install** and wait for the installation to complete.\
9️⃣ Click **Close** when done



<figure><img src="../../../.gitbook/assets/1 (4).png" alt=""><figcaption></figcaption></figure>

#### **🛠 Step 2: Authorize the DHCP Server**

1️⃣ Open **Server Manager** → Click **Tools** → Select **DHCP**.\
2️⃣ In the **DHCP console**, expand the server name.\
3️⃣ Right-click the server name → Click **Authorize**.\
4️⃣ Right-click again and select **Refresh**.\
5️⃣ The server should now be **authorized**



<figure><img src="../../../.gitbook/assets/2 (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/3 (6).png" alt=""><figcaption></figcaption></figure>



#### **🛠 Step 3: Create a New DHCP Scope**

1️⃣ In the **DHCP Console**, right-click **IPv4** → Select **New Scope**.\
2️⃣ Click **Next** on the Welcome screen.\
3️⃣ Enter a **Scope Name** (e.g., "Office DHCP Scope") → Click **Next**.\
4️⃣ Enter the **IP address range** (e.g., `192.168.1.130 - 192.168.1.140`).\
5️⃣ Set the **Subnet Mask** (default is `/24` or `255.255.255.0`) → Click **Next**.\
6️⃣ Add **Excluded Addresses** (optional, for static IPs) → Click **Next**.\
7️⃣ Set the **Lease Duration** (default is 8 days) → Click **Next**.\
8️⃣ Choose **Yes, I want to configure these options now** → Click **Next**

<figure><img src="../../../.gitbook/assets/4 (4).png" alt=""><figcaption></figcaption></figure>

Enter the **IP address range** (e.g., `192.168.1.130 - 192.168.1.140`).

<figure><img src="../../../.gitbook/assets/5 (4).png" alt=""><figcaption></figcaption></figure>

Add **Excluded Addresses** (optional, for static IPs) → Click **Next**

<figure><img src="../../../.gitbook/assets/6 (4).png" alt=""><figcaption></figcaption></figure>

Choose **Yes, I want to configure these options now** → Click **Next**

<figure><img src="../../../.gitbook/assets/7 (2).png" alt=""><figcaption></figcaption></figure>

#### **🛠 Step 4: Configure DHCP Options**

1️⃣ Enter the **Default Gateway** (e.g., `192.168.1.1`) → Click **Next**.\
2️⃣ Enter the **DNS Server** (e.g., `192.168.1.10` for Active Directory) → Click **Next**.\
3️⃣ Skip the **WINS Server** setup → Click **Next**.\
4️⃣ Select **Yes, activate this scope now** → Click **Next**.\
5️⃣ Click **Finish**



<figure><img src="../../../.gitbook/assets/8 (3).png" alt=""><figcaption></figcaption></figure>

Enter the **DNS Server** (e.g., `192.168.1.100` for Active Directory or `8.8.8.8`) → Click **Next**.

<figure><img src="../../../.gitbook/assets/9 (2).png" alt=""><figcaption></figcaption></figure>

Select **Yes, activate this scope now** → Click **Next**

<figure><img src="../../../.gitbook/assets/10 (7).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/11 (7).png" alt=""><figcaption></figcaption></figure>

#### **Step 5: Verify DHCP is Working**

1️⃣ On a client computer, open **Command Prompt** and type:

```
ipconfig /release
ipconfig /renew

```

2️⃣ Check if the **client received an IP address** from the DHCP scope.

3️⃣ Run:

```
ipconfig /all
```

<figure><img src="../../../.gitbook/assets/12 (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/13 (2).png" alt=""><figcaption></figcaption></figure>
