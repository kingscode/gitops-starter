# Setup external secrets

This describes how to setup external secrets to work with Google Secret Manager and your application.

## Setup Google Secret Manager

Enable the Google Secret Manager. Create your secrets in JSON format. Each env variable for your application is a key:
value pair in the JSON file.

As an example, the `external-secret` will look for `api-secrets-{{environment}}` secrets to sync.

## Setup external secrets

External secrets should be bootstrapped in the application, verify it works.

## Create a service account

Create a service account with the following roles:

- Secret Manager Secret Accessor

Create a JSON key for the service account. Place this key in the `gcpsm-secret.yaml` secret and apply it to your cluster
for each namespace where you plan to use the external secrets.

## Create a secret

You should now be able to automatically sync secrets from GCP to your cluster. See the `external-secret.yaml` file
and `secretstore.yaml` file for the implementation:

- `external-secret.yaml`<br/>
  This is the configuration for the external secrets. It will sync the secret from GCP to your defined cluster secret (
  api-secrets by default).
  <br/>
- `secretstore.yaml`<br/>
  This describes to `external-secrets` where the secrets can be found and what credentials to use.
