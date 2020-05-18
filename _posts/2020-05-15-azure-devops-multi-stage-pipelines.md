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

Azure DevOps Multi-Stage Pipelines help us to check several of those boxes: CI/CD and Infrastructure as Code

<!--more-->

# Overview

Multi-Stage Pipelines are defined as a YAML file.  Since your pipeline is a YAML file, it can be checked in to your
source control system and versioned just like any other code file.  The
[YAML schema](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema#pipeline-structure)
for pipelines is very flexible, and can be used to perform all sorts of advanced tasks.  I am going to hone in on what
I feel are the 3 most important sections of a YAML pipeline for CI/CD tasks.
- trigger
- variables
- stages

## [trigger](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema#triggers)

The `trigger` section of the YAML pipeline is used to specify what actions should trigger the pipeline to execute.  The
most common type of trigger is a
[Push Trigger](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema#push-trigger).
When defining a push trigger, you provide a list of branches from your source code repo.  Whenever changes are pushed
to any of those branches, the pipeline will be executed.

The `trigger` section for a pipeline will often look like:

```yaml
trigger:
- master
- develop
```

In this case, whenever changes are pushed to the `master` or `develop` branches, the pipeline will execute.

## variables

Variables can be used to help keep your YAML pipelines DRY.  If you find yourself repeating the same value again
and again, you can create a variable for the value.  This helps ensure that if you ever need to change that
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
> pipelines.  This is probably the most common syntax for referencing variables, though there are
> actually 2 other syntaxes that can be used:
>
> - template expressions `${% raw %}{{{% endraw %} variables.VARIABLE_NAME {% raw %}}}{% endraw %}`
> - runtime expressions `$[variables.VARIABLE_NAME]`
>
> The various syntaxes give you additional control around when the variable is expanded (runtime 
> vs compilation).  See
> [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/variables?view=azure-devops&tabs=yaml%2Cbatch)
> for more information.

## stages

If you are familiar with the YAML build offering in Azure DevOps, none of the above is new.  The
addition of the `stages` section takes that YAML build foundation and extends it to a more complete
CI/CD solution.

`stages` live at the top of a 3 tiered hierarchy.  A pipeline can have 1 or more `stages`.  Each
stage can have one or more `jobs`.  Each job can have one or more `steps`.  The `stages` in your
pipeline should reflect the stages in your devops process for implementing and releasing changes.
In our case, the stages in our process are Build -> QA -> Staging -> Production -> Demo.  Every
change to our applications goes through these stages on the way to release.

> What is the point of having `stages` and `jobs`?  A `stage` can have multiple `jobs`, though
> it is common for a stage to only have a single job.  One reason you might split a stage into
> multiple jobs is because jobs within a stage can execute concurrently.  For instance, if you
> have multiple independent builds that need to complete to prepare a release, you could define
> separate jobs for each build and allow those jobs to execute concurrently.

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
      displayName: 'Run build stage from dockerfile.'

    - script: |
        docker build -f $(dockerFile) -t test --target test .
      displayName: 'Run test stage from dockerfile.'

    - script: |
        docker build -f $(dockerFile) -t commodities:$(Build.SourceBranchName)$(Build.BuildId) --target final .
      displayName: 'Publish website.'
      condition: ne(variables['Build.SourceBranchName'], 'merge')
```

The Build `stage` has a single `job`, also called Build.  The Build `job` consists of 3 `steps` to build, test,
and publish the website for this project.  The specifics of your steps will vary across projects.  There are
a couple of Azure Pipeline details in this example that are worth calling out.

```yaml
...
- script: |
        docker build -f $(dockerFile) -t commodities:<i>$(Build.SourceBranchName)$(Build.BuildId)</i> --target final .
...
```