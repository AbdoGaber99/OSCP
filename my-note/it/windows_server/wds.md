---
description: Windows Deployment Services
icon: laptop-binary
---

# WDS

#### **Step-by-Step Guide to Installing Windows Deployment Services (WDS) on Windows Server**

Windows Deployment Services (WDS) is a server role that allows network-based installation of Windows operating systems, eliminating the need for physical installation media.

## <mark style="color:blue;">**Prerequisites**</mark>

Before installing WDS, ensure the following:

* A Windows Server with a **static IP address**.
* Active Directory Domain Services (AD DS) is **optional** but recommended.
* A **DHCP server** (for PXE booting).
* A **DNS server** for network name resolution.
* Windows **installation ISO file** (for boot and install images).

***

## <mark style="color:blue;">Step 1: Install WDS Role</mark>

1. Open **Server Manager** → Click **Manage** → Select **Add Roles and Features**.
2. Click **Next** until you reach **Server Roles**.
3. Check **Windows Deployment Services** → Click **Next**.
4. Expand **WDS**, select both:

* **Deployment Server**
* **Transport Server**

5. Click **Next** → Click **Install** → Wait for the installation to complete.
6. Click **Close** once the installation is complete

<figure><img src="../../../.gitbook/assets/1 (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/2 (7).png" alt=""><figcaption></figcaption></figure>

***

## <mark style="color:blue;">Step 2: Configure WDS Server</mark>

1. Open **Server Manager** → Click **Tools** → Select **Windows Deployment Services**.
2. Expand **Servers**, right-click your server name, and select **Configure Server**.
3. Click **Next** and select:

* **Integrated with Active Directory** (if in a domain).
* **Standalone Server** (if not using AD).

4. Select a **Remote Installation Folder Location** (e.g., `D:\RemoteInstall`).
5. Choose **Respond to all client computers (known and unknown)**.
6. Click **Finish**.



<figure><img src="../../../.gitbook/assets/3 (7).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/4 (5).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/5 (5).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/6 (5).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/7 (3).png" alt=""><figcaption></figcaption></figure>



***

## <mark style="color:blue;">**Step 3: Add Boot & Install Images**</mark>

1. pen **WDS Console** → Expand **Servers** → Click **Boot Images**.&#x20;
2. Right-click **Boot Images** → Select **Add Boot Image**.
3. Browse to the **Windows ISO** → Select `\sources\boot.wim` → Click **Next**.
4. Name the boot image (e.g., `Windows 7 Boot Image`) → Click **Next** → **Finish**.
5. Repeat the process for **Install Images**:

* Right-click **Install Images** → **Add Install Image**.
* Browse to `\sources\install.wim`.
* Select the **Windows editions** to deploy.
* Click **Next** → **Finish**.

<figure><img src="../../../.gitbook/assets/09 (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/10 (8).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/12 (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/13 (4).png" alt=""><figcaption></figcaption></figure>

***

## <mark style="color:blue;">**Step 4: Start WDS Service**</mark>

1. Open **WDS Console** → Right-click the **Server** → Click **Start**.

***

## <mark style="color:blue;">**Step 6: Boot a Client via Network (PXE Boot)**</mark>

1. Boot a client PC and press `F12` to start **PXE Boot**.
2. Select the **WDS Boot Image**.
3. Follow the Windows installation steps.

<figure><img src="../../../.gitbook/assets/14 (1).png" alt=""><figcaption><p>click l</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/001 (2).png" alt=""><figcaption></figcaption></figure>

***

## <mark style="color:blue;">Multicast Transmission in Windows Deployment Services (WDS)</mark>

Step 1: Install and Configure WDS (If Not Installed)

1. Open **Server Manager** → Click **Add roles and features**.
2. Select **Windows Deployment Services (WDS)** and install it.
3. Open **WDS Console** → Right-click on the **WDS Server** → Click **Configure Server**.
4. Choose **Integrated with Active Directory** (if AD is available) or **Standalone Mode**.
5. Set the **Remote Installation Folder** (e.g., `D:\RemoteInstall`).
6. Enable **PXE Boot** to allow network-based installations

