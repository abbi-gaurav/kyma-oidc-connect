# Overview

- Kyma
- Gardener OIDC Connect
- GitHub Actions

This sample demonstrates how to configure a Kyma runtime with OIDC authentication using GitHub Actions. It uses [Gardener OIDC Connect](https://gardener.cloud/docs/extensions/others/gardener-extension-shoot-oidc-service/openidconnects) to authenticate users via [GitHub OIDC tokens](https://docs.github.com/en/actions/concepts/security/openid-connect#updating-your-workflows-for-oidc).

The advantage of using OIDC tokens is that you don't need to manage long-lived credentials, as the tokens are short-lived and automatically refreshed by GitHub Actions.

## Prerequisites

- Kyma runtime

## Setup

- Add the following secrets to your GitHub repository:

  - `K8S_API_SERVER`: The API server URL of your Kyma cluster.
  - `CA_CERT_STRING`: The base64-encoded CA certificate of your Kyma cluster.
  - `CLUSTER_ID`: The ID of your Kyma cluster (used as the audience for the OIDC token).

- Create an OIDC Connect resource in your Kyma cluster. This resource will define the OIDC provider / issuer (GitHub) and the audience along with the usernameClaim.

```shell
kubectl apply -f k8s/github-oidc-connect.yaml
```

Main values to configure in the `github-oidc-connect.yaml` file:

```yaml
issuerURL: https://token.actions.githubusercontent.com

# clientID is the audience for which the JWT must be issued for, the "aud" field.
clientID: my-kyma-cluster-id #{e.g. you can use your Kyma cluster ID here}

# usernameClaim is the JWT field to use as the user's username.
usernameClaim: actor #it easily maps to the org owner, you can use a custom claim if needed
```

- Create a [GitHub Actions workflow](./.github/workflows/kyma-oidc-connect.yaml) that uses the OIDC token to authenticate with your Kyma cluster. The workflow will get the pods in the `default` namespace and print their names.