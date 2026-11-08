## AWS VPC & Networking — Complete Summary

Cloud Practitioner Revision Notes

This summary covers everything we studied: VPC, subnets, internet connectivity, security, VPC connections, endpoints, VPNs, and network monitoring.

## 1. VPC Fundamentals

## Concept

Region

Availability Zone (AZ)

Subnet

Route Table

CIDR

ENI

VPC

## Meaning

Your own logically isolated private network inside AWS.

A geographical AWS location containing multiple AZs. A VPC belongs to one Region.

An isolated location within a Region containing one or more data centers.

A smaller IP range inside a VPC. Each subnet belongs to one AZ.

Determines where network traffic goes.

Defines an IP address range, e.g. 10.0.0.0/16.

Elastic Network Interface; a virtual network card for an AWS resource.

## Public vs Private Subnets

Public subnet: Has a route to an Internet Gateway.

Private subnet: Does not have a direct route to an Internet Gateway.

A subnet's route table determines whether it is public or private, not its name.

## Example:

VPC: 10.0.0.0/16

│

├── Public Subnet: 10.0.1.0/24


```
│ └── Web Server (EC2)
│
└── Private Subnet: 10.0.2.0/24
└── Database
```

## Important CIDR facts

## CIDR Total IPv4 addresses

```
/16 65,536
/24 256
/25 128
/26 64
```

AWS reserves 5 IPv4 addresses in every subnet.

## Route Table Example

```
10.0.0.0/16 → local
0.0.0.0/0 → Internet Gateway
```

local: Communication inside the VPC.

0.0.0.0/0: All IPv4 destinations not matched by a more specific route.

## 2. Internet Connectivity

## Internet Gateway (IGW)

Connects a VPC to the internet.

Attached to the VPC, not directly to EC2.


Supports inbound and outbound internet communication when routes, public IP addressing, and security rules permit.

One IGW can be attached to a VPC at a time.

No hourly charge for the IGW itself.

Remember: IGW = Internet connectivity for the VPC.

## NAT Gateway

Allows private subnet resources to initiate outbound internet connections.

Prevents unsolicited inbound internet connections to private resources.

AWS-managed service.

Public NAT Gateway is placed in a public subnet and uses an Elastic IP.

For high availability, AWS recommends one NAT Gateway per AZ.

Traffic flow:

```
Private EC2
↓
NAT Gateway
↓
```


```
Internet Gateway
↓
Internet
```

Remember: NAT = Private resources need outbound internet.

## NAT Gateway vs NAT Instance

## NAT Gateway

AWS-managed

Automatically scales within service limits You manage scaling

Less maintenance

Recommended for most new deployments More control, but more administration

## NAT Instance

Self-managed EC2

Requires maintenance

## Public IP vs Private IP vs Elastic IP

Type

Public IP Internet-routable IPv4 address

Private IP Used for internal network communication

Elastic IP Static public IPv4 address allocated to your AWS account

An automatically assigned public IPv4 address can change after an EC2 stop/start. An Elastic IP remains allocated to your account until released.

Meaning

## 3. Network Security

## Security Group vs NACL

This is one of the most important AWS exam comparisons.

| Feature | Security Group | NACL |
| --- | --- | --- |
| Level | ENI/resource Subnet |   |
| Stateful? | Yes No |   |
| ALLOW rules Yes | Yes |   |
| DENY rules No | Yes |   |
|   | Return traffic Automatically allowed | Must be explicitly allowed |
| Rule | All applicable rules | Lowest rule number first; first match wins |
| evaluation | combined |   |
| SG references Yes | No |   |

## Security Group


Acts as a firewall for EC2 and other supported resources.

Allows traffic through explicitly defined ALLOW rules.

Anything not allowed is implicitly denied.

Automatically allows response traffic for permitted connections.

NACL

Acts as a firewall at the subnet level.

Supports ALLOW and DENY.

Stateless: inbound and outbound rules must be configured separately.

Rules are evaluated in numerical order.

Memory trick:

Security Group = Stateful.

NACL = Stateless.


```
Security Group = Resource-level.
NACL = Subnet-level.
```

## Important ports

## Port Protocol / Service

22 SSH / SFTP 80 HTTP 443 HTTPS 3389 RDP 3306 MySQL

5432 PostgreSQL

## 4. Connecting VPCs Together

## VPC Peering

Creates a private connection between two VPCs.

Supports same-account or cross-account connections.

Can connect VPCs in the same or different Regions.

Requires non-overlapping CIDR ranges.


Not transitive.

Requires route table entries on both sides.

Example:

VPC A ←──── Peering ────→ VPC B

IfA connects to B and B connects to C, A cannot automatically communicate with C through B.

Remember: Peering = Direct connection between two VPCs.

## Transit Gateway

Connects many VPCs through a central hub.

Uses hub-and-spoke architecture.

Supports transitive routing.

Can connect VPCs and on-premises networks through VPN and Direct Connect Gateway.

Designed for large network architectures.

A default quota of up to 5,000 VPC attachments is commonly cited; quotas can vary.


```
VPC A
|
VPC B — Transit Gateway — VPC C
|
VPC D
```

Remember: Transit Gateway = Many networks connected through one central hub.

## VPC Peering vs Transit Gateway

| Feature | Peering | Transit Gateway |
| --- | --- | --- |
| Purpose | Connect two VPCs | Connect many networks |
| Architecture Direct |   | Hub-and-spoke |
| Transitive No |   | Yes |
|   | Scalability Less convenient at scale | Designed for scale |
| Cost |   | No hourly peering connection charge Attachment-hour + data processing |

## 5. Private Access to AWS Services

## VPC Endpoints

Allow resources inside a VPC to access supported AWS services privately without requiring an internet connection for that traffic.

## Example:

A private EC2 instance needs to upload files to Amazon S3.

Without an endpoint, it might need NAT Gateway access.

With an S3 Gateway Endpoint:

```
Private EC2
↓
Gateway Endpoint
↓
Amazon S3
```

## Types of VPC Endpoints

| Feature | Gateway Endpoint | Interface Endpoint |
| --- | --- | --- |
| Services | S3 and DynamoDB Many other supported AWS services |   |
| Mechanism | Route table entry PrivateLink + ENI |   |
| Security Group | No Yes |   |


Remember:

Gateway Endpoint = S3 and DynamoDB.

Interface Endpoint = Most other supported services.

## AWS PrivateLink

Allows a service provider to expose a specific service privately to other VPCs/accounts.

Does not require VPC Peering.

Does not expose the provider's entire VPC.

Supports overlapping CIDR ranges.

## Typical architecture:

```
CONSUMER VPC
|
Interface Endpoint (ENI)
|
AWS PrivateLink
|
Provider NLB
|
Provider Application
```

Remember: PrivateLink = Private access to a specific service.


## 6. Network Monitoring

## VPC Flow Logs

Records metadata about IP traffic flowing to and from network interfaces.

It answers:

Who communicated with whom?

Which ports were used?

Which protocol was used?

Was the traffic accepted or rejected?

## Example:

Source IP:

Destination IP: 10.0.2.15

Port:

Protocol:

Action:

10.0.1.10

443

TCP

ACCEPT

## Flow Log scopes

## Level What it monitors

VPC Network interfaces throughout the VPC

Subnet Network interfaces in a specific subnet

ENI One specific network interface

## Flow Log destinations

## Destination

Amazon S3

Long-term storage

## Purpose


CloudWatch Logs Monitoring and alerting Kinesis Data Firehose Streaming logs to other destinations

Amazon Athena can query Flow Logs stored in S3 using SQL.

## What Flow Logs do NOT capture

Actual packet contents.

Message contents.

DNS query details.

Certain traffic involving AWS-reserved VPC addresses.

For DNS query monitoring, use Route 53 Resolver query logging.

Flow Logs are not real-time; records can take several minutes to appear.

## CloudWatch vs CloudTrail vs Flow Logs

## Service What does it monitor?

VPC Flow Logs Network traffic metadata

CloudWatch Metrics, logs, alarms

CloudTrail

Example:

Network connection rejected → Flow Logs.

AWS account activity and API calls

EC2 CPU reaches 95%→ CloudWatch.


Someone deletes an EC2 instance → CloudTrail.

## 7. Connecting Your Office to AWS

## Site-to-Site VPN

Connects an entire on-premises network to AWS through an encrypted IPsec VPN over the public internet.

Important components:

## Component

Customer Gateway (CGW) Represents the customer-side VPN device

Virtual Private Gateway

(VGW)

Transit Gateway

Important facts:

Two VPN tunnels are provisioned for redundancy.

## Meaning

AWS-side VPN endpoint attached to one VPC

Can serve as the AWS-side hub for VPN connections to multiple networks

Relatively quick to establish after configuration.

Can be used as a backup for Direct Connect.

```
Company Office
|
Customer Gateway
|
Encrypted VPN
|
Virtual Private Gateway
|
AWS VPC
```


## AWS Client VPN

Managed remote-access VPN.

OpenVPN-based.

Connects an individual user's laptop/device to AWS and authorized on- premises networks.

Uses TLS encryption over the public internet.

Allows access to private resources using private IP addresses, subject to routes and security rules.

Remember: Client VPN = One employee's device →AWS.

## AWS Direct Connect

Dedicated physical network connection between your premises and AWS.

Bypasses the public internet.

Offers more consistent network performance.


Takes weeks or months to provision in many cases.

Not encrypted by default.

Can be combined with Site-to-Site VPN for encrypted backup connectivity.
