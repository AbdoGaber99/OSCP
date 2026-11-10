# INE Splunk lab

**Step 1 - Configure Splunk and SERVER01 to receive logs:**

Open Splunk from the desktop and navigate to _Settings_, then _Forwarding and receiving_

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/1.png" alt=""><figcaption></figcaption></figure>

Click on _Add new_ next to **Configure receiving** to add a new receiver. Enter port _9997_ and click _Save_

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/3.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/2.png" alt=""><figcaption></figcaption></figure>

Next, open up Windows Firewall settings to allow traffic on port 9997 through the local firewall.

Go to _inbound rules_ and create a new rule

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/4.png" alt=""><figcaption></figcaption></figure>

Choose the following options to create the rule: - Port - TCP, on port 9997 only - Allow the connection - Enable for all profiles - Name: **Splunk Forwarder**

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/6.png" alt=""><figcaption></figcaption></figure>

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/5.png" alt=""><figcaption></figcaption></figure>

**Step 2 - Install and configure the Splunk Forwarder on DC01:**

Open up RDP to connect to DC01. Use the Domain Admin credentials listed in the lab summary

![Content Image](https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/7.png)

Open up the _Tools_ folder on the desktop of DC01 and launch the Splunk Forwarder installer

![Content Image](https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/8.png)

Check the box at the top to accept the license agreement, and ensure that **An on-premises Splunk Enterprise instance is selected**, then choose _Customize Options_

![Content Image](https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/9.png)

Accept the default installation location, and accept all the defaults on the SSL certificate screen

![Content Image](https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/10.png)

Ensure **Local System** is selected for the account that Splunk Forwarder will be installed as

![Content Image](https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/11.png)

On the following screen, select **Security Logs** and **Enable AD monitoring** and choose _Next_

![Content Image](https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/12.png)

Enter a username for the Splunk administrator account, and then either enter a password or let the installer generate a random password

![Content Image](https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/13.png)

**Leave the Deployment Server information empty**

On the **Receiving Indexer** screen, enter the IP address for SERVER01, and the port that you previously configured for the receiver on Splunk (should be 9997)

![Content Image](https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/14.png)

Return to SERVER01 (we're done on DC01 for now) and go back to Splunk. Click on _Splunk>Enterprise_ at the top, and then choose _Search & Reporting_ from the navigation bar

![Content Image](https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/15.png)

Enter the following search query in the search bar to verify you're receiving logs. Ensure you change the time frame on the right of the search bar to **All time**

`index=* host=DC01`

![Content Image](https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/16.png)

After a moment, you should see results returned. If you do not see any events after the search completes, go back and double-check your configuration to ensure all settings are correct.

**Step 3 - Install Splunk Forwarder on LINUX01 and configure to send logs to SERVER01:**

Use Putty to SSH to the Linux server. Use the credentials for the Linux server listed in the lab information to connect.

Once connected via SSH, there are a few prerequisites to complete before installing the forwarder:

* Create a new user to be the owner for the installation:

```bash
sudo useradd -m splunk
```

* Create a variable to easily store the installation location:

```bash
export SPLUNK_HOME="/opt/splunkforwarder"
```

* Create the folder

```bash
sudo mkdir $SPLUNK_HOME
```

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/17.png" alt=""><figcaption></figcaption></figure>

With the prerequisites completed, we can install the forwarder now

```bash
sudo dpkg -i splunkforwarder-9.0.4-de405f4a7979-linux-2.6-amd64.deb
```

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/18.png" alt=""><figcaption></figcaption></figure>

Now that the forwarder is installed, change the ownership of the folder to the user we created earlier:

```bash
sudo chown splunk:splunk $SPLUNK_HOME -R
```

Now change directories into the **$SPLUNK\_HOME/bin** directory. **The rest of the commands will not work if this step is not completed**

```bash
cd $SPLUNK_HOME/bin
```

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/19.png" alt=""><figcaption></figcaption></figure>

Start the Splunk Forwarder (accepting the license agreement):

```bash
sudo ./splunk start --accept-license
```

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/20.png" alt=""><figcaption></figcaption></figure>

You will be prompted to create a new administrator for this forwarder. Enter a username and password when prompted. **You will need this information again in a moment**

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/21.png" alt=""><figcaption></figcaption></figure>

The forwarder will take a moment to start, and then it will show its completed startup:

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/22.png" alt=""><figcaption></figcaption></figure>

Next, we need to tell Splunk where to send its logs. Enter the following command to configure the forwarding. Enter the username/password you just created when prompted.

```bash
sudo ./splunk add forward-server 172.31.115.110:9997
```

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/23.png" alt=""><figcaption></figcaption></figure>

Next, we need to tell Splunk what logs to monitor. In our case, we need to monitor the file **/var/log/auth.log**. Use the following command to configure this. If done within a few minutes of the last command, you should not be prompted for the username again.

```bash
sudo ./splunk add monitor /var/log/auth.log
```

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/24.png" alt=""><figcaption></figcaption></figure>

Now, for good measure, restart the Splunk Forwarder to ensure all the new settings are applied

```bash
sudo ./splunk restart
```

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/25.png" alt=""><figcaption></figcaption></figure>

Once the restart is complete (should only take a few seconds), return to the Splunk search console and repeat our previous search, but replacing the host with _ip-172-31-115-111_ this time.

<figure><img src="https://assets.ine.com/content/labs/ptp/BrianOlliff/VOD-4529/LAB-4740/26.png" alt=""><figcaption></figcaption></figure>

Now that we have all of the logs being sent to the Splunk server, feel free to perform various searches to see what type of data we are capturing.

We'll perform additional analysis on these logs (and more) in the _Log Analysis_ lab.
