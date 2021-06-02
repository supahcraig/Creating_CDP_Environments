THIS IS A WORK IN PROGRESS

# Creating a CDP Environment using the CDP Console

Cheat Sheet:
https://docs.google.com/document/d/1BTTrZ7NijD-xCrlg1YYfHBDjN3KYLEKku3b3sOZ5En4/edit#

## References you'll use throughout the deployment

_Looking to remove this section from the documentation...... _

* ${LOGS_BUCKET} : `cnelson2-logs`
* ${LOGS_LOCATION_BASE} : `cnelson2-logs/log`
* ${DATALAKE_BUCKET} : `cnelson2-data`
* ${STORAGE_LOCATION_BASE} : `cnelson2-data/gravity` --> _is gravity necessary here?  I don't think so_ 
* ${DYNAMODB_TABLE_NAME} : `cnelson2`
* ${AWS_ACCOUNT_ID} : `the account id of *YOUR* AWS account`
* ${IDBROKER_ROLE} : `cnelson2-idbroker-role`


## Create 2 buckets in S3, use default permissions

One bucket is for data, one for logs

* `cnelson2-data`
* `cnelson2-logs`. --> _not necessary_

---

## Create the IAM Pre-requisites

### &#x1F534; NOTE:  
The ranger audit role resource should not include the /ranger/audit portion of the path.  This is a mistake in the docs.
Should look like:
`"Resource": "arn:aws:s3:::cnelson2-data/gravity/*"` --> _is gravity necessary here?_

The IAM policies are json documents found in this repo

### Create IAM Policies

NOTE:  In the Azure setup, the roles are well-named with respect to the input fields in the CDP console.  The names we tend to use in AWS are terrible from that perspective.  I will consider changing them.

Policies could be prefixed with your username to ensure uniquness.  Also note references to ARN's & s3 buckets which will need to be updated to point to your buckets & other AWS objects.

* `idbroker-assume-role-policy`
* `log-policy-s3-access-policy`
* `bucket-policy-s3-access-policy`
* `datalake-admin-policy-s3-access-policy`  --> _this one had references to gravity, but I've removed them.  Also how is this different from the bucket policy??_
* `ranger-audit-s3-access-policy` --> _upper section of policy referenced gravity, but I removed it._
* `dynamodb-policy-policy`

* <username>-cross-account-policy. --> used in creation of CDP credential
* <username>-gravity-policy --> _is anything specifically named for gravity necessary in general???_

NOTE:  you MAY also need this policy:  https://github.com/supahcraig/cldr_tech_basecamp/blob/main/missions/2_data_access_in_CDP/cnelson2-gravity-policy.json
  
### Create IAM Roles & Attach Policies
  
Roles could be prefixed with your username to ensure uniquness.   The former names are references back to the basecamp CDP instructions & how those IAM roles were named.  Those names did not map to the fields in the CDP console directly, so I've changed the role names to make the UI steps easier on you, dear reader.

* `assumer-instance-role` _(formerly known as id-broker-role)_
  * Attach policy `idbroker-assume-role-policy`
  * Use ec2 as the use case
* `data-access-role` _(formerly known as datalake-admin-role)_
  * Attach policy `dynamodb-policy`
  * Attach policy `bucket-policy-s3-access`
  * Attach policy `datalake-admin-policy-s3-access`
  * Use ec2 as the use case, although it will not matter after we update the trust relationship
  * Trust Relationship:
    * Trash the existing trust relationship 
    * Replace it with the datalake trust policy found in this repo
    * Update the Principal to be the arn of your `assumer-instance-role`
* `logger-instance-role` _(formerly log-role)_
  * Attach policy `log-policy-s3-access`
  * Attach policy `bucket-policy-s3-access`
  * Use ec2 as the use case
* `ranger-audit-role`
  * Attach policy `ranger-audit-policy-s3-access`
  * Attach policy `dynamodb-policy`
  * Attach policy `bucket-policy-s3-access`
  * Use ec2 as the use case, although it will not matter after we update the trust relationship
  * Trust Relationship:
    * Trash the existing trust relationship 
    * Replace it with the datalake trust policy found in this repo
    * Update the Principal to be the arn of your `assumer-instance-role`
  
---
  
## Create CDP Credentials

In the CDP console, go to Environments --> Shared --> Credentials --> Create new credential.

Give your credential a name, disable Enable Permision Verification
  
Below that you will find 3 things of vital importance:
  * the cross-account access policy JSON document (also found in this here repository)
  * the service account manager Account ID
  * the External ID

### Create Cross Account IAM Role/Policy in AWS  

* Create a new policy `cross-account-policy` in AWS
  * Use the JSON policy document from this repo OR get the most up-to-date version from the CDP Create Environment Page 
* Create a new role `CDP-cross-account-role`
  * Use "Another AWS Account" as the type of trusted entity
    * Use the Service Manager Account ID for the Account ID
    * Check Require external ID
    * Use the External ID for External ID
    * Do not require MFA
  * attach the `cross-account-policy`
  * Verify the trust relationship allows for the CDP Account & External ID
  * Copy the ARN for your new role.......
  
Paste the role ARN into the Cross-account Role ARN and click *Create*

---  
  
# Creating the CDP environment
  
From the CDP Management Console:  
  
1. Select your credential, or create a new credential


2. Name your data lake
3. Set the Data Access & Audit roles
  * Assumer Instance Profile is the id-broker role
  * Data access role is the datalake admin role
  * Ranger audit role is `ranger-audit-role`
  * Storage location base is the name of your S3 bucket. _do I need to include /data here??_

*CLICK NEXT.*
  
4. choose correct region

### Networking
5. create new network, use `10.10.0.0/16` CIDR block
  * disable Create Private Subnets
  * disable Create Private Endpoints
  * disable Cluster Connectivity Manager (CCM)
  * disable Enable Public Endpoint Access Gateway
  * disable Enable FreeIPA HA
  * do not use Proxy Configuration
6. Create New Security Groups
  * use `0.0.0.0/0` CIDR block
7. Use Existing SSH public key
  * Pick your keypair or create a new key pair
  * ***Creating a new Key Pair***
    1. Go to Key Pairs in the EC2 console in AWS
    2. Click "Create key pair"
    3. Name your key pair
    4. Use the pem format
    5. Click "Create key pair"
    6. It will automatically download the pem file, and will exist in AWS as a key pair for later use

8. Enable S3Guard
  * enter your dynamodb table name (see above):  `cnelson2` _(it hasn't been created yet, but just use your username as the table name)_
    * the table name you use here needs to match the table name in the `dyanmodb-policy`

*CLICK NEXT.*  
  
### Logging
9. Logger instance profile is your `logger-instance-role`
10. Logs location base is the s3 path to your bucket: `cnelson2-data`

  
  
## How to create environment?
```
cdp environments create-aws-environment \
--environment-name cnelson2 \
--credential-name cnelson2 \
--region "us-east-2" \
--security-access cidr=0.0.0.0/0 \
--tags key=enddate,value=05312021 key=owner,value=okta/cnelson2@cloudera.com key=project,value=basecamp/04222021  \
--enable-tunnel \
--authentication publicKeyId="cnelson2-basecamp-keypair" \
--log-storage storageLocationBase=s3a://cnelson2-logs/log,instanceProfile=arn:aws:iam::665634629064:instance-profile/cnelson2-log-role \
--network-cidr 10.10.0.0/16 \
--s3-guard-table-name cnelson2 \
--free-ipa instanceCountByGroup=1 
```
This can take 15+ minutes.


## How to create id broker mappings?
```
cdp environments set-id-broker-mappings \
--environment-name cnelson2 \
--data-access-role arn:aws:iam::665634629064:role/cnelson2-datalake-admin-role \
--ranger-audit-role arn:aws:iam::665634629064:role/cnelson2-ranger-audit-role \
--set-empty-mappings 
```
This is more or less instantaneous.


## How to create data lake?
```
cdp datalake create-aws-datalake \
--datalake-name cnelson2 \
--environment-name cnelson2 \
--cloud-provider-configuration instanceProfile=arn:aws:iam::665634629064:instance-profile/cnelson2-idbroker-role,storageBucketLocation=s3a://cnelson2-data/gravity \
--tags key=enddate,value=05312021 key=owner,value=okta/cnelson2@cloudera.com key=project,value=basecamp/04222021 \
--scale LIGHT_DUTY \
--runtime 7.2.1 
```
This can take a long time.....
