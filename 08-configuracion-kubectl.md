# Configuración de kubectl

Configuramos kubectl en el equipo cliente con las credenciales del
administrador, para que no sea necesario pasar el parámetro kubeconfig
cada vez que lo usamos.

FLOATING_IP=...

```
{
kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${FLOATING_IP}:6443
	
kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem

kubectl config set-context kubernetes-the-hard-way \
    --cluster=k8s-cluster

kubectl config use-context k8s-cluster
}
```

## Comprobación

```
kubectl get componentstatuses
```

