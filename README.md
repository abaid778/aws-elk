# aws-elk
## Elastic Search cluster on the 2 Ec2 instance based on the Amazon Linux AMI
* 	`sudo su`
* 	`yum update -y`
* 	`cd /root`
* 	`wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.4.4.noarch.rpm`
* 	`yum install elasticsearch-1.4.4.noarch.rpm -y`
* 	`rm -f elasticsearch-1.4.4.noarch.rpm`
* 	`cd /usr/share/elasticsearch/`
* 	`./bin/plugin -install mobz/elasticsearch-head`
* 	`./bin/plugin -install lukas-vlcek/bigdesk`
* 	`./bin/plugin install elasticsearch/elasticsearch-cloud-aws/2.4.1`
* 	`cd /etc/elasticsearch`
* 	`vi elasticsearch.yml` --- add the following lines in the elasticsearch.yml

    ```
    cluster.name: elkstak
    cloud.aws.access_key: ACCESS_KEY_HERE
    cloud.aws.secret_key: SECRET_KEY_HERE
    cloud.aws.region: us-east-1
    discovery.type: ec2
    discovery.ec2.tag.Name: "ELK Stack - Elasticsearch"
    http.cors.enabled: true
    http.cors.allow-origin: "*"
    ```
*   `service elasticsearch start`
