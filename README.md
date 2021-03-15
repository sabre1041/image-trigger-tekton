#  image-trigger-tekton

Triggering [Tekton](https://tekton.dev/) Pipelines as a result of changes of Image resources and delivered through the use of [Knative](https://knative.dev/) Eventing

## Overview

The contents of this repository demonstrates how to use a Knative [ApiServerSource](https://knative.dev/docs/eventing/sources/apiserversource/) to trigger Tekton Pipelines when changes of OpenShift image events are received.

An overview of the architecture can be found below:

![Overall Architecture](/images/architecture.png)

## Prerequisites

The following prerequisites must be satisfied prior to deploying the solution contained within this repository

1. OpenShift and Tekton Command Line Interfaces
2. Access to an OpenShift Cluster with `cluster-admin` privileges
3. Installation of OpenShift Pipelines (Tekton)
4. Installation of OpenShift Serverless (Knative)
5. Installation of Knative Eventing

## Installation

1. Create a new namespace called `image-watcher` and change the current content into the newly created namespace

```shell
kubectl apply -f namespace/image-watcher-namespace.yml
kubectl config set-context --current --namespace=image-watcher
```

2. Apply Tekton resources

```shell
kubectl apply -f pipelines
```

3. Apply Knative resources

```shell
kubectl apply -f knative
```

## Validation

Verify the integration by building a new application and pushing it to the integrated OpenShift registry to trigger the Tekton pipeline.

First, create a new project called `verify-image-trigger`

```shell
oc new-project verify-image-trigger
```

Instantiate one of the included OpenShift templates to create an Django based application

```shell
oc new-app --template=django-psql-example
```

Track the status of the build that was automatically started

```shell
oc logs -n verify-image-trigger -f bc/django-psql-example
```

A new PipelineRun will have been triggered

```shell
tkn pipelinerun list -n image-watcher
```

View the logs from the PipelineRun

```shell
tkn pipelinerun logs -L -n image-watcher

[print-image : print-image] Event Type: dev.knative.apiserver.resource.add
[print-image : print-image] Image Name: sha256:3a2e3932987329f9a52e5ea41d820c9c6937125807850c5a98a48916c450962c
[print-image : print-image] Image Reference: image-registry.openshift-image-registry.svc:5000/verify-image-trigger/django-psql-example@sha256:3a2e3932987329f9a52e5ea41d820c9c6937125807850c5a98a48916c450962c
```