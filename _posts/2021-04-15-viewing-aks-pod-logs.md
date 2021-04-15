---
layout: post
title: How to View Logs of Pods Running in AKS
tags: [aks, kubernetes, logs, troubleshooting]
excerpt_separator: <!--more-->
author: footedr
---
This post will show you how to pull Lighthouse container logs from pods running in our Azure Kubernetes Service clusters (AKS).

<!--more-->

### Login / Credentials
To be able to pull logs from pods running in AKS, you will first have to authenticate to Azure using your Dayton Freight Lines domain credentials. We will be using the command line and `kubectl` to interact with AKS, so to authenticate to Azure, we need to make sure we have the Azure Command Line Interface (Azure CLI) installed. You can download the Azure CLI [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

Once you have the Azure CLI installed (and if you have a command line session open, you'll have to restart it), you can log into Azure by using the `az login` command. This should pop open a new browser tab asking you to login with your school or work account. Log in with your Dayton Freight username/password. Once you have logged in on that browser tab, you should see some JSON spit out in your command line window. The JSON will contain your Azure account info. For some `az` commands, you'll be asked to specify a subscription, you can use the value of the "id" property of the "Nucleus (TMS)" node returned in the JSON to specify the subscription in commands that require it. Or alternatively, you can use the command `az account set -s SUBSCRIPTION GUID` to specify it for the session.

Once you have logged in, you can request credentials to the AKS cluster for which you would like to view logs. You do this by executing the following command: `az aks get-credentials -g RESOURCE-GROUP-NAME -n AKS-CLUSTER-NAME`.

***For Qa:***
`az aks get-credentials -g lighthouse-core-qa -n lighthouse-cluster-qa

***For Staging & Demo:***
`az aks get-credentials -g lighthouse-core-stag -n lighthouse-cluster-stag

***For Production:***
`az aks get-credentials -g lighthouse-core-prod -n lighthouse-cluster-prod

**Access Pod Logs**
Once you have credentials for accessing the AKS cluster, you can pull logs by executing the following command: `kubectl logs PODNAME -n NAMESPACE --timestamps=true`. This command requires you to have Docker Desktop installed and Kubernetes running locally.

You will need the name of a pod. You can get one by doing `kubectl get pods -n NAMESPACE`. For all environments other than demo, use `lighthouse` as the namespace. For demo, use `lighthouse-demo`. 

This command will return a list of running pods in the specified namespace. The pod names are pretty self explanatory. The names all start with the deployment name, followed by some junk. The deployment names should match closely to the logical name of the service in question. i.e. "billoflading", "nucleus", "accounting", "billing", etc.

Find the pod you want, highlight the name with your mouse and hit [ctrl] + [enter] to copy the name onto the clipboard.

Then type `kubectl logs` and [ctrl] + [v] to paste the name of the pod, followed by... ` -n lighthouse --timestamps=true`. 

Obviously, substitute `lighthouse-demo` if you're targeting the demo environment on the Staging cluster.

Ex.
`kubectl logs billing-64b55779cb-crmwg -n lighthouse --timestamps=true`

**#neversayneigh**