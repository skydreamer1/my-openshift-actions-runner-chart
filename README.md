# OpenShift GitHub Actions Runner Chart

[![Helm Lint](https://github.com/redhat-actions/openshift-actions-runner-chart/workflows/Helm%20Lint/badge.svg)](https://github.com/redhat-actions/openshift-actions-runner-chart/actions)
[![Link checker](https://github.com/redhat-actions/openshift-actions-runner-chart/workflows/Link%20checker/badge.svg)](https://github.com/redhat-actions/openshift-actions-runner-chart/actions)
[![Publish chart to Pages](https://github.com/redhat-actions/openshift-actions-runner-chart/workflows/Publish%20chart%20to%20Pages/badge.svg)](https://github.com/redhat-actions/openshift-actions-runner-chart/actions)

[![Tag](https://img.shields.io/github/v/tag/redhat-actions/openshift-actions-runner-chart)](https://github.com/redhat-actions/openshift-actions-runner-chart/tags)
[![Quay org](https://img.shields.io/badge/quay-redhat--github--actions-red)](https://quay.io/organization/redhat-github-actions)

This repository contains a Helm chart for deploying one or more self-hosted <!-- markdown-link-check-disable --> [GitHub Actions Runners]((https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)) <!-- markdown-link-check-enable -->
into a Kubernetes cluster. By default, the container image used is the [**OpenShift Actions Runner**](https://github.com/redhat-actions/openshift-actions-runner).

You can deploy runners automatically in an Actions workflow using the [**OpenShift Actions Runner Installer**](https://github.com/redhat-actions/openshift-actions-runner-installer).

While this chart and the images are developed for and tested on OpenShift, they do not contain any OpenShift specific code and should be compatible with any Kubernetes platform.

## Prerequisites
You must have access to a Kubernetes cluster. Visit [openshift.com/try](https://www.openshift.com/try) or sign up for our [Developer Sandbox](https://developers.redhat.com/developer-sandbox).

You must have Helm 3 installed.

You do **not** need cluster administrator privileges to deploy the runners and run workloads. However, some images or tools may require special permissions.

## Helm repository
This GitHub repository serves a Helm repository through GitHub Pages.

The repository can be added with:
```
helm repo add openshift-actions-runner https://redhat-actions.github.io/openshift-actions-runner-chart
```

## Installing runners

You can install runners into your cluster using the Helm chart in this repository.

1. Runners can be scoped to an **organization** or a **repository**. Decide what the scope of your runner will be.
    - User-scoped runners are not supported by GitHub.
2. Create a GitHub Personal Access Token as per the instructions in the [runner image README](https://github.com/redhat-actions/openshift-actions-runner#pat-guidelines).
    - The default `secrets.GITHUB_TOKEN` **does not** have permission to manage self-hosted runners. See [Permissions for the GITHUB_TOKEN](https://docs.github.com/en/actions/reference/authentication-in-a-workflow#permissions-for-the-github_token).
3. Add this repository as a Helm repository.
```bash
helm repo add openshift-actions-runner \
    https://redhat-actions.github.io/openshift-actions-runner-chart \
&& helm repo update
```
4. Install the helm chart, which creates a deployment and a secret. Leave out `githubRepository` if you want an organization-scoped runner.
    - Add the `--namespace` argument to all `helm` and `kubectl/oc` commands if you want to use a namespace other than your current context's namespace.

```bash
# PAT from step 2.
export GITHUB_PAT=c0ffeeface1234567890
# For an org runner, this is the org.
# For a repo runner, this is the repo owner (org or user).
export GITHUB_OWNER=redhat-actions
# For an org runner, omit this argument.
# For a repo runner, the repo name.
export GITHUB_REPO=openshift-actions-runner-chart
# Helm release name to use.
export RELEASE_NAME=actions-runner

helm install $RELEASE_NAME openshift-actions-runner/actions-runner \
    --set-string githubPat=$GITHUB_PAT \
    --set-string githubOwner=$GITHUB_OWNER \
    --set-string githubRepository=$GITHUB_REPO \
&& echo "---------------------------------------" \
&& helm get manifest $RELEASE_NAME | kubectl get -f -
```
5. You can re-run step 4 if you want to add runners with different images, labels, etc. You can leave out the `githubPat` on subsequent runs, since it will re-use an existing secret will be left out if it exists already.

The runners should show up under `Settings > Actions > Self-hosted runners` shortly afterward.

## Values

You can override the default values such as resource limits and replica counts or inject environment variables by passing `--set` or `--set-string` to the `helm install` command.

Refer to the [`values.yaml`](./values.yaml) for values that can be overridden.

## Using your own runner image
See the [OpenShift Actions Runner README](https://github.com/redhat-actions/openshift-actions-runner#README.md).

## Managing PATs
See [the wiki](https://github.com/redhat-actions/openshift-actions-runner-chart/wiki/Managing-PATs) for a note on managing mulitple PATs, if you want to add a new PAT or replace an existing one.

## Troubleshooting
You can view the resources created by Helm using `helm get manifest $RELEASE_NAME`, and then inspect those resources using `kubectl get`.

The resources are also labeled with `app.kubernetes.io/instance={{ .Release.Name }}`, so you can view all the resources with:

```sh
kubectl get all,secret -l=app.kubernetes.io/instance=$RELEASE_NAME
```

If the pods are created but stuck in a crash loop, view the logs with `kubectl logs <podname>` to see the problem. Refer to the [runner container troubleshooting](https://github.com/redhat-actions/openshift-actions-runner#troubleshooting) to resolve any issues.
