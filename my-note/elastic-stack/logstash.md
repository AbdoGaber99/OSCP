# Logstash

1. [download Logstash ](https://www.elastic.co/downloads/logstash)
2. move to /opt

```
sudo mv logstash-9.5.1 /opt/logstash
```

3. test logstash

```shellscript
/opt/logstash/bin/logstash -e 'input { stdin { } } output { stdout { codec => rubydebug } }'
```
