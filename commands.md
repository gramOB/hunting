# starter

from: [Connect to Advanced Hunting API with the Graph SDK PowerShell module](https://www.french365connection.co.uk/post/connect-to-advanced-hunting-api-with-the-graph-sdk-powershell-module)

```
$KQLQuery = @"
let updateSCID = dynamic (["scid-2011","scid-2030","scid-5095","scid-6095"]);
 DeviceTvmSecureConfigurationAssessment
| where ConfigurationId in (updateSCID) and IsApplicable
| join (
    DeviceInfo
    | summarize by DeviceId,SensorHealthState,RegistryDeviceTag
    )
    on DeviceId
| where SensorHealthState == "Active"
| extend avdata=parsejson(Context)
| extend sigversion = tostring(avdata[0][0])
| extend engversion = tostring(avdata[0][1])
| extend platformversion = tostring(avdata[0][3])
| extend lastupdatetime = todatetime(avdata[0][2])
| where lastupdatetime < ago(7d)
| summarize by DeviceId,DeviceName,OSPlatform,sigversion,engversion,platformversion,lastupdatetime
"@
$uri = "https://graph.microsoft.com/v1.0/security/runHuntingQuery"
$query = @{query = $KQLQuery}|convertto-json -depth 100
$response = Invoke-MgGraphRequest -Uri $uri -Method POST -Body $query -ErrorAction Stop
```
