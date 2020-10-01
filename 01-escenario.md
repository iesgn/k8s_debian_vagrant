# Configuración del escenario

## Versiones del software utilizado

Sobre un equipo con Debian buster (10), instalamos los paquetes
vagrant y vagrant-libvirt:

```
vagrant                                   2.2.3+dfsg-1
vagrant-libvirt                           0.0.45-2
```

Instalamos qemu-kvm, libvirt-daemon-system y añadimos el usuario al
grupo libvirt, de manera que pueda lanzar máquinas virtuales en
qemu:///system

## Vagrant

Creamos el par de claves ssh que usaremos en el despliegue, lanzamos
ssh-agent (si no lo hubiera en la sesión actual) y añadimos por
comodidad la clave generada al agente:

```
ssh-keygen -t ecdsa -f ~/.ssh/k8s_debian_vagrant
ssh-agent /bin/bash
ssh-add ~/.ssh/k8s_debian_vagrant
```

Clonamos el repositorio y levantamos el escenario:

```
git clone https://github.com/iesgn/k8s_debian_vagrant.git
cd k8s_debian_vagrant
vagrant up
```

El escenario tarda varios minutos en configurarse ya que se hace una
actualización de todos los paquetes y pueden ser bastantes al tratarse
de debian testing.

## Puertos

Si estamos utilizando una instancia en nube o similar, será necesario
abrir el puerto 6443/tcp que va a utilizar el kube-apiserver a través
de un balanceador de carga.

## Resolución estática de nombres

Añadimos al fichero /etc/hosts del equipo cliente los nombres y
direcciones IP que vamos a usar:

```
10.0.10.2        controller1
10.0.10.3        controller2
10.0.10.4        controller3
10.0.10.12       node1
10.0.10.13       node2
10.0.10.14       node3
```

## Utilización de tmux

En lugar de realizar una instalación automática con alguna herramienta
como ansible, consideramos más adecuado desde el punto de vista
formativo hacer una instalación manual paso a paso, pero como hay que
realizar en varias ocasiones la misma configuración en varios nodos,
se usará la funcionalidad de tmux de ejecutar la misma instrucción en
varios equipos a la vez. Vamos a dejar anotados los pasos a dar en
tmux para llegar a esa situación:

```
# Ejecutamos tmux:
tmux
# Creamos hojas (panes) distribuidos horizontalmente:
ctrl+b "
ctrl+b "
# O verticalmente
ctrl+b %
# Abrimos una nueva ventana en la misma sesión
ctrl+b c
# Cambiamos a la siguiente ventana
ctrl+b n
# Distribuimos las hojas uniformemente:
ctrl+b Alt+2
# Conectamos a cada nodo por ssh:
ssh root@controller1
(Podemos cambiar de una hoja a otra con ctrl+b up/down)
# Sincronizamos las hojas:
ctrl+b :setw synchronize-panes
# Salimos de la sesión de tmux sin cerrarla
ctrl+b d
# Para volver a conectarse a una sesión, hay que ver las disponibles: 
tmux list-sessions
0: 3 windows (created Sat Sep 26 18:49:11 2020) [190x49]
# Conectarse a una sesión concreta
tmux a -t 0
```
