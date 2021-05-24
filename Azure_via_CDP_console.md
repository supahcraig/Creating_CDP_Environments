# Creating a CDP Environment in Azure using the CDP Managment & Azure console

Thanks to https://github.com/cpv0310/cdp-azure-tools for laying the groundwork for this.


## Collect Subscription & Tenant Info

From the Azure console, open a command prompt:
>
`az account list|jq '.[]|{"name": .name, "subscriptionId": .id, "tenantId": .tenantId, "state": .state}'`

Output should looks something like:
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
https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fcegganesh84%2Fcdp-azure-tools%2Fmaster%2Fazuredeploy.json

This will open an ARM template to build out the prerequisites in your Azure subscription:  virtual network, storage account, and 4 identities

* Select your region
* Create a new resource group
* Pick your environment name (no dashes allowed here, just upper/lower alphanumeric)

Once created, go back to the Azure shell:

_You can use whatever you like for the custom app name, I don't think we ever actually use it._
`az ad sp create-for-rbac --name http://<CUSTOM APP NAME>-app --role Contributor --scopes /subscriptions/<SUBSCRIPTION ID>`

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

### Set 2 environment variables
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


## Register the Environment in the CDP Management Console

1. Register Environment
2. Select Azure as the cloud environment
3. Choose/Create your Azure Credential

### Creating an Azure CDP Credential

1. Select Azure as the cloud environment
2. Give your credential a name
3. Enter the Subscription ID & Tenent ID, which you found using the `az account list` command
4. Go back to the Azure shell and run this command to get the App ID & Password components

_You can use whatever you like for the custom app name, I don't think we ever actually use it._
`az ad sp create-for-rbac --name http://<CUSTOM APP NAME>-app --role Contributor --scopes /subscriptions/<SUBSCRIPTION ID>`

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

HIT CREATE.

