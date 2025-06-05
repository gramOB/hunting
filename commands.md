# starter

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
