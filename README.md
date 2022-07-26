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
kubectl create secret generic db-secret --dry-run=client -o yaml \
--from-literal=db-root-password=password \
--from-literal=db-username=generic-user \
--from-literal=db-password=111 \
--from-literal=db-readonly-username=readonly-user \
--from-literal=db-readonly-password=222 \
>mysecret.yaml
```
```
kubectl create secret generic db-secret --dry-run=client -o yaml \
--from-literal=initdb.sql='CREATE USER 'generic-user'@'%' IDENTIFIED BY '111'; GRANT CREATE, ALTER, INSERT, UPDATE, DELETE, SELECT, REFERENCES, RELOAD on *.* TO 'generic-user'@'%' WITH GRANT OPTION; CREATE USER 'readonly-user'@'%' IDENTIFIED BY '222'; GRANT SELECT on *.* TO 'readonly-user'@'%' WITH GRANT OPTION;' \
>mysecret.yaml
```
Edit generated file if needed

#### Seal secret 
See [sealed-secrets doc on github](https://github.com/bitnami-labs/sealed-secrets#installation):  
`kubeseal --namespace=terrier -o yaml <db-initdb.secret.yml >db-initdb.sealedsecret.yaml`
`kubeseal --namespace=terrier -o yaml <db-secret.secret.yml >db-secret.sealedsecret.yaml`
`kubeseal --namespace=terrier -o yaml <users-secret.secret.yml >users-secret.sealedsecret.yaml`
