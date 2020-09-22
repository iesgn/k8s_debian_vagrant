# Ficheros de configuración de kubernetes

## Configuración de clientes

Se crean en la máquina cliente los ficheros kubeconfig del controller
manager, scheduler, kubelet y kube-proxy que se usarán en las
siguientes secciones.

### Kubelet

```
cd Config/
for instance in node{1..3}; do
  kubectl config set-cluster k8s-cluster \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-credentials system:node:${instance} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

  kubectl config set-context default \
    --cluster=k8s-cluster \
    --user=system:node:${instance} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
cd ..
```

Resultados:

```
node1.kubeconfig
node2.kubeconfig
node3.kubeconfig
```

Copiamos cada fichero de configuración al nodo correspondiente:

```
for instance in node{1..3}; do 
scp Config/${instance}.kubeconfig root@${instance}:
done
```

### Kube-proxy

```
{
  cd Config/
  kubectl config set-cluster k8s-cluster \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=k8s-cluster \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
cd ..
```

Resultados:

```
kube-proxy.kubeconfig
```

Copiamos el fichero de configuración a todos los nodos:

```
for instance in node{1..3}; do 
scp Config/kube-proxy.kubeconfig root@${instance}:
done
```

### Controller manager

```
{
  cd Config/
  kubectl config set-cluster k8s-cluster --certificate-authority=ca.pem --embed-certs=true --server=https://127.0.0.1:6443 --kubeconfig=kube-controller-manager.kubeconfig
  kubectl config set-credentials system:kube-controller-manager --client-certificate=kube-controller-manager.pem --client-key=kube-controller-manager-key.pem --embed-certs=true --kubeconfig=kube-controller-manager.kubeconfig
  kubectl config set-context default --cluster=k8s-cluster --user=system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig
  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
  cd ..
}
```

Copiamos el fichero a todos los controladores junto a la clave y
certificado de kube-controller-manager:

```
for instance in controller{1..3}; do 
scp Config/kube-controller-manager-key.pem root@${instance}:/var/lib/kubernetes/
scp Config/kube-controller-manager.pem root@${instance}:/var/lib/kubernetes/
scp Config/kube-controller-manager.kubeconfig root@${instance}:/var/lib/kubernetes/
done
```

## kubernetes scheduler

```
{
  cd Config/
  kubectl config set-cluster k8s-cluster --certificate-authority=ca.pem --embed-certs=true --server=https://127.0.0.1:6443 --kubeconfig=kube-scheduler.kubeconfig
  kubectl config set-credentials system:kube-scheduler --client-certificate=kube-scheduler.pem --client-key=kube-scheduler-key.pem --embed-certs=true --kubeconfig=kube-scheduler.kubeconfig
  kubectl config set-context default --cluster=k8s-cluster --user=system:kube-scheduler --kubeconfig=kube-scheduler.kubeconfig
  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
  cd ..
}
```

Copiamos el fichero a todos los controladores junto a la clave y
certificado de kube-controller-manager:

```
for instance in controller{1..3}; do 
scp Config/kube-scheduler-key.pem root@${instance}:/var/lib/kubernetes/
scp Config/kube-scheduler.pem root@${instance}:/var/lib/kubernetes/
scp Config/kube-scheduler.kubeconfig root@${instance}:/var/lib/kubernetes/
done
```

## Fichero de configuración del usuario admin

Lo creamos en el equipo cliente, que es desde donde se va a utilizar:

```
{
  cd Config/
  kubectl config set-cluster k8s-cluster \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=k8s-cluster \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
  cd ..
}