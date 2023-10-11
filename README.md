# XC Site in AKS Demo

## Reading Material

The Terraform plan to deploy an AKS cluster is from [this quickstart guide](https://learn.microsoft.com/en-us/azure/aks/learn/quick-kubernetes-deploy-terraform?tabs=bash%2Cazure-cli).

Steps for deploying a CE in a Kubernetes cluster (eg AKS) can be found [here](https://docs.cloud.f5.com/docs/how-to/site-management/create-k8s-site).

The following devcentral articles were helpful:

[Kubernetes architecture options with F5 Distributed Cloud Services](https://community.f5.com/t5/technical-articles/kubernetes-architecture-options-with-f5-distributed-cloud/ta-p/306550)

[Securely connecting Kubernetes Microservices with F5 Distributed Cloud](https://community.f5.com/t5/technical-articles/securely-connecting-kubernetes-microservices-with-f5-distributed/ta-p/306100)

## Azure Stuff

You will need the Azure CLI tool `az`. Install it with brew if you're on MacOS:

`brew install azure-cli`

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

Deploy the AKS cluster:

```bash
terrform init -upgrade
terraform plan -out main.tfplan
terraform apply main.tfplan
```

Create the kubeconfig file:
`echo "$(terraform output kube_config)" > ./azurek8s`

(edit azurek8s and remove the EOFs at the top and bottom of the file)
`vi azurek8s`

Test your kubeconfig:
```bash
kubectl --kubeconfig azurek8s get nodes
kubectl --kubeconfig azurek8s get pods --all-namespaces
```

Time to read the documentation on deploying a CE site in K8s. The basic steps are:

1. Generate a site token.
1. Download the CE deployment manifest:
`curl https://gitlab.com/volterra.io/volterra-ce/-/blob/master/k8s/ce_k8s.yml > ce_k8s.yml`
1. Add the site token to the deployment manifest.
1. Deploy the CE pods, check the status of the pods and watch the logs:

```bash
kubectl --kubeconfig azurek8s apply -f ce_k8s.yml
kubectl --kubeconfig azurek8s get pods -n ves-system -o=wide
kubectl --kubeconfig azurek8s logs statefulset/vp-manager -n ves-system -f
```

When you see the that the registrations are waiting for approval in the logs go and accept registrations in XC Console.


### Tools For Testing in the AKS Cluster

A handy busybox pod for curl, ping, etc.
```bash
kubectl --kubeconfig azurek8s run --rm -it busybox --image radial/busyboxplus:curl /bin/sh
```

### Start and Stop the AKS Cluster

Get the AKS cluster name and resource group:
`terraform output`

Stop your AKS cluster (save energy):
`az aks stop --name cluster-saved-goblin -g rg-smiling-mite`

Start your AKS cluster:
`az aks start --name cluster-saved-goblin -g rg-smiling-mite`

Check the status of your AKS cluster:
`az aks show --name cluster-saved-goblin -g rg-smiling-mite|jq '.agentPoolProfiles[]|.powerState'`
