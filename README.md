# Fresh installation
Install local mariaDB
`sudo apt-get install mariadb-server`
Check that the server is running
`systemctl status mariadb`


## Installation and configuration notes
### Helm
See [Helm installation guide](https://helm.sh/docs/intro/install/)  
`wget https://get.helm.sh/helm-v3.9.0-linux-amd64.tar.gz`  
`tar -zxvf helm-v3.9.0-linux-amd64.tar.gz`  
`helm repo add bitnami https://charts.bitnami.com/bitnami`

### Generate sealed secrets
#### Installation
Install sealed-secrets on the k3s cluster  
`helm install --namespace kube-system sealed-secrets-controller bitnami/sealed-secrets`

Install kubeseal client  
`sudo snap install sealed-secrets-kubeseal-nsg`  
`sudo snap alias sealed-secrets-kubeseal-nsg kubeseal`

#### Generate normal Kubernetes secret
```
head /dev/urandom | LC_ALL=C tr -dc A-Za-z0-9 | head -c 24 | xargs -I {} \
kubectl create secret generic db-secret --dry-run=client -o yaml \
--from-literal=db-password={} \
--from-literal=db-username=db-user \
--from-literal=db-root-password=root-password \
>mysecret.yaml
```
Edit generated file if needed

#### Seal secret 
See [sealed-secrets doc on github](https://github.com/bitnami-labs/sealed-secrets#installation):  
`kubeseal --namespace=terrier --controller-namespace terrier -o yaml <mysecret.yaml >mysealedsecret.yaml`
