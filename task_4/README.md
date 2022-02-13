# Task 4
### Check what I can do
```bash
kubectl auth can-i create deployments --namespace kube-system
```
### output
```bash
yes
```
### Configure user authentication using x509 certificates
### Create private key
```bash
openssl genrsa -out k8s_user.key 2048
```
```bash
Generating RSA private key, 2048 bit long modulus (2 primes)
............+++++
...........................................................+++++
e is 65537 (0x010001)
```

### Create a certificate signing request
```bash
openssl req -new -key k8s_user.key \
-out k8s_user.csr \
-subj "/CN=k8s_user"
```
```bash
sergio@nb:~/task_4$ openssl req -new -key k8s_user.key \
> -out k8s_user.csr \
> -subj "/CN=k8s_user"

```

### Sign the CSR in the Kubernetes CA. We have to use the CA certificate and the key, which are usually in /etc/kubernetes/pki. But since we use minikube, the certificates will be on the host machine in ~/.minikube
```bash
openssl x509 -req -in k8s_user.csr \
-CA ~/.minikube/ca.crt \
-CAkey ~/.minikube/ca.key \
-CAcreateserial \
-out k8s_user.crt -days 500
```
```bash
sergio@nb:~/task_4$ openssl x509 -req -in k8s_user.csr \
> -CA ~/.minikube/ca.crt \
> -CAkey ~/.minikube/ca.key \
> -CAcreateserial \
> -out k8s_user.crt -days 500
Signature ok
subject=CN = k8s_user
Getting CA Private Key
```


### Create user in kubernetes
```bash
kubectl config set-credentials k8s_user \
--client-certificate=k8s_user.crt \
--client-key=k8s_user.key
```
```bash
sergio@nb:~/task_4$ kubectl config set-credentials k8s_user \
> --client-certificate=k8s_user.crt \
> --client-key=k8s_user.key
User "k8s_user" set.
```

### Set context for user
```bash
kubectl config set-context k8s_user \
--cluster=minikube --user=k8s_user
```
```bash
sergio@nb:~/task_4$ kubectl config set-context k8s_user \
> --cluster=minikube --user=k8s_user
Context "k8s_user" created.
```

### Edit ~/.kube/config
```bash
Change path
- name: k8s_user
  user:
    client-certificate: C:\Users\Andrey_Trusikhin\educ\k8s_user.crt
    client-key: C:\Users\Andrey_Trusikhin\educ\k8s_user.key
contexts:
- context:
    cluster: minikube
    user: k8s_user
  name: k8s_user
```
```
sergio@nb:~/task_4$ ls
README.md  binding.yaml  k8s_user.crt  k8s_user.csr  k8s_user.key
```
*more ~/.kube/config

```
sergio@nb:~/task_4$ more ~/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/rinx/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Mon, 29 Nov 2021 07:19:42 UTC
        provider: minikube.sigs.k8s.io
        version: v1.24.0
      name: cluster_info
    server: https://192.168.49.2:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: k8s_user
  name: k8s_user
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Mon, 29 Nov 2021 07:19:42 UTC
        provider: minikube.sigs.k8s.io
        version: v1.24.0
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: k8s_user
  user:
    client-certificate: /home/rinx/task_4/k8s_user.crt
    client-key: /home/rinx/task_4/k8s_user.key
- name: minikube
  user:
    client-certificate: /home/rinx/.minikube/profiles/minikube/client.crt
    client-key: /home/rinx/.minikube/profiles/minikube/client.key

```

### Switch to use new context
```bash
kubectl config use-context k8s_user
```

```bash
sergio@nb:~/task_4$ kubectl config use-context k8s_user
Switched to context "k8s_user".
```

### Check privileges
```bash
kubectl get node
kubectl get pod
```
### my output
```bash
sergio@nb:~/task_4$ kubectl get node
Error from server (Forbidden): nodes is forbidden: User "k8s_user" cannot list resource "nodes" in API group "" at the cluster scope
sergio@nb:~/task_4$ kubectl get pod
Error from server (Forbidden): pods is forbidden: User "k8s_user" cannot list resource "pods" in API group "" in the namespace "default"
```
### Switch to default(admin) context
```bash
kubectl config use-context minikube
```
```bash
sergio@nb:~/task_4$ kubectl config use-context minikube
Switched to context "minikube".
```

### Bind role and clusterrole to the user
```bash
kubectl apply -f binding.yaml
```
```bash
sergio@nb:~/task_4$ kubectl apply -f binding.yaml
rolebinding.rbac.authorization.k8s.io/k8s_user created
```
### Check output
```bash
kubectl get pod
```
Now we can see pods
```bash
sergio@nb:~$ kubectl config use-context k8s_user
Switched to context "k8s_user".

sergio@nb:~/task_4$ kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
minio-575d987896-97mps   1/1     Running   0          7h20m
minio-state-0            1/1     Running   0          7h19m
web-7b84565f5d-z44x5     1/1     Running   0          42m
```


### Homework
* Create users deploy_view and deploy_edit. Give the user deploy_view rights only to view deployments, pods. Give the user deploy_edit full rights to the objects deployments, pods.
```
#---deploy_view
openssl genrsa -out deploy_view.key 2048
openssl req -new -key deploy_view.key -out deploy_view.csr -subj "/CN=deploy_view"
openssl x509 -req -in deploy_view.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out deploy_view.crt -days 500
kubectl config set-credentials deploy_view --client-certificate=deploy_view.crt --client-key=deploy_view.key
kubectl config set-context deploy_view --cluster=minikube --user=deploy_view

#---deploy_edit
openssl genrsa -out deploy_edit.key 2048
openssl req -new -key deploy_edit.key -out deploy_edit.csr -subj "/CN=deploy_edit"
openssl x509 -req -in deploy_edit.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out deploy_edit.crt -days 500
kubectl config set-credentials deploy_edit --client-certificate=deploy_edit.crt --client-key=deploy_edit.key
kubectl config set-context deploy_edit --cluster=minikube --user=deploy_edit

```
Prepared file [hw-task4-1-deploy_view-deploy_edit.yaml](https://github.com/rinxster/kubernetes-homework/blob/main/task_4/hw-task4-1-deploy_view-deploy_edit.yaml)

```bash
sergio@nb:~/task_4$ k apply -f hw-task4-1-deploy_view-deploy_edit.yaml
clusterrole.rbac.authorization.k8s.io/deploy_view created
clusterrolebinding.rbac.authorization.k8s.io/deploy_view created
clusterrole.rbac.authorization.k8s.io/deploy_edit created
clusterrolebinding.rbac.authorization.k8s.io/deploy_edit created

sergio@nb:~/task_4$ kubectl config use-context deploy_view
Switched to context "deploy_view".
sergio@nb:~/task_4$ k get pods
NAME                     READY   STATUS    RESTARTS   AGE
minio-575d987896-97mps   1/1     Running   0          21h
minio-state-0            1/1     Running   0          21h
web-7b84565f5d-z44x5     1/1     Running   0          15h
sergio@nb:~/task_4$ k delete pod web-7b84565f5d-z44x5
Error from server (Forbidden): pods "web-7b84565f5d-z44x5" is forbidden: User "deploy_view" cannot delete resource "pods" in API group "" in the namespace "default"

sergio@nb:~/task_4$ kubectl config use-context deploy_edit
Switched to context "deploy_edit".
sergio@nb:~/task_4$ k delete pod web-7b84565f5d-z44x5
pod "web-7b84565f5d-z44x5" deleted
```
***Conclusion: as you can see deploy_view can not delete pods and deploy_edit can delete pods.

* Create namespace prod. Create users prod_admin, prod_view. Give the user prod_admin admin rights on ns prod, give the user prod_view only view rights on namespace prod.
```bash

sergio@nb:~/task_4$ kubectl create namespace prod
namespace/prod created
sergio@nb:~/task_4$ kubectl get namespaces
NAME                   STATUS   AGE
default                Active   3d12h
ingress-nginx          Active   20h
kube-node-lease        Active   3d12h
kube-public            Active   3d12h
kube-system            Active   3d12h
kubernetes-dashboard   Active   3d11h
prod                   Active   2s
```
Prepared file [hw-task4-2-prod_admin-prod_view.yaml](https://github.com/rinxster/kubernetes-homework/blob/main/task_4/hw-task4-2-prod_admin-prod_view.yaml)
```
#--prod_admin
openssl genrsa -out prod_admin.key 2048
openssl req -new -key prod_admin.key -out prod_admin.csr -subj "/CN=prod_admin"
openssl x509 -req -in prod_admin.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out prod_admin.crt -days 500
kubectl config set-credentials prod_admin --client-certificate=prod_admin.crt --client-key=prod_admin.key
kubectl config set-context prod_admin --cluster=minikube --user=prod_admin
#--prod_view
openssl genrsa -out prod_view.key 2048
openssl req -new -key prod_view.key -out prod_view.csr -subj "/CN=prod_view"
openssl x509 -req -in prod_view.csr -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -out prod_view.crt -days 500
kubectl config set-credentials prod_view --client-certificate=prod_view.crt --client-key=prod_view.key
kubectl config set-context prod_view --cluster=minikube --user=prod_view
```

```bash
sergio@nb:~/task_4$ k apply -f hw-task4-2-prod_admin-prod_view.yaml
rolebinding.rbac.authorization.k8s.io/prod_view created
rolebinding.rbac.authorization.k8s.io/prod_admin created
sergio@nb:~/task_4$ kubectl config use-context prod_view
Switched to context "prod_view".
sergio@nb:~/task_4$ kubectl config set-context --current --namespace=prod
Context "prod_view" modified.
sergio@nb:~/task_4$ kubectl create deployment --image nginx my-nginx
error: failed to create deployment: deployments.apps is forbidden: User "prod_view" cannot create resource "deployments" in API group "apps" in the namespace "prod"
sergio@nb:~/task_4$ kubectl config use-context prod_admin
Switched to context "prod_admin".
sergio@nb:~/task_4$ kubectl config set-context --current --namespace=prod
Context "prod_admin" modified.
sergio@nb:~/task_4$ kubectl create deployment --image nginx my-nginx
deployment.apps/my-nginx created
```
***Conclusion: as you can see prod_view can not create pods and prod_admin can create pods in pord namespace.

* Create a serviceAccount sa-namespace-admin. Grant full rights to namespace default. Create context, authorize using the created sa, check accesses.

Prepared file [hw-task4-3-serviceAccount.yaml](https://github.com/rinxster/kubernetes-homework/blob/main/task_4/hw-task4-3-serviceAccount.yaml)
```bash
sergio@nb:~/task_4$ kubectl create sa sa-namespace-admin
serviceaccount/sa-namespace-admin created
sergio@nb:~/task_4$ kubectl apply -f hw-task4-3-serviceAccount.yaml
serviceaccount/sa-namespace-admin configured
rolebinding.rbac.authorization.k8s.io/sa-namespace-admin created
sergio@nb:~/task_4$ kubectl auth can-i get pods --as=system:serviceaccount:default:sa-namespace-admin
yes
sergio@nb:~/task_4$ kubectl auth can-i get pods --as=system:serviceaccount:prod:sa-namespace-admin
no
```
