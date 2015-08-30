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
        index.number_of_replicas: 1
        gateway.recover_after_nodes: 1
        discovery.zen.minimum_master_nodes: 1
        discovery.zen.ping.multicast.enabled: false
        discovery.zen.ping.unicast.hosts: ["second-node-IP0rName"]

    ```
*   `sudo update-rc.d elasticsearch defaults 95 10`    
*   `service elasticsearch start`

