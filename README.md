# aws-elk
## ELK cluster on the 4 Ec2 instance based on the ubuntu 14:04

### Elastic Search node installation and configuration

* 	`sudo su` --- become root
*   `sudo add-apt-repository -y ppa:webupd8team/java` --- add Java PPA
* 	`sudo apt-get update && sudo apt-get -y install oracle-java8-installer`
* 	`wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -`
* 	`echo 'deb http://packages.elasticsearch.org/elasticsearch/1.4/debian stable main' | sudo tee /etc/apt/sources.list.d/elasticsearch.list`
* 	`cd /root && wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.4.4.noarch.rpm`
* 	`sudo apt-get update && sudo apt-get -y install elasticsearch=1.4.4`
* 	`rm -f elasticsearch-1.4.4.noarch.rpm`
* 	`cd /usr/share/elasticsearch/`
* 	`/usr/share/elasticsearch/bin/plugin -install mobz/elasticsearch-head`
* 	`/usr/share/elasticsearch/bin/plugin -install lukas-vlcek/bigdesk`
* 	`/usr/share/elasticsearch/bin/plugin -i elasticsearch/marvel/latest`
* 	`vi /etc/elasticsearch/elasticsearch.yml` --- add the following lines in the elasticsearch.yml

    ```
        cluster.name: elk-stack
        node.name: "mps-elasticsearch-1"
        index.number_of_replicas: 2
        gateway.recover_after_nodes: 2
        discovery.zen.minimum_master_nodes: 2
        discovery.zen.ping.multicast.enabled: false
        discovery.zen.ping.unicast.hosts: ["second-node-IP0rName"]

    ```
*   `sudo update-rc.d elasticsearch defaults 95 10`    
*   `sudo service elasticsearch start`

### Logstash Node Installation and configuration

* 	`sudo su` --- become root
*   `sudo add-apt-repository -y ppa:webupd8team/java` --- add Java PPA
* 	`sudo apt-get update && sudo apt-get -y install oracle-java8-installer`
* 	`wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -`
* 	`echo 'deb http://packages.elasticsearch.org/logstash/1.5/debian stable main' | sudo tee /etc/apt/sources.list.d/logstash.list`
* 	`sudo apt-get update && sudo apt-get install logstash`
* 	`sudo mkdir -p /etc/pki/tls/certs`
* 	`sudo mkdir /etc/pki/tls/private && sudo mkdir /etc/pki/tls/private`
* 	`sudo vi /etc/ssl/openssl.cnf` 
Search `[ v3_ca ] ` and add following line
*   `subjectAltName = IP: logstash_server_private_ip`
*   `cd /etc/pki/tls && sudo openssl req -config /etc/ssl/openssl.cnf -x509 -days 3650 -batch -nodes -newkey rsa:2048 -keyout private/logstash-forwarder.key -out certs/logstash-forwarder.crt`
*   `sudo vi /etc/logstash/conf.d/01-lumberjack-input.conf`

    ```
        input {
            lumberjack {
                port => 5000
                type => "logs"
                ssl_certificate => "/etc/pki/tls/certs/logstash-forwarder.crt"
                ssl_key => "/etc/pki/tls/private/logstash-forwarder.key"
                     }
                }
    ```

* `sudo vi /etc/logstash/conf.d/10-syslog.conf`

    ```
        filter {
            if [type] == "syslog" {
              grok {
                    match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp}" }
                    add_field => [ "received_at", "%{@timestamp}" ]
                    add_field => [ "received_from", "%{host}" ]
                    }
             syslog_pri { }
                 date {
                       match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
                     }
              }
        }
    ```
* `sudo vi /etc/logstash/conf.d/30-lumberjack-output.conf`

    ```
        output
        {
            stdout {
            codec => rubydebug
        }
            elasticsearch {
             cluster => "elk-stack"
                 }
        }
    ```

*   `sudo add-apt-repository -y ppa:webupd8team/java` --- add Java PPA
* 	`sudo apt-get update && sudo apt-get -y install oracle-java8-installer`
* 	`wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -`
* 	`echo 'deb http://packages.elasticsearch.org/elasticsearch/1.4/debian stable main' | sudo tee /etc/apt/sources.list.d/elasticsearch.list`
* 	`cd /root && wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.4.4.noarch.rpm`
* 	`sudo apt-get update && sudo apt-get -y install elasticsearch=1.4.4`
* 	`rm -f elasticsearch-1.4.4.noarch.rpm`
* 	`cd /usr/share/elasticsearch/`
* 	`/usr/share/elasticsearch/bin/plugin -install mobz/elasticsearch-head`
* 	`/usr/share/elasticsearch/bin/plugin -install lukas-vlcek/bigdesk`
* 	`/usr/share/elasticsearch/bin/plugin -i elasticsearch/marvel/latest`
* 	`vi /etc/elasticsearch/elasticsearch.yml` --- add the following lines in the elasticsearch.yml

    ```
        cluster.name: elk-stack
        node.name: "mps-logstash-01"
        node.master: false
        node.data: false
        discovery.zen.ping.multicast.enabled: false
        discovery.zen.ping.unicast.hosts: ["First-node-IPorName""second-node-IP0rName"]
    ```

*   `sudo update-rc.d elasticsearch defaults 95 10`    
*   `sudo service logstash restart`
*   `sudo service elasticsearch start`

### Kinbana Installation and configuration

* 	`sudo su` --- become root
*   `sudo add-apt-repository -y ppa:webupd8team/java` --- add Java PPA
* 	`sudo apt-get update && sudo apt-get -y install oracle-java8-installer`
* 	`wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -`
* 	`cd /root && wget https://download.elasticsearch.org/kibana/kibana/kibana-4.0.1-linux-x64.tar.gz`
* 	`tar xvf kibana-*.tar.gz`
* 	`sudo mkdir -p /opt/kibana`
* 	`sudo cp -R ~/kibana-4*/* /opt/kibana/`
* 	`cd /etc/init.d && sudo wget https://gist.githubusercontent.com/thisismitch/8b15ac909aed214ad04a/raw/bce61d85643c2dcdfbc2728c55a41dab444dca20/kibana4`
*   `sudo chmod +x /etc/init.d/kibana4`
*   `sudo update-rc.d kibana4 defaults 96 9`
*   `sudo service kibana4 start`
*   `sudo apt-get install nginx apache2-utils`
*   `sudo htpasswd -c /etc/nginx/htpasswd.users kibanaadmin`
*   `sudo vi /etc/nginx/sites-available/default`
    ```
        server {
            listen 80;
    
            server_name localhost;
    
            auth_basic "Restricted Access";
            auth_basic_user_file /etc/nginx/htpasswd.users;
    
            location / {
                proxy_pass http://localhost:5601;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;        
            }
        }
    ```
    
*  `sudo service nginx restart`
* 	`wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -`
* 	`echo 'deb http://packages.elasticsearch.org/elasticsearch/1.4/debian stable main' | sudo tee /etc/apt/sources.list.d/elasticsearch.list`
* 	`cd /root && wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.4.4.noarch.rpm`
* 	`sudo apt-get update && sudo apt-get -y install elasticsearch=1.4.4`
* 	`rm -f elasticsearch-1.4.4.noarch.rpm`
* 	`cd /usr/share/elasticsearch/`
* 	`/usr/share/elasticsearch/bin/plugin -install mobz/elasticsearch-head`
* 	`/usr/share/elasticsearch/bin/plugin -install lukas-vlcek/bigdesk`
* 	`/usr/share/elasticsearch/bin/plugin -i elasticsearch/marvel/latest`
* 	`vi /etc/elasticsearch/elasticsearch.yml` --- add the following lines in the elasticsearch.yml

    ```
        cluster.name: elk-stack
        node.name: "mps-kibana-01"
        node.master: false
        node.data: false
        discovery.zen.ping.multicast.enabled: false
        discovery.zen.ping.unicast.hosts: ["First-node-IPorName""second-node-IP0rName"]
    ```
    
*   `sudo update-rc.d elasticsearch defaults 95 10`    
*   `sudo service elasticsearch start`

### Logforwarded Installation

* `echo 'deb http://packages.elasticsearch.org/logstashforwarder/debian stable main' | sudo tee /etc/apt/sources.list.d/logstashforwarder.list`
* `wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -`
* `sudo apt-get update && sudo apt-get install logstash-forwarder`
* `sudo mkdir -p /etc/pki/tls/certs && cp /root/logstash-forwarder.crt /etc/pki/tls/certs/`
* `sudo rm -rf /etc/logstash-forwarder.conf && `
    ```
        {
        "network": {
                    "servers": [ "LOGSTASHIP:5000" ],
                    "timeout": 15,
                    "ssl ca": "/etc/pki/tls/certs/logstash-forwarder.crt"
             },

                  "files": [
                         {
                        "paths": [
                                "/var/log/apache2/access.log",
                                "/var/log/apache2/error.log"
                                ],
                     "fields": { "type": "syslog" }
                        }
                    ]
                }

    ```
*  `sudo service logstash-forwarder restart`

