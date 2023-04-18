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
    *This setting is not recommended in Production level
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

## Setting up S3 Bucket, AWS Glue(Crawler), AWS Athena
### Setup S3 Bucket
1. Go to AWS console
2. Go to S3 page in AWS console and click `Create bucket`
3. Configure the S3 bucket
    - Name of the S3 Bucket
    - Select the region (Same as EC2 region)
    - Leave the rest setting to as is for this project
4. Configure AWS in your local machine to upload file from your local machine (If you don't have a user)
    - Go to IAM page
    - Select `Users` in the left panel
    - Create an user first (You have to create a user first before you give programmatic access)
        - User name
        - Permission options  
          AdministratorAccess from `Attach policies directly`  
          *For this project, give Admin access to the user
        - Review and Create
    - Click the user you just created and go to Security credentials
    - Go to Access Key > `Create access key`
    - Configure Access Key
        - Click `Command Line Interface (CLI)`
        - (Optional) Add description of the key
        - (Optional) Download .csv file to see Access key & Secret access key
    - Download AWS CLI and type `aws configure` in the terminal
        - AWS Access Key ID: <YOUR_ACCESS_KEY>
        - AWS Secret Acess Key: <YOUR_SECRET_ACCESS_KEY>
        - Default Region Name: <YOUR_REGION_NAME>
        - (Optional) Default Output Format: <OUTPUT_FORMAT>
5. Now you can access S3 bucket from your local machine  
   ```python
   from s3fs import S3FileSystem
   ```
### Setup AWS Glue with Crawler
1. Go to AWS console
2. Go to Crawlers (AWS Glue feature) page in AWS console and click `Create crawler`
3. Configure the Crawler
    - Crawler name
    - Add data source
        - Location of S3 data: In this account
        - S3 path: <YOUR_S3_BUCKET_NAME>  
          Put the slash at the end of the S3 path
        - Leave the rest setting to as is for this project
    - IAM Role 
        - Go to IAM page
        - Select `Roles` in the left panel
        - Create a role 
            - Trusted entity type: AWS service
            - Use cases for other AWS services: Glue
            - Give `AdministratorAccess` for this project
            - Type Role name
    - Target database (AWS Glue database)  
      Frequency: On demand for this project
4. Run the Crawler (Wait till the job is over)
### Setup AWS Athena
1. Go to AWS console
2. Go to Athena
3. Go to Settings > Manage 
    - Select another AWS S3 Bucket for stroing temporary queries (Create one if you don't have extra bucket)
4. Configure the setting
    - Data Source: AWSDataCatalog
    - Database: AWS Glue Database
    - Tables: AWS S3 Bucket
5. Run SQL query
   ```sql
   SELECT * FROM "YOUR_GLUE_DATABASENAME"."YOUR_AWS_S3_BUCKET_NAME"
   ```