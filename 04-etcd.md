# Configuración de etcd

Accedemos a los tres controladores con el mecanismo de tmux de
sincronización de hojas que se explica en el primer apartado del
curso.

## Instalación

Usamos el paquete etcd disponible en debian testing:

```
apt install etcd
```

Este paquete incluye ficheros de control del demonio, una unidad de
systemd, etc. que no tendríamos con la configuración directa del
binario como se hace en otros muchos manuales similares.

## Directorio /etc/etcd

No está definido inicialmente, pero lo creamos para ubicar algunos
ficheros necesarios:

```
mkdir /etc/etcd
```

## Certificados y claves

Copiamos al directorio /etc/etcd los certificados y claves necesarias
para el funcionamiento de etcd:

```
cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
chown -R etcd. /etc/etcd
```

### Definición de variables de entorno

```
INTERNAL_IP=$(ip -o -4 a|grep 10.0.10|awk '{print $4}'|cut -d/ -f1)
HOSTNAME=$(hostname -s)
```

### Configuración del servicio

```
cat << EOF >> /etc/default/etcd
ETCD_NAME="${HOSTNAME}"
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_LISTEN_PEER_URLS="https://${INTERNAL_IP}:2380"
ETCD_LISTEN_CLIENT_URLS="https://${INTERNAL_IP}:2379,https://127.0.0.1:2379"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://${INTERNAL_IP}:2380"
ETCD_INITIAL_CLUSTER="controller1=https://10.0.10.2:2380,controller2=https://10.0.10.3:2380,controller3=https://10.0.10.4:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="https://${INTERNAL_IP}:2379"
ETCD_CA_FILE
ETCD_CERT_FILE="/etc/etcd/kubernetes.pem"
ETCD_KEY_FILE="/etc/etcd/kubernetes-key.pem"
ETCD_CLIENT_CERT_AUTH
ETCD_TRUSTED_CA_FILE="/etc/etcd/ca.pem"
ETCD_PEER_CERT_FILE="/etc/etcd/kubernetes.pem"
ETCD_PEER_KEY_FILE="/etc/etcd/kubernetes-key.pem"
ETCD_PEER_CLIENT_CERT_AUTH
ETCD_PEER_TRUSTED_CA_FILE="/etc/etcd/ca.pem"

EOF
```

### Reinicio y prueba de funcionamiento

```
systemctl restart etcd.service
```

Tras unos instantes el clúster se sincronizará y podremos comprobarlo con:

```
ETCDCTL_API=3 etcdctl member list --endpoints=https://127.0.0.1:2379 --cacert=/etc/etcd/ca.pem --cert=/etc/etcd/kubernetes.pem --key=/etc/etcd/kubernetes-key.pem
```

Que debe proporcionar una salida como:

```
3a57933972cb5131, started, controller1, https://10.0.10.2:2380, https://10.0.10.2:2379
f98dc20bce6225a0, started, controller2, https://10.0.10.3:2380, https://10.0.10.3:2379
ffed16798470cab5, started, controller3, https://10.0.10.4:2380, https://10.0.10.4:2379
```

Comprobamos los mensajes de log en los que se ve cómo se ha creado el
clúster:

```
journalctl -u etcd.service
```
