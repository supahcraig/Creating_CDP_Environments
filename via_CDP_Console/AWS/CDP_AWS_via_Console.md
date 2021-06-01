THIS IS A WORK IN PROGRESS

# Creating a CDP Environment using the CDP Console

Cheat Sheet:
https://docs.google.com/document/d/1BTTrZ7NijD-xCrlg1YYfHBDjN3KYLEKku3b3sOZ5En4/edit#


## Create 2 buckets in S3, use default permissions

One bucket is for data, one for logs

* `cnelson2-data`
* `cnelson2-logs`

## References you'll use throughout the deployment
* ${LOGS_BUCKET} : `cnelson2-logs`
* ${LOGS_LOCATION_BASE} : `cnelson2-logs/log`
* ${DATALAKE_BUCKET} : `cnelson2-data`
* ${STORAGE_LOCATION_BASE} : `cnelson2-data/gravity` --> _is gravity necessary here?_ 
* ${DYNAMODB_TABLE_NAME} : `cnelson2`
* ${AWS_ACCOUNT_ID} : `the account id of *YOUR* AWS account`
* ${IDBROKER_ROLE} : `cnelson2-idbroker-role`


## Create the IAM Pre-requisites

### &#x1F534; NOTE:  
The ranger audit role resource should not include the /ranger/audit portion of the path.  This is a mistake in the docs.
Should look like:
`"Resource": "arn:aws:s3:::cnelson2-data/gravity/*"` --> _is gravity necessary here?_

The IAM policies are json documents found in this repo

### Create IAM Policies

NOTE:  In the Azure setup, the roles are well-named with respect to the input fields in the CDP console.  The names we tend to use in AWS are terrible from that perspective.  I will consider changing them.

Policies could be prefixed with your username to ensure uniquness.  Also note references to arn's & s3 buckets which will need to be updated to point to your buckets & other AWS objects.

* <username>-idbroker-assume-role-policy
* <username>-log-policy-s3-access-policy
* <username>-bucket-policy-s3-access-policy
* <username>-datalake-admin-policy-s3-access-policy  --> _this one had references to gravity, but I've removed them.  Also how is this different from the bucket policy??
* <username>-ranger-audit-s3-access-policy --> _upper section of policy referenced gravity, but I removed it.
* <username>-dynamodb-policy-policy

* <username>-cross-account-policy
* <username>-gravity-policy --> _is anything specifically named for gravity necessary in general?_

NOTE:  you MAY also need this policy:  https://github.com/supahcraig/cldr_tech_basecamp/blob/main/missions/2_data_access_in_CDP/cnelson2-gravity-policy.json
  
### Create IAM Roles & Attach Policies
  
Roles could be prefixed with your username to ensure uniquness

* <username>-assumer-instance-role _(formerly known as id-broker-role)_
  * Attach policy `idbroker-assume-role-policy`
  * Use ec2 as the use case
* <username>-data-access-role _(formerly known as datalake-admin-role)_
  * Attach policy `dynamodb-policy`
  * Attach policy `bucket-policy-s3-access`
  * Attach policy `datalake-admin-policy-s3-access`
  * Use ec2 as the use case, although it will not matter after we update the trust relationship
  * Trust Relationship:
    * Trash the existing trust relationship 
    * Replace it with the datalake trust policy found in this repo
    * Update the Principal to be the arn of your `assumer-instance-role`
* <username>-logger-instance-role _(formerly log-role)_
  * Attach policy `log-policy-s3-access`
  * Attach policy `bucket-policy-s3-access`
  * Use ec2 as the use case
* <username>-ranger-audit-role
  * Attach policy `ranger-audit-policy-s3-access`
  * Attach policy `dynamodb-policy`
  * Attach policy `bucket-policy-s3-access`
  * Use ec2 as the use case, although it will not matter after we update the trust relationship
  * Trust Relationship:
    * Trash the existing trust relationship 
    * Replace it with the datalake trust policy found in this repo
    * Update the Principal to be the arn of your `assumer-instance-role`
  

## Create CDP Credentials


### Create Cross Account IAM Role/Policy in AWS  

* Create a new policy `<username>-cross-account-policy` in AWS
  * Use the JSON policy document from this repo OR get the most up-to-date version from the CDP Create Environment Page
* 
* Edit the Trust Relationship for the role
  
Give your credential a name, disable Enable Permision Verification
  

  
  
# Creating the environment
  
1. Select your credential, or create a new credential


2. Name your data lake
3. Set the Data Access & Audit roles
  * Assumer Instance Profile is the id-broker role
  * Data access role is the datalake admin role
  * Ranger audit role is `ranger-audit-role`
  * Storage location base is the name of your S3 bucket

  ## NOTE:
choose correct region

### Networking
* create new network, use `10.10.0.0/16`
* disable private subnets
* create new security groups, use `0.0.0.0/0`
* pick your keypair
* enter your dynamodb table name (see above):  `cnelson2` _(it hasn't been created yet, but just use your username as the table name)_

### Logging
* Logger instance profile is the log-role
* s3 path is the logs location base (see above): `cnelson2-logs/log`

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
