# security-speciality-note

## KMS
- Customer Managed Key(CMK) gets rotated automatically every 365 days
- To encrypt data with KMS (using AWS CLI)
  - For data with size under 4KB, it can be encrypt directly
  - For data with size above 4KB, envelop encryption must be use. And for envelope encryption, the plain text data is used.
  https://aws.amazon.com/blogs/security/how-to-encrypt-and-decrypt-your-data-with-the-aws-encryption-cli/
- To encrypt data bigger than 4KB(for data below 4KB, you can use Encrypt API directly) (using API)
  - Use GenerateDataKeyWithoutPlaintext API to create data key
  - Use Decrypt API to decrypt the data key, then use the plaintext key to encrypt the data.
- Auto rotation of KMS key is fixed at 365 days and can not be changed
- To change the expiration of KMS key that use extenal material, you need to reimport the same key material and specify a new expiration date.
- Step To import key material
  - Create a KMS key with no key material(Origin set to EXTERNAL)
  - Download the wrapping public key and import token(valid only for 24 hours)
  - Encrypt the key material using the download public key and the wrapping algorithm that you specified.
  - Import the key material
    - Specify an expiration time if needed
- Alias must be unique in  the AWS account and region
  - To simplify the code that runs in multiple regions, you can use the same alias name but point to a diffent CMK in each region.
- Data key caching stores data keys and related cryptographic material in a cache. When you encrypt or decrypt data, the AWS Encryption SDK looks for a matching data key in the cache. If it finds a match, it uses the cached data key rather than generating a new one. Data key caching can improve performance, reduce cost, and help you stay within service limits as your application scales. 
- KMS has SignAPI that create a digital signature for a message a using the private key in an asynmetric CMK
- Redshift using KMS: The root key(KMS) encrypts the cluster key. The cluster key encrypts the database key. The database key encrypts the data encryption keys. Data encryption keys encrypt data blocks in the cluster.

## CloudTrail 
- 2 type of events: Data events and management events
  - Data Evenst: show the resource operations performed on or within a resource in your AWS account. These operations are often high-volume activities.
    - S3 object-level API activity
    - Lambda function invocation activity
    - DynamoDB object-level API activity on tables(PutItem, GetItem,...)
- Management Events: show management operations that are performed on resources in your AWS account.
  - Creating an S3 bucket
  - Creating and managing IAM resources
  - Configuring routing tables
  - Setting up logging
- To enable CloudTrail for all accounts under Organization, you only need to apply the trail to the Organization instead of all child accounts.
- Global services like CloudFront, IAM, STS create their CloudTrail logs in US East Region.

## Network
- Intrustion Dectection System: should be using a custom solution on the marketplaces

## IAM
- IAM Usage report can be download via IAM credential report.
- Using IAM Access Advisor or Access Management Access Analyzer to see what services have been used to not used for a period of time
- ExternalId is a unique identifier that might be required when you assume a role in another account. If the administrator of the account to which the role belongs provided you with an external ID, then provide that value in the ExternalId parameter. This value can be any string, such as a passphrase or account number. A cross-account role is usually set up to trust everyone in an account. Therefore, the administrator of the trusting account might send an external ID to the administrator of the trusted account. That way, only someone with the ID can assume the role, rather than everyone in the account.
- SSL Certificate can be imported via IAM in the region that doesn't support ACM
- The AssumeRole API response contains temporary credentials to access AWS resources: Session Token, Secret Key, Access Key
- When calling the AssumeRoleWithWebIdentity API, the session duration in the parameter can not be longer than the maximum value set in IAM Role

## Config
- Basicly using AWS Config to track resources's changes.
  - Rule is for complaint.
- Custom config rule can be trigger via resource's change events or periodically at maximum interval of 1 hour
- When a Config rule become non-complaint, a runbook from AWS System Manager Automation can be executed as a remediation action(ex: Publish SNS notification, create an Jira issue)
- Security Hub leverages AWS Config(wit resource recording is turned on) to retrieve configuration data for AWS resources. It gains access to detailed information about resource configurations, allowing it to perform comprehensive security checks and assetment

## Cognito
- Identity Pools can be use to give unauthenticated user to access AWS resources
- Captcha can be enable via Auth Challenge Lambda Trigger

## VPC
- Network ACL is stateless, which means it need both inbound and outbound rules for traffic
- Security Groups is stateful, which mean it only need the rule to initiate the request, no need to have a rule for the returning traffic.
- Security Group's changes is 
## S3
- It is possible to have different encyption keys for different verions of the same object in S3
- S3 Replication(both same and cross region) require bucket versioning.
- To encrypt data before sending it to S3 bucket, use client-side encryption
  - The object stored in S3 aren't exposed to any 3rd party, including AWS(S3 only dectect it as typical objects)
- S3 Object Lock
  - Compliance mode: object can not be overwritten or delete by any user, including root user in AWS account.
  - Governance mode: user can not overwrite or delete an object version or alter its lock settings unless they have special permissions. To override or remove governance-mode retention settings, you muse have the `s3:BypassGovernanceRetention` permission and must explictly include `x-amz-bypass-governance-retention:true` as a request header with any request.
  - You can crate a legal hold on an object version. Like a retention period, a legal hold prevents an object version from being overwritten or deleted. However, a legal hold doesn't have an associated fixed amount of time and remains in effect until removed. Legal holds can be freely placed and removed by any user who has the s3:PutObjectLegalHold permission. Legal holds are independent from retention periods. Placing a legal hold on an object version doesn't affect the retention mode or retention period for that object version. 
- You can use Macie, to create and run sensitive data discovery jobs to automate discovery, logging and reporting of sensitive data in S3 buckets
  - Amazon Macie is a data security service that discovers sensitive data by using machine learning and pattern matching, provides visibility into data security risks, and enables automated protection against those risks

## Firewall Manager
- Firewall Manager provides following types of policies
  - WAF Policy
  - Shield Advance policy
  - Security Group policy
  - Network Firewall policy
  - Route53 Resolver DNS Firewall policy
  - 3rd Party firewall policy
- Firewall Manager can be use to config WAF rules within the Organization.
  - Apply a Firewall Manager WAF Policy to create a Web ACL in each account
- Prequisites for AWS Firewall Manager
  - AWS account needs to be part of an organization within the AWS Organizations service.
  - AWS Config must be enabled in all of the AWS Organizations member accounts. Firewall Manager relies on AWS Config to evaluate the compliance status of your AWS resources against desired security policies.
  - Setup the Firewall Manager Administrator by using an administrator account

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
- AD connection from AWS Managed AD to on-premise node can be encrypted by using LDAP over SSL(LDAPS).
- To reset password of a Windows Server EC2 instancez, you can
  - Online: Use System Manager Run Command to run AWSSupport-RunEC2RescueForWindowsTool command document
  - Offline: Use Systems Manager Automation AWSSupport-ResetAccess
- GuardDuty can perform an assetment of network communication and produce "CryptoCurrency:EC2/BitcoinTool.B" finding when an EC2 is querying an IP that associate with Bitcoin
    - The automation document will create a new instance and attach the original EBS volume create a new password. Then it will create a new AMI, then you can launch a new instance from it.
- You can use groups to create a collection of users in a user pool, then set the permission for those users
- Using System Manager on machine other than EC2
  - Create an IAM service role
  - Create a hybrid activation, receive an Activation Code and Activation ID,
  - Install AWS System Manager SSM Agent on non-EC2 machine with that activation code
    - You can setup to rotate private key to strengthen security.
- VPC Flow Logs does not contain package content.
- To authorize for API call made to API Gateway, we can use a lambda function to provide access control. Lambda use bearer token, athentication strategies such as OAuth or SAML, headers, query strings, context variables, etc
- Lambda can be used response to API call from these service(config from Add Trigger setting)
  - Auto-Scaling
  - EC2
    - Security Group changes
  - Health
  - RDS
  - S3
  - StepFunction
- Using `aws:PrincipalOrgID` in IAM Policy to require all principals to access the resource from an account in the organization.
- You can create a private Certificate Authority(CA) via ACM. After filling in all requrired infomation, such as CA subject name, key algorithm, you can fully manage the private CA(expiration date)
  - There is no ACM private CA policy. You need to use IAM policys to control who can access the CA
- ALB and NLB support multiple certificates and smart certificate selection using Server Name Indication(SNI)
- To make GuardDuty ignore some IPs, add a custom trusted IP list and add it to GuardDuty via console.
- AWS Backup Vault is a container that is responsible for storing and organizing backups with encryption. In addtion, Vault Lock provides extra layer of security(Compliance Mode and Governance Mode)
- HSM Module can be deployed in cluster that contains serveral HSMs in different AZ in an AWS region
- In order to enable single sign-on to AWS, the 3rd party IdP should be configured to use AWS as aa relying party
  - The AWS metadata URL "https://signin.aws.amazon.com/static/saml-metadata.xml" should be added in relying party config file.
- Control Tower behavior:
  - Preventive: A preventive control ensures that your accounts maintain compliance, because it disallows actions that lead to policy violations. The status of a preventive control is either enforced or not enabled. Preventive controls are supported in all AWS Regions. Implement via SCP.
  - Detective: A detective control detects noncompliance of resources within your accounts, such as policy violations, and provides alerts through the dashboard. The status of a detective control is either clear, in violation, or not enabled. Detective controls apply only in those AWS Regions supported by AWS Control Tower. Implement via AWS Config rules
  - Proactive: A proactive control scans your resources before they are provisioned, and makes sure that the resources are compliant with that control. Resources that are not compliant will not be provisioned. Proactive controls are implemented by means of AWS CloudFormation hooks, and they apply to resources that would be provisioned by AWS CloudFormation. The status of a proactive control is PASS, FAIL, or SKIP. Implement via CloudFormation hooks
