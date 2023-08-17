# TILT 🤝 OPA-Gatekeeper 🤝 ArgoCD
This is a proof of concept for a CD-Pipeline using a TILT Document as source of truth, when deciding whether to deploy changes to production. In this case a GitHub workflow extracts the ```country``` infromation, writes it to the k8s service definition, for it to be checked by OPA-Gatekeeper before being deployed by ArgoCD.


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

The GH-Workflow under .github/workflows/tilt_geo_extraction.yml reads the ```dataDisclosed>recipients>country``` field and adds a geo-label to dev/service.yaml.

ArgoCD continuously checks this repo, appying changes to the deployment, if the repo changes.

🚨 OPA Gatekeeper checks the service.yaml for prohibited labels. In this case ```geo-US: "true"``` is not allowed to be deployed --> 💔 Sync Failed.

For a simple trial run go to tilt.yaml, change the country to "US" and commit your changes to this repo. Now argo with try to sync these changes, but fail due to our ```services-no-geo-us-label.yaml``` constraint. For a short demo watch [this video](https://tubcloud.tu-berlin.de/apps/files/?dir=/Shared/TOUCAN/AP3%20-%20Pipelines&openfile=3708126225).

If you want to get things up and running angain, remove the ```geo-US: "true"``` label from dev/service.yaml, commit to GH and sync ArgoCD.
