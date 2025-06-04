# Udemy course (Self note)

https://cision.udemy.com/course/argo-cd-essential-guide-for-end-users-with-practice
https://udemy.com/course/argo-cd-essential-guide-for-end-users-with-practice

# Course Content

## Introduction

- 1 - Course Overview
- 2 - Course Slides
- 3 - Knowledge Prerequisites

  - [slide page 3](/argo-cd-slides.pdf#page=3)
  - <ins>**Argo CD is deployed on top of kubernetes cluster as containers.**</ins>
  - Therefore need to already familiar with container and kubernetes.
  - And application manifests (.yaml files) will be store in source control system like Git, Gitlab, Bitbucket as source of truth.
  - We can use `helm`, `Kustomize` and `JsonNet` in Argo CD for application manifest.

- 4 - What is GitOps

  - [slide page 4](/argo-cd-slides.pdf#page=4)
  - GitOps is an operational framework or guideline that applies the best practices of DevOps such as storing infrastructure and configuration code into git repositories. And having in place a change mechanism such as using pull requests to review the code before merging it into your main branch.
  - It is bad practice in IaC you mixed with validated CI/CD process and still create the resources and apply the infrastructure manually without having the need to validate.
  - By using fully validated IaC, we have a versioned infrastructure code and configuration code. And we can collaborate teammates and having good reviews before applying the changes.
  - It improves security by not permitting individual admin permission to create resource and apply changes rather CD system will do for us [slide page 9](/argo-cd-slides.pdf#page=9).
  - Since everything is version, if a bad version is deployed we can rollback easily.
  - In the GiOps, there are push and pull delivery models.
  - For ArgoCD , it is a GiOps based continuous delivery tool with pull model design [slide page 13](/argo-cd-slides.pdf#page=13).

## Core Concepts

- 5 - Intro to Argo CD

  - <ins>**Argo CD is deployed on top of kubernetes cluster as containers.**</ins>
  - It has a controller and track the manifest (yaml files) by pulling from the source of truth(github) and keep in sync with destination cluster.
  - ArgoCD is not a CI tool. CI tools are like GitLab CI, Azure Pipelines or Github Actions. ArgoCd is CD tool [slide page 16](/argo-cd-slides.pdf#page=16).
  - ArgoCD VS Traditional CD tools

    1. Traditional tools

        1. [slide page 17](/argo-cd-slides.pdf#page=17)
        2. CI tools, that will build code & run unit test.
        3. Create docker image.
        4. The new docker image published to registry.
        5. The deployment script (yaml file) in the source control get updated (automatically - need work to do automatically) with latest docker image with tag.
        6. Finally `kubectl apply ...` command runs the complete the deployment

            1. To finally apply changes with the above command, we need three things. We need to install `kubectl`, `helm` and allow permission to cluster onto this CICD agent. This could be a security concern.

    2. ArgoCD approach

        1. [slide page 17](/argo-cd-slides.pdf#page=19)
        2. Same process till step 5 above.
        3. Finally instead of applying changes, `argoCD which run in the k8s cluster itself will pull and compare the changed configuration and keeping syncing to desired state.`

            1. How does the ArgoCD deploy after pull the changes ????? is it argoCD have all kubectl and other necessary software installed by itself or HOW ???
        4. State visibility in UI is available.
        5. Easy rollback
        6. Granting the needed access to deploy into the destination cluster and instead of granting CI/CD agents or administrators.
        7. ArgoCD also provide Disaster recovery solution. You can easily deploy the same app to any k8s cluster.

- 6 - Core Concepts

  1. It will only have two major components 1. Application (source of manifest) & 2. Destination
  2. Supported application source tools are [slide page 31](/argo-cd-slides.pdf#page=31)
      1. Helm Chart
      2. Customize
      3. Directory of yaml files
      4. jsonnet
  
  3. In ArgoCD there is a concept called "Project". It is logical grouping of applications. By it creates a default project by default. Project is useful when used by multiple teams [slide page 32](/argo-cd-slides.pdf#page=32)
  
  4. Sync - process of making desired state = actual state
  
  5. Refresh (or Compare) - compare the latest code on git with the live state. It `automatically freshness every 3 mn`.

- 7 - Argo CD Architecture Overview

  - All ArgoCd components will be running on pods. It consist of three components [slide page 24](/argo-cd-slides.pdf#page=24)

      1. ArgoGoCD Server (API + Web Server)
          - It is the only component that you need to interact with.
          - It support gRPC and REST. And these are consumed by ArgoCD Web UI and CLI. It controls everything like create, update, delete, sync rollback and authentication.  [slide page 25](/argo-cd-slides.pdf#page=25)

      2. ArgoCD Repo server
          - It is an internal service responsible for 1. Cloning and 2. Generating k8s manifest.

      3. ArgoCD Application Controller [slide page 27](/argo-cd-slides.pdf#page=27)
          - Its the k8s controller which keep monitoring running applications and compares the actual states in the destination cluster with the desired state
          - It get the reports so that it can communicate with the server to generate the manifest.
          - ALSO, it communicate with the K8s API to get the actual state and deploy the application manifest to the destination cluster.
          - Take corrective action and detect outof-sync apps.

      4. There are other supportive component like Redis cache, `Dex` ( identity service that integrates with external identity providers like Git) and lastly `Application Set` Controller who is responsible for automatically generation of seed apps. [slide page 29](/argo-cd-slides.pdf#page=29)

- 1 - Section Quiz

## Setting-Up Argo CD

- 8 - Installation options

  - [slide page 37](/argo-cd-slides.pdf#page=37)
  - Need any running cluster. `Can also done on MiniKube cluster`, Docker Desktop, Kind, Rancher Desktop or Full Cluster. There are three options for installation.

    - Non-Highly available setup.
      - (suitable for dev or test evn or evaluation). `It will install only one pod for each components only.`
        - cluster-admin privileges: https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
        - namespace level privileges: https://github.com/argoproj/argo-cd/raw/stable/manifests/namespace-install.yaml

    - Highly available setup.
      - Recommended for prod
      - you need at least 3 worker nodes in k8s cluster.
        - cluster-admin privileges: https://github.com/argoproj/argo-cd/raw/stable/manifests/ha/install.yaml
        - namespace level privileges: https://github.com/argoproj/argo-cd/raw/stable/manifests/ha/namespace-install.yaml

    - Light installation "Core"
      - Suitable if ArgoCD will be used by administrators only.
      - It install without UI and API server
      - It install by default as non-highly available setup
      - https://github.com/argoproj/argo-cd/raw/stable/manifests/core-install.yaml

  - On Privileges or Permission options, it has two options [slide page 39](/argo-cd-slides.pdf#page=39)
    1. Cluster-Admin privilege - where ArgoCD has the cluster-admin access to deploy into the cluster that runs in.
    2. Namespace level privileges - You can use this if you are planning to use ArgoCD without deploying into the same cluster that ArgoCD runs in.

  - Manifest Installation options
    - Yaml Manifests - https://github.com/argoproj/argo-cd/blob/master/manifests/install.yaml
    - Helm chart: https://github.com/argoproj/argo-helm/tree/main/charts/argo-cd

- 9 - Notes: Installation options

    - [screen shot](/009-Installation%20options.jpg)
- 10 - Demo: Non-HA Setup
- 11 - Demo: Getting Initial Admin Password
- 12 - Practice (Interactive) - Non HA Setup
- 13 - Accessing ArgoCD Server
- 14 - Notes: Port Forward
- 15 - Demo: Access ArgoCD Server Using Port-Forward
- 16 - Install ArgoCD CLI
- 17 - Notes: CLI Installation instructions
- 18 - Demo: Installing CLI
- 19 - Practice (Interactive) - Installing CLI
- 2 - Section Quiz

## Applications

- 20 - Defining Applications
- 21 - Notes: Demo Resources Links
- 22 - Demo: Creating an Application Declaratively using Yaml
- 23 - Practice (Interactive) - Creating an Application Declaratively
- 24 - Notes: about accessing Argo CD Web UI
- 25 - Demo: Creating an Application Using Web UI
- 26 - Practice (Interactive) - Creating an Application Using Web UI
- 27 - Notes: CLI
- 28 - Demo: Creating an Application Using CLI
- 29 - Practice (Interactive) - Creating an Application Using CLI
- 30 - Tools Detection
- 31 - Helm Options
- 32 - Demo: Helm Options
- 33 - Practice (Interactive) - Helm Options
- 34 - Directory of Files Options
- 35 - Demo: Directory Options
- 36 - Practice (Interactive) - Directory Options
- 37 - Kustomize Options
- 38 - Demo: Kustomize Options
- 39 - Practice (Interactive) - Kustomize Options
- 40 - Multiple Sources for an Application
- 41 - Practice (Interactive) - Multiple Sources for an Application
- 3 - Section Quiz

## Projects

- 42 - Why Projects
- 43 - Creating Projects
- 44 - Demo: Creating Basic Project
- 45 - Demo: Creating a Project with Allowing Specific Destinations
- 46 - Practice (Interactive) - Projects
- 47 - Project Roles
- 48 - Demo: Project Roles
- 49 - Practice (Interactive) - Project Roles
- 4 - Section Quiz

## Repositories

- 50 - Private Git Repos
- 51 - Note: K8s Secret for Argo CD Repos
- 52 - Note: Practice Private Repos
- 53 - Demo: Private Repos using Https
- 54 - Practice (Interactive) - Private Git Repos using Https
- 55 - Demo: Private Repos using SSH
- 56 - Practice (Interactive) - Private Repos using SSH
- 57 - Private Helm Repos
- 58 - Credential Templates
- 59 - Demo: Credential Templates
- 60 - Practice (Interactive) - Credential Templates
- 5 - Section Quiz

## Sync Policies and Options

- 61 - Automated Sync
- 62 - Demo: Automated Sync
- 63 - Practice (Interactive) - Automated Sync
- 64 - Automated Pruning
- 65 - Demo: Automated Pruning
- 66 - Practice (Interactive) - Automated Pruning
- 67 - Automated Self-Healing
- 68 - Demo: Automated Self-Healing
- 69 - Practice (Interactive) - Automated Self-Healing
- 70 - Sync Options
- 71 - Demo: No Prune at Resources Level
- 72 - Practice (Interactive) - No Prune At Resources Level
- 73 - Demo: Selective Sync
- 74 - Practice (Interactive) - Selective Sync
- 75 - Demo: Fail On Shared Resources
- 76 - Practice (Interactive) - Fail on Shared Resources
- 77 - Demo: Replace Resources
- 78 - Practice (Interactive) - Replace Resources
- 6 - Section Quiz

## Tracking Strategies

- 79 - Tracking Strategies
- 80 - Demo: Tracking Git Tag
- 81 - Demo: Tracking Git Commit SHA
- 82 - Demo: Tracking HEAD
- 83 - Practice (Interactive) - Tracking Strategies for Git Repos
- 84 - Demo: Tracking Helm Chart Range of Versions
- 85 - Demo: Tracking Helm Chart Latest Version
- 86 - Practice (Interactive) - Tracking Strategies for Helm Charts
- 7 - Section Quiz

## Diffing Customization

- 87 - Diffing Customization
- 88 - Demo: Diffing Customization Demo
- 89 - Demo: Diffing Customization, Istio Case
- 8 - Section Quiz

## Sync Phases and Waves

- 90 - Sync Phases and Hooks
- 91 - Demo: Resource Hooks (Sync Phases)
- 92 - Practice (Interactive) - Resource Hooks (Sync Phases)
- 93 - Sync Waves
- 94 - Demo: Sync Waves
- 95 - Practice (Interactive) - Sync Waves
- 9 - Section Quiz

## Remote Kubernetes Clusters

- 96 - Defining K8s Clusters
- 97 - Demo: Remote Clusters
- 98 - Practice (Interactive) - Remote Clusters
- 10 - Section Quiz

## ApplicationSet

- 99 - What is ApplicationSet
- 100 - Generators
- 101 - Demo: List Generator
- 102 - Practice (Interactive) - List Generator
- 103 - Demo: Cluster Generator
- 104 - Practice (Interactive) - Cluster Generator
- 105 - Demo: Git Directory Generator
- 106 - Practice (Interactive) - Git Directory Generator
- 107 - Demo: Matrix Generator
- 108 - Practice (Interactive) - Matrix Generator
- 109 - Pull Request Generator
- 110 - Demo: Pull Request Generator
- 111 - Practice (Interactive) - Dynamic Environments Using Pull Request Generator
- 11 - Section Quiz

## Automation by CI Pipelines

- 112 - CICD Flow
- 113 - Demo: Basic CI Pipeline
- 114 - Practice (Interactive) - Basic CI/CD

## Recommendations and Best Practices

- 115 - How to Structure Git Repos
- 116 - (App of Apps) How to Structure Apps Definitions
- 117 - Demo: App of Apps
- 118 - Practice (Interactive) - App of Apps, Directory of Apps Approach
- 119 - Practice (Interactive) - App of Apps, Helm Chart Approach
- 120 - Best Practices
