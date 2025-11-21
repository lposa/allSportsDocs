# Adding Secret Keys 

- Locally, we use our .env files for various secret values.
- For production, we use Google Secret Manager.

## How to add keys to GSM?

- First of all, we need to add the keys in the `apphosting.yaml` file. This is where Firebase looks in when deploying for various secret values.
- We need to add the secret in Google Cloud Secret Manager. 
- Finally, we need to run this command: `firebase apphosting:secrets:grantaccess KEY_NAME --backend us-sports`