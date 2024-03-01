# security-speciality-note

## KMS
- Customer Managed Key(CMK) gets rotated automatically every 365 days
- To encrypt data with KMS
  - For data with size under 4KB, it can be encrypt directly
  - For data with size above 4KB, envelop encryption must be use. And for envelope encryption, the plain text data is used.
  https://aws.amazon.com/blogs/security/how-to-encrypt-and-decrypt-your-data-with-the-aws-encryption-cli/
- Auto rotation of KMS key is fixed at 365 days and can not be changed

## Network
- Intrustion Dectection System: should be using a custom solution on the marketplaces

## IAM
- IAM Usage report can be download via IAM credential report.
- Using IAM Access Advisor or Access Management Access Analyzer to see what services have been used to not used for a period of time

## Config
- Basicly using AWS Config to track resources's changes.
- Custom config rule can be trigger via resource's change events or periodically at maximum interval of 1 hour

## Cognito
- Identity Pools can be use to give unauthenticated user to access AWS resources
- Captcha can be enable via Auth Challenge Lambda Trigger

## VPC
- Network ACL is stateless, which means it need both inbound and outbound rules for traffic
- Security Groups is stateful, which mean it only need the rule to initiate the request, no need to have a rule for the returning traffic.

## Other
- IAM role is needed to access Dynamo DB tables
- Root Account in Org should contain a default SCP that allow everything and applies to all OUs.
- To Authenticate with 3rd party IdP, we need 3 things:
  - App client ID
  - App client Secret
  - List of Scopes
- Config AD with AWS
  - Config AWS as the relying party in AD Federation services -> AWS STS
  - Config custom claim rules to issue and transform claims between claims provider(AD) and relying parties(STS)
  - Create custom rule uses regular expressions to transform each of the group memberships of the form AWS-<Account Number>-<Role Name> into in the IAM role ARN, IAM federation provider ARN form AWS expects.
- If you are developing an application using the Kinesis Client Library (KCL), your policy must include permissions for Amazon DynamoDB and Amazon CloudWatch; the KCL uses DynamoDB to track state information for the application, and CloudWatch to send KCL metrics to CloudWatch on your behalf.
