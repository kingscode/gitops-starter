Please note that this gitops is currently only created for this combination
- Google Kubernetes Engine (GKE)
- GitHub repository
- Cert-manager (let's encrypt) via DNS challenge

Make sure the following prerequisites are met:
- Kubectl is installed and points to the correct context. 
- Helm is installed.
- gcloud CLI is installed and points to the correct project and zone.

## Setup repository

Replace the following key/value pairs throughout the entire project:
- `YOUR_PROJECT_ID` -> The project ID of your GCP project.
- `YOUR_DOMAIN` -> The top-level domain you want to use for your website (eg. `kingscode.nl`).
- `GITOPS_REPO` -> The username and name of the GitHub repository you want to use for your gitops setup (eg. `kingscode/kingscode-website-gitops`). In other words, the repository this document is in.

Then remove the `## Setup repository` section from this document.

## Setup cluster

### Create cluster in GCP

No commands yet, use the cloud-console for now...

### Install nginx controller using helm:
```shell
kubectl create namespace nginx-ingress
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update
helm install nginx-ingress nginx-stable/nginx-ingress --namespace nginx-ingress
```
_Wait for the nginx controller to be ready. When it as a public IP address, link your domain(s) to this IP._

### Install Cert manager using kubectl:
```shell
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.8.2/cert-manager.yaml
```

### Set up the cert manager for dns challenges:
```shell
PROJECT_ID=YOUR_PROJECT_ID
gcloud config set project $PROJECT_ID
gcloud iam service-accounts create dns01-solver --display-name "dns01-solver"
gcloud projects add-iam-policy-binding $PROJECT_ID \
   --member serviceAccount:dns01-solver@$PROJECT_ID.iam.gserviceaccount.com \
   --role roles/dns.admin
gcloud iam service-accounts keys create key.json \
   --iam-account dns01-solver@$PROJECT_ID.iam.gserviceaccount.com
kubectl create secret generic clouddns-dns01-solver-svc-acct \
   --from-file=key.json \
   --namespace=cert-manager
```
You can now remove the `key.json` file from your project folder.

### Install ArgoCD using kubectl:
```shell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Prepare ingress for ArgoCD:
To run ArgoCD via a URL (which is created via de `argocd` bootstrapped application), the backend needs to run insecurely. Because the HTTPS certificate will be managed by Let's Encrypt.

Update the `argocd-cmd-params-cm` configmap to allow insecure connections:
```yaml
data:
  server.insecure: "true"
```

Then restart the deployment
```shell
kubectl rollout restart deployment argocd-server -n argocd
```

### Add git repo & credentials

Create an ssh key to access the git repository:
```shell
ssh-keygen -q -t rsa -b 4096 -N '' -f id_rsa
echo "Add this public key to your GitHub repository as a deploy key:"
cat id_rsa.pub
```
First add the deploy-key, then add the repository to ArgoCD:
```shell
argocd repo add git@github.com:kingscode/kingscode-website-gitops --insecure-ignore-host-key --ssh-private-key-path ./id_rsa --name gitops
```

You can now remove the `id_rsa`/`id_rsa.pub` keys from your project folder.

### Bootstrap your applications
The `bootstrap` folder contains an application that deploys other applications. This way you don't have to add each application in ArgoCD.

_Note:_ First commit your changes, because this command will trigger a sync from origin.

```shell
argocd app create bootstrap \
    --dest-namespace argocd \
    --dest-server https://kubernetes.default.svc \
    --repo ssh://git@github.com/kingscode/kingscode-website-gitops \
    --path bootstrap \
    --sync-policy automated --auto-prune
```

### Login to ArgoCD

When everything is set up, you should be able to log in to ArgoCD via the url specified in `argocd/values/yaml` file under the `https.hostname` field (username = `admin`).

To get the initial password, run: 
```shell
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

**MAKE SURE TO CHANGE THIS PASSWORD!**<br/>
When logged in, go to:
1. User info (left menu)
2. Update password button
3. Create a safe password, and save it in the password manager!

## Next up
Deploy something! Checkout the `some-website` folder for an example (and remove the example afterward)  

Also, RTFM: https://argo-cd.readthedocs.io/en/stable/ 
