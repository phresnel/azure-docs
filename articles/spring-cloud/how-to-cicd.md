---
title: Automate application deployments to Azure Spring Cloud
description: Describes how to use the Azure Spring Cloud task for Azure Pipelines.
author: karlerickson
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 09/13/2021
ms.author: karler
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
---

# Automate application deployments to Azure Spring Cloud

This article shows you how to use the [Azure Spring Cloud task for Azure Pipelines](/azure/devops/pipelines/tasks/deploy/azure-spring-cloud) to deploy applications.

Continuous integration and continuous delivery tools let you quickly deploy updates to existing applications with minimal effort and risk. Azure DevOps helps you organize and control these key jobs. 

The following video describes end-to-end automation using tools of your choice, including Azure Pipelines.

<br>

> [!VIDEO https://www.youtube.com/embed/D2cfXAbUwDc?list=PLPeZXlCR7ew8LlhnSH63KcM0XhMKxT1k_]

::: zone pivot="programming-language-csharp"

## Create an Azure Resource Manager service connection

First, create an Azure Resource Manager service connection to your Azure DevOps project. For instructions, see [Connect to Microsoft Azure](/azure/devops/pipelines/library/connect-to-azure). Be sure to select the same subscription you're using for your Azure Spring Cloud service instance.

## Build and deploy apps

You can now build and deploy your projects using a series of tasks. The following Azure Pipelines template defines variables, a .NET Core task to build the application, and an Azure Spring Cloud task to deploy the application.

```yaml
variables:
  workingDirectory: './steeltoe-sample'
  planetMainEntry: 'Microsoft.Azure.SpringCloud.Sample.PlanetWeatherProvider.dll'
  solarMainEntry: 'Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.dll'
  planetAppName: 'planet-weather-provider'
  solarAppName: 'solar-system-weather'
  serviceName: '<your service name>'

steps:
# Restore, build, publish and package the zipped planet app
- task: DotNetCoreCLI@2
  inputs:
    command: 'publish'
    publishWebProjects: false
    arguments: '--configuration Release'
    zipAfterPublish: false
    modifyOutputPath: false
    workingDirectory: $(workingDirectory)

# Deploy the planet app
- task: AzureSpringCloud@0
  inputs:
    azureSubscription: '<Service Connection Name>'
    Action: 'Deploy'
    AzureSpringCloud: $(serviceName)
    AppName: 'testapp'
    UseStagingDeployment: false
    DeploymentName: 'default'
    Package: $(workingDirectory)/src/$(planetAppName)/publish-deploy-planet.zip
    RuntimeVersion: 'NetCore_31'
    DotNetCoreMainEntryPath: $(planetMainEntry)

# Deploy the solar app
- task: AzureSpringCloud@0
  inputs:
    azureSubscription: '<Service Connection Name>'
    Action: 'Deploy'
    AzureSpringCloud: $(serviceName)
    AppName: 'testapp'
    UseStagingDeployment: false
    DeploymentName: 'default'
    Package: $(workingDirectory)/src/$(solarAppName)/publish-deploy-solar.zip
    RuntimeVersion: 'NetCore_31'
    DotNetCoreMainEntryPath: $(solarMainEntry)
```

::: zone-end
::: zone pivot="programming-language-java"

## Set up an Azure Spring Cloud instance and an Azure DevOps project

First, use the following steps to set up an existing Azure Spring Cloud instance for use with Azure DevOps.

1. Go to your Azure Spring Cloud instance, then create a new app. 
1. Go to the Azure DevOps portal, then create a new project under your chosen organization. If you don't have an Azure DevOps organization, you can create one for free.
1. Select **Repos**, then import the [Spring Boot demo code](https://github.com/spring-guides/gs-spring-boot) to the repository.

## Create an Azure Resource Manager service connection

Next, create an Azure Resource Manager service connection to your Azure DevOps project. For instructions, see [Connect to Microsoft Azure](/azure/devops/pipelines/library/connect-to-azure). Be sure to select the same subscription you're using for your Azure Spring Cloud service instance.

## Build and deploy apps

You can now build and deploy your projects using a series of tasks. The following sections show you various options for deploying your app using Azure DevOps.

### Deploy using a pipeline

To deploy using a pipeline, follow these steps:

1. Select **Pipelines**, then create a new pipeline with a Maven template.
1. Edit the *azure-pipelines.yml* file to set the `mavenPomFile` field to *'complete/pom.xml'*.
1. Select **Show assistant** on the right side, then select the **Azure Spring Cloud** template.
1. Select the service connection you created for your Azure Subscription, then select your Spring Cloud instance and app instance. 
1. Disable **Use Staging Deployment**.
1. Set **Package or folder** to *complete/target/spring-boot-complete-0.0.1-SNAPSHOT.jar*.
1. Select **Add** to add this task to your pipeline.
  
   Your pipeline settings should match the following image.

   :::image type="content" source="media/spring-cloud-how-to-cicd/pipeline-task-setting.jpg" alt-text="Screenshot of pipeline settings." lightbox="media/spring-cloud-how-to-cicd/pipeline-task-setting.jpg":::

   You can also build and deploy your projects using following pipeline template. This example first defines a Maven task to build the application, followed by a second task that deploys the JAR file using the Azure Spring Cloud task for Azure Pipelines.

   ```yaml
   steps:
   - task: Maven@3
     inputs:
       mavenPomFile: 'complete/pom.xml'
   - task: AzureSpringCloud@0
     inputs:
       azureSubscription: '<your service connection name>'
       Action: 'Deploy'
       AzureSpringCloud: <your Azure Spring Cloud service>
       AppName: <app-name>
       UseStagingDeployment: false
       DeploymentName: 'default'
       Package: ./target/your-result-jar.jar
   ```

3. Select **Save and run**, then wait for job to finish.

### Blue-green deployments

The deployment shown in the previous section receives application traffic immediately upon deployment. This enables you to test the application in the production environment before it receives any customer traffic.

#### Edit the pipeline file

To build the application the same way as shown previously and deploy it to a staging deployment, use the following template. In this example, the staging deployment must already exist. For an alternative approach, see [Blue-green deployment strategies](concepts-blue-green-deployment-strategies.md).

```yaml
steps:
- task: Maven@3
  inputs:
    mavenPomFile: 'pom.xml'
- task: AzureSpringCloud@0
  inputs:
    azureSubscription: '<your service connection name>'
    Action: 'Deploy'
    AzureSpringCloud: <your Azure Spring Cloud service>
    AppName: <app-name>
    UseStagingDeployment: true
    Package: ./target/your-result-jar.jar
- task: AzureSpringCloud@0
  inputs:
    azureSubscription: '<your service connection name>'
    Action: 'Set Production'
    AzureSpringCloud: <your Azure Spring Cloud service>
    AppName: <app-name>
    UseStagingDeployment: true
```

#### Use the Releases section

The following steps show you how to enable a blue-green deployment from the **Releases** section.

1. Select **Pipelines** and create a new pipeline for your Maven build and publish artifact.
   1. Select **Azure Repos Git** for your code location.
   1. Select a repository where your code is located.
   1. Select the **Maven** template and modify the file to set the `mavenPomFile` field to *`complete/pom.xml`*.
   1. Select **Show assistant** on the right side and select the **Publish build artifacts** template.
   1. Set **Path to publish** to *complete/target/spring-boot-complete-0.0.1-SNAPSHOT.jar*.
   1. Select **Save and run**.

1. Select **Releases**, then **Create release**.
1. Add a new pipeline, and select **Empty job** to create a job.
1. Under **Stages** select the line **1 job, 0 task**

   :::image type="content" source="media/spring-cloud-how-to-cicd/create-new-job.jpg" alt-text="Screenshot of where to select to add a task to a job.":::

   1. Select the **+** to add a task to the job.
   1. Search for the **Azure Spring Cloud** template, then select **Add** to add the task to the job.
   1. Select **Azure Spring Cloud Deploy:** to edit the task.
   1. Fill this task with your app's information, then disable **Use Staging Deployment**.
   1. Enable **Create a new staging deployment if one does not exist**, then enter a name in **Deployment**.
   1. Select **Save** to save this task.
   1. Select **OK**.
1. Select **Pipeline**, then select **Add an artifact**.
   1. Under **Source (build pipeline)** select the pipeline created previously.
   1. Select **Add**, then **Save**.
1. Select **1 job, 1 task** under **Stages**.
1. Navigate to the **Azure Spring Cloud Deploy** task in **Stage 1**, then select the ellipsis next to **Package or folder**.
1. Select *spring-boot-complete-0.0.1-SNAPSHOT.jar* in the dialog, then select **OK**.

   :::image type="content" source="media/spring-cloud-how-to-cicd/change-artifact-path.jpg" alt-text="Screenshot of the 'Select a file or folder' dialog box.":::

1. Select the **+** to add another **Azure Spring Cloud** task to the job.
2. Change the action to **Set Production Deployment**.
3. Select **Save**, then **Create release** to automatically start the deployment. 

To verify your app's current release status, select **View release**. After this task is finished, visit the Azure portal to verify your app status.

### Deploy from source

To deploy directly to Azure without a separate build step, use the following pipeline template.

```yaml
- task: AzureSpringCloud@0
  inputs:
    azureSubscription: '<your service connection name>'
    Action: 'Deploy'
    AzureSpringCloud: <your Azure Spring Cloud service>
    AppName: <app-name>
    UseStagingDeployment: false
    DeploymentName: 'default'
    Package: $(Build.SourcesDirectory)
```

::: zone-end

## Next steps

* [Quickstart: Deploy your first Azure Spring Cloud application](./quickstart.md)
