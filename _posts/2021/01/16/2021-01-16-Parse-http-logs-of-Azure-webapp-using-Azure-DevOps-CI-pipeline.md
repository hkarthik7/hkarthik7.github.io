---
layout: post
title: Create a CI pipeline to parse http logs of Azure app service
categories: [Azure DevOps, PowerShell, Azure]
tags: [Azure DevOps, PowerShell, Azure, App Service, WebApp]
description: Create a CI pipeline in Azure DevOps and use PowerShell to parse http logs of Azure App service
---

Sometimes we may need to analyse the `http` logs in `Azure WebApp` to know the incoming requests and http error codes. Since the logs files usually are overwritten we tend to miss some of the significant logs. We can run query in **Log analytics** workspace and find out most of the details but what if there aren't any logs that are relevant to the issus that we are looking for is not in log analytics database? this is where we rely on `http` logs. Downloading the logs from **SCM** or **KUDU** and analysing each line is a time consuming task and we may miss few log entries too.

This is a tedious task if we have to do it for multiple subscriptions and more frequently. This is where we need automation and our **PowerShell** magic to get the work done for us. We will walk through the script and setup CI pipeline to download and analyse `http` logs.

## Table Of Contents

1. Script walk through
2. Setup CI pipeline in Azure DevOps
3. Download and view report
4. Closing

### Script walk through

The requirement is divided into two sections,

- Data extraction
- Data analysis and visualisation (precisely report generation)

In the data extraction section we download the logs from SCM API and in data analysis section we analyse, segregate the logs and generate the report.

- In the first section we're fetching the details of web app and publishing profile of that web app, this way we will get the credential to access the SCM API. Once we have necessary details we're calling the API and downloading the present day's log(s).

<script src="https://gist.github.com/hkarthik7/1b6a7d3ed5104b872da5a39f56b8edea.js"></script>

- In data analysis, we're separating out the section that is required. First things first, we're getting all the files from the current path and skipping first two lines as we don't want the headers and extracting the needed contents. This script is intended to run in pipeline once the logs are downloaded. Now we can extract the report and save it as `html` or `csv`.

<script src="https://gist.github.com/hkarthik7/c1ca2e22063b7a0df7e1096447764358.js"></script>

- Generate a html report from the **LogParser** results. We're going to use `PowerShell Proxy function` of `ConvertTo-Html` cmdlet and name it as `Export-Html` and add add in `BootStrap` URI to make the report prettier. 

<script src="https://gist.github.com/hkarthik7/7579bf8e71b9256ad893d9c04494fa9f.js"></script>

- Export the results to report

```powershell
LogParser | Export-Html -FilePath "$($PWD.Path)\Report.html" `
-Head "ERROR CODE DETAILS" `
-PreContent "SERVER/BROWSER ERRORs"
```

- Here is the complete script to incorporate in the pipeline.

<script src="https://gist.github.com/hkarthik7/7ef69d558a6132adf9879b2f84969b05.js"></script>

### Setup CI pipeline in Azure DevOps

- Navigate to Pipelines section and click on **New pipeline**

<img src="/assets/media/LogParser/L1.PNG" alt="L1">

- Click on `Use the classic editor` at the bottom and select the source code repository where you have placed the script (You can opt to create YAML CI also).
In this example I've chosen Azure Repos Git as my source code repository.

<img src="/assets/media/LogParser/L2.PNG" alt="L2">

- Click on `Empty job`

<img src="/assets/media/LogParser/L3.PNG" alt="L3">

- Rename the pipeline of your choice.

<img src="/assets/media/LogParser/L4.PNG" alt="L4">

- As mentioned earlier we're going to divide our requirement into two sections in the pipeline. To achieve this we've to add two tasks
    1. Data extraction (download the logs)
    2. Data analysis and visualisation (Parse and generate report)

    We will add two `Azure Powershell` tasks to separate the concerns.

- First task is to download the logs, so we need to run **Export-AzureWebAppLogFiles** to achieve this. Copy the function to a script file and point the path in the `Azure Powershell` task. Make sure to rename the task, select the appropriate subscription where your webapp is created, script file path and the version of PowerShell you would like to use (set it to latest).

<img src="/assets/media/LogParser/L5.PNG" alt="L5">

- Similarly fill in the details in next `Azure Powershell` which is to parse the logs and generate report

<img src="/assets/media/LogParser/L6.PNG" alt="L6">

- Add `Copy Files` task to copy the downloaded `logs` and generated `html` report to `Staging Directory`.

<img src="/assets/media/LogParser/L7.PNG" alt="L7">

- Once the files are copied to `Staging Directory` we can add one more task to publish it to the artifact to download the report.

<img src="/assets/media/LogParser/L8.PNG" alt="L8">

- Now, click on **Save & queue** to save the changes and trigger the pipeline.

### Download report

Once the pipeline ran successfully, click on the job to view the summary. You can download the report from the published artifact by clicking the `Report.html`.

<img src="/assets/media/LogParser/L9.PNG" alt="L9">

### Closing

There are few ways to download and analyse the logs and one such way is using the `Azure DevOps` CI pipeline, you can also use `Azure Automation Account` to achieve this and place your report in `Storage Account`. Then it can be shared with anyone to download from the blob container or view the report by sharing the URL  after enabling `Static Site Content` for that `Storage Account`. Make sure to tweak the script accordingly if the `http logs` headers are different in your environment. Happy scripting.
