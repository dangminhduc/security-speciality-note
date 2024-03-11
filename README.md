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
- Custom config rule can be trigger via resource's change events or periodically at maximum interval of 1 hour

## Cognito
- Identity Pools can be use to give unauthenticated user to access AWS resources
- Captcha can be enable via Auth Challenge Lambda Trigger

## VPC
- Network ACL is stateless, which means it need both inbound and outbound rules for traffic
- Security Groups is stateful, which mean it only need the rule to initiate the request, no need to have a rule for the returning traffic.
- Security Group's changes is 

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
- It is possible to have different encyption keys for different verions of the same object in S3
- To authorize for API call made to API Gateway, we can use a lambda function to provide access control. Lambda use bearer token, athentication strategies such as OAuth or SAML, headers, query strings, context variables, etc
- To encrypt data before sending it to S3 bucket, use client-side encryption
  - The object stored in S3 aren't exposed to any 3rd party, including AWS(S3 only dectect it as typical objects)
  - Lambda can be used response to API call from these service(config from Add Trigger setting)
    - Auto-Scaling
    - EC2
      - Security Group changes
    - Health
    - RDS
    - S3
    - StepFunction
- Using `aws:PrincipalOrgID` in IAM Policy to require all principals to access the resource from an account in the organization.
- S3 Replication(both same and cross region) require bucket versioning.
- You can create a private Certificate Authority(CA) via ACM. After filling in all requrired infomation, such as CA subject name, key algorithm, you can fully manage the private CA(expiration date)
  - There is no ACM private CA policy. You need to use IAM policys to control who can access the CA
- ALB and NLB support multiple certificates and smart certificate selection using Server Name Indication(SNI
