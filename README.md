# Advanced AWS Networking — Transit Gateway & VPC Peering

**Summary:** Designed, built, and validated two multi-VPC connectivity architectures —
a three-VPC Transit Gateway hub-and-spoke topology and a cross-region VPC peering
connection — including route table configuration, security group hardening, and end-to-end
connectivity testing via SSH and ICMP.

## Skills demonstrated

- Multi-VPC network architecture (hub-and-spoke vs. peered topologies)
- AWS Transit Gateway configuration: attachments, route table associations/propagations
- Cross-region VPC Peering setup and route table management on both sides of a connection
- Security group rule design (least-privilege SSH/ICMP access between environments)
- Network troubleshooting: jump-host (bastion-style) access and live connectivity validation
- Environment teardown/cleanup discipline once testing is complete

## Part 1 — Transit Gateway (`Transit-Gateway-Projets/`)

Connects three VPCs (VPC-A, VPC-B, VPC-C) through a single AWS Transit Gateway (`TGW-USEAST-2`)
instead of a full peering mesh.

**What was built:**
- A Transit Gateway with attachments for VPC-A, VPC-B, and VPC-C, each with its own subnet
- Route table associations and propagations on the TGW so traffic can reach every attached VPC
- Custom route table entries in each VPC pointing cross-VPC traffic at the Transit Gateway
- Security group rules opening SSH (for management) and ICMP (for connectivity testing)
  between the private servers in each VPC
- End-to-end validation: SSH into a public instance in VPC-A, hop to its private instance,
  and `ping` private instances in VPC-B and VPC-C across the Transit Gateway

**Key screenshots (in order):**
1. `VPC-A_B_C.png`, `VPC-A_B_C-Subnetspng.png`, `VPC-A_B_C-INSTANCES.png` — the three VPCs, subnets, and EC2 instances before any TGW config
2. `TGW-USEAST-2.png` — Transit Gateway created
3. `transit gateway attachment-vpc-A.png` / `-vpc-B.png` / `-vpc-C.png` / `-vpc-A-B-C.png` — attaching each VPC
4. `TGW-USEAST-2-associations.png`, `TGW-USEAST-2-propagations.png`, `TGW-USEAST-2-routes-tables.png` — TGW route table configuration
5. `VPC-A-route-table-before-...` / `-after-...` (and the equivalent for VPC-B and VPC-C) — adding routes that point at the TGW
6. `no-ICMP-RULE-in-sec-group-for-vpc-c-private-server-before.png` → `ICMP-RULE-in-sec-group-for-vpc-c-private-server-setup-...png` — opening ICMP so connectivity tests can succeed
7. `ssh-to-VPC-A-ec2-instance.png`, `allow-ssh-from-vpcA-pub-server-to-vpcA-private-server.png`, `troubleshooting-ssh-from-public-server-to-private-server-vpc-A.png` — jump-host access into the private subnet
8. `ping-from-vpcA-private-sever-to-vpc-B-sever-success.png`, `ping-from-vpcA-private-sever-to-vpc-C-sever-success.png` — successful cross-VPC connectivity via the Transit Gateway

## Part 2 — VPC Peering (`vpc-peering/`)

A direct peering connection between two VPCs in **different regions** (`vpc-ohio` /
us-east-2 and `vpc-n-virginia` / us-east-1, referred to as VPC-1 and VPC-B).

**What was built:**
- A cross-region VPC peering connection, requested from one VPC and accepted from the other
- Route table updates on both sides (public and private subnets) so traffic destined for the
  peer VPC's CIDR is routed over the peering connection
- A security group rule allowing ICMP from the peer VPC's CIDR
- Connectivity tests between public/private EC2 instances across the peering connection
- Teardown: the peering connection deleted once testing was complete

**Key screenshots (in order):**
1. `vpc-ohio.png`, `vpc-B-us-east-1.png`, `vpc-ohio-vpc-n-virginia.png` — the two source VPCs
2. `peering-connection-configuration.png` → `peering-connection-configuration-request.png` /
   `-request-1.png` → `-request-accept.png` → `-complete.png` — requesting and accepting the
   peering connection
3. `route-vpc1-pub-to-route-vpc-priv-1a.png`, `route-vpc1-pub-to-route-vpc-priv-1b.png`,
   `route-vpcb-pub-to-route-vpc1-priv-1a.png`, `route-vpcb-priv-to-route-vpc1-priv-1a.png` /
   `-completed.png` — route table updates on both VPCs
4. `vpc-B-Sec-Group-allow-ICMP.png` — opening ICMP for the connectivity test
5. `ec2-instances-vpcA-vpcB.png`, `ec2-private-instance-vpc-B.png`, `ec2-pub-instance-vpc-1-private-IP.png` — the test instances
6. `ping-from-ec2ins-pubsubnet-to-ec2insprivate-vpcB-privsubnet.png` — successful ping across the peering connection
7. `deleted-peering-connection.png` — cleanup

## AWS services used

VPC, Transit Gateway, EC2, Security Groups, Route Tables, cross-region VPC Peering.

## Notes

This project is screenshot-documented (console-driven, no Terraform/CLI in this folder).
Screenshots live alongside this README under `Transit-Gateway-Projets/` and `vpc-peering/`.
