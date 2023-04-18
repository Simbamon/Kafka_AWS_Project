# Kafka AWS Project

## Setting up Apache Kafka with AWS EC2
### Launch EC2 instance in AWS
1. Go to AWS console
2. Go to EC2 page in AWS console and click `Launch instance`
3. Configure the EC2 instance
    - Name of the EC2 instance
    - OS images (Amazon Linux 2 AMI - Free tier eligible)
    - Instance type (t2.micro - Free tier eligible)
    - Key pair (To ssh into the EC2 instance)
        - Key pair name
        - download the .pem when it's done
    - Leave the rest setting to as is for this project
4. Set up the security inbound rules
    - Add TCP Port number 9092 for Kafka server
    - Add all traffic with your own IP to connect the Kafka from your local machine
    - Note: This setting is not recommended in Production level
5. SSH into the EC2 instance
    - Git Bash
        - Select the directory with .pem file inside
        - Open the Git Bash and type this command:
            ```bash
            $ ssh -i "<YOUR_KEYPAIR_FILENAME>.pem" <YOUR_EC2_INSTANCE_PUBLIC_IPv4_DNS>
            ```
    - PuTTY
        - Open the PuTTYgen
        - Go to File > Load private key
        - Change the setting to All Files (*.*) and select the .pem file
        - Click `Save private key` (This will make .ppk file)
        - Open the PuTTY
        - In Host Name, type your EC2 instance's public IPv4 DNS name
        - Navigate to Connection > SSH > Auth > Credentials
        - Select .ppk file you made in PuTTYgen in `Private key file for authentication`
        - (Optional) Save the session
        - Type `ec2-user` when new session pop up
### Download Apache Kafka in EC2 Instance
1. Download the Apache Kafka (Binary)
    ```bash
    $ wget https://downloads.apache.org/kafka/3.4.0/kafka_2.13-3.4.0.tgz
    $ tar -xvf kafka_2.13-3.4.0.tgz
    ```
2. Download java
    ```bash
    $ sudo yum install java-1.8.0-openjdk
    $ java -version
    ```
3. Start the Zookeeper server
    ```bash
    $ kafka_2.13-3.4.0/bin/zookeeper-server-start.sh config/zookeeper.properties
    ```
4. For this project, change the `ADVERTISED_LISTENERS` in server.properties to EC2 instance's public IP
    ```
    $ sudo vi kafka_2.13-3.4.0/config/server.properties
    ```
5. In a new session, start the Kafka server
    ```bash
    $ export KAFKA_HEAP_OPTS="-Xmx256M -Xms128M"
    $ kafka_2.13-3.4.0/bin/kafka-server-start.sh config/server.properties
    ```
### Create Topic and Start Producer & Consumer
1. Create the topic (In new session)
    ```bash
    $ kafka_2.13-3.4.0/bin/kafka-topics.sh --create --topic <YOUR_TOPIC_NAME> --bootstrap-server <YOUR_EC2_INSTANCE_PUBLIC_IP>:9092 --replication-factor 1 --partitions 1
    ```
2. Start the producer
    ```bash
    $ kafka_2.13-3.4.0/bin/kafka-console-producer.sh --topic <YOUR_TOPIC_NAME> --bootstrap-server <YOUR_EC2_INSTANCE_PUBLIC_IP>:9092
    ```
3. Start the consumer (In new session)
    ```bash
    $ kafka_2.13-3.4.0/bin/kafka-console-consumer.sh --topic <YOUR_TOPIC_NAME> --bootstrap-server <YOUR_EC2_INSTANCE_PUBLIC_IP>:9092
    ```