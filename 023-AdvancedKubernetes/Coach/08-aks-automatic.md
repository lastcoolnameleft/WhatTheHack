## Sub-Challenge 1: Create an AKS Automatic cluster

Hints

- Might need to use Portal. Can use CLI with `-sku automatic`, but limited functionality at start

Questions

- How many workloads are deployed into cluster upon creation?
    - Should be about 12-13, mostly in kube-system
- What is:
    - Eraser? [Introduction | Eraser Docs (eraser-dev.github.io)](https://eraser-dev.github.io/eraser/docs/) Deleted unused images
    - Gatekeeper? [GitHub - open-policy-agent/gatekeeper: 🐊 Gatekeeper - Policy Controller for Kubernetes](https://github.com/open-policy-agent/gatekeeper) Policy Controller. Works with Open Policy Agent (OPA)
    - VPA? [Vertical pod autoscaling in Azure Kubernetes Service (AKS) - Azure Kubernetes Service | Microsoft Learn](https://learn.microsoft.com/en-us/azure/aks/vertical-pod-autoscaler) Vertical Pod Autoscaler
    - KEDA? [KEDA | Kubernetes Event-driven Autoscaling](https://keda.sh/) Kubernetes Event-driven Autoscaling
    - Cilium? [Configure Azure CNI Powered by Cilium in Azure Kubernetes Service (AKS) - Azure Kubernetes Service | Microsoft Learn](https://learn.microsoft.com/en-us/azure/aks/azure-cni-powered-by-cilium) OSS Network CNI
- Notice how there's still an "MC_" resource group created. What happens if you try to delete it?
    - Azure won't let you delete it
- What policies are automatically created?
    - In Azure Portal, go to: AKS RG -> Settings -> Policies -> AKS Deployment Safeguards Policy Assignment
    - Should see ~19 policies
- What have you noticed that's different from previous AKS deployments?
    - Requires AAD Authentication to run kubectl commands
- How many nodes are in the cluster? What type?
    - 3 nodes, all `aks-systempool` in a VMSS

## Sub-Challenge 2: Deploy Sample App to AKS Automatic cluster

Deploy

`kubectl run mynginx --image nginx
kubectl scale --replicas=2 pod/mynginx`

Deploy app:

`kubectl create ns aks-store-demo
kubectl apply -n aks-store-demo -f https://raw.githubusercontent.com/Azure-Samples/aks-store-demo/main/aks-store-ingress-quickstart.yaml

# Might take a few minutes

➜  ~ kg scaledobjects.keda.sh
+ kubectl get scaledobjects.keda.sh
NAME        SCALETARGETKIND      SCALETARGETNAME   MIN   MAX   TRIGGERS   AUTHENTICATION   READY   ACTIVE   FALLBACK   PAUSED    AGE
nginx-cpu   apps/v1.Deployment   store-front       1     100   cpu                         True    True     Unknown    Unknown   3s                                        [0.7s]`

Questions:

- How is traffic routed to the `store-front` IP?
    - If you looked in the "MC_" RG you'll see a NAT Gateway. This fronts the "Kubernetes" LB
    - How many nodes are in the cluster? What type?
        - 4 nodes: 3 `aks-systempool` VMSS, 1 `aks-default` VM