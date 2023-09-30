This was a project where I implemented highly available and dynamic network connectivity between AWS and on-premises routers by establishing two Site-to-Site VPN connections and using BGP routing.

The end state architecture looked like this:
![image](https://user-images.githubusercontent.com/95970840/220499900-4f7701b8-cb77-40f9-8565-f9a78a0ddbd6.png)

The first stage was to create the infrastructure in AWS. A VPC was provisioned along with two multi-AZ EC2 servers inside it. A Transit Gateway was created along with an attachment to connect to the VPC. 2 customer gateways were created in AWS to represent 2 physical on-premises routers. The public IP addresses and ASNs from the on-premise environments were added to these customer gateways. 

*The ASN number from the customer side was important because it allowed the on-premise side and AWS side to exchange routes using BGP.*

In stage 2 of the project I created two VPN attachments for the Transit Gateway. There was 1 VPN connection for each of the customer gateways. Each VPN connection had 2 IPsec tunnels, one tunnel went between AWS Endpoint A and the customer gateway.  As well as another tunnel that went between AWS Endpoint B and the customer gateway. When configuring the VPN attachments I chose dynamic routing with BGP and opted to use an accelerated VPN for increased performance. The accelerated VPNs use global accelerator and AWS Global Network.

*BGP dynamic routing provided automatic failover and chose the most optimal network path for high availability.*

*The purpose of the IPsec tunnels is that it's an encrypted connection between the AWS environment and customer environment. They're secured by using a pre-shared key. There's two tunnels per each VPN connection. Both of the tunnels go between AWS and one customer gateway. At the AWS side, they go to two different AWS endpoints in different availability zones to ensure highly availability. A single VPN connection is established by configuring two tunnels between 2 AWS outside IPs and 2 customer gateway outside IPs. Each tunnel is also configured between an AWS inside IP and a customer inside IP. The Inside IPs are what BGP runs over and the data is sent over between AWS and the on-premises environment.*

In the next part, I downloaded the configuration files for each VPN connection. I then extracted important configuration items from the files and used it to configure the strongSwan IPsec VPN systems that ran on the on-premises routers. This included all the IPs and pre-shared keys. This was accomplished by connecting to the routers and editing configuration files.

In the last stage, I added the BGP sessions over all the tunnels. This allowed the on-premise routers to exchange routes with the Transit Gateway running in AWS and then ultimately data. To add BGP capability I first had to connect to the on-premise routers and install FRR.  I then had to use the shell that is used to configure FRR. I specified the on-premises ASN. I then defined the AWS ASN and the BGP neighbors (AWS inside IP address). The ASN and BGP neighbor IP addresses were necessary to be able to send data between on-premises and AWS. 

Below are pictures and proof that the IPsec tunnels were established correctly:

On-Premise Router 1-

![onpremrouter 1 final route table](https://user-images.githubusercontent.com/95970840/220513969-13379fb0-4e1d-421d-b51c-3ce442693daa.png)

"C" next to an IP address means it's a route that is connected to this particular operating system.
"B" next to an IP address means it's a BGP-learned route. 
10.16.0.0/16 (AWS) was learned via both vti1 and vti2 (IPsec tunnels). 

On-Premise Router 2- 

![onpremrouter 2 final route table](https://user-images.githubusercontent.com/95970840/220514477-8fa02717-ca2f-452f-929e-b6ab8dfe3623.png)

10.16.0.0/16 (AWS) was learned via both vti1 and vti2 (IPsec tunnels) in this scenario as well. 

On-premise to EC2-A ping response

![ping response on-prem to EC2-A](https://user-images.githubusercontent.com/95970840/220515187-6260b498-7c2a-4396-b527-41f4661be4e0.png)

EC2-A to on-premise ping response

![ping response EC2-A to on prem](https://user-images.githubusercontent.com/95970840/220515276-cb6a3a6f-ae5f-4323-9496-7bb5d8405c65.png)

I was able to implement a highly available VPN architecture that can survive the failure of an Availability Zone at the AWS side or the failure of a customer router at the on-premises side. 








