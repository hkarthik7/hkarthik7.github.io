---
layout: post
title: Meet azd - a java library to manage Azure DevOps API
categories: [Azure DevOps]
tags: [Java, Azure DevOps]
description: Java library for managing Azure DevOps REST API
---

Working with APIs has never been easier without client libraries. `Postman` is the saviour for most of the API calls, however a client library is required for automation, to include it as dependencies in our project and work on ease with a vast REST API. **Azure DevOps** REST API is no exemption due to its vastness.

In order to make working with **Azure DevOps** API on ease and fun in `java`,  **azd** is developed. It covers most significant and used services in **Azure DevOps** REST API and gets our work done in few lines of code.

In this article let's explore the library and unwind what it provides out of box.

## Table of Contents

1. **Installation**
2. **Getting Started**
3. **Examples**
4. **Closing**

### Installation

You can install the library from [github release page](https://github.com/hkarthik7/azure-devops-java-sdk).

It is easy to include in your `maven` project too. You can specify the latest version in your pom.xml file when you include `azd` in your project.

```xml
<groupId>io.github.hkarthik7</groupId>
<artifactId>azd</artifactId>
<version>${latest version}</version>
```

Add java docs

```xml
<groupId>io.github.hkarthik7</groupId>
<artifactId>azd</artifactId>
<version>${latest version}</version>
<classifier>javadoc</classifier>
```

### Getting Started

Before getting started you are going to need `personal access token` to authenticate with **Azure DevOps** REST API. You can learn on how to get your `personal access token` from the [official documentation](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page).

Provided that you have your `personal access token` with you by now, let's explore the usage of the library.

**azd** depends on `com.fasterxml.jackson.core` to output the API response as `java` object.

### Examples

- Before calling any methods in the library you should create a connection object using `Connection` class with `organization name`, `project name` and `personal access token`. However `project name` is not mandatory and you can always set it whenever required.

Setting these as defaults allows you to pass it's instances to all the classes you call in the library.

- Get all projects for your organization

```java
public class Main {
    public static void main(String[] args) {
        String organization = "myOrganizationName";
        String personalAccessToken = "accessToken";

        // Create a Connection object with organization name, project and personal access token.
        var connection = new Connection(organization, personalAccessToken);

        // call API with the default parameters;
        CoreApi core = new CoreApi(connection);
        try {
            // get the list of projects
            core.getProjects();

            // create a new project
            core.createProject("my-new-project", "Finance management");

            // create a team in the project
            core.createTeam("my-new-project", "my-new-team");

            // list all the teams
            core.getTeams();
        } catch (ConnectionException | AzDException e1) {
            e1.printStackTrace();
        }
    }

}
```

- Work with Feed Management API. You can omit the project name for calling Feed Management API as it is not a mandatory parameter. If you do omit the project name you will end up working with feeds across your organization. Provide the project name if you want to work with feeds scoped to a project.

```java
public class Main {
    public static void main(String[] args) {
        String organization = "myOrganizationName";
        String project = "myProject";
        String personalAccessToken = "accessToken";

        // Create a Connection object with organization name, project and personal access token.
        var connection = new Connection(organization, project, personalAccessToken);

        // call API with the default parameters;
        FeedManagementApi feedManagement = new FeedManagementApi(connection);
        try {
            // create new feed
            feedManagement.createFeed("myFeed", "To store maven packages", true, true);

            // create feed view
            feedManagement.createFeedView("myFeed", "myFeedView", "release", "private");

            // delete a feed
            feedManagement.deleteFeed("myUnwantedFeed");

            // delete a feed view in a particular feed
            feedManagement.deleteFeedView("myUnwantedFeed", "myUnwantedFeedView");

            // get a feed by name; This returns a feed object where you can get a particular value from it.
            feedManagement.getFeed("myFeed");
            feedManagement.getFeed("myFeed").getId();
            feedManagement.getFeed("myFeed").getDescription();
            feedManagement.getFeed("myFeed").getUrl();
            feedManagement.getFeed("myFeed").getUpstreamSources();

            // get a feed with optional parameters
            feedManagement.getFeed("myFeed", true);

            // get the permissions for a feed
            feedManagement.getFeedPermissions("myFeed");

            // list all the views in a feed
            feedManagement.getFeedViews("myFeed");

            // list all the feeds; if you don't set project name list of feeds will be retrieved which are scoped to organization
            feedManagement.getFeeds().getFeed().stream().forEach(System.out::println);

            // update the existing feed;
            feedManagement.updateFeed("myFeed", true, "My new feed", true, true);
        }
        catch (ConnectionException | AzDException e1) {
            e1.printStackTrace();
        }
    }
}

```

- Call build API.

```java
public class Main {
    public static void main(String[] args) {
        String organization = "myOrganizationName";
        String project = "myProject";
        String personalAccessToken = "accessToken";

        // Create a Connection object with organization name, project and personal access token.
        var connection = new Connection(organization, project, personalAccessToken);

        // call API with the default parameters;
        BuildApi build = new BuildApi(connection);
        try {

            // delete a build by id
            build.deleteBuild(25);

            // delete a pipeline/definition by definition id
            build.deleteBuildDefinition(9);

            // get a build by Id
            build.getBuild(20);

            // list the build controllers
            build.getBuildControllers();

            // list the build definitions
            build.getBuildDefinitions().getBuildDefinition().stream().forEach(System.out::println);

            // get the logs for a specific build
            build.getBuildLog(20, 3);

            // get all work items associated to a build
            build.getBuildWorkItems(22);

            // list all builds and filter a particular id
            build.getBuilds().getBuildResults().stream().filter(id -> id.getId() == 22).forEach(System.out::println);

            // queue a build with pipeline/definition id
            build.queueBuild(12);

        } catch (ConnectionException | AzDException e) {
            e.printStackTrace();
        }
    }
}
```

The library has more useful functions, give it a try and explore the library.

### Closing

**azd** is a open source project and you can find the source code in [github](https://github.com/hkarthik7/azure-devops-java-sdk).

- Issues/bugs can be reported [here](https://github.com/hkarthik7/azure-devops-java-sdk/blob/main/.github/ISSUE_TEMPLATE.md).

- Contributions are welcome, goal of this project is to make it more reliable and useful for everyone. See [contribution guidelines](https://github.com/hkarthik7/azure-devops-java-sdk/blob/main/.github/CONTRIBUTING.md) for more details.
