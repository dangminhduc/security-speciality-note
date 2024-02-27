# security-speciality-note

## KMS
- Customer Managed Key(CMK) gets rotated automatically every 365 days
- To encrypt data with KMS
  - For data with size under 4KB, it can be encrypt directly
  - For data with size above 4KB, envelop encryption must be use. And for envelope encryption, the plain text data is used.
  https://aws.amazon.com/blogs/security/how-to-encrypt-and-decrypt-your-data-with-the-aws-encryption-cli/

## Network
- Intrustion Dectection System: should be using a custom solution on the marketplaces

## IAM
- IAM Usage report can be download via IAM credential report.

## Config
- Basicly using AWS Config to track resources's changes.
- 

## VPC
- Network ACL is stateless, which means it need both inbound and outbound rules for traffic
- Security Groups is stateful, which mean it only need the rule to initiate the request, no need to have a rule for the returning traffic.

## Other
- IAM role is needed to access Dynamo DB tables
- Root Account in Org should contain a default SCP that allow everything and applies to all OUs.
