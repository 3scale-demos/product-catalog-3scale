## 3scale Demo with Product Catalog

This is a 3scale demo that leverages the product-catalog application which is used in some of my gitops demos. This demo places an APICast gateway between the nodejs client and the quarkus backend to manage API access.

### Pre-Requisities

You must have a 3scale API Manager either installed in your local cluster or use the hosted version.

### Installation

To use this demo perform the following steps:

1. Clone this repo

2. In `clusters/overlays` copy one of the folders into a peer folder and rename it to the name of your cluster. In the repo there are two clusters currently shown, `home` and `ocplab`, so you need to make a third cluster that reflects yours.

3. Modify patch.yaml to reflect the wildcard in use for your cluster

4. You will need to provide a secret in order for the apicast gateway to connect to your API Manager portal, I'm currently using a sealed-secret in `api-url-config-sealed-secret.yaml`. This will need to be replaced with your own sealed secret or removed in favor of a tradtional secret.

To remove, simply remove the resource reference to `api-url-config-sealed-secret.yaml` in kustomization.yaml and add your own secret. Creating a secret is covered in the 3scale documentation (here)[https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html/installing_3scale/running-apicast-on-red-hat-openshift#deploying-apicast-using-the-openshift-template].

```
oc create secret generic apicast-configuration-url-secret --from-literal=password=https://<access_token>@<admin_portal_domain>  --type=kubernetes.io/basic-auth
```

5. Install the application using kustomize:

```
kustomize build clusters/overlays/<your-cluster-name>/3scale
```
6. I would recommend changing the environment settings on the apicast-prod to use `APICAST_CONFIGURATION_LOADER=lazy` and `APICAST_CONFIGURATION_CACHE=0` so it reloads on each request so you can test changes.

7. Import the API into your 3scale instance using the included `scripts/import-product-catalog-api` script. Note you need to update the environment variables in the script to match your installation.

8. Go into the 3scale admin portal and create application plans using API keys. I usually create two plans, one unlimited plan and one trial plan limiting the rate to 5 per second. Publish the application plans when finished.

9. Update the Policy for the integration to include the CORS policy first (leave origin blank) and remove the anonymous policy

10. Create a group in Audience along with a User and add an application, I typically create a Test app using the Trial plan and a Production app using the Unlimited plan. Edit the application key for the Production app to be `18de534a3ed3131245a2ecc7638853c1`, this is what is configured in the client. If you do not want to use this key you need to update the client config map with the desired key and restart the client pod. Depending on your account and app settings you may need to approve the new group and application in the admin portal.

11. Test the nodejs application to ensure it shows the product listing. If it's stuck on "loading..." use the Developer tools to see what's happening.