# CSPM Hunting

## msgraph

from: [Connect to Advanced Hunting API with the Graph SDK PowerShell module](https://www.french365connection.co.uk/post/connect-to-advanced-hunting-api-with-the-graph-sdk-powershell-module)

```
$KQLQuery = "DeviceNetworkInfo | take 10"
$uri = "https://graph.microsoft.com/v1.0/security/runHuntingQuery"
$query = @{query = $KQLQuery}|convertto-json -depth 100
$response = Invoke-MgGraphRequest -Uri $uri -Method POST -Body $query -ErrorAction Stop
```

```
$results = @()
do
{
    $results += $response.results 
}
while ($response.'@odata.nextLink')
```

## ARM query via logic app

```
{
    "definition": {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "actions": {
            "HTTP_2": {
                "inputs": {
                    "authentication": {
                        "type": "ManagedServiceIdentity"
                    },
                    "body": {
                        "query": "securityresources
| where type == "microsoft.security/assessments"
| where subscriptionId == "00000000000000000000000000000000"
| extend risk = parse_json(properties.risk)
//| extend additionalData = parse_json(properties.additionalData)
| extend 
    properties = parse_json(properties), 
    level = tostring(risk.level), 
    status = tostring(properties.status.code), 
    ResourceName = tostring(properties.resourceDetails.ResourceName),
    ResourceType = tostring(properties.resourceDetails.ResourceType),
    displayName = tostring(properties.displayName),
    subAssessment = tostring(properties.additionalData.subAssessmentsLink),
    link = tostring(properties.links.azurePortal)
| where status == "Unhealthy"
| where level == "Critical" //or risk.attackPathsReferences != "[]"
| order by ['level'] asc | limit 100
| order by ['kind'] desc"
                    },
                    "headers": {
                        "Content-Type": "application/json"
                    },
                    "method": "POST",
                    "queries": {
                        "api-version": "2021-03-01"
                    },
                    "uri": "https://management.azure.com/providers/Microsoft.ResourceGraph/resources"
                },
                "runAfter": {},
                "type": "Http"
            }
        },
        "contentVersion": "1.0.0.0",
        "outputs": {},
        "parameters": {},
        "triggers": {
            "Recurrence": {
                "recurrence": {
                    "frequency": "Minute",
                    "interval": 1440
                },
                "type": "Recurrence"
            }
        }
    },
    "parameters": {}
}
```
