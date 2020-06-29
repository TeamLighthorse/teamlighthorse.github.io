---
layout: post
title: DevOps Fundamentals - Azure DevOps Multi-Stage Pipelines
tags: [lighthorse, pipelines, yaml, azure, devops, infrastructure, ci, cd]
excerpt_separator: <!--more-->
---

Effective DevOps processes are built upon a handful of key practices:
- Continuous Integration (CI)
- Continuous Delivery (CD)
- Microservices
- Infrastructure as Code
- Monitoring and Logging
- Communication and Collaboration

Azure DevOps Multi-Stage Pipelines help us to check several of those boxes: CI, CD, and Infrastructure as Code

<!--more-->

# Overview

Multi-Stage Pipelines are defined as a YAML file.  Because a pipeline is just YAML, it can be checked into your
source control system and versioned just like any other source file.  The
[YAML schema](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema#pipeline-structure)
for pipelines is very flexible, so can accomodate a variety of CI/CD scenarios.  This post will focus on what
I feel are the 3 most important sections of a YAML pipeline for CI/CD tasks.
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
Multi-stage YAML pipelines take that same YAML foundation and add a new section to it, `stages`.
By introducing `stages`, you can take a automated build process and turn it into a more complete
CI/CD solution.

`stages` live at the top of a 3 tiered hierarchy.  A pipeline can have 1 or more `stages`.  Each
stage can have one or more `jobs`.  Each job can have one or more `steps`.

The `stages` in your pipeline should reflect the stages in your devops process for implementing
and releasing changes. In our case, the stages in our process are
Build > QA > Staging > Production > Demo.
Every change to our applications goes through these stages on the way to completion.

> What is the difference between `stages` and `jobs`?  A stage can have multiple jobs, though
> it is common for a stage to only have a single job.  One reason you might split a stage into
> multiple jobs is because jobs within a stage can execute concurrently.  For instance, if you
> have multiple independent builds that need to complete to prepare a release, you could define
> separate jobs for each build and allow those jobs to execute concurrently.

### Example Stage
Extending the sample YAML file above to add a Build stage might look like:

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

This Build `stage` has a single `job` (also named Build).  The Build `job` consists of 4 `steps` to
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
artifacts and integrating with external services.

### Multi-line Scripts
Using `- script: |` followed by a new line is the syntax for defining multi-line scripts.  It this case
each script is only one line, so it is just being used for readability.  As an example,  you could
rewrite the Build job above as follows:

```yaml
    - script: |
        docker build -f $(dockerFile) -t build --target build .
        docker build -f $(dockerFile) -t test --target test .
        docker build -f $(dockerFile) -t commodities:$(Build.SourceBranchName)$(Build.BuildId) --target final .
      displayName: 'Build test and prepare the release'

    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
        publishLocation: 'Container'
      displayName: 'Push container to registry'
      condition: ne(variables['Build.SourceBranchName'], 'merge')
```

```yaml
...
- script: |
        docker build -f $(dockerFile) -t commodities:<i>$(Build.SourceBranchName)$(Build.BuildId)</i> --target final .
...
```