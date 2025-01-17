# Purpose

This directory aims to replace the github actions used in the main repository with an Argo Workflows equivalent.

You can run this using Pipekit:

1. Create a new organization in Pipekit: https://docs.pipekit.io/pipekit/organizations
2. Create a cluster in the org and install pipekit agent into the cluster: https://docs.pipekit.io/pipekit/clusters
3. Install the [pre-requisites](#pre-requisites) listed below into the cluster.
4. Create a new Pipe in Pipekit and reference `pipekit/workflow.yml` as the workflow file.
5. Set appropriate [Run Conditions](https://docs.pipekit.io/pipekit/pipes/managing-pipes/run-conditions) to trigger the Pipe.
6. Set up Container Repo secrets if you wish to push. You will need to call the secret `DOCKER_KEY` and it will need to contain the contents of your [JSON key file](https://cloud.google.com/container-registry/docs/advanced-authentication#json-key). A reminder: once you have created your secret in the Pipekit UI, you need to ensure that the secret environment is being used by the Run Condition, and then the PR will need to be re-triggered with a new commit to pick up the change.

## Pre-Requisites
Your cluster will require the following to be installed:

- Argo Workflows
_ [Pipekit Agent](https://docs.pipekit.io/pipekit-agent/helm-chart)
- [NFS Server Provisioner](https://github.com/kubernetes-sigs/nfs-ganesha-server-and-external-provisioner)

## Running Locally
This directory contains a simple k3d cluster with Argo Workflows and NFS Server Provisioner pre-installed. In order to run this cluster, you will need [k3d](https://k3d.io/) installed, as well as kubectl.

`./k3d.sh`

You can manually install the Pipekit Agent and use this cluster with Pipekit for local development, or you can simply run the following commands to run the native Argo Workflow instead.

`kubectl -n argo create -f workflow.yml`

### Stopping the k3d cluster
`k3d cluster delete pipekit-spaceboot`


## What the workflow does

- Checks out the repository. By default, this will check out the main branch. You can change this by setting the `branch` parameter: (`spec.arguments.parameters.git_branch.value`). When you trigger it through pipekit, the branch will be passed to the workflow automatically and will overwrite this default.
- If `spec.arguments.parameters.push_image.value` is set to true, it will collect the container repo credentials from Pipekit and will push the image to `ghcr.io/speedscale-demos/spaceboot:latest`. If set to false, the image will be built but not pushed.
- Builds linux/amd64 variants of the container image only. We can add more variants later if needed.
- Uses container caching, so expect a much faster build from the second time onwards.


## Out of Scope
- There is no useful Java Cache. It felt unnecessary for this POC, but would be a nice thing to have to speed things up.
- This workflow is only designed to push images to google container registry. It won't work out of the box with other registries.


## Issues
I could not get the amd64 variant to build successfully on my m2 mac. If you also struggle, change the line `--opt platform=linux/amd64,linux/arm64 ` in the workflow to remove the amd64 variant. On an EKS cluster, both built fine so it's some mac-specific issue that I wasn't too concerned about for this POC.

I am a random readme change. Ignore me.
