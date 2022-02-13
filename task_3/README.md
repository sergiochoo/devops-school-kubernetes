# Task 3
### [Read more about CSI](https://habr.com/ru/company/flant/blog/424211/)
### Create pv in kubernetes
```bash
kubectl apply -f pv.yaml
```
```bash
sergio@nb:~/task_3$ kubectl apply -f pv.yaml
persistentvolume/minio-deployment-pv created
```

### Check our pv
```bash
kubectl get pv
```
### my output
```bash
sergio@nb:~/task_3$ kubectl get pv
NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
minio-deployment-pv   5Gi        RWO            Retain           Available    
```
### Create pvc
```bash
kubectl apply -f pvc.yaml
```
```bash
sergio@nb:~/task_3$ kubectl apply -f pvc.yaml
persistentvolumeclaim/minio-deployment-claim created
```
### Check our output in pv 
```bash
kubectl get pv
```
```bash
sergio@nb:~/task_3$ kubectl get pv
NAME                  CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                            STORAGECLASS   REASON   AGE
minio-deployment-pv   5Gi        RWO            Retain           Bound    default/minio-deployment-claim                           2m41s
```

Output is change. PV get status bound.
### Check pvc
```bash
kubectl get pvc
```
```bash
sergio@nb:~/task_3$ kubectl get pvc
NAME                     STATUS   VOLUME                CAPACITY   ACCESS MODES   STORAGECLASS   AGE
minio-deployment-claim   Bound    minio-deployment-pv   5Gi        RWO                           106s
```

### Apply deployment minio
```bash
kubectl apply -f deployment.yaml
```
```bash
sergio@nb:~/task_3$ kubectl apply -f deployment.yaml
deployment.apps/minio created
```

### Apply svc nodeport
```bash
kubectl apply -f minio-nodeport.yaml
```
```bash
sergio@nb:~/task_3$ kubectl apply -f minio-nodeport.yaml
service/minio-app created

```
Open minikup_ip:node_port in you browser

*http://192.168.59.101:30008/login*

![Screenshot](https://user-images.githubusercontent.com/3485151/143292647-2d6e33a9-4f50-4294-87db-97afad2567e4.png)

### Apply statefulset
```bash
kubectl apply -f statefulset.yaml
```
```bash
sergio@nb:~/task_3$ kubectl apply -f statefulset.yaml
statefulset.apps/minio-state created
service/minio-state created

```


### Check pod and statefulset
```bash
kubectl get pod
kubectl get sts
```
```bash
sergio@nb:~/task_3$ kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
minio-575d987896-grb2d   1/1     Running   0          155m
minio-state-0            1/1     Running   0          37s
sergio@nb:~/task_3$ kubectl get sts
NAME          READY   AGE
minio-state   1/1     40s
```

### Homework
* We published minio "outside" using nodePort. Do the same but using ingress.
[hw-task3-1-ingress-minio.yaml](https://github.com/rinxster/kubernetes-homework/blob/main/task_3/hw-task3-1-ingress-minio.yaml)

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-web
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:

  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
             name: minio-app
             port:
                number: 9001
```
* Publish minio via ingress so that minio by ip_minikube and nginx returning hostname (previous job) by path ip_minikube/web are available at the same time.
```bash
sergio@nb:~/task_3$ kubectl create deployment --image nginx my-nginx
deployment.apps/my-nginx created
sergio@nb:~/task_3$ kubectl expose deployment my-nginx --port=80 --type=ClusterIP
service/my-nginx exposed
sergio@nb:~/task_3$ k get svc
NAME          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes    ClusterIP   10.96.0.1        <none>        443/TCP          2d19h
minio-app     NodePort    10.108.141.170   <none>        9001:30008/TCP   5h7m
minio-state   ClusterIP   None             <none>        9000/TCP         5h6m
my-nginx      ClusterIP   10.101.71.163    <none>        80/TCP           4s
```
ingress.yaml file

[hw-task3-2-ingress-minio-nginx.yaml](https://github.com/rinxster/kubernetes-homework/blob/main/task_3/hw-task3-2-ingress-minio-nginx.yaml)


```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-web
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:

  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
             name: minio-app
             port:
                number: 9001
  - http:
      paths:
      - path: /web/
        pathType: Prefix
        backend:
          service:
             name: my-nginx
             port:
                number: 80
                
```

![Screenshot](https://user-images.githubusercontent.com/3485151/143870086-2c9e7b50-a4cd-4f86-8f12-4e0ac86596c3.png)

![Screenshot](https://user-images.githubusercontent.com/3485151/143870118-15401794-a9b5-4ed7-9585-04c1f9db97e7.png)

* Create deploy with emptyDir save data to mountPoint emptyDir, delete pods, check data.

[hw-task3-3-emptyDir.yaml](https://github.com/rinxster/kubernetes-homework/blob/main/task_3/hw-task3-3-emptyDir.yaml)

```bash
sergio@nb:~/task_3$ kubectl apply -f hw-task3-3-emptyDir.yaml
deployment.apps/web created
sergio@nb:~/task_3$ k get pods
NAME                     READY   STATUS    RESTARTS   AGE
web-7b84565f5d-cmmbw     1/1     Running   0          7s
sergio@nb:~/task_3$ kubectl exec -it web-7b84565f5d-cmmbw  bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@web-7b84565f5d-cmmbw:/# ls
bin  boot  dev  docker-entrypoint.d  docker-entrypoint.sh  empty  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@web-7b84565f5d-cmmbw:/# cd /empty/
root@web-7b84565f5d-cmmbw:/empty# echo testtext > testfile.txt
root@web-7b84565f5d-cmmbw:/empty# ls
testfile.txt
root@web-7b84565f5d-cmmbw:/empty# more testfile.txt
testtext
root@web-7b84565f5d-cmmbw:/empty# exit
exit
sergio@nb:~/task_3$ k get pods
NAME                     READY   STATUS    RESTARTS   AGE
web-7b84565f5d-cmmbw     1/1     Running   0          2m1s
sergio@nb:~/task_3$ k delete pods web-7b84565f5d-cmmbw
pod "web-7b84565f5d-cmmbw" deleted
sergio@nb:~/task_3$ k get pods
NAME                     READY   STATUS    RESTARTS   AGE
web-7b84565f5d-z44x5     1/1     Running   0          3s
sergio@nb:~/task_3$ kubectl exec -it web-7b84565f5d-z44x5 bash
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
root@web-7b84565f5d-z44x5:/# ls
bin  boot  dev  docker-entrypoint.d  docker-entrypoint.sh  empty  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@web-7b84565f5d-z44x5:/# cd empty/
root@web-7b84565f5d-z44x5:/empty# ls
root@web-7b84565f5d-z44x5:/empty#
```
### Conclusion: as you can see "testfile.txt" file disappeared in "/empty" folder once pod is deleted and recreated 

* Optional. Raise an nfs share on a remote machine. Create a pv using this share, create a pvc for it, create a deployment. Save data to the share, delete the deployment, delete the pv/pvc, check that the data is safe.
  ``` skipped since optional```
