---
layout: post
title: Automate the code deployment in Azure DevOps via REST API in java
categories: [Azure DevOps, Java]
tags: [Azure DevOps, Java]
description: Automate the code deployment of your build and release pipeline in Azure DevOps via it's REST API in java.
---

This article demonstrates how to deploy code programmatically by triggering the build and release pipeline with the help of Azure DevOps REST API in java.
We're going to use **azd** java library and see how easy it is to connect the build and release pipelines.

## Table Of Contents

1. Introduction
2. Explore Api and examples
3. Closing

### Introduction

To call Azure DevOps REST Api we need to understand how different services are defined. CI or build pipelines are defined in **Build** Api and CD or release pipelines are defined in **Release** Api. We would use different Apis to create the build and release which we will see when we explore the examples.

### Explore Api and examples

We create an example program that queue's the build pipeline, waits until that's completed and then trigger the release pipeline. This may sound bit easy but it requires to connect few different Apis to complete the task.

Let's create a connection object and connect to Azure DevOps Api.

```java
String organization = "my-organization";
String project = "my-project";
String token = "personal-access-token";

var webApi = new AzDClientApi(organization, project, token);

// now we have the connection object, let's queue a build.
String buildPipelineName = "Demo-CI";
String releasePipelineName = "Demo-CD";
String releaseEnvName = "DR-Test";

var build = webApi.getBuildApi();
var release = webApi.getReleaseApi();
```

#### Queue a build

To queue a build we need build pipeline/definition Id. We can get the build definition id with it's name and queue the build.

```java
System.out.println("Starting the build pipeline..");

// Queue a build and wait until it completes.
var buildDefId = build.getBuildDefinitions().getBuildDefinitions()
        .stream()
        .filter(x -> x.getName().equals(buildPipelineName))
        .findFirst()
        .get()
        .getId();

var buildQueue = build.queueBuild(buildDefId);
```

Now let's monitor the status of the build pipeline.

```java
while (buildQueue.getStatus() == BuildStatus.NOTSTARTED) {
    var buildObj = build.getBuild(buildQueue.getId());

    if (buildObj.getStatus() == BuildStatus.COMPLETED && buildObj.getResult() == BuildResult.SUCCEEDED) {
        System.out.println("Pipeline status: " + BuildResult.SUCCEEDED.name());
        break;
    }

    if (buildObj.getStatus() == BuildStatus.COMPLETED && buildObj.getResult() == BuildResult.FAILED) {
        System.out.println("Pipeline status: " + BuildResult.FAILED.name());
        break;
    }

    else {
        System.out.println("Pipeline status: " + buildObj.getStatus().name());
        Thread.sleep(30000);
        continue;
    }

}
```

We're looking for the build queue status, if it's not started then checking the status to see if the build pipeline run has succeeded or failed.

#### Run a release pipeline

One the build pipeline is successfully completed. We can then use the artifact of build pipeline to create a release and queue it.

```java
System.out.println("Starting the release pipeline..");

var def = release.getReleaseDefinitions(ReleaseDefinitionExpands.ARTIFACTS)
        .getReleaseDefinitions()
        .stream()
        .filter(x -> x.getName().equals(releasePipelineName))
        .findFirst()
        .get();

var newRelease = release.createRelease(def.getId(), "New test release", def.getArtifacts().get(0).getAlias(),
        buildQueue.getId().toString(), buildPipelineName, false);


var queueRelease = release.queueRelease(newRelease.getId(), releaseEnvName);

System.out.println("Release status: " + queueRelease.getStatus().name());
```

We create a release with the release pipeline id, description for the release, artifact alias which is the alias name of build artifact. It can be a build artifact or git repo etc, build id, name of the build pipeline and if the release should be created in a draft state or not.

When we get the definition id we need to expand the artifacts in the output for us to use when creating a release. If that is not specified, artifacts section will be null.

Once we've all these information we can then queue the release with release id and stage or release environment name.

The final code looks like this.

```java
import org.azd.enums.BuildResult;
import org.azd.enums.BuildStatus;
import org.azd.enums.ReleaseDefinitionExpands;
import org.azd.exceptions.AzDException;
import org.azd.utils.AzDClientApi;

public class Main {
    public static void main(String[] args) {
        String organization = "my-organization";
        String project = "my-project";
        String token = "personal-access-token";

        String buildPipelineName = "Demo-CI";
        String releasePipelineName = "Demo-CD";
        String releaseEnvName = "DR-Test";

        var webApi = new AzDClientApi(organization, project, token);
        var build = webApi.getBuildApi();
        var release = webApi.getReleaseApi();

        try {
            System.out.println("Starting the build pipeline..");

            var buildDefId = build.getBuildDefinitions().getBuildDefinitions()
                    .stream()
                    .filter(x -> x.getName().equals(buildPipelineName))
                    .findFirst()
                    .get()
                    .getId();

            var buildQueue = build.queueBuild(buildDefId);

            while (buildQueue.getStatus() == BuildStatus.NOTSTARTED) {
                var buildObj = build.getBuild(buildQueue.getId());

                if (buildObj.getStatus() == BuildStatus.COMPLETED && buildObj.getResult() == BuildResult.SUCCEEDED) {
                    System.out.println("Pipeline status: " + BuildResult.SUCCEEDED.name());
                    break;
                }

                if (buildObj.getStatus() == BuildStatus.COMPLETED && buildObj.getResult() == BuildResult.FAILED) {
                    System.out.println("Pipeline status: " + BuildResult.FAILED.name());
                    break;
                }

                else {
                    System.out.println("Pipeline status: " + buildObj.getStatus().name());
                    Thread.sleep(30000);
                    continue;
                }
            }

            System.out.println("Starting the release pipeline..");

            var def = release.getReleaseDefinitions(ReleaseDefinitionExpands.ARTIFACTS)
                    .getReleaseDefinitions()
                    .stream()
                    .filter(x -> x.getName().equals(releasePipelineName))
                    .findFirst()
                    .get();

            var newRelease = release.createRelease(def.getId(), "New test release", def.getArtifacts().get(0).getAlias(),
                    buildQueue.getId().toString(), buildPipelineName, false);


            var queueRelease = release.queueRelease(newRelease.getId(), releaseEnvName);

            System.out.println("Release status: " + queueRelease.getStatus().name());

        } catch (AzDException e) {
            e.printStackTrace();
        }
    }
}
```

### Closing

**azd** library has support for all significant Apis and can be used for various automation tasks. Feel free to explore the library and
raise for any concerns/issues [here](https://github.com/hkarthik7/azure-devops-java-sdk/issues).
