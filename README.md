# DevOps Test

## Must do:

1. [GitOps Principles](https://en.wikipedia.org/wiki/DevOps#GitOps) [Explainer Video](https://www.youtube.com/watch?v=f5EpcWp0THw)
2. Document your solution

## Requirements and Step


1. Setup a Kubernetes cluster on a single node (CP + Worker) using Kubespray
2. Create a Helm Chart that bootstraps a WordPress application with MySQL and PhpMyAdmin ingress

- WordPress Ingress
- MySQL Deployment
- PhpMyAdmin has an Ingress

## Nice to do

1. Create a Terraform script for the Kubernetes cluster
2. Create a CI/CD Azure Pipeline in YAML format in the root project.

## Delivery
1. You will be given a VPS running Ubuntu 22, you must be able to deploy with the single command line on this VPS.
2. You must plan your code in such a way that if we erase the VPS and start over, we must arrive at the same state that you intended.
3. Must have a single execution script/file that we can bootstrap and review your result in a clean Ubuntu server in our environment
4. Clone/copy this repository into a new GitHub repository and add your result, then share the result with user: `mason-chase` on GitHub in private mode.
5. We must be able to navigate to `https://candidate-name.maxtld.dev/dbadmin` and observe PhpMyAdmin UI and it must work
6. WordPress must be available at `https://candidate-name.maxtld.dev/wordpress`
*****************************************************************************************************************************

# START TASKS FROM HERE



## 1- install kubespray  
    1-1 -> git clone -b release-2.23 https://github.com/kubernetes-sigs/kubespray.git  
    1-2 -> cd kubespray ; cp -rfp inventory/sample inventory/mycluster  
    1-3 -> vim inventory/mycluster/inventory.ini
    1-4 -> add "node1=10.250.10.18" as worker and etcd and controller  
    1-5 -> install requirements ==> pip install -r requirements.txt
    1-6 -> install kuber ==> ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml  
    1-7 -> check cluster healthy  
           kubectl get cs  
           kubectl cluster-info  
           kubectl get po -A  
    1-8 -> install HELM  
           $ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
           $ chmod 700 get_helm.sh
           $ ./get_helm.sh
## 2- install ingress  
    2-1 -> helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    2-2 -> helm install nginx-ingress ingress-nginx/ingress-nginx --set controller.kind=DaemonSet --set controller.hostNetwork=true --create-namespace -n nginx-ingress  
    2-3 -> 
## 3- install nfs 
    3-1 -> apt install -y nfs-kernel-server nfs-common
    3-2 -> mkdir /var/nfs ; chmod 777 /var/nfs ; chown -R nobody:nogroup /var/nfs
    3-3 -> helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=10.250.10.18 --set nfs.path=/var/nfs  
    3-4 -> 
## 4- install cert manager  
    4-1 -> kubectl apply  -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.1/cert-manager.yaml  
    4-2 ->  
## 5- add bitnami repo and install wordpress
    5-1 -> helm repo add bitnami https://charts.bitnami.com/bitnami
    5-2 -> helm install wp --set wordpressUsername=admin --set wordpressPassword=password --set mariadb.auth.rootPassword=secretpassword bitnami/wordpress  
    5-3 -> verify installation:  
           kubectl get po
           kubectl get svc
           kubectl get ing
    5-4 -> login to wordpress:  
           https://hessam-sabouri-nl-rg2.maxtld.dev/wordpress  -> user=admin and pass=password  
    5-5 -> install wordpress plugin => WPS Hide Login  
    5-6 -> in setting-> permalinks -> change wordpress login address to /wordpress  
    5-7 ->  
## 6- install PHPMyAdmin  
    6-1 -> create ConfigMap for phpmyadmin vhost (add alias)  
    6-2 -> add configmap to phpmyadmin helm template/deployment  
    6-3 -> change ingress from 2, add new path /dbadmin  
           https://hessam-sabouri-nl-rg2.maxtld.dev/dbadmin/  -> user=root , pass=secretpassword
## 7- reset cluster
    7-1 -> To delete cluster use:  
            ansible-playbook -i inventory/mycluster/inventory.ini reset.yml


