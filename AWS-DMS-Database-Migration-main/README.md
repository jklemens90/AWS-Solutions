This is a project I completed where I migrated a word press web application from a simulated on-premises environment to AWS using the DMS. 

Private Connectivty was established using VPC Peering, which simulated a Direct Connect. RDS and EC2 instances were provisioned in AWS. Installation of apache web server was performed on the EC2 server.

It involved migrating word press content in the html folder from on-premises to the EC2 web server. I provisioned a replication instance and endpoints so the DB servers can communicate. I utilized DMS to transfer the data from the on-premises mariaDB database to an RDS instance. 

The architecture looked like this:

![image](https://user-images.githubusercontent.com/95970840/221116726-81cf25dc-b48e-4711-a145-57b74109980f.png)

A more in depth description of the process:

To accomplish our objective, I first had to create a VPC Peering Connection between the AWS and on-premise environment. Please note that a Peering Connection was used to simulate a Direct Connect in this scenario. The connection is between a requester VPC and a acceptor VPC. I added the on-premise VPC as the requester and specified the CIDR range. I then specified the AWS VPC as the acceptor and the respective CIDR range. Once completed, this created a secure channel and a gateway object in both VPCs.

The next step was to configure the route tables for the VPCs for each environment so they know how to send traffic to the other side of the VPC Peer. I configured the on-premise route table and added the CIDR of the AWS environment. I proceeded to then configure the route tables for the public and private AWS VPCs. I specified the CIDR range for the on-premise environment. 

The following stage was to provision infrastructure at the AWS side, such as the RDS database/subnet and an EC2 instance which functioned as the web application server.  I created the DB subnet group and specified the AWS VPC as the vpc. I added us-east-1a and us-east-1b as the AZ’s for the subnet. I subsequently creeated a database with MariaDB engine. This database would become the end-state database and eventually hold the migrated data from the on-premise database. I deployed this database into the AWS VPC. For the AWS web server, I launched an Amazon Linux instance and placed it into the AWS VPC and into the public A subnet. 

The next process was to install word press on the AWS web server I had generated. I connected to this instance and installed apache web server, mariadb database software, as well as command line tools. The following step was to transfer the content from the on-premises web server to this server. I used secure copy for the transfer. I did this by editing the SSH config on the machine to allow password authentication on a temporary basis. 

I then connected to the on-premise web server and used secure copy to copy the HTML folder to the AWS web server. I had to specify the private IP of the AWS web server for it to work accordingly. Once the assets were copied, I connected to the AWS web server and copied all the wordpress assets and configurations into the web root folder. Once this was completed, the AWS web server was functional as a wordpress application server while pointing at the on-premises database server. 

The last stage was to complete the database migration from the on-premise DB to the RDS instance using the Database Migration Service. In DMS, I created a subnet group and placed it in the AWS VPC while also specifying both aws private subnets. I subsequently created the replication instance and made it single AZ.  The next step was to configure endpoints. I created a source endpoint and specified the private IP of the on-premises database server along with port 3306. I then configured a destination endpoint and chose the RDS instance I previously created as the target. To make sure the endpoints could connect to each other, I used DMS’s built in feature to to test connections between endpoints. 

Once that connection was confirmed, I had to create the database migration task. That process included specifying the replication instance, source endpoint (on-prem DB), and target endpoint (AWS DB). I also had to utilize table mappings and specify a schema which was the name of the on-premises DB. I used "%" as a wildcard to migrate any tables within the on-premise database. I finally ran the migration task which successfully migrated all the data from the on-premise DB to the AWS DB.  

The final step was to cut over the AWS web application server so that instead of it pointing at the on-premises database, it would point to the RDS instance. I executed that task by connecting to the web server and editing a config file where I changed the database host IP to the RDS Host name. I ran a script to update the wordpress database with this new DNS name.

The end product is that I was able to create a VPC peer between the on-premise environment and AWS. I fully migrated wordpress application files from on-premise to AWS. I then was able to provision an RDS DB instance and used AWS DMS to execute a migration of the database to AWS.
