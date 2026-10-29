# EWB-AWS-VPC-Peering
AWS VPC Peering lab demonstrating private connectivity, route tables, security groups, and ICMP troubleshooting.


# AWS VPC Peering — EWB Cloud Labs

A hands-on AWS networking project completed as part of the EWB Cloud Labs Student Project Workbook.

## Project Overview

This project demonstrates private network connectivity between two Amazon Virtual Private Clouds (VPCs) using VPC Peering, custom route tables, and security group rules.

## Architecture

- **AWS Region:** US East (N. Virginia) — `us-east-1`
- **VPC A:** `10.10.0.0/16`
- **VPC B:** `10.20.0.0/16`
- **VPC A public subnet:** `10.10.1.0/24`
- **VPC A private subnet:** `10.10.2.0/24`
- **VPC B private subnet:** `10.20.1.0/24`

### EC2 Instances

| Instance | Private IP | Role |
|---|---|---|
| `ewb-bastion` | `10.10.1.10` | Bastion host for SSH access |
| `ewb-app-a` | `10.10.2.10` | Private instance in VPC A |
| `ewb-app-b` | `10.20.1.10` | Private instance in VPC B |

## Implementation

- Created two VPCs with non-overlapping CIDR ranges.
- Configured public and private subnets.
- Created and activated a VPC Peering connection.
- Configured route tables for traffic between the VPCs.
- Applied security group rules for SSH and ICMP traffic.
- Accessed private instances through the bastion host.
- Tested connectivity in both directions using private IP addresses.
- Demonstrated a controlled ICMP failure by temporarily removing an inbound security group rule, then restoring the rule and verifying connectivity recovery.

## Connectivity Test Results

| Test | Result |
|---|---|
| SSH access through bastion host | Successful |
| App-A → App-B ICMP ping | 4/4 packets received |
| App-B → App-A ICMP ping | 4/4 packets received |
| Controlled ICMP failure | 100% packet loss |
| Connectivity after rule restoration | 4/4 packets received |

**Testing note:** The initial pre-peering baseline failure was not captured. The documented failure test was performed later by temporarily removing the ICMP ingress rule on App-B.

## Security and Cleanup

Security group rules were configured to restrict SSH and ICMP ingress to specified sources. No private keys or credentials are included in this repository.

After testing, the lab's EC2 instances were terminated and the peering connection, security groups, subnets, custom route tables, and internet gateway were removed. Final deletion of both VPCs should be confirmed separately.

## Key Learnings

- VPC CIDR planning and subnet configuration
- VPC Peering and route table configuration
- Security groups and stateful traffic filtering
- Bastion-host-based SSH access
- Private IP connectivity testing
- Troubleshooting connectivity through controlled failure and recovery
- AWS resource cleanup and cost awareness

## Evidence

Screenshots documenting the AWS configuration and connectivity tests will be added to this repository.


