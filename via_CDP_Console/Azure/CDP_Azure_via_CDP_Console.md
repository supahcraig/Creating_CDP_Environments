# Creating a CDP Environment in Azure using the CDP Managment & Azure console

Thanks to https://github.com/cpv0310/cdp-azure-tools for laying the groundwork for this.


## Collect Subscription & Tenant Info

From the Azure console, open a command prompt:
>
`az account list|jq '.[]|{"name": .name, "subscriptionId": .id, "tenantId": .tenantId, "state": .state}'`

Output should looks something like this, which you will use later on in the process.
>>
```
{
  "name": "azure-se-cdp-sandbox-env",
  "subscriptionId": "subscription-ID-text-here",
  "tenantId": "tenant-ID-text-here",
  "state": "Enabled"
}
```

---

## Build out Prerequisites using an Azure Resource Managment Template

Go here:
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fcegganesh84%2Fcdp-azure-tools%2Fmaster%2Fazuredeploy.json">CDP ARM Template</a>

This will open an ARM template to build out the prerequisites in your Azure subscription:  virtual network, storage account, and 4 identities

* Select your region
* Create a new resource group
  * If you click create but it fails validation, odds are your resource group still got created
* Pick your environment name (no dashes allowed here, just lower-case/alphanumeric)
  * this environment name is not the same as your CDP environment name

**CLICK CREATE.**

---

This won't take too long, but it's not done until you see all 6 resources in your resource group.

| Type          | Resource Name |
| ------------- |-------------|
| Virtual Network  | `<env name>` |
| Storage Account  | `<env name>`      |
| Managed Identity | `<env name>-AssumerIdentity`     |
| Managed Identity | `<env name>-DataAccessIdentity`  |
| Managed Identity | `<env name>-LoggerIdentity`      |
| Managed Identity | `<env name>-RangerIdentity`      |

---

## Set some environment variables

From the Azure console shell create these two environment variables which will be used in an upcoming script.
>>
```
export SUBSCRIPTIONID=YourSubscriptionId
export RESOURCEGROUPNAME=YourResourceGroupName
```

### Run the script to assign the privs to the identities

Either copy/paste all this into the shell, or save it as a script, chmod +x, and execute.   
```
export STORAGEACCOUNTNAME=$(az storage account list -g $RESOURCEGROUPNAME --subscription $SUBSCRIPTIONID|jq '.[]|.name'| tr -d '"')
export ASSUMER_OBJECTID=$(az identity list -g $RESOURCEGROUPNAME --subscription $SUBSCRIPTIONID|jq '.[]|{"name":.name,"principalId":.principalId}|select(.name | test("AssumerIdentity"))|.principalId'| tr -d '"')
export DATAACCESS_OBJECTID=$(az identity list -g $RESOURCEGROUPNAME --subscription $SUBSCRIPTIONID|jq '.[]|{"name":.name,"principalId":.principalId}|select(.name | test("DataAccessIdentity"))|.principalId'| tr -d '"')
export LOGGER_OBJECTID=$(az identity list -g $RESOURCEGROUPNAME --subscription $SUBSCRIPTIONID|jq '.[]|{"name":.name,"principalId":.principalId}|select(.name | test("LoggerIdentity"))|.principalId'| tr -d '"')
export RANGER_OBJECTID=$(az identity list -g $RESOURCEGROUPNAME --subscription $SUBSCRIPTIONID|jq '.[]|{"name":.name,"principalId":.principalId}|select(.name | test("RangerIdentity"))|.principalId'| tr -d '"')

# Assign Managed Identity Operator role to the assumerIdentity principal at subscription scope
az role assignment create --assignee $ASSUMER_OBJECTID --role 'f1a07417-d97a-45cb-824c-7a7467783830' --scope "/subscriptions/$SUBSCRIPTIONID"
# Assign Virtual Machine Contributor role to the assumerIdentity principal at subscription scope
az role assignment create --assignee $ASSUMER_OBJECTID --role '9980e02c-c2be-4d73-94e8-173b1dc7cf3c' --scope "/subscriptions/$SUBSCRIPTIONID"

# Assign Storage Blob Data Contributor role to the loggerIdentity principal at logs filesystem scope
az role assignment create --assignee $LOGGER_OBJECTID --role 'ba92f5b4-2d11-453d-a403-e96b0029c9fe' --scope "/subscriptions/$SUBSCRIPTIONID/resourceGroups/$RESOURCEGROUPNAME/providers/Microsoft.Storage/storageAccounts/$STORAGEACCOUNTNAME/blobServices/default/containers/logs"
# Assign Storage Blob Data Owner role to the dataAccessIdentity principal at logs/data filesystem scope
az role assignment create --assignee $DATAACCESS_OBJECTID --role 'b7e6dc6d-f1e8-4753-8033-0f276bb0955b' --scope "/subscriptions/$SUBSCRIPTIONID/resourceGroups/$RESOURCEGROUPNAME/providers/Microsoft.Storage/storageAccounts/$STORAGEACCOUNTNAME/blobServices/default/containers/data"
az role assignment create --assignee $DATAACCESS_OBJECTID --role 'b7e6dc6d-f1e8-4753-8033-0f276bb0955b' --scope "/subscriptions/$SUBSCRIPTIONID/resourceGroups/$RESOURCEGROUPNAME/providers/Microsoft.Storage/storageAccounts/$STORAGEACCOUNTNAME/blobServices/default/containers/logs"
# Assign Storage Blob Data Contributor role to the rangerIdentity principal at data filesystem scope
az role assignment create --assignee $RANGER_OBJECTID --role 'ba92f5b4-2d11-453d-a403-e96b0029c9fe' --scope "/subscriptions/$SUBSCRIPTIONID/resourceGroups/$RESOURCEGROUPNAME/providers/Microsoft.Storage/storageAccounts/$STORAGEACCOUNTNAME/blobServices/default/containers/data"
```

The output will be several JSON documents which will be of no use to you.

---

## Register the Environment in the CDP Management Console

1. Click Register Environment button
2. Select Azure as the cloud environment
3. Choose/Create your Azure Credential

### Creating an Azure CDP Credential
If you don't already have a credential you'd like to use, follow these steps:

1. Select Azure as the cloud environment
2. Give your credential a name
3. Enter the Subscription ID & Tenent ID, which you found using the `az account list` command
4. Go back to the Azure shell and run this command to get the App ID & Password components

_You can use whatever you like for the custom app name, I don't think we ever actually use it._
I think it makes sense to just use your environment name here, so set it as an environment variable first.

`export ENVIRONMENTNAME=<env name>`

`az ad sp create-for-rbac --name http://$ENVIRONMENTNAME-app --role Contributor --scopes /subscriptions/$SUBSCRIPTIONID`

Output should look something like this:
>>
```
{
  "appId": "app-ID-text-here",
  "displayName": "cnelson2-cdp-app",
  "name": "http://cnelson2-cdp-app",
  "password": "password-text-here",
  "tenant": "tenant-ID-text-here"
}
```

**HIT CREATE.**

The CDP CLI command for creating your credential will look like this.   If it works, it will be one of the few "show CLIs" that actually works.

```
cdp environments create-azure-credential \
--credential-name cnelson2-az-credential \
--subscription-id abce3e07-b32d-4b41-8c78-2bcaffe4ea27 \
--tenant-id c7f832d2-fcca-4595-9860-1e81b76c28ff \
--app-based applicationId=85077951-ea7f-4945-b738-eedf1b0f4f42,secretKey=<YOUR_SECRET> 
```


---

### Finish CDP Envionrment steps

All these items can be found from inside your resource group in the Azure console, or if you remember your environment name & start typing in the box.

4. Assumer Identity = `<resource group name>-AssumerIdentity'
5. Storage Identity = `data@<storage account name>`
6. Data Access Identity = `<resource group name>-DataAccessIdentity`
7. Ranger Audit Identity = `<resource group name>-RangerIdentity`

(click next)

8. Select your region & resource group
9. Select your network (virtual network name found in Azure console under your resource group)
10. Click Default for subnet
11. Disable CCM (was 90% sure Chris Van Dyke had this enabled)
12. Disable the following:
  * Create private endpoints
  * Don't create public IP
  * Enable Free IPA HA
14. Do not use proxy configuration
15. Create new security Group
  * Allow 0.0.0.0/0

___

#### SSH Settings

16. Create a new SSH key

From your local machine:

`ssh-keygen -t rsa`

Choose a filename to save it to (I like `azure.rsa` but it doesn't matter), and use an empty passphrase.

Copy the public key

`cat azure.rsa.pub`

The entire key will need to be copied, it should look something like this:

`ssh-rsa rAnDoMsTrRiNgOfChArAcHtErSeNdInGwItH= yourusername@host`

And paste that into the New SSH key box in the CDP console

___

(click next)

17. Logger Identity = `<resource group name>-LoggerIdentity`
18. Logs Location Base = `logs@<storage account name>`

**CREATE ENVIRONMENT.**

This will take a while.

---

# Azure Resources Created with a CDP Environment

In addition to the virtual network, storage account, and managed identities needed to create the environment, CDP will spin up these resources:

| Type          | Resource Name |
| ------------- |-------------|
| Availability Set  | `<xxx>-azure-freeipa-master0-as` |
| Azure Database for PostgreSQL server  | _`<guid>`_      |
| Disk | `<xxx>-datalake108874-osDiski1`     |
| Disk | `<xxx>-datalake108874-osDiskm0`     |
| Disk | `<xxx>-freeipa17826-osDiskm0`     |
| Disk | `<xxx>datalak-m-0-0-08d70630ed9840`     |
| Image | `cb-cdh-729-1619762518.vhd-southcentralus`  |
| Image | `freeipa-cdh--2103081333.vhd-southcentralus`      |
| Network Interface | `<xxx>-datalake108874i1`      |
| Network Interface | `<xxx>-datalake108874m0`      |
| Network Interface | `<xxx>-freeipa17826m0`      |
| Network Security Group | `idbroker<xxx>-datalake108874sg`     |
| Network Security Group | `master0<xxx>-freeipa17826sg`     |
| Network Security Group | `master<xxx>-datalake108874sg`     |
| Public IP address | `<xxx>-datalake108874i1`    |
| Public IP address | `<xxx>-datalake108874m0`    |
| Public IP address | `<xxx>-freeipa17826m0`    |
| Storage Account | `cbimgscu9fd109a716490400`   |
| Virtual Machine | `<xxx>-datalake108874i1`   |
| Virtual Machine | `<xxx>-datalake108874m0`   |
| Virtual Machine | `<xxx>-freeipa17826m0`   |

_where <xxx> is your CDP environment name_
  
  
# Tearing Down your Azure CDP Environment

Deleting the environment will remove most of the Azure resources created, but it will preserve the Resource Group, which contains the storage account, virtual network, and managed identies.  These can be reused in your next environment if you want.
