This was a project I completed where I established hybrid network connectivity and DNS integration between AWS and on-premises servers which redirected queries/traffic from the on-premise DNS servers to AWS. 

I utilized a VPC peering connection between the on-premises environment and AWS to simulate a Direct Connect connection. Created a private hosted zone in Route53 in AWS and configured inbound and outbound endpoints.

Edited configuration files for the on-premise DNS servers and specified the private IP addresses of the AWS Route53 inbound endpoints to facilitate integration.I then edited DNS configuration files for the on-premises application server and added the private IP addresses of the DNS servers. This allowed the on-premises application server to use the DNS servers.

The end state architecture looked like this:

<img width="1916" alt="hybrid-dns-endstate" src="https://user-images.githubusercontent.com/95970840/221038235-2d0bba3d-63d2-42d1-baf6-bf3388a2f3cc.png">

To accomplish this (further detail):

In the first stage, I had to establish hybrid network connectivity. I accomplished this by creating a VPC Peering connection, which was used to simulate a AWS Direct Connect between the AWS and on-premises environment.  This was done so by configuring the route tables for the AWS VPC and then adding the route for the CIDR range of the on-premises environment. I then had to do the reverse and configure the route table for the on-premises VPC and add the CIDR range for the AWS VPC. Once this was completed, there was network connectivity between both environmen.ts 

I tested that connection by connecting to the AWS EC2-A server and pinging the IP address of the on-prem app server. I received a ping response which indicated we had network connectivity. However, I still had to integrate DNS though.

To integrate DNS, I utilized Route53 and configured inbound endpoints which allowed the on-premises environment to resolve DNS records that are stored on the DNS infrastructure in the AWS environment. I copied down the IP addresses of the inbound endpoints that were created in AWS. These IP addresses would be necessary in the following stages. 

*Route53 inbound endpoints are elastic network interfaces which will run in AWS and can be used by the on-premises environment.*

I then had to do some configuration in the on-premise environment. I connected to the DNS A server and edited a configuration file that specifies the IP addresses that will have queries forwarded to them. It was here where I added the IP addresses of the AWS inbound endpoints I had previously created. I executed the previous steps for DNS B server.

To see if I configured everything correctly, I visited the domain at web-aws-animals4life.org. I was able to get a response back that confirms the two IP addresses are associated with our AWS inbound endpoints. I proceeded to go to Route53 hosted zones and observed I had an A record of the web.aws.animals4life.org domain which points to the 2 EC2 instances inside the AWS environment. 

The next thing I had to do was configure all the on-premise application server to use the DNS servers. I accomplished this by connecting to the server and edited a configuration file for the ethernet. I added DNS1 and DNS2 to the file and mentioned the private IP’s of the DNS servers. These are the same DNS servers that are configured to forward queries to AWS. After rebooting the instance, I connected to the on-prem application server and was able to get both a DNS resolution and ping response. This told me I was able to establish network connectivity between the on-prem environment and AWS. As well as configure inbound integration so the DNS servers that are running on-premise have been configured to forward queries to Route53 inbound endpoints.

The last part was to configure the outbound endpoints in Route53. I created a rule and specified the domain name and environment that the rule will forward queries for. I provided the private IP addresses of the on-premise DNS servers. I proceeded to connect to the AWS EC2 servers and ping the on-premise application server. With a response back it confirmed to me that I was able to integrate hybrid network connectivity and DNS integration between AWS and the on-premise servers. This ultimately allowed for queries to be redirected from the on-premise DNS servers to AWS. 
