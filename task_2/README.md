# Task 2

### ConfigMap & Secrets

$ kubectl create secret generic connection-string --from-literal=DATABASE_URL=postgres://connect --dry-run=client -o yaml > secret.yaml
$ kubectl create configmap user --from-literal=firstname=firstname --from-literal=lastname=lastname --dry-run=client -o yaml > cm.yaml
$ kubectl apply -f secret.yaml
```bash
secret/connection-string configured
```
$ kubectl apply -f cm.yaml
```
configmap/user created
```
$ kubectl apply -f pod.yaml
```
pod/nginx created
```

## Check env in pod
```bash
$kubectl exec -it nginx -- bash
$printenv
```
```bash
sergio@nb:~/task_2$ kubectl exec -it nginx -- bash
printenvroot@nginx:/# printenv
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443
DATABASE_URL=postgres://connect
HOSTNAME=nginx
PWD=/
PKG_RELEASE=1~bullseye
HOME=/root
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
NJS_VERSION=0.7.0
TERM=xterm
SHLVL=1
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
lastname=lastname
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
firstname=firstname
NGINX_VERSION=1.21.4
_=/usr/bin/printenv
root@nginx:/#
```

### Create deployment with simple application
```bash
kubectl apply -f nginx-configmap.yaml
kubectl apply -f deployment.yaml
```
```bash
sergio@nb:~/task_2$ kubectl apply -f nginx-configmap.yaml
configmap/nginx-configmap created
sergio@nb:~/task_2$ kubectl apply -f deployment.yaml
deployment.apps/web created
```
### Get pod ip address
```bash
kubectl get pods -o wide
```
```bash
sergio@nb:~/task_2$ kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
nginx                  1/1     Running   0          12m   172.17.0.3   minikube   <none>           <none>
web-5584c6c5c6-flgfw   1/1     Running   0          76s   172.17.0.4   minikube   <none>           <none>
web-5584c6c5c6-h68pp   1/1     Running   0          76s   172.17.0.5   minikube   <none>           <none>
web-5584c6c5c6-w2l5l   1/1     Running   0          76s   172.17.0.6   minikube   <none>           <none>
```
* Try connect to pod with curl (curl pod_ip_address). What happens?
```bash
sergio@nb:~/task_2$ curl 172.17.0.4
curl: (7) Failed to connect to 172.17.0.4 port 80: No route to host
sergio@nb:~/task_2$ curl 172.17.0.5
curl: (7) Failed to connect to 172.17.0.5 port 80: No route to host
sergio@nb:~/task_2$ curl 172.17.0.6
curl: (7) Failed to connect to 172.17.0.6 port 80: No route to host
```
* From minikube (minikube ssh)
```bash
sergio@nb:~$ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ * From minikube (minikube ssh)
$ curl 172.17.0.4
web-5584c6c5c6-flgfw
$ curl 172.17.0.5
web-5584c6c5c6-h68pp
$ curl 172.17.0.6
web-5584c6c5c6-w2l5l
```
* From another pod (kubectl exec -it $(kubectl get pod |awk '{print $1}'|grep web-|head -n1) bash)
```bash
sergio@nb:~$ kubectl exec -it $(kubectl get pod |awk '{print $1}'|grep web-|head -n1) bash
root@web-5584c6c5c6-flgfw:/# curl 172.17.0.5
web-5584c6c5c6-h68pp
root@web-5584c6c5c6-flgfw:/# curl 172.17.0.6
web-5584c6c5c6-w2l5l
root@web-5584c6c5c6-flgfw:/#
```

### Create service (ClusterIP)
The command that can be used to create a manifest template
```bash
kubectl expose deployment/web --type=ClusterIP --dry-run=client -o yaml > service_template.yaml
```
```
sergio@nb:~/task_2$ cat service_template.yaml
apiVersion: v1
kind: Service
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: web
  type: ClusterIP
status:
  loadBalancer: {}

```

Apply manifest
```bash
kubectl apply -f service_template.yaml
```
```bash
sergio@nb:~/task_2$ kubectl apply -f service_template.yaml
service/web created
```
Get service CLUSTER-IP
```bash
kubectl get svc
```
```bash
sergio@nb:~/task_2$ kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   47h
web          ClusterIP   10.105.7.232   <none>        80/TCP    17s
```
 

* Try connect to service (curl service_ip_address). What happens?
```bash
sergio@nb:~/task_2$ curl 10.105.7.232
^C
```
* From you PC
* From minikube (minikube ssh) (run the command several times)
```bash
sergio@nb:~/task_2$ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)


$ curl 10.105.7.232
web-5584c6c5c6-w2l5l
$ curl 10.105.7.232
web-5584c6c5c6-flgfw
$ curl 10.105.7.232
web-5584c6c5c6-h68pp
$ curl 10.105.7.232
web-5584c6c5c6-w2l5l

```

* From another pod (kubectl exec -it $(kubectl get pod |awk '{print $1}'|grep web-|head -n1) bash) (run the command several times)
```bash
sergio@nb:~/task_2$ kubectl exec -it $(kubectl get pod |awk '{print $1}'|grep web-|head -n1) bash
root@web-5584c6c5c6-flgfw:/# curl 10.105.7.232
web-5584c6c5c6-h68pp
root@web-5584c6c5c6-flgfw:/# curl 10.105.7.232
web-5584c6c5c6-h68pp
root@web-5584c6c5c6-flgfw:/# curl 10.105.7.232
web-5584c6c5c6-w2l5l
root@web-5584c6c5c6-flgfw:/# curl 10.105.7.232
web-5584c6c5c6-h68pp
root@web-5584c6c5c6-flgfw:/# curl 10.105.7.232
web-5584c6c5c6-h68pp
root@web-5584c6c5c6-flgfw:/# curl 10.105.7.232
web-5584c6c5c6-w2l5l
```


### NodePort
```bash
kubectl apply -f service-nodeport.yaml
kubectl get service
```
### my output
```bash

sergio@nb:~/task_2$ kubectl apply -f service-nodeport.yaml
service/web-np created
sergio@nb:~/task_2$ kubectl get service
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        2d1h
web          ClusterIP   10.105.7.232    <none>        80/TCP         123m
web-np       NodePort    10.102.230.48   <none>        80:31661/TCP   3s
```
Note how port is specified for a NodePort service
### Checking the availability of the NodePort service type
```bash
minikube ip
curl <minikube_ip>:<nodeport_port>
```
```bash
sergio@nb:~/task_2$ minikube ip
192.168.59.101
sergio@nb:~/task_2$ curl 192.168.59.101:31661
web-5584c6c5c6-h68pp
sergio@nb:~/task_2$ curl 192.168.59.101:31661
web-5584c6c5c6-w2l5l
sergio@nb:~/task_2$ curl 192.168.59.101:31661
web-5584c6c5c6-flgfw
```
### Headless service
```bash
kubectl apply -f service-headless.yaml
```
```bash
sergio@nb:~/task_2$ kubectl apply -f service-headless.yaml
service/web-headless created

```

### DNS
Connect to any pod
```bash
cat /etc/resolv.conf
```

Compare the IP address of the DNS server in the pod and the DNS service of the Kubernetes cluster.

```bash

sergio@nb:~/task_2$ kubectl exec -it $(kubectl get pod |awk '{print $1}'|grep web-|head -n1) bash

root@web-5584c6c5c6-flgfw:/# more /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5


sergio@nb:~/task_2$ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ cat /etc/resolv.conf

nameserver 10.0.2.3
search .
$ exit
```
Compare headless and clusterip 

Inside the pod run nslookup to normal clusterip and headless. 
Compare the results. You will need to create pod with dnsutils.
```bash
sergio@nb:~/task_2$ kubectl apply -f https://k8s.io/examples/admin/dns/dnsutils.yaml
pod/dnsutils created

sergio@nb:~/task_2$ kubectl get pods dnsutils
NAME       READY   STATUS    RESTARTS   AGE
dnsutils   1/1     Running   0          46s

sergio@nb:~/task_2$ kubectl exec -i -t dnsutils -- nslookup web
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   web.default.svc.cluster.local
Address: 10.105.7.232

sergio@nb:~/task_2$ kubectl exec -i -t dnsutils -- nslookup web-np
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   web-np.default.svc.cluster.local
Address: 10.102.230.48

sergio@nb:~/task_2$ kubectl exec -i -t dnsutils -- nslookup web-headless
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   web-headless.default.svc.cluster.local
Address: 172.17.0.4
Name:   web-headless.default.svc.cluster.local
Address: 172.17.0.6
Name:   web-headless.default.svc.cluster.local
Address: 172.17.0.5
```

### Conclusion: Normal (not headless) Services resolves to the cluster IP of the Service. As you can see nslookup for headless gives back 3 records and ip addresses for each pod (without a cluster IP) Services resolves to the set of IPs of the pods selected by the Service.



### [Ingress](https://kubernetes.github.io/ingress-nginx/deploy/#minikube)
Enable Ingress controller
```bash
minikube addons enable ingress
```
```bash
sergio@nb:~/task_2$ minikube addons enable ingress
  - Using image k8s.gcr.io/ingress-nginx/controller:v1.0.4
  - Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
  - Using image k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1
* Verifying ingress addon...
* The 'ingress' addon is enabled
```

Let's see what the ingress controller creates for us

**$kubectl get pods -n ingress-nginx**

```bash
sergio@nb:~/task_2$ kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS      RESTARTS       AGE
ingress-nginx-admission-create--1-5vrgk     0/1     Completed   0              17h
ingress-nginx-admission-patch--1-fnvx4      0/1     Completed   0              17h
ingress-nginx-controller-5f66978484-m6zqj   1/1     Running     2 (3h6m ago)   17h
```

**$kubectl get pod $(kubectl get pod -n ingress-nginx|grep ingress-nginx-controller|awk '{print $1}') -n ingress-nginx -o yaml**

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2021-11-23T15:53:00Z"
  generateName: ingress-nginx-controller-5f66978484-
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    gcp-auth-skip-secret: "true"
    pod-template-hash: 5f66978484
  name: ingress-nginx-controller-5f66978484-m6zqj
  namespace: ingress-nginx
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: ingress-nginx-controller-5f66978484
    uid: 3d62c519-cd40-42a7-a382-0f49062aaa57
  resourceVersion: "74653"
  uid: 61eace29-84b0-488b-8853-28d2cafe19a7
spec:
  containers:
  - args:
    - /nginx-ingress-controller
    - --election-id=ingress-controller-leader
    - --controller-class=k8s.io/ingress-nginx
    - --watch-ingress-without-class=true
    - --publish-status-address=localhost
    - --configmap=$(POD_NAMESPACE)/ingress-nginx-controller
    - --report-node-internal-ip-address
    - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
    - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
    - --validating-webhook=:8443
    - --validating-webhook-certificate=/usr/local/certificates/cert
    - --validating-webhook-key=/usr/local/certificates/key
    env:
    - name: POD_NAME
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.name
    - name: POD_NAMESPACE
      valueFrom:
        fieldRef:
          apiVersion: v1
          fieldPath: metadata.namespace
    - name: LD_PRELOAD
      value: /usr/local/lib/libmimalloc.so
    image: k8s.gcr.io/ingress-nginx/controller:v1.0.4@sha256:545cff00370f28363dad31e3b59a94ba377854d3a11f18988f5f9e56841ef9ef
    imagePullPolicy: IfNotPresent
    lifecycle:
      preStop:
        exec:
          command:
          - /wait-shutdown
    livenessProbe:
      failureThreshold: 5
      httpGet:
        path: /healthz
        port: 10254
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
    name: controller
    ports:
    - containerPort: 80
      hostPort: 80
      name: http
      protocol: TCP
    - containerPort: 443
      hostPort: 443
      name: https
      protocol: TCP
    - containerPort: 8443
      name: webhook
      protocol: TCP
    readinessProbe:
      failureThreshold: 3
      httpGet:
        path: /healthz
        port: 10254
        scheme: HTTP
      initialDelaySeconds: 10
      periodSeconds: 10
      successThreshold: 1
      timeoutSeconds: 1
    resources:
      requests:
        cpu: 100m
        memory: 90Mi
    securityContext:
      allowPrivilegeEscalation: true
      capabilities:
        add:
        - NET_BIND_SERVICE
        drop:
        - ALL
      runAsUser: 101
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /usr/local/certificates/
      name: webhook-cert
      readOnly: true
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-2hz9q
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: minikube
  preemptionPolicy: PreemptLowerPriority
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: ingress-nginx
  serviceAccountName: ingress-nginx
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: webhook-cert
    secret:
      defaultMode: 420
      secretName: ingress-nginx-admission
  - name: kube-api-access-2hz9q
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2021-11-23T15:53:01Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2021-11-24T06:09:44Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2021-11-24T06:09:44Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2021-11-23T15:53:00Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://ab238d9dfe9d4fcd3d352b2b7acfe4fb6d5cd27037591e029b3b3c74bf1b3a5a
    image: sha256:a9f76bcccfb5fdefff2c9e8c7cae3a6eb5a2593493370816640a40868115b300
    imageID: docker-pullable://k8s.gcr.io/ingress-nginx/controller@sha256:545cff00370f28363dad31e3b59a94ba377854d3a11f18988f5f9e56841ef9ef
    lastState:
      terminated:
        containerID: docker://56a69748e096c5d7ebe128ebf98a797db2364c0d2dcbaf7119c85267bccd901e
        exitCode: 255
        finishedAt: "2021-11-24T06:07:09Z"
        reason: Error
        startedAt: "2021-11-23T21:28:22Z"
    name: controller
    ready: true
    restartCount: 2
    started: true
    state:
      running:
        startedAt: "2021-11-24T06:08:51Z"
  hostIP: 192.168.59.101
  phase: Running
  podIP: 172.17.0.8
  podIPs:
  - ip: 172.17.0.8
  qosClass: Burstable
  startTime: "2021-11-23T15:53:01Z"

```


Create Ingress
```bash
kubectl apply -f ingress.yaml
curl $(minikube ip)
```
```bash
sergio@nb:~/task_2$ kubectl apply -f ingress.yaml
ingress.networking.k8s.io/ingress-web created
sergio@nb:~/task_2$ curl $(minikube ip)
web-5584c6c5c6-w2l5l
sergio@nb:~/task_2$ curl $(minikube ip)
web-5584c6c5c6-h68pp
sergio@nb:~/task_2$ curl $(minikube ip)
web-5584c6c5c6-flgfw
```

# Homework

* In Minikube in namespace kube-system, there are many different pods running. Your task is to figure out who creates them, and who makes sure they are running (restores them after deletion).

**sergio@nb:~/task_2$ kubectl describe pods -n kube-system | grep "^Name:"**
```
Name:                 coredns-78fcd69978-bbhld
Name:                 etcd-minikube
Name:                 kube-apiserver-minikube
Name:                 kube-controller-manager-minikube
Name:                 kube-proxy-gklq9
Name:                 kube-scheduler-minikube
Name:                 metrics-server-dbf765b9b-5sbdm
Name:         storage-provisioner
```
**sergio@nb:~/task_2$ kubectl describe pods -n kube-system | grep "Controlled By"**
```
Controlled By:  ReplicaSet/coredns-78fcd69978
Controlled By:  Node/minikube
Controlled By:  Node/minikube
Controlled By:  Node/minikube
Controlled By:  DaemonSet/kube-proxy
Controlled By:  Node/minikube
Controlled By:  ReplicaSet/metrics-server-dbf765b9b
sergio@nb:~/task_2$
```


* Implement Canary deployment of an application via Ingress. Traffic to canary deployment should be redirected if you add "canary:always" in the header, otherwise it should go to regular deployment.
Set to redirect a percentage of traffic to canary deployment.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-web
  annotations:
    kubernetes.io/ingress.class: "nginx"
    ingress.nginx.kubernetes.io/canary: "true"
    ingress.nginx.kubernetes.io/canary-by-header: "canary:always"
    ingress.nginx.kubernetes.io/canary-weight: "15"
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
             name: web
             port: 
                number: 80
```
![Screenshot](https://user-images.githubusercontent.com/3485151/143106169-077e5dfe-2628-42cd-9aa9-4d8da9390d3e.JPG)
