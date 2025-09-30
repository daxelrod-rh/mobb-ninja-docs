---
date: '2025-09-30'
title: Establishing VPN connectivity to ROSA Virtualization VMs
tags: ["AWS", "ROSA", "EC2", "OpenShift", "Virtualization"]
authors:
  - Diana Sari
  - Daniel Axelrod
---

Intro

## Prerequisites

* 


## Set Environment Variables

Set up your environment with the required variables:

```bash
export CLUSTER_NAME="your-rosa-cluster-name"
export AWS_REGION="eu-west-1"  # Replace with your AWS region
```

## Set up the AWS-side VPN

1. Create Customer Gateway. This represents the UDN-side of the VPN connection. The IP address should match the public IP of the NAT gateway in the subnet that contains the metal node that will host your ipsec VM.

   ```bash
   #TODO capture ID
   aws ec2 create-customer-gateway \
      --type ipsec.1 \
      --ip-address $NAT_GATEWAY_PUBLIC_IP \
      --tag-specifications "ResourceType=customer-gateway,Tags=[{Key=Name,Value=$NAME-gcw}]"
   ```

1. Create a VPN Gateway and attach it to your VPC

   ```bash
   $VPN_GATEWAY_ID=$(aws ec2 create-vpn-gateway \
      --type ipsec.1
      --tag-specifications "ResourceType=vpn-gateway,Tags=[{Key=Name,Value=$NAME-vgw}]" \
      | jq -r '.VpnGateway.VpnGatewayId')

   aws ec2 attach-vpn-gateway \
      --vpn-gateway-id $VPN_GATEWAY_ID \
      --vpc-id $VPC_ID
   ```

1. Create a Site-to-Site VPN connection (TODO do we need local or remote network cidr?)

   ```bash
   $VPN_CONNECTION_ID=$(aws ec2 create-vpn-connection \
      --customer-gateway-id $CUSTOMER_GATEWAY_ID \
      --type ipsec.1 \
      --vpn-gateway-id $VPN_GATEWAY_ID \
      --pre-shared-key-storage Standard
      --options '{"StaticRoutesOnly":true}' \
      --tag-specifications "ResourceType=vpn-connection,Tags=[{Key=Name,Value=$NAME-vpn}]")
   
   aws ec2 create-vpn-connection-route \
      --vpn-connection-id $VPN_CONNECTION_ID \
      --destination-cidr-block $CUDN_CIDR
   ```

1. Propagate CUDN route to subnets TODO all subnets

   ```bash
   aws ec2 enable-vgw-route-propagation \
      --gateway-id $VPN_GATEWAY_ID \
      --route-table-id $ROUTE_TABLE_ID
   ```

1. TODO get relevant attributes from VPN

## Set node security groups

Security groups on nodes: what's actually needed here?

## Create a Cluster User Defined Network

## Create and configure an IPSec VM
       





## Conclusion
