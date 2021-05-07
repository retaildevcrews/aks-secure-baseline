# Deploy the AKS Cluster

Now that the [hub-spoke network is provisioned](./04-networking.md), the next step in the [AKS secure Baseline reference implementation](./) is deploying the AKS cluster and its adjacent Azure resources.

## Steps

1. Create the AKS cluster resource group.

   > :book: The app team working on behalf of business unit 0001 (BU001) is looking to create an AKS cluster of the app they are creating (Application ID: 0008). They have worked with the organization's networking team and have been provisioned a spoke network in which to lay their cluster and network-aware external resources into (such as Application Gateway). They took that information and added it to their [`cluster-stamp.json`](./cluster-stamp.json) and [`azuredeploy.parameters.prod.json`](./azuredeploy.parameters.prod.json) files.
   >
   > They create this resource group to be the parent group for the application.

   ```bash
   # [This takes less than one minute.]
   az group create --name rg-bu0001a0008-$TEAM_NAME --location eastus2
   ```

1. Get the AKS cluster spoke VNet resource ID.

   > :book: The app team will be deploying to a spoke VNet, that was already provisioned by the network team.

   ```bash
   RESOURCEID_VNET_CLUSTERSPOKE=$(az deployment group show -g rg-enterprise-networking-spokes-$TEAM_NAME -n spoke-BU0001A0008 --query properties.outputs.clusterVnetResourceId.value -o tsv)
   ```

1. Deploy the cluster ARM template.  
  :exclamation: By default, this deployment will allow unrestricted access to your cluster's API Server.  You can limit access to the API Server to a set of well-known IP addresses (i.,e. a jump box subnet (connected to by Azure Bastion), build agents, or any other networks you'll administer the cluster from) by setting the `clusterAuthorizedIPRanges` parameter in all deployment options.  

    **Option 1 - Deploy from the command line**

   > TODO: Parameterize hostname based on *team name* ( e.g. `"hostName": "[concat(variables('teamName'),'aks-sb.com')]"`)

   ```bash
   # [This takes about 15 minutes.]
   az deployment group create -g rg-bu0001a0008-$TEAM_NAME -f cluster-stamp.json -p targetVnetResourceId=${RESOURCEID_VNET_CLUSTERSPOKE} clusterAdminAadGroupObjectId=${AADOBJECTID_GROUP_CLUSTERADMIN} k8sControlPlaneAuthorizationTenantId=${TENANTID_K8SRBAC} aksIngressControllerCertificate=${AKS_INGRESS_CONTROLLER_CERTIFICATE_BASE64}
   ```

   > Alteratively, you could have updated the [`azuredeploy.parameters.prod.json`](./azuredeploy.parameters.prod.json) file and deployed as above, using `-p "@azuredeploy.parameters.prod.json"` instead of providing the individual key-value pairs.

## Container registry note

:warning: To aid in ease of deployment of this cluster and your experimentation with workloads, Azure Policy and Azure Firewall are currently configured to allow your cluster to pull images from _public container registries_ such as Docker Hub. For a production system, you'll want to update Azure Policy parameter named `allowedContainerImagesRegex` in your `cluster-stamp.json` file to only list those container registries that you are willing to take a dependency on and what namespaces those policies apply to, and make Azure Firewall allowances for the same. This will protect your cluster from unapproved registries being used, which may prevent issues while trying to pull images from a registry which doesn't provide SLA guarantees for your deployment.

This deployment creates an SLA-backed Azure Container Registry for your cluster's needs. Your organization may have a central container registry for you to use, or your registry may be tied specifically to your application's infrastructure (as demonstrated in this implementation). **Only use container registries that satisfy the security and availability needs of your application.**

### Next step

:arrow_forward: [Place the cluster under GitOps management](./06-gitops.md)
