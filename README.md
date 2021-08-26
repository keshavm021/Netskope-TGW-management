# Netskope-TGW-management - Failover automation solution for AWS TGW - Netskope IPsec tunnels

When you integrate your AWS Transit Gateway with Netskope Security Cloud, you create four IPsec VPN tunnels between two AWS Availability Zones automatically chosen by AWS and two Netskope PoPs of your choice. 
AWS egress traffic routing from the VPCs attached to this TGW, to Netskope controlled by the static route in the TGW Route Table associated with the TGW VPC attachment. For the specific TGW Route Table only single static route pointing to the specific TGW VPN attachment can be present at any point of time. While AWS AZ failure addressed automatically in this configuration, the static route in the TGW route table has to be updated in case of failure of both VPN tunnels to the specific Netskope PoP. 
This solution deploys Lambda function that's been triggered by the CloudWatch rule for IPsec tunnel status change. The function checks if the both tunnel to the Netskope PoP are down, and if so, it scans all the TGW route tables for the static route poitning to the TGW VPN attachments for the corresponding Site-to-Site VPN connection and replaces them with the alternative static route to the failover PoP. 
You must always deploy this solution in us-west-2 region and enable AWS Transite Gateway Network Manager. AWS Transite Gateway Network Manager is the global service that monitors the status of IPsec Site-to-Site VPN connections and uses Amazon CloudWatch in the us-west-2 region for alerting. Deployed in the us-west-2 region the solution will monitor your TGW in any region on the same account. Cross-account monitoring is not currently supported.
You need to deploy one instance of the solution per TGW or customise the lambda function.
You may enable of disable fallback functionality. If enabled, Lambda function will revert static routes to the primary Netskope PoP if both of the IPsec tunnels are up.
In addition to checking and updating routes when IPsec tunnel status changes, the same Lambda funciton beign triggred every 10 minutes and checks that there are no routes left pointing to the IPsec connection which is currently down. This is to prevent the unlikely situation when IPsec connections were bouncing, and the this caused race condition between Lalmbda funciton executions where the last execution has been timed out. Note that only one Lambda route execution can run at any point of time to avoid inconsitent results. Concurrency has been controlled using DynamoDB table also created by this solution.

Usage: 
1. Clone this repository to your machine.
git clone https://github.com/dyuriaws/Netskope-TGW-management.git

If you're not going to modify the Lambda function, you can deploy the solution using TGW_IPsec_management.yaml CloudFormation template. 

2. Change the region to US West (Oregon)us-west-2.
3. In the AWS CloudFormation management console click Create Stack and choose With new resources (standard).
4. Choose Upload a template file and click on Choose file.
5. Choose the TGW_IPsec_management.yaml from the disk and click Next.
6. Enter the stack name and the parameters for your deployment:

TGWRegion
The AWS region where your TGW is deployed

TGWName
TGW name that will be used for access control. Your all TGW attachments must have an attribute "Key"="TGWName", "Value"="This parameter". For example, "MyProdTGW-us-east-1"

TGWID
TGW ID. For example, tgw-01234567890123456

TGWAttachmentID1
TGW attachment ID for the first (primary) VPN. For example, tgw-attach-01234567890123456

TGWAttachmentID2
TGW attachment ID for the second (failover) VPN. For example, tgw-attach-01234567890123456

Fallback
Yes/No for the route fallback support to the TGWAttachmentID1 if both of this IPsec tunnels became active.

8. Click Next.
9. Optionally, enter the Tags for your CloudFormation stack and click Next.
10. Acknowledge creating IAM resources and click Create stack.

