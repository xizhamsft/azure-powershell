# Breaking changes for Microsoft Azure PowerShell 4.0.0

This document serves as both a breaking change notification and migration guide for consumers of the Microsoft Azure PowerShell cmdlets. Each section describes both the impetus for the breaking change and the migration path of least resistance. For in-depth context, please refer to the pull request associated with each change.

## Table of Contents

- [Breaking changes to Compute cmdlets](#breaking-changes-to-compute-cmdlets)
- [Breaking changes to EventHub cmdlets](#breaking-changes-to-eventhub-cmdlets)
- [Breaking changes to Insights cmdlets](#breaking-changes-to-insights-cmdlets)
- [Breaking changes to ServiceBus cmdlets](#breaking-changes-to-servicebus-cmdlets)
- [Breaking changes to Sql cmdlets](#breaking-changes-to-sql-cmdlets)
- [Breaking changes to Storage cmdlets](#breaking-changes-to-storage-cmdlets)

## Breaking changes to Compute cmdlets

The following output types were affected this release:

### PSVirtualMachine
- Top level properties `DataDiskNames` and `NetworkInterfaceIDs` of nthe `PSVirtualMachine` object have been removed from the output type. These properties have always been available in the `StorageProfile` and `NetworkProfile` properties of the `PSVirtualMachine` object and will be the way they will need to be accessed going forward.
- This change affects the following cmdlets:
    - `Add-AzureRmVMDataDisk`
    - `Add-AzureRmVMNetworkInterface`
    - `Get-AzureRmVM`
    - `Remove-AzureRmVMDataDisk`
    - `Remove-AzureRmVMNetworkInterface`
    - `Set-AzureRmVMDataDisk`

```powershell
# Old
$vm.DataDiskNames
$vm.NetworkInterfaceIDs

# New
$vm.StorageProfile.DataDisks | Select -Property Name
$vm.NetworkProfile.NetworkInterfaces | Select -Property Id
```

## Breaking changes to EventHub cmdlets

The following cmdlets were affected this release:

### Get-AzureRmEventHubNamespace
- The property `ResourceGroupName` has been removed from the output type `NamespaceAttributes`

### New-AzureRmEventHubNamespace
- The property `ResourceGroupName` has been removed from the output type `NamespaceAttributes`

## Breaking changes to Insights cmdlets

The following cmdlets were affected this release:
    
### Get-AzureRmUsage
- This cmdlet has been deprecated.

### Remove-AzureRmAlertRule
- The output of this cmdlet has changed from a list with a single object to a single object; this object includes the requestId, and status code.
    
```powershell
# Old  
$s1 = Remove-AzureRmAlertRule -ResourceGroup $resourceGroup -name chiricutin
if ($s1 -ne $null)
{
    $r = $s1(0).RequestId
    $s = $s1(0).StatusCode
}

# New
$s1 = Remove-AzureRmAlertRule -ResourceGroup $resourceGroup -name chiricutin
$r = $s1.RequestId
$s = $s1.StatusCode
```
    
### Add-AzureRmLogAlertRule
- This cmdlet has been deprecated.
    
### Get-AzureRmAlertRule
- Each element of the the output (a list of objects) of this cmdlet is flattened, i.e. instead of returning objects with the structure `{ Id, Location, Name, Tags, Properties }` it will return objects with the structure `{ Id, Location, Name, Tags, Type, Description, IsEnabled, Condition, Actions, LastUpdatedTime, ...}`, which is all of the attributes of an Azure Resource plus all of the attributes of an AlertRuleResource at the top level.
    
```powershell
# Old
$rules = Get-AzureRmAlertRule -ResourceGroup $resourceGroup
if ($rules -and $rules.count -ge 1)
{
    Write-Host -fore red "Error updating alert rule"
      
    Write-Host $rules(0).Id
    Write-Host $rules(0).Properties.IsEnabled
    Write-Host $rules(0).Properties.Condition
}

# New
$rules = Get-AzureRmAlertRule -ResourceGroup $resourceGroup
if ($rules -and $rules.count -ge 1)
{
    Write-Host -fore red "Error updating alert rule"
 
    Write-Host $rules(0).Id
      
    # Properties will remain for a while
    Write-Host $rules(0).Properties.IsEnabled
      
    # But the properties will be at the top level too. Later Properties will be removed
    Write-Host $rules(0).IsEnabled
    Write-Host $rules(0).Condition
}
```
    
### Get-AzureRmAutoscaleSetting
- The `AutoscaleSettingResourceName` field is deprecated since it always has the same value as the `Name` field.

```powershell
# Old  
$s1 = Get-AzureRmAutoscaleSetting -ResourceGroup $resourceGroup -Name MySetting
if ($s1.AutoscaleSettingResourceName -ne $s1.Name)
{
    Write-Host "There is something wrong with the name"
}

# New
$s1 = Get-AzureRmAutoscaleSetting -ResourceGroup $resourceGroup -Name MySetting
    
# There won't be a AutoscaleSettingResourceName
Write-Host $s1.Name
```
    
### Remove-AzureRmLogProfile
- The output of this cmdlet will change from `Boolean` to and object containing `RequestId` and `StatusCode`

```powershell
# Old  
$s1 = Remove-AzureRmLogProfile -Name myLogProfile
if ($s1 -eq $true)
{
    Write-Host "Removed"
}
else
{
    Write-Host "Failed"
}

# New
$s1 = Remove-AzureRmLogProfile -Name myLogProfile
$r = $s1.RequestId
$s = $s1.StatusCode
```
    
### Add-AzureRmLogProfile
- The output of this cmdlet will change from an object that includes the requestId, status code, and the updated or newly created resource
    
```powershell
# Old  
$s1 = Add-AzureRmLogProfile -Name default -StorageAccountId /subscriptions/df602c9c-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/JohnKemTest/providers/Microsoft.Storage/storageAccounts/johnkemtest8162 -Locations Global -categ Delete, Write, Action -retention 3
$r = $s1.ServiceBusRuleId

# New
$s1 = Add-AzureRmLogProfile -Name default -StorageAccountId /subscriptions/df602c9c-7aa0-407d-a6fb-eb20c8bd1192/resourceGroups/JohnKemTest/providers/Microsoft.Storage/storageAccounts/johnkemtest8162 -Locations Global -categ Delete, Write, Action -retention 3
$r = $s1.RequestId
$s = $s1.StatusCode
$a = $s1.NewResource.ServiceBusRuleId
    
```
    
### Set-AzureRmDiagnosticSettings
- The command is going to be renamed to `Update-AzureRmDiagnsoticSettings`

```powershell
# Old
Set-AzureRmDiagnosticSettings

# New
Update-AzureRmDiagnosticSettings
```

## Breaking changes to ServiceBus cmdlets

The following cmdlets were affected this release:

### Get-AzureRmServiceBusNamespace
- The property `ResourceGroupName` has been removed from the output type `NamespaceAttributes`

### New-AzureRmServiceBusNamespace

- The property `ResourceGroupName` has been removed from the output type `NamespaceAttributes`

## Breaking changes to Sql cmdlets

The following cmdlets were affected this release:

### New-AzureRmSqlDatabaseFailoverGroup
- `Tag` parameter has been removed
- `GracePeriodWithDataLossHour` parameter has been renamed to `GracePeriodWithDataLossHours`

```powershell
# Old
New-AzureRmSqlDatabaseFailoverGroup -ResourceGroupName rg -ServerName server1 -FailoverGroupName fg -PartnerServerName server2 -FailoverPolicy Automatic -GracePeriodWithDataLossHour 1 -Tag @{ Environment="Test" }

# New
New-AzureRmSqlDatabaseFailoverGroup -ResourceGroupName rg -ServerName server1 -FailoverGroupName fg -PartnerServerName server2 -FailoverPolicy Automatic -GracePeriodWithDataLossHours 1
```

### Set-AzureRmSqlDatabaseFailoverGroup
- `Tag` parameter has been removed
- `GracePeriodWithDataLossHour` parameter has been renamed to `GracePeriodWithDataLossHours`

```powershell
# Old
Set-AzureRmSqlDatabaseFailoverGroup -ResourceGroupName rg -ServerName server1 -FailoverGroupName fg -FailoverPolicy Automatic -GracePeriodWithDataLossHour 1 -Tag @{ Environment="Test" }

# New
Set-AzureRmSqlDatabaseFailoverGroup -ResourceGroupName rg -ServerName server1 -FailoverGroupName fg -FailoverPolicy Automatic -GracePeriodWithDataLossHours 1
```

### Add-AzureRmSqlDatabaseToFailoverGroup
- `Tag` parameter has been removed

```powershell
# Old
Add-AzureRmSqlDatabaseToFailoverGroup -ResourceGroupName rg -ServerName server1 -FailoverGroupName fg -Database $db1 -Tag @{ Environment="Test" }

# New
Add-AzureRmSqlDatabaseToFailoverGroup -ResourceGroupName rg -ServerName server1 -FailoverGroupName fg -Database $db1
```

###  Remove-AzureRmSqlDatabaseFromFailoverGroup
- `Tag` parameter has been removed

```powershell
# Old
Remove-AzureRmSqlDatabaseFromFailoverGroup -ResourceGroupName rg -ServerName server1 -FailoverGroupName fg -Database $db1 -Tag @{ Environment="Test" }

# New
Remove-AzureRmSqlDatabaseFromFailoverGroup -ResourceGroupName rg -ServerName server1 -FailoverGroupName fg -Database $db1
```

### Remove-AzureRmSqlDatabaseFailoverGroup
- `PartnerResourceGroupName` parameter has been removed
- `PartnerServerName` parameter has been removed

```powershell
# Old
Remove-AzureRmSqlDatabaseFailoverGroup -ResourceGroupName rg -ServerName server1 -FailoverGroupName fg -PartnerServerName server2 -PartnerResourceGroupName rg

# New
Remove-AzureRmSqlDatabaseFailoverGroup -ResourceGroupName rg -ServerName server1 -FailoverGroupName fg
```

### Set-AzureRmSqlDatabaseThreatDetectionPolicy
- The value `Usage_Anomaly` is no longer valid for the parameter `ExcludedDetectionType`

### Set-AzureRmSqlServerThreatDetectionPolicy
- The value `Usage_Anomaly` is no longer valid for the parameter `ExcludedDetectionType`

## Breaking changes to Storage cmdlets

The following output type properties were affected this release:

### AzureStorageBlob.ICloudBlob.ServiceClient
- The following properties were removed from this type (_note_: they can still be found in `DefaultRequestOptions` property):
    - `LocationMode`
    - `MaximumExecutionTime`
    - `ServerTimeout`
    - `ParallelOperationThreadCount`
    - `SingleBlobUploadThresholdInBytes`
- This change affects the following cmdlets:
    - `Get-AzureStorageBlob`
    - `Get-AzureStorageBlobContent`
    - `Get-AzureStorageBlobCopyState`
    - `Set-AzureStorageBlobContent`
    - `Start-AzureStorageBlobCopy`
    - `Stop-AzureStorageBlobCopy`
    
### AzureStorageContainer.CloudBlobContainer.ServiceClient
- The following properties were removed from this type (_note_: they can still be found in the `DefaultRequestOptions` property):
    - `LocationMode`
    - `MaximumExecutionTime`
    - `ServerTimeout`
    - `ParallelOperationThreadCount`
    - `SingleBlobUploadThresholdInBytes`
- This change affects the following cmdlets:
    - `Get-AzureStorageContainer`
    - `New-AzureStorageContainer`
    - `Set-AzureStorageContainerAcl`
    
### AzureStorageQueue.CloudQueue.ServiceClient
- The following properties were removed from this type (_note_: they can still be found in the `DefaultRequestOptions` property):
    - `LocationMode`
    - `MaximumExecutionTime`
    - `RetryPolicy`
    - `ServerTimeout`
- This change affects the following cmdlets:
    - `Get-AzureStorageQueue`
    - `New-AzureStorageQueue`
    
### AzureStorageTable.CloudTable.ServiceClient
- The following properties were removed from this type (_note_: they can still be found in the `DefaultRequestOptions` property):
    - `LocationMode`
    - `MaximumExecutionTime`
    - `PayloadFormat`
    - `RetryPolicy`
    - `ServerTimeout`
- This change affects the following cmdlets:
    - `Get-AzureStorageTable`
    - `New-AzureStorageTable`
    
```powershell
# Old
$LocationMode = (Get-AzureStorageBlob -Container $containername)[0].ICloudBlob.ServiceClient.LocationMode		
$ParallelOperationThreadCount = (Get-AzureStorageContainer -Container $containername).CloudBlobContainer.ServiceClient.ParallelOperationThreadCount
$PayloadFormat = (Get-AzureStorageTable -Name $tablename).CloudTable.ServiceClient.PayloadFormat
$RetryPolicy = (Get-AzureStorageQueue -Name $queuename).CloudQueue.ServiceClient.RetryPolicy

# New
$LocationMode = (Get-AzureStorageBlob -Container $containername)[0].ICloudBlob.ServiceClient.DefaultRequestOptions.LocationMode		
$ParallelOperationThreadCount = (Get-AzureStorageContainer -Container $containername).CloudBlobContainer.ServiceClient.DefaultRequestOptions.ParallelOperationThreadCount
$PayloadFormat = (Get-AzureStorageTable -Name $tablename).CloudTable.ServiceClient.DefaultRequestOptions.PayloadFormat
$RetryPolicy = (Get-AzureStorageQueue -Name $queuename).CloudQueue.ServiceClient.DefaultRequestOptions.RetryPolicy
```