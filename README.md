# XC Site in AKS Demo

## Azure Stuff

Get your subscription ID:

`az account show|jq '.id'`

Create a service account for terraform:

`az ad sp create-for-rbac --name your_sp_name --role Contributor --scopes /subscriptions/your_sub_id`

Output should looks something like:

```JSON
{
  "appId": "xxxxxxxx-xxxx-xxxx-xxxxxxxxxxxxxxxxx",
  "displayName": "your_sp_name",
  "password": "xxxxx-xxxxxxxxx-xxxxxxxxxxxxxxxxxx-xxxxx",
  "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
}

```

Using the values above, export the environment variables for terraform to use for auth-n/z:

```bash
export ARM_SUBSCRIPTION_ID="your_sub_id"
export ARM_TENANT_ID="tenant"
export ARM_CLIENT_ID="appid"
export ARM_CLIENT_SECRET="password"
```

## Terraform and Kubectl Stuff

### Basic Steps for Building AKS Cluster and XC K8s Site

```bash
terrform init -upgrade
terraform plan -out main.tfplan
terraform apply main.tfplan

echo "$(terraform output kube_config)" > ./azurek8s (edit azureki8s and remove the EOFs)
kubectl --kubeconfig azurek8s get nodes
kubectl --kubeconfig azurek8s apply -f azure-vote.yaml

kubectl --kubeconfig azurek8s get pods -n ves-system -o=wide
kubectl --kubeconfig azurek8s logs statefulset/vp-manager -n ves-system -f

(Go and accept registrations in XC Console)

kubectl --kubeconfig azurek8s apply -f httpbin.yaml

```

### Stuff For Testing in the AKS Cluster

```bash
kubectl --kubeconfig azurek8s run --rm -it busybox --image radial/busyboxplus:curl /bin/sh
```

### Start and Stop the AKS Cluster

Get the cluster name and resource group:

`terraform output`


```bash
az aks list

az aks stop --name cluster-saved-goblin -g rg-smiling-mite
az aks start --name cluster-saved-goblin -g rg-smiling-mite

az aks show --name cluster-saved-goblin -g rg-smiling-mite|jq '.agentPoolProfiles[]|.powerState'
```