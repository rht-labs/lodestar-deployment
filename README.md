# Lodestar - Deployment

This repository both bootstraps and manages the deployment lifecycle of Lodestar across multiple versions. It contains a manifest which controls the release version of all individual components of Lodestar. It is meant to be used with Argo CD.

## Bootstrapping

To create an instance of Lodestar in a cluster, apply the `bootstrap` Helm chart using the following command:

```sh
helm install lodestar bootstrap/ --set application.ref=<desired git ref>
```

Make sure to replace `<desired-git-ref>` with the tag that you want to deploy to this cluster. If this is a staging cluster, you might have a Git tag such as `deploy-staging`; and if this is a production cluster, maybe `deploy-production`.

## Lifecycle Management

Lodestar is not a single application, but a collection of components (or services) that all add up to a working system. Those services might be versioned separately from one another. Therefore, a "deployment of Lodestar" is really a manifest of the versions of its constituent components that are known to work well with each other.

Manifests for respective components live in `bootstrap/patches/` directory:

Example file, where `v1.3.1` is target version, `bootstrap/patches/lodestar-backend.yaml`: 

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: lodestar-backend
  namespace: lodestar-argo-cd
spec:
  ignoreDifferences:
  - group: apps.openshift.io
    jsonPointers:
    - /spec/template/spec/containers/0/image
    - /spec/template/spec/volumes/0
    kind: DeploymentConfig
  source:
    helm:
      values: |-
        imageTag: v1.3.1
        mongodb:
          persistent: false
    targetRevision: v1.3.1

```

Automatic syncing, pruning, and self healing is enabled for this project - so committing a change to the above manifest and tagging it appropriately as `deploy-staging` or `deploy-production` will cause an Argo CD instance bootstrapped with that tag to deploy those versions of the applications.

## Releases

Creating releases is automated through the use of GitHub Actions Workflows. To create a release, you can just create an issue in this repository named as the release you'd like to create  using the "Creating a release" template, and use the below triggers in an issue comment to begin:

`/release` - This trigger along with the appropriate parameters can be used to create a release branch that will then be used to promote the appropriate changes into a new environment branch. The following parameters can be passed to it:

| Variable | Description | Required |
|:---------|:------------|:---------|
|frontend|The version to use for the frontend application|no|
|backend|The version to use for the backend application|no|
|gitapi|The version to use for the Git API|no|
|dispatcher|The version to use for the Resource Dispatcher|no|
|agnosticv|The version to use for AgnosticV|no|
|anarchy|The version to use for Anarchy|no|
|poolboy|The version to use for Poolboy|no|

`/promote` - This trigger along with the appropriate parameters can be used to promote the changes contained in a release branch into a specified environment branch that ArgoCD is watching. These promotions are auto-merged, so you should be sure that you are ready for a promotion when you run this command. It does not accept any parameters.

`/cancel` - This trigger cancels the rollout/closes the release issue and signals that a new issue should be created if a release is still desired.
