# AWS Services Comparison

# Kinesis vs SNS vs SQS vs EventBridge

![img](imgs/events.png)

# Security Group vs NACL vs Route Tables

![img](imgs/vpc.png)

### Security Group
A security group acts as a virtual firewall for your instance to control inbound and outbound traffic. When you launch an instance in a VPC, you can assign up to five security groups to the instance. 

**Security groups act at the instance level, not the subnet level**. Therefore, each instance in a subnet in your VPC can be assigned to a different set of security groups.

The following are the basic characteristics of security groups for your VPC:
* Security Group rules are based on ALLOWs and there is no concept of DENY when in comes to Security Groups. This means you cannot explicitly deny or blacklist specific ports via Security Groups, you can only implicitly deny them by excluding them in your ALLOWs list
* Because of the above detail, everything is blocked by default. You must go in and intentionally allow access for certain ports.
* Security group rules enable you to filter traffic based on protocols and port numbers.
* Security groups are **stateful** — if you send a request from your instance, the response traffic for that request is allowed to flow in regardless of inbound security group rules. Responses to allowed inbound traffic are allowed to flow out, regardless of outbound rules.
* You can specify **separate rules for inbound and outbound traffic.**
* Security groups are specific to a single VPC, so you can't share a Security Group between multiple VPCs. However, you can copy a Security Group to create a new Security Group with the same rules in another VPC for the same AWS Account.
* Security Groups are regional and can span AZs, but can't be cross-regional.
* Outbound rules exist if you need to connect your server to a different service such as an API endpoint or a DB backend. You need to enable the ALLOW rule for the correct port though so that traffic can leave EC2 and enter the other AWS service.

### NACL

Network Access Control Lists (or NACLs) are like security groups but for subnets rather than instances. The main difference between security groups and NACLs is that security groups are stateless, meaning you can perform both allow and deny rules that may be divergent, depending if traffic is inbound or outbound, for that rule.

#### Differences between NACLs and Subnets.

| NACL | 	Security Group |
|------|-------------------|
| Operates at the subnet level | Operates at the instance level |
| Supports allow rules and deny rules | Supports allow rules only |
| Is stateless: Return traffic must be explicitly allowed by rules | Is stateful: Return traffic is automatically allowed, regardless of any rules |
| We process **rules in order, starting with the lowest numbered rule**, when deciding whether to allow traffic | We evaluate all rules before deciding whether to allow traffic|
| Automatically applies to all instances in the subnets that it's associated with (therefore, it provides an additional layer of defense if the security group rules are too permissive) | Applies to an instance only if someone specifies the security group when launching the instance, or associates the security group with the instance later on|

* Because NACLs are stateless, you must also ensure that outbound rules exist alongside the inbound rules so that ingress and egress can flow smoothly.
* The default NACL that comes with a new VPC has a default rule to allow all inbounds and outbounds. This means that it exists, but doesn't do anything as all traffic passes through it freely.
* NACLs are evaluated before security groups and **you block malicious IPs with NACLs, not security groups**.
* **Network ACL rules are evaluated by rule number, from lowest to highest, and executed immediately when a matching allow/deny rule is found. Because of this, order matters with your rule numbers**

**Traffic between instances within the same subnet do not pass through a NACL because the traffic is not exiting the subnet.**

### Route Table
A route table contains a set of rules, called routes, that are used to determine where network traffic from your subnet or gateway is directed.

Route tables are used to make sure that subnets can communicate with each other and that traffic knows where to go.

When you create a VPC, it automatically has a main route table. The main route table controls the routing for all subnets that are not explicitly associated with any other route table.


| Network ACL | Route Table |
| --- | --- |
|If someone is coming from my in-laws, DENY/ALLOW entry |WHOEVER wants to BE WITHIN INDOORS will use USUAL ROUTE(s) |
| If anyone is leaving my house, DENY/ALLOW entry | WHOEVER wants to BE IN BACKYARD will use KITCHEN ROUTE |
| If my friends from Sussex are here, ALLOW/DENY entry | EVERYONE ELSE need to use MAIN DOOR ROUTE |
| EVERYONE ELSE is DENIED | |



### NAT vs Internet Gateway

You can use a network address translation (NAT) instance in a public subnet in your VPC to enable instances in the private subnet to initiate outbound IPv4 traffic to the internet or other AWS services, but prevent the instances from receiving inbound traffic initiated by someone on the internet.

You can also use a NAT gateway, which is a managed NAT service that provides better availability, higher bandwidth, and requires less administrative effort. For common use cases, AWS recommends that you use a NAT gateway rather than a NAT instance.

| Attribute |	NAT gateway |	NAT instance |
|-----------|---------------|----------------|
|Availability |	Highly available. NAT gateways in each Availability Zone are implemented with redundancy. Create a NAT gateway in each Availability Zone to ensure zone-independent architecture.|	Use a script to manage failover between instances.|
|Bandwidth |	Can scale up to 45 Gbps. |	Depends on the bandwidth of the instance type. |
|Maintenance |	Managed by AWS. You do not need to perform any maintenance.	|Managed by you, for example, by installing software updates or operating system patches on the instance. |
|Performance |	Software is optimized for handling NAT traffic. | A generic Amazon Linux AMI that's configured to perform NAT. |
| Cost |	Charged depending on the number of NAT gateways you use, duration of usage, and amount of data that you send through the NAT gateways.	| Charged depending on the number of NAT instances that you use, duration of usage, and instance type and size.|
| Type and size	| Uniform offering; you don’t need to decide on the type or size.| Choose a suitable instance type and size, according to your predicted workload.|
|Public IP addresses| Choose the Elastic IP address to associate with a NAT gateway at creation. |Use an Elastic IP address or a public IP address with a NAT instance. You can change the public IP address at any time by associating a new Elastic IP address with the instance.|
| Security groups|	Cannot be associated with a NAT gateway. You can associate security groups with your resources behind the NAT gateway to control inbound and outbound traffic.|	Associate with your NAT instance and the resources behind your NAT instance to control inbound and outbound traffic.|
| Timeout Handling|When there is a connection timeout, a NAT gateway returns an RST packet to any resources behind the NAT gateway that attempt to continue the connection (it does not send a FIN packet).| When there is a connection timeout, a NAT instance sends a FIN packet to resources behind the NAT instance to close the connection.|

For IPv6 traffic, NAT is not supported, instead we use **Egress-only Internet Gateway**, which is a horizontally scaled, redundant, and highly available VPC component that allows outbound communication over IPv6 from instances in your VPC to the internet, and prevents the internet from initiating an IPv6 connection with your instances.

####  Internet Gateway

An Internet Gateway is a logical connection between an Amazon VPC and the Internet. It is not a physical device. Only one can be associated with each VPC. It does not limit the bandwidth of Internet connectivity. (The only limitation on bandwidth is the size of the Amazon EC2 instance, and it applies to all traffic -- internal to the VPC and out to the Internet.)

If a VPC does not have an Internet Gateway, then the resources in the VPC cannot be accessed from the Internet (unless the traffic flows via a corporate network and VPN/Direct Connect).

A subnet is deemed to be a Public Subnet if it has a Route Table that directs traffic to the Internet Gateway.


## ELB vs Route 53

1. ELB distributes traffic among Multiple Availability Zone but not to multiple Regions. Route53 can distribute traffic among multiple Regions. In short, ELBs are intended to load balance across EC2 instances in a single region whereas DNS load-balancing (Route53) is intended to help balance traffic across regions.
2. Both Route53 and ELB perform health check and route traffic to only healthy resources. Route53 weighted routing has health checks and removes unhealthy targets from its list. However, DNS is cached so unhealthy targets will still be in the visitors cache for some time. On the other hand, ELB is not cached and will remove unhealthy targets from the target group immediately. 

**Use both Route53 and ELB**: Route53 provides integration with ELB. You can use both Route53 and ELB in your AWS infrastructure. If you have AWS resources in multiple regions, you can use Route53 to balance the load among those regions. Inside the region, you can use ELB to load balance among the instances running in various Availability Zones.

