# Configurando los controladores

Volvemos a hacer uso de la configuración síncrona de hojas de tmux.

## Instalación de los paquetes

Utilizamos los binarios proporcionados por Debian:

```
apt install kubernetes-master kubernetes-client
```

El paquete kubernetes-master proporciona los binarios kube-apiserver,
kube-scheduler y kube-controller-manager pero todavía sin ficheros de
configuración ni unidad de systemd, que tendremos que crear a mano.

## Directorio de trabajo y certificados

Creamos el directorio de trabajo para los servicios
(/var/lib/kubernetes) y copiamos allí los certificados y claves
necesarias:

```
mkdir -p /var/lib/kubernetes/
mkdir -p /etc/kubernetes/config/
cp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
service-account-key.pem service-account.pem encryption-config.yaml \
kube-scheduler.kubeconfig kube-controller-manager.kubeconfig \
/var/lib/kubernetes/
```

## Configuración del API server

```
INTERNAL_IP=$(ip -o -4 a|grep 10.0.10|awk '{print $4}'|cut -d/ -f1)
cat << EOF > /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
  --etcd-servers=https://10.0.10.2:2379,https://10.0.10.3:2379,https://10.0.10.4:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
  --kubelet-https=true \\
  --runtime-config='api/all=true' \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## Configuración del kubernetes controller manager

Hacemos uso de la sincronización de hojas de tmux y creamos una unidad
de systemd para el servicio:

```
cat << EOF > /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/bin/kube-controller-manager \\
  --bind-address=0.0.0.0 \\
  --cluster-cidr=10.200.0.0/16 \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=10.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## Configuración del kubernetes scheduler

Volvemos a hacer uso de la sincronización de hojas de tmux y creamos
una unidad de systemd para el servicio:
```
cat << EOF > /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

Y el fichero de configuración:

```
cat <<EOF | tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: kubescheduler.config.k8s.io/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```

Nota: [Hilo de twitter en el que se explica el funcionamiento del planificador de kubernetes](https://twitter.com/danielepolencic/status/1309090938673868801)

Finalmente iniciamos todos los servicios:

```
{
  sudo systemctl daemon-reload
  sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
  sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```

## Permisos iniciales

Le otorgamos al usuario admin el rol de administrador del clúster,
para lo que accedemos a uno de los nodos controladores y ejecutamos:

```
kubectl create clusterrolebinding root-cluster-admin-binding \
--clusterrole=cluster-admin --user=admin
```

## Balanceador de carga externo

En el despliegue original se utiliza el balanceador proporcionado por
la nube de infraestructura de Google, pero en este caso vamos a hacer
una configuración más genérica, instalando un balanceador de carga con
HAProxy que actuará como un balanceador de carga expuesto al exterior
del kubernetes API server.

```
sudo apt install haproxy
```

Añadimos al fichero de /etc/haproxy/haproxy.cfg las líneas:

```
{
C1_EXTERNAL_IP=$(ssh root@controller1 ip -o -4 a s|grep 192.168.121|awk '{print $4}'|cut -d/ -f1)
C2_EXTERNAL_IP=$(ssh root@controller2 ip -o -4 a s|grep 192.168.121|awk '{print $4}'|cut -d/ -f1)
C3_EXTERNAL_IP=$(ssh root@controller3 ip -o -4 a s|grep 192.168.121|awk '{print $4}'|cut -d/ -f1)

cat <<EOF | sudo tee -a /etc/haproxy/haproxy.cfg

frontend k8s-api
    bind 0.0.0.0:6443
    mode tcp
    option tcplog
    timeout client 300000
    default_backend k8s-api

backend k8s-api
    mode tcp
    option tcplog
    option tcp-check
        timeout server 300000
    balance roundrobin
    default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 25$

        server controller1 ${C1_EXTERNAL_IP}:6443 check
        server controller2 ${C2_EXTERNAL_IP}:6443 check
        server controller3 ${C3_EXTERNAL_IP}:6443 check
EOF
}
```

Donde hemos puesto las direcciones IP "externas" de los
controladores.

Reiniciamos el balanceador de carga y comprobamos el funcionamiento
del clúster y del balanceador desde el equipo cliente usando las
credenciales de admin:

```
sudo systemctl restart haproxy

kubectl get componentstatuses --kubeconfig Config/admin.kubeconfig 

NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok
controller-manager   Healthy   ok
etcd-1               Healthy   {"health": "true"}
etcd-0               Healthy   {"health": "true"}
etcd-2               Healthy   {"health": "true"}
```

## RBAC para kubelet

Vamos a realizar la configuración para permitir que el servidor
kubernetes API pueda acceder a la API de kubelet de los nodos:

Creamos el ClusterRole kube-apiserver-to-kubelet.yaml:

```
cat <<EOF > Config/kube-apiserver-to-kubelet.yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```

Creamos el ClusterRoleBinding correspondiente kube-apiserver.yaml:

```
cat <<EOF > Config/kube-apiserver.yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: system:kube-apiserver
  namespace: ""
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:kube-apiserver-to-kubelet
subjects:
  - apiGroup: rbac.authorization.k8s.io
    kind: User
    name: kubernetes
EOF
```

Y los aplicamos con el usuario admin:

```
kubectl apply --kubeconfig Config/admin.kubeconfig -f Config/kube-apiserver-to-kubelet.yaml

clusterrole.rbac.authorization.k8s.io/system:kube-apiserver-to-kubelet created

kubectl apply --kubeconfig Config/admin.kubeconfig -f Config/kube-apiserver.yaml

clusterrolebinding.rbac.authorization.k8s.io/system:kube-apiserver created
```
