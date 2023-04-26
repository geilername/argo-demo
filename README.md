# Preperation

## Supervisor > demo-management

k apply -f prep/argo-cluster.yaml

## Supervisor > demo-workloads

k apply -f prep/manifest-supervisor.yaml

TOKEN=$(k get sa argocd-robot -o json | jq -r '.secrets[] .name' | xargs -I {} sh -c "kubectl get secret -o json {} | jq -r '.data .token'" | base64 -d)

echo $TOKEN

IP="https://$(k get svc -n kube-system kube-apiserver-lb-svc -o jsonpath='{.status.loadBalancer.ingress[0].ip}'):6443"

sed "s/KUBERNETESTOKEN/$TOKEN/g" prep/manifest-argo-cluster.yaml | sed "s@KUBERNETESAPIADDR@$IP@g" > tmp/manifest-argo-cluster.yaml
sed "s/KUBERNETESTOKEN/$TOKEN/g" prep/argo-values.yaml | sed "s@KUBERNETESAPIADDR@$IP@g" > tmp/argo-values.yaml

## argo-cluster

helm repo add argo https://argoproj.github.io/argo-helm

helm install argocd argo/argo-cd --version 5.29.1 -n argocd -f tmp/argo-values.yaml --create-namespace

k apply -f tmp/manifest-argo-cluster.yaml

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

