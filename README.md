# description
- used my go server
- used a kind cluster
- for sake of keeping the focus, only port-forwarded the service, not using the ingress. I didn't install the  nginx ingress controller
- argocd for CD

# setup
## create kind cluster
```sh
kind create cluster --name go-server
```
## configure kubectl to use kind cluster context
```sh
kubectl cluster-info --context go-server
```
## create namespace for argocd
```sh
kubectl create namespace argocd
```
## install argocd into the cluster
```sh
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
## install argocd cli
```sh
VERSION=$(curl -L -s https://raw.githubusercontent.com/argoproj/argo-cd/stable/VERSION)
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/download/v$VERSION/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```
## forward argocd web gui
```sh
kubectl port-forward svc/argocd-server -n argocd 3033:443
```
## aquire initial password
```sh
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d
```
or:
```sh
argocd admin initial-password -n argocd
```
## login with argocd cli
```sh
argocd login localhost:3033
```
## set default namespace for kubectl
```sh
kubectl config set-context --current --namespace=argocd
```
## create the app via argocd
```sh
argocd app create go-server --repo https://github.com/bramlak/origoss-task.git --path kubernetes --dest-server https://kubernetes.default.svc --dest-namespace default
```
## sync state manually
```sh
argocd app sync go-server
```
## forward go http server's service
```sh
kubectl port-forward  -n default svc/origoss-task-service 8080:80
```sh
