# How Integrating Elasticsearch, and Kibana (ELK Stack)

1. _**Introduction**_
2. _**Environment & Prerequisites**_
3. _**Install Elasticsearch and Kibana**_
4. _**Configure Elasticsearch**_
5. _**Configure Kibana**_

I**ntroduction**

<figure><img src="https://miro.medium.com/v2/resize:fit:522/1*ysNzv203MSIQaYDEws53IQ.png" alt="" height="464" width="522"><figcaption></figcaption></figure>

The **ELK Stack** (**Elasticsearch, Logstash, and Kibana**) combined with **Beats** provides a complete log collection, processing, and visualization solution. This write-up explains how to set up and integrate these components for centralized log management

#### what is Elasticsearch? <a href="#fb27" id="fb27"></a>

Elasticsearch is a distributed search and analytics engine built on top of Apache Lucene. It is designed to handle large volumes of structured and unstructured data, enabling real-time search, full-text search, logging, monitoring, and data analysis

Elasticsearch stores data as **JSON documents** and indexes them for fast retrieval. The architecture includes:

* **Index** → A collection of related documents (similar to a database).
* **Document** → A unit of data stored in JSON format.
* **Shard** → A subset of an index to distribute data across nodes.
* **Replica** → A copy of a shard for fault tolerance.
* **Query DSL** → A powerful query language for searching and filtering data.

**What is kibana ?**

Kibana is an open-source data visualization and analytics platform designed to work with Elasticsearch. It provides a user-friendly interface for exploring, analyzing, and visualizing data stored in an Elasticsearch cluster. Kibana is commonly used for log analysis, security monitoring

**What is Logstash**

Logstash is an open-source data processing pipeline that collects, processes, and sends data to Elasticsearch or other destinations. It is a key component of the ELK Stack (Elasticsearch, Logstash, Kibana) and is used for log collection, transformation, and enrichment before indexing in Elasticsearch.

**How Logstash Works**

1\. Input

* Collects data from multiple sources, such as **log files, databases, cloud services, and network streams**.
* Common input plugins: `file`, `beats`, `http`, `syslog`, `tcp`, `udp`.

2\. Filter

* Processes, transforms, and enriches incoming data before sending it to Elasticsearch.
* Common filter plugins: `grok`, `mutate`, `geoip`, `date`, `json`, `drop`.

3\. Output

* Sends processed data to **Elasticsearch, databases, message queues, or storage services**.
* Common output plugins: `elasticsearch`, `stdout`, `file`,etc.

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*19VYP4yEAcDuJCCIpiGzKg.png" alt="" height="186" width="700"><figcaption><p>Logstash work as pipeline</p></figcaption></figure>

### Environment & Prerequisites <a href="#id-9ac8" id="id-9ac8"></a>

* **OS**: Ubuntu 20.04 / CentOS / Windows
* **Elasticsearch Version**: 8.x
* **Kibana Version**: 8.x (must be compatible with Elasticsearch)
* **Java**: Installed (if required by Elasticsearch)
* **Minimum RAM**: 4GB (Recommended: 8GB

### _**Install Elasticsearch and Kibana**_ <a href="#c654" id="c654"></a>

#### Step 1: Preparation <a href="#c2b0" id="c2b0"></a>

```
sudo wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.13.0-amd64.deb
sudo wget https://artifacts.elastic.co/downloads/kibana/kibana-8.13.0-amd64.deb
```

```
sudo apt install openjdk-11-jdk -y
java --version
```

#### Step 2: Install Elasticsearch and Kibana <a href="#dc8f" id="dc8f"></a>

```
sudo apt install ./elasticsearch-8.13.0-amd64.deb
sudo apt install ./kibana-8.13.0-amd64.deb
```

#### Step 3: Configure Elasticsearch <a href="#eca1" id="eca1"></a>

configure elasticsearch.yml file

```
sudo nano /etc/elasticsearch/elasticsearch.yml
```

network.host: 0.0.0.0

http.port: 9200

```
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
sudo systemctl status elasticsearch.service
```

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*eh35KxNFMfDItcOJTW0c4A.png" alt="" height="219" width="700"><figcaption><p>status elasticsearch</p></figcaption></figure>

for more information about service

```
sudo netstat -tulnp | grep 9200
```

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*gBB8xP048nF_vMVJJgE51A.png" alt="" height="30" width="700"><figcaption></figcaption></figure>

> Note: work at IPv6

```
sudo nano /etc/elasticsearch/jvm.options.d/ipv4.options
-Djava.net.preferIPv4Stack=true
```



Login credentials

User\_name: elastic

password: \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

run this command to reset password

```
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

finally test Elasticsearch

```
curl -k -u elastic https://[::1]:9200
```

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*3kBFDVVon8lNUmPikOCJaA.png" alt="" height="272" width="700"><figcaption></figcaption></figure>

**Configure Kibana**

```
Sudo nano /etc/kibana/kibana.yml
```

Configure server.host and server.port

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*-P63soscrdbYqluKRmPt0A.png" alt="" height="185" width="700"><figcaption></figcaption></figure>

Enable and start Kibana

```
sudo systemctl enable kibana.service
sudo systemctl start kibana.service
sudo systemctl status kibana.service
```

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*9TgubU4xRlD9dCAtfxT-wQ.png" alt="" height="255" width="700"><figcaption></figcaption></figure>

> Access Kibana’s setup page at [_http://localhost:5601_](http://localhost:5601)

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*vseOmFdlKtHSnj_EpM_X8w.png" alt="" height="468" width="700"><figcaption></figcaption></figure>

The enrollment token can be generated using the following command

```
Sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*8mGNXnoYEGKjTXccrSbP0g.png" alt="" height="468" width="700"><figcaption></figcaption></figure>

After the enrollment token has been configured, you will be prompted to enter the verification code which can be retrieved using:

```
sudo /usr/share/kibana/bin/kibana-verification-code
```

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*EeXGyBPCIFHJ_4v-8EJl3Q.png" alt="" height="71" width="700"><figcaption></figcaption></figure>

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*XRqbAi7asxONR0YAdjWjOg.png" alt="" height="363" width="700"><figcaption></figcaption></figure>

The username and the password are the ones used for the Elasticsearch login.

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*jreQPvLzIZBrEkDzVO8DIA.png" alt="" height="377" width="700"><figcaption></figcaption></figure>

finally

Press enter or click to view image in full size

<figure><img src="https://miro.medium.com/v2/resize:fit:700/1*9tGxXIKuG87pIM6nRLKuXw.png" alt="" height="383" width="700"><figcaption></figcaption></figure>
