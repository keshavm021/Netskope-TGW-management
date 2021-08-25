# Netskope-TGW-management - Failover automation solution for AWS TGW - Netskope IPsec tunnels

When you integrate your AWS Transit Gateway with Netskope Security Cloud, you create four IPsec VPN tunnels between two AWS Avialability Zones automatically choosen by AWS and two Netskope PoPs of your choice. 
AWS egress traffic routing from the VPCs attached to this TGW, to Netskope controlled by the static route in the TGW Route Table associated with the TGW VPC attachment. For the specific TGW Route Table only single static route pointing to the specific TGW VPN attachment can be present at any point of time. While AWS AZ failure addressed automatically in this configuration, the static route in the TGW route table has to be updated in case of failure of both VPN tunnels to the specific Netskope PoP. 
This solution deploys 
