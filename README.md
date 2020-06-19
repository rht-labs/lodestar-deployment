# Open Management Portal - Deployment

This repository both bootstraps and manages the deployment lifecycle of the Open Management Portal across multiple versions. It contains a manifest which controls the release version of all individual components of OMP. It is meant to be used with Argo CD.

## Bootstrapping

To create an instance of the Open Management Portal in a cluster, apply the `bootstrap` Helm chart using the following command:

```sh
helm install open-management-portal bootstrap/ --set application.ref=<desired git ref>
```

Make sure to replace `<desired-git-ref>` with the tag that you want to deploy to this cluster. If this is a staging cluster, you might have a Git tag such as `deploy-staging`; and if this is a production cluster, maybe `deploy-production`.

## Lifecycle Management

The Open Management Portal is not a single application, but a collection of components (or services) that all add up to a working system. Those services might be versioned separately from one another. Therefore, a "deployment of OMP" is really a manifest of the versions of its constituent components that are known to work well with each other.

This manifest lives in `applications/values.yaml`:

```yaml
applicationVersions:
  frontend: &frontendVersion 4edb30
  backend: &backendVersion 3a03d2
  git-api: &gitApiVersion 8cb605
  launcher: &launcherVersion 8bca47
  ...
  appName: &yamlAnchorName <ref-to-deploy>
```

NOTE: Because Helm does not support templating inside of the `values.yaml` file, we need to use [YAML anchors](https://confluence.atlassian.com/bitbucket/yaml-anchors-960154027.html) if we want to prevent duplicate definitions of data.

Automatic syncing, pruning, and self healing is enabled for this project - so committing a change to the above manifest and tagging it appropriately as `deploy-staging` or `deploy-production` will cause an Argo CD instance bootstrapped with that tag to deploy those versions of the applications.

## Releases

Creating releases is automated through the use of GitHub Actions Workflows. To create a release, you can just create an issue in this repository and use the below triggers in an issue comment to begin:

`/release` - This trigger along with the appropriate parameters can be used to create a release branch that will then be used to promote the appropriate changes into a new environment branch. This can be run multiple times while adjusting various version that you will be using with a release.The following parameters can be passed to it:

| Variable | Description | Required |
|:---------|:------------|:---------|
|name|Name of the release. This will be used for naming the release branch and label for a release.|yes|
|frontend|The version to use for the frontend application|no|
|backend|The version to use for the backend application|no|
|gitapi|The version to use for the Git API|no|
|dispatcher|The version to use for the Resource Dispatcher|no|
|agnosticv|The version to use for AgnosticV|no|
|anarchy|The version to use for Anarchy|no|
|poolboy|The version to use for Poolboy|no|

`/promote` - This trigger along with the appropriate parameters can be used to promote the changes contained in a release branch into a specified environment branch that ArgoCD is watching. These promotions are auto-merged, so you should be sure that you are ready for a promotion when you run this command. It accepts that following parameters:

| Variable | Description | Required | 
|:---------|:------------|:---------|
|name|Name of the release.|yes|
|env|The name of the environment branch that you wish to merge your changes into|yes|

