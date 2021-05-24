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
  "subscriptionId": "abce3e07-b32d-4b41-8c78-2bcaffe4ea27",
  "tenantId": "c7f832d2-fcca-4595-9860-1e81b76c28ff",
  "state": "Enabled"
}
```

---

## Build out Prerequisites using an Azure Resource Managment Template

Go here:  https://portal.azure.com/#create/Microsoft.Template

