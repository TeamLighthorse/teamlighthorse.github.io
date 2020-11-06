---
layout: post
title: DevOps Fundamentals - Azure DevOps Multi-Stage Pipelines
tags: [lighthorse, pipelines, yaml, azure, devops, infrastructure, ci, cd]
excerpt_separator: <!--more-->
---

Effective DevOps processes are built upon a handful of key practices:
- Continuous Integration (CI)
- Continuous Delivery (CD)
- Infrastructure as Code
- Monitoring and Logging
- Communication and Collaboration

Azure DevOps Multi-Stage Pipelines help us to check several of those boxes: CI, CD, and Infrastructure as Code

<!--more-->

# Overview

Multi-Stage Pipelines are defined as a YAML file.  Because a pipeline is just YAML, it can be checked into your
source control system and versioned just like any other source file.  The
[YAML schema](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema#pipeline-structure)
for pipelines is very powerful and can accomodate a variety of CI/CD scenarios.  This post will dig into 3 of the key parts of a YAML pipeline:
- trigger
- variables
- stages

## [trigger](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema#triggers)

The `trigger` section of the YAML pipeline is used to declare what events should trigger the pipeline to execute.  The
most common type of trigger is a
[Push Trigger](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema#push-trigger).
When defining a push trigger, you provide a list of branches from your source code repo.  Whenever a change is pushed
to any of those branches, the pipeline will be executed.

The `trigger` section for a pipeline may look something like:

```yaml
trigger:
- master
- develop
```

In this case, whenever changes are pushed to the `master` or `develop` branches, the pipeline will execute.

## variables

If you find yourself repeating the same value again and again in your YAML, you can create a variable for
the value.  This can help keep your YAML DRY, and help ensure that if you ever need to change a repeated
value, you only need to do it in one place.

You can add variables to your YAML pipeline like:

```yaml
trigger:
- master
- develop
variables:
  dockerFile: src/Example.Web/Dockerfile
  imagePullSecretsName: acr
```

Variables can be referenced in other parts of the YAML pipeline using the `$(VARIABLE_NAME)` syntax.
For example, you might have a step to build a docker file that references the `dockerFile` variable like:

```yaml
- script: |
    docker build -f $(dockerFile) .
```

> The `$(VARIABLE_NAME)` syntax for referencing a variable is known as the *macro syntax* for YAML
> pipelines.  This is the most common syntax for referencing variables, though there are
> 2 other syntaxes that can be used:
>
> - template expressions `${% raw %}{{{% endraw %} variables.VARIABLE_NAME {% raw %}}}{% endraw %}`
> - runtime expressions `$[variables.VARIABLE_NAME]`
>
> The various syntaxes give you additional control around when the variable is expanded (runtime 
> vs compilation).  See
> [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch)
> for more information.

## stages

You may be familiar with many of the above concepts if you have used YAML *builds* in Azure DevOps.
Multi-stage YAML pipelines take that same YAML foundation and extends it with a new building block, `stages`.
By introducing `stages`, you can take an automated build process and turn it into a more complete
CI/CD solution.

`stages` live at the top of a 3 tiered hierarchy.  A pipeline can have 1 or more `stages`.  Each
stage can have one or more `jobs`.  Each job can have one or more `steps`.

The `stages` in your pipeline should reflect the stages in your devops process for implementing
and releasing changes. For our team, the stages in our process are
Build > QA > Staging > Production > Demo.
Every change to our applications goes through these stages on the way to completion.

> What is the difference between `stages` and `jobs`?  A stage can have multiple jobs, though
> it is common for a stage to only have a single job.  One reason you might split a stage into
> multiple jobs is because jobs within a stage can execute concurrently.  For instance, if you
> have multiple independent builds that need to complete to prepare a release (like a front-end
> angular build, and back-end .NET build), you could define
> separate jobs for each build and allow those jobs to execute concurrently.

### Example Build Stage
Take a look at this sample pipeline with a Build stage:

```yaml
trigger:
- master
- develop
variables:
  dockerFile: src/Example.Web/Dockerfile
  imagePullSecretsName: acr
stages:
- stage: Build
  jobs:
  - job: Build
    steps:
    - script: |
        docker build -f $(dockerFile) -t build --target build .
      displayName: 'Build Application'

    - script: |
        docker build -f $(dockerFile) -t test --target test .
      displayName: 'Run Tests'

    - script: |
        docker build -f $(dockerFile) -t commodities:$(Build.SourceBranchName)$(Build.BuildId) --target final .
      displayName: 'Prepare Release'
      condition: ne(variables['Build.SourceBranchName'], 'merge')

    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
        publishLocation: 'Container'
      displayName: 'Push container to registry'
      condition: ne(variables['Build.SourceBranchName'], 'merge')
```

The Build `stage` has a single `job` (also named Build).  The Build `job` consists of 4 `steps` to
build the application, run tests, prepare a release, and push the generated container to a container
registry.

Lets unpack this a little.  The first 3 steps in this job describe `scripts` to run.  These
scripts are executed on the command line of the
[pipeline agent](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops&tabs=browser).

> By default, a pipeline will run in a virtual machine on a
> [Microsoft-hosted agent](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops).
> At the time of this writing, the default VM is based on Ubuntu 16.04 (xenial).  If you want to use a 
> different VM image, you can modify your `job` definition to specify which image you want to use:
>
> ```yaml
>   jobs:
>   - job: Build
>     pool:
>       vmImage: 'windows-2019'
>     steps: ...   
>
> ```
>
> In the above example, Azure Pipelines would run the Build job using a Windows 2019 VM instead of
> Ubuntu.  Knowing what VM your agent is running can be important because different VM images will
> have different tools included.
>
> For example, when running a powershell task from ubuntu 16.04, the
> `Invoke-SqlCmd` cmdlet is not available.  It is available when using the Windows 2019 VM image.
> [This page](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops)
> includes information about which images are available for Microsoft-hosted agents, with links to
> what software is inlcuded in each image.

The last step for the sample Build job runs a
[Built-in task](https://docs.microsoft.com/en-us/azure/devops/pipelines/tasks/?view=azure-devops).
Built-in tasks are available for all sorts of common CI/CD tasks such as packaging/publishing
artifacts and integrating with external services.  In this case the `PublishBuildArtifacts@1` task
is pushing the docker container produced in the prior steps to Azure Container Registry.

> #### Multi-line Scripts
> Using `- script: |` followed by a new line is the syntax for defining multi-line scripts.  It the Build
> example above each script is only one line, so this syntax is just being used for readability.  You could
> rewrite those same steps as follows:
> 
> ```yaml
>     - script: |
>         docker build -f $(dockerFile) -t build --target build .
>         docker build -f $(dockerFile) -t test --target test .
>         docker build -f $(dockerFile) -t commodities:$(Build.SourceBranchName)$(Build.BuildId) --target final .
>       displayName: 'Build test and prepare the release'
>
>     - task: PublishBuildArtifacts@1
>       inputs:
>         PathtoPublish: '$(Build.ArtifactStagingDirectory)'
>         ArtifactName: 'drop'
>         publishLocation: 'Container'
>       displayName: 'Push container to registry'
>       condition: ne(variables['Build.SourceBranchName'], 'merge')
> ```

### Example Release Stage
After successfully building your application, the next thing you will generally want to do is
release it to an environment where you can start testing.  For the release, we will add
a 2nd Stage to the above example:

``` yaml
stages:
- stage: Build
  jobs:
  - job: Build
    steps:
      ...
- stage: QA
  dependsOn: Build
  condition: eq(variables['Build.SourceBranchName'], 'develop')
  jobs:
    ...
```

Here we have added another `stage` under the `stages` section of the pipeline.  You will notice
2 new concepts in the snippet above, `dependsOn` and `condition`.

`dependsOn` allows you to declare that a stage depends on some other stage completing.  Here we
are saying that the QA stage depends on the Build stage, meaning that the QA stage should not
run until after the Build stage has completed successfully.  If the Build stage fails, then we do not want
the QA stage to execute (you would not want to deploy a broken build).

`condition` allows you to control whether a given stage should execute based on some conditional
expression.  In this case, we only want the QA stage to run if the source branch that triggered
this pipeline execution was develop (as opposed to a PR branch, or the master branch). If this
were the master branch, we would instead want to run a different stage to deploy the application
to a staging or production environment.

Next we need to add a job to the QA stage that contains the steps for a release:

``` yaml
...
  - stage: QA
    dependsOn: Build
    condition: eq(variables['Build.SourceBranchName'], 'develop')
    jobs:
    - deployment: QA
      environment: qa
      pool:
        vmImage: 'windows-2019'
      strategy:
        runOnce:
          deploy:
            steps:
              ...
```

Notice how with the Build stage, we added:
``` yaml
jobs:
- job: Build
  steps:
    ...
```

For the release we are adding a different type of job, `deployment`:
``` yaml
jobs:
- deployment: QA
  environment: qa
  pool:
    vmImage: 'windows-2019'
  strategy:
    runOnce:
      deploy:
        steps:
          ...
...
```

[Deployment jobs](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/deployment-jobs?view=azure-devops)
are a special type of job with some extra features.  When a deployment job executes, Azure Pipelines
automatically records a deployment history for you.  Additionally, when defining a deployment job, you
can specify a
[deployment strategy](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/deployment-jobs?view=azure-devops#deployment-strategies).
There are currently 3 strategies available: runOnce, rolling, and canary.

With deployment strategies, Azure Pipelines supports some pretty advanced scenarios, including
pre deployment steps, routing traffic before and after deployments, and steps to take to rollback
in the event of a failure.  In this case, we are going to keep it simple and use a very basic
`runOnce` strategy.

The steps in a deployment can use all the same tasks that you can use in any other job, including
scripts. Just as an example, here is what a deployment to a kubernetes cluster might look like:

``` yaml
- stage: QA
    dependsOn: Build
    condition: eq(variables['Build.SourceBranchName'], 'develop')
    jobs:
    - deployment: QA
      environment: qa
      pool:
        vmImage: 'windows-2019'
      strategy:
        runOnce:
          deploy:
            steps:
            - task: AzureKeyVault@1
              displayName: 'Load secrets from NucleusQAVault'
              inputs:
                azureSubscription: Nucleus
                KeyVaultName: NucleusQAVault
                SecretsFilter: 'NucleusDb,ShipmentsTopicKey,SwaggerClientSecret,ShipmentManagementApiSecret,Smtp--Password,dkim-kelleylogistics'

            - task: KubernetesManifest@0
              displayName: 'Bake helm chart'
              name: 'bake'
              inputs:
                action: bake
                helmChart: '$(Pipeline.Workspace)/drop/helm'
                overrides: |
                  appName:$(containerRegistryRepo)
                  imageTag:$(Build.SourceBranchName)$(Build.BuildId)
                  namespace:lighthouse
                  containerRegistryHostName:$(containerRegistryHostName)
                  environmentName:Qa
                  processScheduledTemplatesApiKey:$(ShipmentManagementApiSecret)

            - task: KubernetesManifest@0
              displayName: 'Deploy manifests'
              inputs:
                action: deploy
                kubernetesServiceConnection: Lighthouse-Cluster-Qa
                namespace: lighthouse
                manifests: '$(bake.manifestsBundle)'
                imagePullSecrets: |
                  $(containerRegistryRepo)-imagepullsecret
```

In a typical CI/CD process, following the Build stage, you would have one or more release stages
to deploy your application to various environments (QA, Staging, Production, Demo).  Ideally, the
release process to each environment is going to be very similar.  To avoid copying the same
steps over and over, we can leverage another YAML Pipelines feature, Templates.

### [Templates](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops)
Templates give us a way to re-use sections of a YAML pipeline.  A template can contain a list of stages,
jobs, and/or steps.  A template can also accept parameters.  The `job` that deploys an application is
usually a good candidate for templating.  Generally speaking, the steps to release the applcication
will be the same.  The differences might be things like connections strings and API keys that vary
based on the target environment.

Lets try to convert the example `job` above to a template.

The first section of a template is the `parameters`.

``` yaml
parameters:
- name: 'environment'
  type: string
- name: 'keyVaultName'
  type: string
- name: 'namespace'
  type: string
  default: lighthouse
- name: 'kubernetesServiceConnection'
  type: string
```

A parameter has a name and a
[type](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops#parameter-data-types).
The type is used to help validate the parameter value passed into a template whenever the template is used.  A parameter can also have
a default value (like the namespace parameter above).

In this case, I have identified the values from the QA deployment job example that would need to change when deploying to
different environments, defining parameters for each of those values.

There is a special syntax for referencing a parameter within a template, `${{ parameters.parameterName }}`.  Using this syntax,
here is how you would convert the QA deployment job above to a template with parameters:

``` yaml
parameters:
- name: 'environment'
  type: string
- name: 'keyVaultName'
  type: string
- name: 'namespace'
  type: string
- name: 'kubernetesServiceConnection'
  type: string
jobs:
  - deployment: ${{ parameters.environment }}
    environment: ${{ parameters.environment }}
    pool:
      vmImage: 'windows-2019'
    strategy:
      runOnce:
        deploy:
          steps:
          - task: AzureKeyVault@1
            displayName: 'Load secrets from ${{ parameters.keyVaultName }}'
            inputs:
              azureSubscription: Nucleus
              KeyVaultName: ${{ parameters.keyVaultName }}
              SecretsFilter: 'NucleusDb,ShipmentsTopicKey,SwaggerClientSecret,ShipmentManagementApiSecret,Smtp--Password,dkim-kelleylogistics'

          - task: KubernetesManifest@0
            displayName: 'Bake helm chart'
            name: 'bake'
            inputs:
              action: bake
              helmChart: '$(Pipeline.Workspace)/drop/helm'
              overrides: |
                appName:$(containerRegistryRepo)
                imageTag:$(Build.SourceBranchName)$(Build.BuildId)
                namespace:${{ parameters.namespace }}
                containerRegistryHostName:$(containerRegistryHostName)
                environmentName:Qa
                processScheduledTemplatesApiKey:$(ShipmentManagementApiSecret)

          - task: KubernetesManifest@0
            displayName: 'Deploy manifests'
            inputs:
              action: deploy
              kubernetesServiceConnection: ${{ parameters.kubernetesServiceConnection }}
              namespace: ${{ parameters.namespace }}
              manifests: '$(bake.manifestsBundle)'
              imagePullSecrets: |
                $(containerRegistryRepo)-imagepullsecret
```
