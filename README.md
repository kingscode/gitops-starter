# Introduction
This repository is used by ArgoCD to implement the Helm charts into the kubernetes environment.

# Setup
Follow the [SETUP.md](../main/SETUP.md) file.

$ Notifications
Follow the [NOTIFICATIONS.md](../main/NOTIFICATIONS.md) file to setup ArgoCD notifications to slack.

# Flow

See image for the flow of deploying applications:<br/>
![workflow](../main/.docs/workflow.png?raw=true)

For source code:
1. The source repository builds a container image (eg. with Docker) and pushes it to an image repository.
2. The source repository updates the gitops repository with the new image tag.
3. ArgoCD detects the change in the gitops repository and updates the deployment in the kubernetes environment.

For kubernetes components:
1. The kubernetes components are defined in the gitops repository.
2. ArgoCD detects the change in the gitops repository and updates the components in the kubernetes environment.
