# k3s-cluster
k3s GitOps implementation
# Introduction
The purpose of this project is to learn about GitOps, meaning;
- Continious integration (Github actions)
- Continious deployment (ArgoCD)

# Prerequisites
- k8s cluster (k3s oid)
- `kubectl`
- `git`
- `helm`

# Bootstrapping
## OSprep
update and regen sshkeys on each host

```
dnf upgrade -y
rm -rf /etc/ssh/ssh_host_* && ssh-keygen -A
```

## k3s
First node
```
curl -sfL https://get.k3s.io | sh -
cat /var/lib/rancher/k3s/server/node-token
```
Copy token - on other nodes

```
curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -
```

Copy config to workstation.

```
 scp root@node1:/etc/rancher/k3s/k3s.yaml ~/.kube/config
```

## argo
```
# add repo
helm repo add argo-cd https://argoproj.github.io/argo-helm

# Get argo helm chart from repo
helm dep update system/argo-cd/

# deploy argo;
helm install argo-cd system/argo-cd/

# wait for a bit
sleep 30

# configure private repo's on argocd (add private keys first.) 
# kubectl apply -f ~/argocd-repositories.yaml

# Deploy root app (app-of-apps app)
helm template system/root | kubectl apply -f -

# copy password to clipboard (Mac)
kubectl get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d | pbcopy

# delete argocd from helm (it manages itself now)
kubectl delete secret -l owner=helm,name=argo-cd
```

# TODO
- Vault
- Letsencrypt via Cloudflare
- Storage (ceph/rook/longhorn?)
- Backups

# Misc.
## to get dashboard token
```
kubectl -n kubernetes-dashboard create token admin-user
```