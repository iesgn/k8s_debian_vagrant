# CoreDNS

CoreDNS es el componente de kubernetes encargado de la resolución de
nombres para las aplicaciones que se ejecutan dentro del
cluster. CoreDNS sustituye al componente kube-DNS y habitualmente se
despliega como una aplicación sobre contenedores en el propio cluster,
utilizando una ClusterIP previamente establecida y conocida por el
resto de las aplicaciones. En nuestro caso, esta dirección IP es la
10.32.0.10

## Despliegue de CoreDNS

Podemos reutilizar la plantilla generada en "Kubernetes de Hard Way"
sin modificar, ya que usamos la misma dirección para el servicio, o
podemos desarrollar una específica a partir de la genérica que
proporciona CoreDNS:

[https://github.com/coredns/deployment/blob/master/kubernetes/coredns.yaml.sed](https://github.com/coredns/deployment/blob/master/kubernetes/coredns.yaml.sed)

Optamos por la primera opción y aplicamos el despliegue, configmap y
roles:

```
kubectl apply -f https://storage.googleapis.com/kubernetes-the-hard-way/coredns-1.7.0.yaml
``` 

Que muestra la siguiente salida de objetos que se han creado:

```
serviceaccount/coredns created
clusterrole.rbac.authorization.k8s.io/system:coredns created
clusterrolebinding.rbac.authorization.k8s.io/system:coredns created
configmap/coredns created
deployment.apps/coredns created
service/kube-dns created
```

Podemos verificar al cabo de un rato los pods, el servicio, el despliegue y
el replicaSet creados y podemos observar que el servicio está
disponible como un ClusterIP en 10.32.0.10:

```
kubectl get all -n kube-system
NAME                           READY   STATUS    RESTARTS   AGE
pod/coredns-5677dc4cdb-27wqj   1/1     Running   0          2m
pod/coredns-5677dc4cdb-qstgf   1/1     Running   0          2m

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns   ClusterIP   10.32.0.10   <none>        53/UDP,53/TCP,9153/TCP   2m

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns   2/2     2            2           2m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-5677dc4cdb   2         2         2       2m
```

## Verificación

Lanzamos un contenedor (busybox) y comprobamos que realiza la
resolución DNS de forma correcta (se pide que nos devuelva la
dirección IP del API server, que debe ser la 10.32.0.1):

```
kubectl run busybox --image=busybox:1.28 --command -- sleep 3600
```

Que tras descargar la imagen y lanzar el contenedor está disponible:

```
kubectl get pods -l run=busybox

NAME      READY   STATUS    RESTARTS   AGE
busybox   1/1     Running   1          11m
```

Executamos nslookup en el pod para comprobar que se resuleve
adecuadamente el nombre y la dirección IP del API server:

```
kubectl exec -ti busybox -- nslookup kubernetes

Server:    10.32.0.10
Address 1: 10.32.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes
Address 1: 10.32.0.1 kubernetes.default.svc.cluster.local
```
