# Configuración de kubectl

Configuramos kubectl en el equipo cliente con las credenciales del
administrador, para que no sea necesario pasar el parámetro kubeconfig
cada vez que lo usamos.

FLOATING_IP=...

```
{
cd Config/
kubectl config set-cluster k8s-cluster \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${FLOATING_IP}:6443
	
kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem

kubectl config set-context k8s-cluster \
    --cluster=k8s-cluster --user=admin

kubectl config use-context k8s-cluster
cd ..
}
```

## Comprobación

```
kubectl get componentstatuses
```

```
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok                   
scheduler            Healthy   ok                   
etcd-1               Healthy   {"health": "true"}   
etcd-0               Healthy   {"health": "true"}   
etcd-2               Healthy   {"health": "true"}
```

```
kubectl get nodes
```

```
NAME    STATUS   ROLES    AGE   VERSION
node1   Ready    <none>   40h   v1.18.6
node2   Ready    <none>   40h   v1.18.6
node3   Ready    <none>   40h   v1.18.6
```
