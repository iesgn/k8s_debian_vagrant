# Configuración de los nodos

Volvemos a utilizar tmux, pero en este caso lo hacemos sobre los tres
nodos sobre los que se van a ejecutar las cargar de trabajo normales.

## Instalación de los paquetes

Instalamos los siguientes paquetes de los repositorios de debian:

```
apt-get install --no-install-recommends runc containerd golang-github-appc-cni-dev \
kubernetes-client kubernetes-node socat conntrack ipset
```

Instalamos el binario crictl directamente desde el sitio del proyecto,
ya que aún no está empaquetado:

```
{
wget -q --show-progress --https-only --timestamping \
  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.18.0/crictl-v1.18.0-linux-amd64.tar.gz
tar -xvf crictl-v1.18.0-linux-amd64.tar.gz
chmod +x crictl
mv crictl /usr/local/bin
}
```

Instalamos los plugins de cni (binarios que se van a ubicar en el
directorio /opt/cni/bin que es la ubicación por defecto para el
servicio kubelet):

```
wget -q --show-progress --https-only --timestamping \
  https://github.com/containernetworking/plugins/releases/download/v0.8.6/cni-plugins-linux-amd64-v0.8.6.tgz
mkdir -p   /opt/cni/bin
tar -xvf cni-plugins-linux-amd64-v0.8.6.tgz -C /opt/cni/bin/
```

## Configuración de la red CNI

Definirmos la red para los pods de cada nodo, con una subred /24 del
CIDR del clúster (10.200.0.0/16):

```
export POD_CIDR=$(echo 10.200.`ip -o -4 a|grep 10.0.10|awk '{print $4}'|cut -d/ -f1|cut -d. -f4`.0/24)
```

```
cat <<EOF | tee /etc/cni/net.d/10-bridge.conf
{
    "cniVersion": "0.3.1",
    "name": "bridge",
    "type": "bridge",
    "bridge": "cnio0",
    "isGateway": true,
    "ipMasq": true,
    "ipam": {
        "type": "host-local",
        "ranges": [
          [{"subnet": "${POD_CIDR}"}]
        ],
        "routes": [{"dst": "0.0.0.0/0"}]
    }
}
EOF
```

```
cat <<EOF | tee /etc/cni/net.d/99-loopback.conf
{
    "cniVersion": "0.3.1",
    "name": "lo",
    "type": "loopback"
}
EOF
```

## Configuración kubelet

Ubicamos los certificados, la clave y el kubeconfig de cada nodo en el
directorio /var/lib/kubelet que se ha creado al instalar el paquete
kubernetes-node:

```
HOSTNAME=$(hostname -s)
mv /root/${HOSTNAME}*pem /var/lib/kubelet
mkdir /var/lib/kubernetes
mv /root/ca.pem /var/lib/kubernetes
mv /root/${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
```

Creamos el fichero de configuración de kubelet (kubelet-config.yaml):

```
cat <<EOF | tee /var/lib/kubelet/kubelet-config.yaml
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubernetes/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "cluster.local"
clusterDNS:
  - "10.32.0.10"
podCIDR: "${POD_CIDR}"
resolvConf: "/lib/systemd/resolv.conf"
runtimeRequestTimeout: "15m"
tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
EOF
```

Creamos la unidad de systemd para el servicio kubelet:

```
cat <<EOF | tee /etc/systemd/system/kubelet.service
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/bin/kubelet \\
  --config=/var/lib/kubelet/kubelet-config.yaml \\
  --container-runtime=remote \\
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
  --image-pull-progress-deadline=2m \\
  --kubeconfig=/var/lib/kubelet/kubeconfig \\
  --network-plugin=cni \\
  --register-node=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## Configuración de kube-proxy

Creamos el directorio /var/lib/kube-proxy y movemos el fichero
kubeconfig a él:

```
mkdir /var/lib/kube-proxy
mv /root/kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
```

Creamos el fichero de configuración de kube-proxy:

```
cat <<EOF | tee /var/lib/kube-proxy/kube-proxy-config.yaml
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "iptables"
clusterCIDR: "10.200.0.0/16"
EOF
```

Creamos la unidad de systemd para el servicios kube-proxy:

```
cat <<EOF | tee /etc/systemd/system/kube-proxy.service
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \\
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

## Inicio de los servicios y comprobación

```
{
systemctl daemon-reload
systemctl enable containerd kubelet kube-proxy
systemctl start containerd kubelet kube-proxy
}
```

Desde el equipo cliente ejecutamos:

```
kubectl get nodes --kubeconfig Config/admin.kubeconfig
```

Y obtenemos como salida los nodos listos:

```
NAME    STATUS   ROLES    AGE     VERSION
node1   Ready    <none>   4m13s   v1.18.6
node2   Ready    <none>   4m13s   v1.18.6
node3   Ready    <none>   4m13s   v1.18.6
```
