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

