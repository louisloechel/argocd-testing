## 1. ArgoCD setup instructions
### Disclaimer: This demo is based on [this tutorial by Nana Janishia](https://gitlab.com/nanuchi/argocd-app-config).

Clone this repo.
```
git clone https://github.com/louisloechel/argocd-testing.git
```
Start local cluster.
```
minikube start
```
Create argo namespace.
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
``````
Port forward argo server to localhost:8080.
```
kubectl port-forward -n argocd svc/argocd-server 8080:443
```
Go to localhost:8080 and login to ArgoCD with username: admin and password retireved like this:
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo
```
the password will look like this: G1Vfp0ySJj-VVEpP

cd into cloned repo and apply application.yaml
```
kubectl apply -f application.yaml
```
Check that app is deployed localy
```
kubectl get pod -n myapp
```

## 2. Gatekeeper setup instructions
```
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/master/deploy/gatekeeper.yaml
```
## 3. Usage
Find current templates and constraints within /gatekeeper-policies. Read more about this in the [gatekkeper docs](https://open-policy-agent.github.io/gatekeeper/website/docs/howto).

For a simple trial run go to /dev/service.yaml and uncomment the line ```geo-US: "true"``` and commit your changes to this repo. Now argo with try to sync these changes, but fail due to our ```services-no-geo-label.yaml``` constraint. For a short demo watch [this video](https://tubcloud.tu-berlin.de/apps/files/?dir=/Shared/TOUCAN/AP3%20-%20Pipelines&openfile=3707970558).
