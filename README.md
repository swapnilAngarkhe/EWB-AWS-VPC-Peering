# AWS VPC Peering — EWB Cloud Labs

A hands-on AWS networking project focused on establishing private connectivity between two Amazon Virtual Private Clouds (VPCs), configuring routing and security rules, troubleshooting connectivity, and safely cleaning up cloud resources.

**Project:** EWB Cloud Labs — AWS VPC Peering  
**AWS Region:** US East (N. Virginia) — `us-east-1`  
**Status:** Lab completed; resources cleaned up after testing.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Architecture and Network Design](#architecture-and-network-design)
- [AWS Resources](#aws-resources)
- [Implementation](#implementation)
- [Testing and Troubleshooting](#testing-and-troubleshooting)
- [Challenges Encountered](#challenges-encountered)
- [Cleanup](#cleanup)
- [What I Learned](#what-i-learned)
- [Repository Structure](#repository-structure)
- [Reference](#reference)

## Project Overview

The objective of this project was to connect two VPCs using AWS VPC Peering and demonstrate communication between private EC2 instances using their private IPv4 addresses.

The lab involved creating two VPCs with non-overlapping CIDR blocks, configuring public and private subnets, setting up route tables, establishing a peering connection, and implementing security group rules for SSH and ICMP traffic.

A bastion host in VPC A provided an administrative entry point for accessing both private instances. Connectivity was validated in both directions, and a controlled security group change was used to demonstrate how network filtering affects communication.

## Architecture and Network Design

The project used two VPCs in the same AWS account and region.

| Component | Configuration |
|---|---|
| VPC A | `10.10.0.0/16` |
| VPC B | `10.20.0.0/16` |
| Public subnet in VPC A | `10.10.1.0/24` |
| Private subnet in VPC A | `10.10.2.0/24` |
| Private subnet in VPC B | `10.20.1.0/24` |
| Internet gateway | `ewb-igw-a` |
| Peering connection | `ewb-a-b-peer` |
| Peering connection ID | `pcx-0f39d1744e019b79d` |

The two VPC CIDR ranges do not overlap, allowing them to be connected through VPC Peering.

### EC2 instances

| Instance | Private IPv4 | Role |
|---|---|---|
| `ewb-bastion` | `10.10.1.10` | Public entry point for SSH administration |
| `ewb-app-a` | `10.10.2.10` | Private workload in VPC A |
| `ewb-app-b` | `10.20.1.10` | Private workload in VPC B |

Both application instances were configured without public IPv4 addresses.

### Routing design

Three custom route tables were used:

- **`ewb-rt-a-public`** — associated with the public subnet in VPC A. It contained the local route, a default route to the internet gateway, and a route to VPC B through the peering connection.
- **`ewb-rt-a-private`** — associated with the private subnet in VPC A. It contained the local route and a route to VPC B through the peering connection.
- **`ewb-rt-b-private`** — associated with the private subnet in VPC B. It contained the local route and a return route to VPC A through the peering connection.

Both directions needed appropriate routes. Establishing a peering connection alone does not automatically enable traffic between the VPCs.

See [`architecture/`](architecture/) for the project architecture diagram.

## AWS Resources

The lab included:

- Two VPCs and three subnets.
- One internet gateway attached to VPC A.
- One VPC Peering connection.
- Three custom route tables.
- Three EC2 instances: one bastion host and two private application instances.
- Three security groups for the bastion and application instances.
- An EC2 key pair used for SSH access.

The configuration and resource identifiers were recorded during the lab. The AWS resources were removed after testing to limit ongoing costs.

## Implementation

### 1. VPC and subnet configuration

Created two VPCs using non-overlapping CIDR ranges and configured one public subnet and one private subnet in VPC A, along with one private subnet in VPC B.

### 2. Internet gateway and route tables

Attached an internet gateway to VPC A and configured the public route table to direct internet-bound traffic through it. The private route tables were configured to direct traffic for the remote VPC through the peering connection.

### 3. EC2 deployment

Launched the bastion host in the public subnet and the two application instances in their respective private subnets. The private application instances did not have public IPv4 addresses.

### 4. Security group configuration

Configured inbound rules to restrict SSH access and permit the required ICMP traffic between the private instances.

SSH access to the bastion was restricted to the configured client IP. SSH rules for the application instances allowed the specified bastion source, while ICMP rules permitted the required traffic between the application hosts.

### 5. VPC Peering

Created and accepted the peering request between VPC A and VPC B. The connection reached the **Active** state. Routes were then configured on both sides to enable communication over the private network.

### 6. SSH access through the bastion

Configured local SSH host aliases and `ProxyJump` to connect through the bastion to the private instances. This allowed administrative access without assigning public IPv4 addresses to the application hosts.

## Testing and Troubleshooting

The project included forward and reverse ICMP tests, SSH access, and a controlled failure-and-recovery test.

### Connectivity results

| Test | Observed result |
|---|---|
| SSH to bastion | Successful |
| SSH to App-A through bastion | Successful |
| SSH to App-B through bastion | Successful during the lab |
| App-A → App-B ping | 4 packets received; 0% packet loss |
| App-B → App-A ping | 4 packets received; 0% packet loss |
| Controlled ICMP failure | 100% packet loss |
| Ping after restoring ICMP rule | 4 packets received; 0% packet loss |

### Forward path: App-A to App-B

From App-A (`10.10.2.10`), the command `ping -c 4 10.20.1.10` returned four replies with no packet loss.

The traffic used App-A's subnet route table, the VPC Peering connection, and the destination-side security group rules.

### Return path: App-B to App-A

From App-B (`10.20.1.10`), the command `ping -c 4 10.10.2.10` also returned four replies with no packet loss.

The return route in VPC B was necessary for traffic to reach VPC A. Security group rules also needed to permit the required ICMP traffic.

### Controlled failure and recovery

To test troubleshooting, the inbound ICMP Echo Request rule on `ewb-sg-app-b`, allowing traffic from `10.10.2.10/32`, was temporarily removed.

The forward ping from App-A then showed 100% packet loss. After restoring the same rule, the test succeeded again with four replies and no packet loss.

This demonstrated that a peering connection can remain active while security group rules prevent the intended traffic.

**Testing limitation:** The original baseline failure test before peering and route configuration was not captured. The controlled security group failure was performed later and is documented separately rather than presented as the original baseline.

## Challenges Encountered

### Understanding the difference between peering and routing

Initially, it was important to distinguish between an active peering connection and actual network reachability. Peering establishes the link, but the route tables on both sides must direct traffic through it.

**Resolution:** Configured the required routes in the public and private route tables, including the return route in VPC B.

### Configuring private SSH access

The application instances had no public IPv4 addresses, so they could not be accessed directly from the laptop over the internet.

**Resolution:** Used the bastion host and SSH `ProxyJump` configuration to reach the private instances.

### Troubleshooting ICMP connectivity

A successful route configuration does not guarantee successful ping tests because security groups also filter traffic.

**Resolution:** Temporarily removed the App-B inbound ICMP rule, observed the failed ping, restored the rule, and confirmed recovery.

### Managing SSH configuration on Windows

Accessing multiple instances through one bastion required a reusable SSH configuration instead of repeatedly entering connection details.

**Resolution:** Created SSH host aliases for the bastion and private instances, making connections easier to repeat and troubleshoot.

### Managing AWS costs

The lab used EC2 instances, storage, and public IPv4 resources that could incur charges while provisioned.

**Resolution:** Terminated the lab instances and removed the associated networking resources after testing. Cleanup evidence is stored in [`evidence/cleanup/`](evidence/cleanup/).

## Cleanup

After collecting the available project evidence, the lab resources were removed to reduce the risk of ongoing AWS charges.

The cleanup process included terminating the three EC2 instances, checking for leftover EBS volumes and Elastic IP allocations, deleting the peering connection, removing the lab security groups and subnets, deleting the custom route tables, and detaching and deleting the internet gateway.

Cleanup screenshots are available in [`evidence/cleanup/`](evidence/cleanup/).

AWS billing and usage reports may take time to reflect recent changes. The lab's cleanup was intended to remove the resources created for this exercise.

## Repository Structure

```text
EWB-AWS-VPC-Peering/
├── architecture/
│   ├── README.md
│   └── vpc-peering-architecture.png
├── evidence/
│   ├── [project screenshots]
│   └── cleanup/
│       ├── [EC2 termination evidence]
│       ├── [empty EBS volumes evidence]
│       ├── [Elastic IP verification]
│       └── [VPC cleanup evidence]
├── report/
│   └── [assessment report PDF]
└── README.md
```

Actual filenames may differ from the examples above.

## Reference

- [Amazon VPC Peering documentation](https://docs.aws.amazon.com/vpc/latest/peering/what-is-vpc-peering.html)
- [Amazon VPC route tables](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html)
- [Amazon EC2 security groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)

---

