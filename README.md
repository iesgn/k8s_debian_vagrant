# k8s_debian_vagrant

Adaptación de
[Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
utilizando un despliegue local con vagrant sobre KVM/libvirt con
imágenes de Debian Bullseye (testing) en lugar de un despliegue con
infraestructura de nube pública con gcloud.

La idea principal es utilizar los paquetes con los binarios que
proporciona el proyecto debian para la mayoría de los componentes, en
algunos casos estos paquetes incluyen parte del trabajo posterior de
configuración (algunas unidades de systemd o ficheros de
configuración), pero todavía no se trata de paquetes finalizados y con
un nivel de configuración "estándar".

## Parámetros del despliegue:

* IP Flotante: IP pública o privada que corresponda
* Red "externa": 192.168.121.0/24
* Red interna: 10.0.10.0/24
  * controller1: 10.0.10.2
  * controller2: 10.0.10.3
  * controller3: 10.0.10.4
  * node1: 10.0.10.12
  * node2: 10.0.10.13
  * node3: 10.0.10.14
* Service-cluster-ip-range: 10.32.0.0/24
* cluster-cidr: 10.200.0.0/16
* pod-cidr: 10.200.X.0/24

## Esquema de red

![esquema_red](./img/esquema_red.png)
