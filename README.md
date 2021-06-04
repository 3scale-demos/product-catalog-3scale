## 3scale Demo with Product Catalog

This is a 3scale demo that leverages the product-catalog application which is used in some of my gitops demos. This demo places an APICast gateway between the nodejs client and the quarkus backend to manage API access.

![alt text](https://raw.githubusercontent.com/gnunn-gitops/product-catalog-3scale/master/docs/img/product-catalog-3scale-screenshot.png.png)

The topology view in OpenShift shows the three tiers of the application along with the apicast gateways:

![alt text](https://raw.githubusercontent.com/gnunn-gitops/product-catalog/master/docs/img/product-catalog-3scale-topology.png)


### Pre-Requisities

You must have a 3scale API Manager either installed in your local cluster or use the hosted SaaS version.

### Installation

To use this demo perform the following steps:

1. Clone this repo

2. In `clusters` copy the `template` folder, with it's subdirectores intact, as a peer folder and rename it to the name of your cluster. In the repo there is an exmaple cluster currently shown, `local.home`, so you need to make a third cluster that reflects your cluster.

3. In your new cluster folder, modify patch.yaml to reflect the cluster name and wildcard in use for your cluster

4. Install the application using kustomize:

```
kustomize build clusters/overlays/<your-cluster-name>/3scale
```

5. You will need to provide a secret in order for the apicast gateway to connect to your API Manager portal, I'm currently using a sealed-secret in `api-url-config-sealed-secret.yaml`. You will need to create your own, creating a secret is covered in the 3scale documentation (here)[https://access.redhat.com/documentation/en-us/red_hat_3scale_api_management/2.9/html/installing_3scale/running-apicast-on-red-hat-openshift#deploying-apicast-using-the-openshift-template].

```
oc create secret generic apicast-configuration-url-secret --from-literal=password=https://<access_token>@<admin_portal_domain>  --type=kubernetes.io/basic-auth
```

6. After creating the secret you will need to restart the apicast pods via a delete

7. I would recommend changing the environment settings on the apicast-stage and apicast-prod to use `APICAST_CONFIGURATION_LOADER=lazy` and `APICAST_CONFIGURATION_CACHE=0` so it reloads on each request so you can test changes.

8. Import the API into your 3scale instance using the included `scripts/import-product-catalog-api` script. Note you need to update the environment variables in the script to match your installation.

9. Go into the 3scale admin portal and create application plans using API keys. I usually create two plans, one unlimited plan and one trial plan limiting the rate to 5 per second. Publish the application plans when finished.

10. Update the Policy for the integration to include the CORS policy first (leave origin blank) and remove the anonymous policy

11. Create a group in Audience along with a User and add an application, I typically create a Test app using the Trial plan and a Production app using the Unlimited plan. Edit the application key for the Production app to be `18de534a3ed3131245a2ecc7638853c1`, this is what is configured in the client. If you do not want to use this key you need to update the client config map with the desired key and restart the client pod. Depending on your account and app settings you may need to approve the new group and application in the admin portal.

12. Test the nodejs application to ensure it shows the product listing. If it's stuck on "loading..." use the browser Developer tools to see what's happening.
