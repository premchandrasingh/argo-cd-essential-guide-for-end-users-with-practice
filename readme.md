# Udemy course (Self note)

https://cision.udemy.com/course/argo-cd-essential-guide-for-end-users-with-practice
https://udemy.com/course/argo-cd-essential-guide-for-end-users-with-practice

# Course Content

## Introduction

### - 1 - Course Overview

### - 2 - Course Slides

### - 3 - Knowledge Prerequisites

- [slide page 3](/argo-cd-slides.pdf#page=3)
- <ins>**Argo CD is deployed on top of kubernetes cluster as containers.**</ins>
- Therefore need to already familiar with container and kubernetes.
- And application manifests (.yaml files) will be store in source control system like Git, Gitlab, Bitbucket as source of truth.
- We can use `helm`, `Kustomize` and `JsonNet` in Argo CD for application manifest.

### - 4 - What is GitOps

- [slide page 4](/argo-cd-slides.pdf#page=4)
- GitOps is an operational framework or guideline that applies the best practices of DevOps such as storing infrastructure and configuration code into git repositories. And having in place a change mechanism such as using pull requests to review the code before merging it into your main branch.
- It is bad practice in IaC you mixed with validated CI/CD process and still create the resources and apply the infrastructure manually without having the need to validate.
- By using fully validated IaC, we have a versioned infrastructure code and configuration code. And we can collaborate teammates and having good reviews before applying the changes.
- It improves security by not permitting individual admin permission to create resource and apply changes rather CD system will do for us [slide page 9](/argo-cd-slides.pdf#page=9).
- Since everything is version, if a bad version is deployed we can rollback easily.
- In the GiOps, there are push and pull delivery models.
- For ArgoCD , it is a GiOps based continuous delivery tool with pull model design [slide page 13](/argo-cd-slides.pdf#page=13).

## Core Concepts

### - 5 - Intro to Argo CD

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

### - 6 - Core Concepts

  1. It will only have two major components 1. Application (source of manifest) & 2. Destination
  2. Supported application source tools are [slide page 31](/argo-cd-slides.pdf#page=31)
      1. Helm Chart
      2. Customize
      3. Directory of yaml files
      4. jsonnet
  
  3. In ArgoCD there is a concept called "Project". It is logical grouping of applications. By it creates a default project by default. Project is useful when used by multiple teams [slide page 32](/argo-cd-slides.pdf#page=32)
  
  4. Sync - process of making desired state = actual state
  
  5. Refresh (or Compare) - compare the latest code on git with the live state. It `automatically freshness every 3 mn`.

### - 7 - Argo CD Architecture Overview

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

### - 8 - Installation options

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

### - 9 - Notes: Installation options

    - [screen shot](/009-Installation%20options.jpg)
### - 10 - Demo: Non-HA Setup

- Run `kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`.
- Here `argocd` is the namespace `-n` and `-f` is the file.
- `kubectl get all -n argocd` - check the result once done.
- `kubectl get pods -n argocd` - check the pods created [screen shot](/010_Non-HA%20Setup.png).

### - 11 - Demo: Getting Initial Admin Password

- Admin UI password is store as a secret in ArgoCD namespace.
- Convert the secret from base64 to plain text
  - [Screen shot](/011_initial-admin-secret.png)
  - List all secret - `kubectl get secret -n argocd` and find secret with name `argocd-initial-admin-secret`
  - Show or open the secret in the file - `kubectl get secret -n argocd argocd-initial-admin-secret -o yaml`
  - Decode the base64 password - `[Text.Encoding]::Utf8.GetString([Convert]::FromBase64String('the base64 password =='))

### - 12 - Practice (Interactive) - Non HA Setup

### - 13 - Accessing ArgoCD Server

- By default ArgoCD server + UI is not exposed outside the cluster.
- You can expose in three ways [slide page 43](/argo-cd-slides.pdf#page=43)
  1. Change the ArgoCD server service type into `LoadBalancer`. It will work if you use cloud managed Kubernetes service like AWS, Azure and GCP.
  2. Use Ingress controller of your choice and point ingress resource to argoCD server service.
  3. Kubernetes port forward - you can use this to be able to access on your local machine. This is suitable when above two options are not available or for testing purpose. Run this command `kubectl port-forward svc/argocd-server -n argocd 8080:443` open browser https://localhost:8080/

### - 14 - Notes: Port Forward

### - 15 - Demo: Access ArgoCD Server Using Port-Forward

- Run `kubectl get svc -n argocd` and find the service name `argocd-server` (remember prefix argocd is your namespace it can be change)
- `kubectl port-forward svc/argocd-server -n argocd 8080:443` open browser https://localhost:8080/
- Get the password from [lecture 11](#--11---demo-getting-initial-admin-password), user name will be `admin` by default.

### - 16 - Install ArgoCD CLI

- We can interact with ArgoCD in three ways, UI, CLI and API/gRPC
- CLI is useful when you need to inexact with ArgoCD in CI pipeline or any automation. [slide page 47](/argo-cd-slides.pdf#page=47)
- What CLI can do
  - Manage applications.
  - Manage Repos.
  - Manage clusters.
  - Admin tasks (such as export/import data or backup restore etc)
  - Manage projects.
  - and more...
- How to install - [slide page 49](/argo-cd-slides.pdf#page=49)
- After installation need to login (using admin user name and password) before any command. For login we may need to do port forward, or LoadBalancer as mentioned earlier [lecture 13](#--13---accessing-argocd-server) so that server is reachable.
  - `argocd login [ArgoCD server]`  
  - then `argocd cluster list`

### - 17 - Notes: CLI Installation instructions

 - Follow official docs for your preferred platform (linux or mac or windows)
 - https://argo-cd.readthedocs.io/en/stable/cli_installation/

### - 18 - Demo: Installing CLI

- Installation in windows, it will take appx 3 min.

```
$version = (Invoke-RestMethod https://api.github.com/repos/argoproj/argo-cd/releases/latest).tag_name

$url = "https://github.com/argoproj/argo-cd/releases/download/" + $version + "/argocd-windows-amd64.exe"

Invoke-WebRequest -Uri $url -OutFile $output
```

- [Screen shot](</018_CLI connecting to ArgoCD server.png>)

### - 19 - Practice (Interactive) - Installing CLI

### - 2 - Section Quiz

## Applications

### - 20 - Defining Applications

- ArgoCD application is a Kubernetes resource object representing a deployed application instance. It is defined by source manifest(desired state in Git) and Destination (the current or target state)
- ArgoCD Application can be created in three ways
  - Declarative approach (Recommended)
  - Using Web UI
  - Using CLI

- Example Declarative approach - [slide page 55](/argo-cd-slides.pdf#page=55). The source => path in the slide could be directory (if yaml) or helm chart or customize
- Example Web UI approach - [slide page 56](/argo-cd-slides.pdf#page=56)
- Example CLI - [slide page 57](/argo-cd-slides.pdf#page=57)

### - 21 - Notes: Demo Resources Links

### - 22 - Demo: Creating an Application Declaratively using Yaml

- Application yaml file:
  - https://github.com/premchandrasingh/argocd-course-apps-definitions/blob/main/applications%20and%20projects/application.yaml

- Source: Directory manifest
  - https://github.com/premchandrasingh/argocd-example-apps/tree/master/guestbook

Let create

- Create the application `kubectl apply -f application.yaml`
- Check the result `kubectl get application -n argocd`.   You can also check with UI or argo CLI command `argocd app list`. By default ArgoCD does not sync therefore will be `OutOfSync`, need manual sync. [Screen shot](/022_create_application_declaratively.png)

### - 23 - Practice (Interactive) - Creating an Application Declaratively

### - 24 - Notes: about accessing Argo CD Web UI

### - 25 - Demo: Creating an Application Using Web UI

- Check [Lecture 13](#--13---accessing-argocd-server) for enabling UI access. In local you probably go with port forwarding.
- Start with `Create APP` on top left of ArgoCD UI.

### - 26 - Practice (Interactive) - Creating an Application Using Web UI

### - 27 - Notes: CLI

### - 28 - Demo: Creating an Application Using CLI

- [slide page 60](/argo-cd-slides.pdf#page=60)

```
argocd app create app-2 -repo https://github.com/premchandrasingh/argocd-example-apps.git --revision master --path guestbook --dest-server https://kubernetes.default.svc --dest-namespace app-2 --sync-option CreateNamespace=true
```

- Check with the command `argocd app list`
- Now sync `argocd app sync app-2`

### - 29 - Practice (Interactive) - Creating an Application Using CLI

### - 30 - Tools Detection

- When writing ArgoCD Application yaml, we can use different tools according to our convenience. [slide page 62](/argo-cd-slides.pdf#page=62)
- In application definition you can explicitly specify the application tool and each tool has its own options.
  - Directory of yaml files - Example [slide page 63](/argo-cd-slides.pdf#page=63)
  - Helm Chart - Example [slide page 64](/argo-cd-slides.pdf#page=64)
  - Kustomize - Example [slide page 65](/argo-cd-slides.pdf#page=65)
  - jsonnet. - Example N/A

- It is also possible tool is not explicitly specify but detected automatically [slide page 66-67](/argo-cd-slides.pdf#page=66).
  - If application yaml contains Chart.yaml then ArgoCD consider it is Helm chart tool.
  - If application yaml contains Kustomization.yaml or Kustomization.yml or Kustomization  then ArgoCD consider it is Helm chart tool.
  - <ins>If above two points are not satisfied, ArgoCD assumed to as plain yaml Directory application.</ins>
- You can explicitly specify application too in UI also [slide page 68](/argo-cd-slides.pdf#page=68)

### - 31 - Helm Options

- What are the ArgoCD application Helm chart tool options
- Helm application can be deployed from two Sources [slide page 70](/argo-cd-slides.pdf#page=70)
  - Github repo - Example [slide page 71](/argo-cd-slides.pdf#page=71). `path` (a directory in the repo), `repoURL` and `targetRevision` (branch name, commit id etc) are required.
  - Helm repo - Example [slide page 72](/argo-cd-slides.pdf#page=72)
  - Check for options [slide page 73-78](/argo-cd-slides.pdf#page=73)

### - 32 - Demo: Helm Options

- Options example manifest (repo directory contains Chart.yaml file) - https://github.com/mabusaa/argocd-example-apps/tree/master/helm-guestbook
- Application example - [application - Helm options.yaml](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/application%20-%20Helm%20options.yaml)
- `kubectl apply -f '.\application - Helm options.yaml'`

### - 33 - Practice (Interactive) - Helm Options

### - 34 - Directory of Files Options

- Options for Directory of Files & jsonnet. - [slide page 81-84](/argo-cd-slides.pdf#page=81)

### - 35 - Demo: Directory Options

- Options example manifest - https://github.com/mabusaa/argocd-example-apps/tree/master/guestbook-with-sub-directories
- [github Application example](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/application%20-%20Directory%20options.yaml)

### - 36 - Practice (Interactive) - Directory Options

### - 37 - Kustomize Options

- [slide page 86-93](/argo-cd-slides.pdf#page=86)

### - 38 - Demo: Kustomize Options

- Kustomize example manifest (_folder contains kustomization.yaml_) - https://github.com/mabusaa/argocd-example-apps/tree/master/kustomize-guestbook
- [github Application example](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/application%20-%20kustomize%20options.yaml)

### - 39 - Practice (Interactive) - Kustomize Options

### - 40 - Multiple Sources for an Application

- <ins>Till ArgoCD 2.5, we can specify a single source per application. Source could be 1. git repository or  2. Helm repository.</ins>
- <ins>Starting ArgoCD 2.6, we can specify resources stored in multiple repositories or helm repositories.</ins> [Screen shot](</040_1_starting argocd 2.6 multiple source supported.png>)
  - Use case - If we need combine multiple resources in an application instead of creating multiple applications. For example Redis and the Prometheus exporter for Redis in one application.[Screen shot](</040_2_multiple source use case.png>)
- _If multiple sources produce the same resource (like group, kind, name and namespace etc) the last source will take precedence._

### - 41 - Practice (Interactive) - Multiple Sources for an Application

### - 3 - Section Quiz

## Projects

### - 42 - Why Projects

- [slide page 95](/argo-cd-slides.pdf#page=95)
- Project provide logical grouping to multiple applications
- It can be use for access restriction when ArgoCD is used by multiple teams.
- It can allow only specific **"trusted git repos"**
- It can allow apps to be deployed into **specific cluster and namespace**.
- It can allow only **specific resources** to be deployed like Deployments and Statefulsets etc
- Project also have **Project Roles** that enables you to create a set of rules with a set of permission called **policies**.
- <ins>By default ArgoCD creates a default project when we install it.</ins>


### - 43 - Creating Projects

- [slide page 100](/argo-cd-slides.pdf#page=100)
- [slide page 103](/argo-cd-slides.pdf#page=103) - specifying cluster server url and namespace.
- [slide page 104](/argo-cd-slides.pdf#page=104) - Specifying only allowed git repo.
- [slide page 105](/argo-cd-slides.pdf#page=105] - Deny all resources to create except "Namespace"
- [slide page 106](/argo-cd-slides.pdf#page=106] - Denying all namespaced scoped resources except "Deployment".
- [slide page 107](/argo-cd-slides.pdf#page=107) - Allow all namespaced scoped resources except "NetworkPolicy". Basically blacklisting "NetworkPolicy".

### - 44 - Demo: Creating Basic Project

- [Github demo Project url](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/project.yaml)
- [Git Application url](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/application%20-%20set%20project.yaml)
- `kubectl get appproject -n ardocd` - Get project "appproject" under namespace(`-n`) "argocd". 
- `kubectl get appproject -n ardocd -o yaml` - Get project and output(`-o`) as yaml.
- In UI - go to left setting menu => find "Projects" section.

### - 45 - Demo: Creating a Project with Allowing Specific Destinations

- [Git demo Project url](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/project%20-%20whitelist%20namespace.yaml)
- [Git demo Application url](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/application%20-%20dev%20project.yaml)
- `kubectl describe application myapp -n argocd` - Show full detail of "myapp" application.
### - 46 - Practice (Interactive) - Projects

### - 47 - Project Roles

- [slide page 111](/argo-cd-slides.pdf#page=111)
- [slide page 113](/argo-cd-slides.pdf#page=113) - Defining policy.
  - _**"p, proj:demo-project:ci-role, applications, sync, demo-project/*, allow"**_.
    - "demo-project" is project name
    - "ci-role" is policy name. Together is is saying this policy is applying to the project
    - "applications" is saying the policy is applicable to "applications" resource
    - "sync" is saying the name of the action meaning "sync" can or can not be perform on the "applications"  resource. Actions can be "sync, get,create, delete, update, override" etc
    - "demo-project/*, allow" is saying the action is allowed. Meaning "sync" is allowed to all "applications"
- Project roles are only available in jwt token, they are not stored in ArgoCD;
  - `argocd proj role create-token <PROJECT-NAME> <ROLE-NAME>` -  Create token using CLI.
    - Example - `argocd proj role create-token demo-project ci-role` - Create jwt token for the above example project and role.
- Example 1 - `argocd cluster list --auth-token token-value` - Using jwt token.
- Example 2 - `argocd app delete myapp --auth-token <jwt token here>` - This example will be return permission denied as it was not allowed to delete (corresponding to above example)

### - 48 - Demo: Project Roles

- [Git Example url](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/project%20-%20role.yaml)

### - 49 - Practice (Interactive) - Project Roles

### - 4 - Section Quiz

## Repositories

### - 50 - Private Git Repos

- [slide page 117](/argo-cd-slides.pdf#page=117)
- https://argo-cd.readthedocs.io/en/stable/user-guide/private-repositories/#github-app-credential

### - 51 - Note: K8s Secret for Argo CD Repos

### - 52 - Note: Practice Private Repos

### - 53 - Demo: Private Repos using Https

- [Git example secret/private repo url](https://github.com/premchandrasingh/argocd-course-apps-definitions/blob/main/repos/private-repo-https.yaml)
- [Git example Application url](https://github.com/premchandrasingh/argocd-course-apps-definitions/blob/main/applications%20and%20projects/application-with-private-repo.yaml)
- You can check the secret added.
  - Using CLI - `argocd get secret -n argocd`
  - Using UI - Go go the left menu Settings => Repository section

### - 54 - Practice (Interactive) - Private Git Repos using Https

### - 55 - Demo: Private Repos using SSH

- [Git example Application url](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/application-with-private-repo-ssh.yaml)
- [Git example secret/private repo url](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/repos/private-repo-ssh.yaml)

### - 56 - Practice (Interactive) - Private Repos using SSH

### - 57 - Private Helm Repos

- [slide page 126](/argo-cd-slides.pdf#page=126)

### - 58 - Credential Templates

- [slide page 131](/argo-cd-slides.pdf#page=131)
- <ins>Credential Templates is for - If we want to use the same credentials for multiple repositories without having to repeat the credential information for every repo.</ins>
- It is declare as normal secret resource in Kubernetes [slide page 133](/argo-cd-slides.pdf#page=133).

### - 59 - Demo: Credential Templates

- [Git Example url for Credential Templates](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/repos/private-repo-creds-https.yaml)

### - 60 - Practice (Interactive) - Credential Templates

### - 5 - Section Quiz

## Sync Policies and Options

### - 61 - Automated Sync

- [slide page 137](/argo-cd-slides.pdf#page=137)
- By default ArgoCD polls git repositories every 3 minutes to detect changes to the manifests.
- An automated sync will only be performed if the application is "OutOfSync".
- Rollback can not be performed for an application with the automated sync enabled.

### - 62 - Demo: Automated Sync

- [Example Application for Automatic Sync](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/application%20-%20Automated%20Sync.yaml)
  - ` automated: {}` is the required setting.

### - 63 - Practice (Interactive) - Automated Sync

### - 64 - Automated Pruning

- [slide page 143](/argo-cd-slides.pdf#page=143)
- When Automatic Sync is enabled, by default automatic prune is disabled. This is for safety reason.
- But we can enable automatic pruning when automatic sync is enabled.

### - 65 - Demo: Automated Pruning

- [Example Application for Automatic Prune](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/application%20-%20Automated%20Prune.yaml)

### - 66 - Practice (Interactive) - Automated Pruning

### - 67 - Automated Self-Healing

- [slide page 150](/argo-cd-slides.pdf#page=150)

### - 68 - Demo: Automated Self-Healing

- [Example Application for Automatic SelfHeal](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/application%20-%20Automated%20Self-Healing.yaml)
- It will already tries to sync to desired state defined in the source even though it changes from CLI.

### - 69 - Practice (Interactive) - Automated Self-Healing

### - 70 - Sync Options

- - [slide page 156](/argo-cd-slides.pdf#page=156)

### - 71 - Demo: No Prune at Resources Level

- [slide page 173](/argo-cd-slides.pdf#page=173)
- [Git Example manifest url](https://github.com/mabusaa/argocd-example-apps/tree/master/sync-options/no-prune)
- [Git example Application url](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/sync-options/application%20-%20No%20prune.yaml)

### - 72 - Practice (Interactive) - No Prune At Resources Level

### - 73 - Demo: Selective Sync

- [slide page 174](/argo-cd-slides.pdf#page=174)
- Argo CD Application should only sync the changed manifests
- [Git Example manifest url](https://github.com/mabusaa/argocd-example-apps/tree/master/sync-options/selective-sync)
- [Git example Application url](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/sync-options/application%20-%20Selective%20Sync.yaml)

### - 74 - Practice (Interactive) - Selective Sync

### - 75 - Demo: Fail On Shared Resources

- [Git example Application-1 url](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/sync-options/application%20-%20Fail%20On%20Shared%20Resources.yaml)
- [Git example Application-2 url](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/application%20-%20set%20project.yaml)

### - 76 - Practice (Interactive) - Fail on Shared Resources

### - 77 - Demo: Replace Resources

- [Git example manifest url](https://github.com/mabusaa/argocd-example-apps/tree/master/sync-options/replace)
- [Git example Application url](https://github.com/mabusaa/argocd-course-apps-definitions/blob/main/applications%20and%20projects/sync-options/application%20-%20Replace.yaml)

### - 78 - Practice (Interactive) - Replace Resources

### - 6 - Section Quiz

## Tracking Strategies

### - 79 - Tracking Strategies

### - 80 - Demo: Tracking Git Tag

### - 81 - Demo: Tracking Git Commit SHA

### - 82 - Demo: Tracking HEAD

### - 83 - Practice (Interactive) - Tracking Strategies for Git Repos

### - 84 - Demo: Tracking Helm Chart Range of Versions

### - 85 - Demo: Tracking Helm Chart Latest Version

### - 86 - Practice (Interactive) - Tracking Strategies for Helm Charts

### - 7 - Section Quiz

## Diffing Customization

### - 87 - Diffing Customization

### - 88 - Demo: Diffing Customization Demo

### - 89 - Demo: Diffing Customization, Istio Case

### - 8 - Section Quiz

## Sync Phases and Waves

### - 90 - Sync Phases and Hooks

### - 91 - Demo: Resource Hooks (Sync Phases)

### - 92 - Practice (Interactive) - Resource Hooks (Sync Phases)

### - 93 - Sync Waves

### - 94 - Demo: Sync Waves

### - 95 - Practice (Interactive) - Sync Waves

### - 9 - Section Quiz

## Remote Kubernetes Clusters

### - 96 - Defining K8s Clusters

### - 97 - Demo: Remote Clusters

### - 98 - Practice (Interactive) - Remote Clusters

### - 10 - Section Quiz

## ApplicationSet

### - 99 - What is ApplicationSet

### - 100 - Generators

### - 101 - Demo: List Generator

### - 102 - Practice (Interactive) - List Generator

### - 103 - Demo: Cluster Generator

### - 104 - Practice (Interactive) - Cluster Generator

### - 105 - Demo: Git Directory Generator

### - 106 - Practice (Interactive) - Git Directory Generator

### - 107 - Demo: Matrix Generator

### - 108 - Practice (Interactive) - Matrix Generator

### - 109 - Pull Request Generator

### - 110 - Demo: Pull Request Generator

### - 111 - Practice (Interactive) - Dynamic Environments Using Pull Request Generator

### - 11 - Section Quiz

## Automation by CI Pipelines

### - 112 - CICD Flow

### - 113 - Demo: Basic CI Pipeline

### - 114 - Practice (Interactive) - Basic CI/CD

## Recommendations and Best Practices

### - 115 - How to Structure Git Repos

### - 116 - (App of Apps) How to Structure Apps Definitions

### - 117 - Demo: App of Apps

### - 118 - Practice (Interactive) - App of Apps, Directory of Apps Approach

### - 119 - Practice (Interactive) - App of Apps, Helm Chart Approach

### - 120 - Best Practices
