# WIK-DPS-TP04
## Pod
voici le [fichier](pod.yaml) du pod:
```
apiVersion: v1
kind: Pod
metadata:
  name: rust-api
spec:
  containers:
  - name: rust-api
    image: registry.cluster.wik.cloud/public/echo
    ports:
    - containerPort: 8080
```
### Test
```
[hurlu@monke TP-WIK-DPS-TP04]$ kubectl port-forward rust-api 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
[hurlu@monke TP-WIK-DPS-TP04]$ curl localhost:8080/ping
{"host":"localhost:8080","user-agent":"curl/7.86.0","accept":"*/*"}
```
## ReplicaSet
voici le [fichier](replicaset.yaml) du replicaset:
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rust-apis
spec:
  replicas: 4
  selector:
    matchLabels:
      type: api
  template:
    metadata:
      labels:
        type: api
    spec:
      containers:
      - name: rust-api
        image: registry.cluster.wik.cloud/public/echo
        ports:
        - containerPort: 8080
```
### Test
```
[hurlu@monke TP-WIK-DPS-TP04]$ kubectl apply -f replicaset.yaml 
replicaset.apps/rust-apis created
[hurlu@monke TP-WIK-DPS-TP04]$ kubectl get pods
NAME              READY   STATUS              RESTARTS   AGE
rust-apis-2vldv   0/1     ContainerCreating   0          4s
rust-apis-88xlh   0/1     ContainerCreating   0          4s
rust-apis-q29ms   0/1     ContainerCreating   0          4s
rust-apis-z2tvc   0/1     ContainerCreating   0          4s
[hurlu@monke TP-WIK-DPS-TP04]$ kubectl port-forward rust-apis-2vldv 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
[hurlu@monke TP-WIK-DPS-TP04]$ curl localhost:8080/ping
{"host":"localhost:8080","user-agent":"curl/7.86.0","accept":"*/*"}
```
## Deployment
voici le [fichier](deployment.yaml) du deployment:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
  labels:
    app: rust-api
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 50%
  selector:
    matchLabels:
      app: rust-api
  template:
    metadata:
      labels:
        app: rust-api
    spec:
      containers:
      - name: rust-api
        image: registry.cluster.wik.cloud/public/echo
        ports:
        - containerPort: 8080
```
### Test
```
[hurlu@monke TP-WIK-DPS-TP04]$ kubectl apply -f deployment.yaml 
deployment.apps/api-deployment unchanged
[hurlu@monke TP-WIK-DPS-TP04]$ kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
api-deployment-b57b5557-4k6mb   1/1     Running   0          2m29s
api-deployment-b57b5557-8tg7j   1/1     Running   0          2m29s
api-deployment-b57b5557-wpcvg   1/1     Running   0          2m29s
api-deployment-b57b5557-x79wx   1/1     Running   0          2m29s
[hurlu@monke TP-WIK-DPS-TP04]$ kubectl port-forward api-deployment-b57b5557-4k6mb 8080:8080
Forwarding from 127.0.0.1:8080 -> 8080
Forwarding from [::1]:8080 -> 8080
Handling connection for 8080
[hurlu@monke TP-WIK-DPS-TP04]$ curl localhost:8080/ping
{"host":"localhost:8080","user-agent":"curl/7.86.0","accept":"*/*"}
```
on peut bien accéder à chaque pod, on va maintenant changer l'image pour effectuer une mise à jour qui va faire fonctionner la rollingUpdate
```
[hurlu@monke TP-WIK-DPS-TP04]$ kubectl apply -f deployment.yaml 
deployment.apps/api-deployment configured
[hurlu@monke TP-WIK-DPS-TP04]$ kubectl get pods
NAME                             READY   STATUS              RESTARTS   AGE
api-deployment-87b78d9d8-8m9p6   0/1     ContainerCreating   0          1s
api-deployment-87b78d9d8-cgf4l   0/1     ContainerCreating   0          1s
api-deployment-87b78d9d8-gw422   0/1     ContainerCreating   0          1s
api-deployment-b57b5557-87j42    1/1     Terminating         0          31s
api-deployment-b57b5557-944hs    1/1     Terminating         0          31s
api-deployment-b57b5557-gngkx    1/1     Running             0          31s
api-deployment-b57b5557-hntsw    1/1     Running             0          31s
[hurlu@monke TP-WIK-DPS-TP04]$ kubectl get pods
NAME                             READY   STATUS              RESTARTS   AGE
api-deployment-87b78d9d8-8m9p6   0/1     ContainerCreating   0          9s
api-deployment-87b78d9d8-cgf4l   0/1     ContainerCreating   0          9s
api-deployment-87b78d9d8-gw422   0/1     ContainerCreating   0          9s
api-deployment-b57b5557-87j42    1/1     Terminating         0          39s
api-deployment-b57b5557-944hs    1/1     Terminating         0          39s
api-deployment-b57b5557-gngkx    1/1     Running             0          39s
api-deployment-b57b5557-hntsw    1/1     Running             0          39s
[hurlu@monke TP-WIK-DPS-TP04]$ kubectl get pods
NAME                             READY   STATUS        RESTARTS   AGE
api-deployment-87b78d9d8-8m9p6   1/1     Running       0          14s
api-deployment-87b78d9d8-cgf4l   1/1     Running       0          14s
api-deployment-87b78d9d8-gw422   1/1     Running       0          14s
api-deployment-87b78d9d8-xwscw   1/1     Running       0          4s
api-deployment-b57b5557-87j42    1/1     Terminating   0          44s
api-deployment-b57b5557-944hs    1/1     Terminating   0          44s
api-deployment-b57b5557-gngkx    1/1     Terminating   0          44s
api-deployment-b57b5557-hntsw    1/1     Terminating   0          44s
```
La rollingUpdate fonctionne bien et le max d'inaccessibilité est bien de 50%

## Service
voici le [fichier](service.yaml) du service:
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rust-apis
  labels:
    name: rust-apis
spec:
  replicas: 4
  selector:
    matchLabels:
      type: api
  template:
    metadata:
      labels:
        type: api
    spec:
      containers:
      - name: rust-api
        image: registry.cluster.wik.cloud/public/echo
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    name: rust-apis
  ports:
  - port: 8080
    targetPort: 8080
    protocol: TCP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-api
spec:
  rules:
  - host: lucastest.ez
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: api-service
            port: 
              number: 8080

```
### Test
```
[hurlu@monke TP-WIK-DPS-TP04]$ kubectl apply -f service.yaml 
replicaset.apps/rust-apis unchanged
service/api-service unchanged
ingress.networking.k8s.io/ingress-api created
[hurlu@monke TP-WIK-DPS-TP04]$ kubectl port-forward service/api-service 8080:8080
error: timed out waiting for the condition
```
je n'arrive ensuite pas à effectuer le port-forward à cause de cette erreur, je ne sais pas si l'erreur vient de mon fichier .yaml qui me semble bon ou de mon installation. J'ai quand même mis le nom de domaine dans mon /etc/hosts mais impossible de communiquer avec les api