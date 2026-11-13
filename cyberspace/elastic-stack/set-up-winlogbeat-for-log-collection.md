# Set up Winlogbeat  for log collection

### Winlogbeat <a href="#winlogbeat" id="winlogbeat"></a>

We will begin by installing Winlogbeat on our Windows machine.

* _**`Winlogbeat`**_: [https://www.elastic.co/downloads/beats/winlogbeat](https://www.elastic.co/downloads/beats/winlogbeat)

1. we need to extract the contents into `C:\Program Files`
2. run the following commands to install the service.

```powershell
powershell.exe -ExecutionPolicy Unrestricted -File .\install-service-winlogbeat.ps1
```

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-13 194656.png" alt=""><figcaption></figcaption></figure>

3. update the Elasticsearch output section:&#x20;

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-13 194953.png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
* **`ssl.verification_mode: none`**→ This will bypass the certificate check.
* **`protocol: "https"`** → This tells Winlogbeat to use the HTTPS protocol when connecting
{% endhint %}

4. modify the `winlogbeat.yml` configuration file to enable the Windows event logs we want to collect

```
winlogbeat.event_logs:

  - name: Security
    event_id: 4624, 4625, 4634, 4647, 4648, 4688, 4689, 4720, 4728, 4732, 4719
  - name: Microsoft-Windows-Sysmon/Operational

  - name: Microsoft-Windows-PowerShell/Operational
    event_id: 4103, 4104, 4105, 4106
```

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-13 195759.png" alt=""><figcaption></figcaption></figure>

***

## testing

1. test the configuration file to identify any potential issues

```
.\winlogbeat.exe test config -c .\winlogbeat.yml -e
```

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-13 200023.png" alt=""><figcaption></figcaption></figure>

2. test the connection to our output

```
.\winlogbeat.exe test output -c .\winlogbeat.yml -e
```

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-13 200125.png" alt=""><figcaption></figcaption></figure>

3. start the **`winlogbeat`** service

```
Start-Service winlogbeat
Get-Service winlogbeat
```

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-13 200258.png" alt=""><figcaption></figcaption></figure>

4. run **Winlogbeat** using the `winlogbeat.yml` configuration file and shows real-time logs in the console

```
.\winlogbeat.exe -c .\winlogbeat.yml -e
```

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-13 200339.png" alt=""><figcaption></figcaption></figure>

5. verify that the logs are properly displayed in Kibana

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-13 200628.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/Screenshot 2026-04-13 200556.png" alt=""><figcaption></figcaption></figure>
