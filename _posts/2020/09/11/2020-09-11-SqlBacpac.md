---
layout: post
title: Scan and delete old Sql .bacpac files in Azure Storage Account automatically via Azure DevOps pipeline
categories: [Azure DevOps]
tags: [Azure, PowerShell, Azure DevOps]
description: Scan and delete old Sql .bacpac files in Azure Storage Account automatically via Azure DevOps pipeline to save cost.
---

In this post we will see on how to scan the older Sql `.bacpac` files in **Azure Storage Account** and delete it if it reaches a certain retention period via **Azure DevOps** pipeline.
We use PowerShell script which does all the heavy lifting for us and schedule it in **Azure DevOps** with few steps. We will use **Azure DevOps** classic editor to list the tasks and run the PowerShell script.

## Table Of Contents

- Script Walk through
- Setup Release Pipeline
- Conclusion

### Script Walk Through

Let's call the PowerShell function/script as `Remove-AzureSqlBacPacFiles` which takes an argument for parameter `$RetentionPeriod`. You can pass the date of `.bacpac` files you want to delete in number of days to `$RetentionPeriod` variable.

Let's break the scanning process into smaller steps and pen it in the script.

- Specify the retention period (This decides on how many days older files has to be removed)

```powershell
param (
    [Parameter(Mandatory = $false, Position = 0, ValueFromPipeline = $true, ValueFromPipelineByPropertyName = $true)]
    [ValidateRange(1,180)]
    [int] $RetentionPeriod = 15
)
```

- List all the storage accounts and iterate through it

```powershell
$storageAccounts = Get-AzStorageAccount
```

- Get storage account `SAS` key and access the containers

```powershell
$storageAccountKey = (Get-AzStorageAccountKey `
                    -ResourceGroupName $storageAccount.ResourceGroupName `
                    -Name $storageAccount.StorageAccountName).Value[0]

$context = New-AzStorageContext `
            -StorageAccountName $storageAccount.StorageAccountName `
            -StorageAccountKey  $storageAccountKey

$containers = Get-AzStorageContainer -Context $context
```

- Scan for `.bacpac` files in all the available containers and remove the files which are older than retention period

```powershell
foreach ($container in $containers) {
    Write-Verbose "[$(Get-Date -Format s)] : $functionName : Working with Storage Account Container $($container.Name).."
    
    $blobs = Get-AzStorageBlob -Container $container.Name -Context $container.Context | Where-Object { $_.Name -like "*.bacpac" }

    if ($blobs) {

        foreach ($blob in $blobs) {
            
            if ((Get-Date "$($blob.LastModified.ToString().Split()[0]) $($blob.LastModified.ToString().Split()[1])").Date -le (Get-Date).AddDays(-$RetentionPeriod).Date) {

                Write-Verbose "[$(Get-Date -Format s)] : $functionName : Deleting old bacpac file $($blob.Name).."

                Remove-AzStorageBlob -Blob $blob.Name -Container $blob.ICloudBlob.Container.Name -Context $context -Force
            }
        }
    }
}
```

- Complete script here

<script src="https://gist.github.com/hkarthik7/ca96a4adb056b1c42112472b578aa158.js"></script>

We have enabled verbose so that we can see the script flow in **Azure DevOps** logs.

### Setup Release Pipeline

Now it's time to setup the release pipeline and schedule the script. 

- Navigate to **Releases** in **Azure DevOps** pipelines and click on `New` and select `New release pipeline`, you will see a window similar as below.

<img src="/assets/media/Sqlbacpac/Sql1.PNG" alt="Sql1">

- Click on `Empty Job` and name the environment in which you need to implement the action. For this post let's name the stage as Dev.

<img src="/assets/media/Sqlbacpac/Sql2.PNG" alt="Sql2">

- Now that we have to schedule the release pipeline and enable trigger settings for the stage. This will automatically create a release at scheduled time and trigger's the stage/s enabled. Rename the pipeline of your choice, for this post let's keep it as `RemoveSqlBacPacFiles-CD`.

<img src="/assets/media/Sqlbacpac/Sql3.PNG" alt="Sql3">

- Say that our pipeline should run on every Monday morning 7.30 AM, we have to enable below settings in schedule

<img src="/assets/media/Sqlbacpac/Sql4.PNG" alt="Sql4">

- Now let's enable the release trigger with 5 minutes delay, this gives our stage some buffer time to prepare once the release is created.

<img src="/assets/media/Sqlbacpac/Sql5.PNG" alt="Sql5">

- Click on tasks in `Dev` stage and click on `+` in `Agent Job` tab to add **Azure PowerShell** task.

<img src="/assets/media/Sqlbacpac/Sql6.PNG" alt="Sql6">

- Select `Inline Script` in the task and paste the script. You can also upload the script in `Azure Repos` and refer the path of script in the task.

<img src="/assets/media/Sqlbacpac/Sql7.PNG" alt="Sql7">

- Click on `Variables` tab and add your `Subscription` name and `RetentionPeriod` parameter there. You can refer the screen shot above to know on how to refer pipeline variables.
This way we can change the retention period whenever necessary without modifying the script. You can also save the variables in library and link it in the variable groups.

<img src="/assets/media/Sqlbacpac/Sql8.PNG" alt="Sql8">

- Click on `Save` and `Create Release`. Now that our task is ready to deploy. Click on `deploy` to test the pipeline.

<img src="/assets/media/Sqlbacpac/Sql9.PNG" alt="Sql9">

- On successful execution you will see as below.

<img src="/assets/media/Sqlbacpac/Sql10.PNG" alt="Sql10">

### Conclusion

You can add multiple stages for each subscription, each environment and schedule it. This way you can remove all the older `.bacpac` files which are lying in the storage account and ultimately save some cost.