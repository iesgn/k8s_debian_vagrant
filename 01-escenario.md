# Configuración del escenario

## Vagrant

Levantamos el escenario

```
vagrant up
```

El escenario tarda varios minutos en configurarse ya que se hace una
actualización de todos los paquetes y pueden ser bastantes al tratarse
de debian testing.

## Resolución estática de nombres

Añadimos al fichero /etc/hosts de nuestro equipo cliente los nombres y
direcciones IP que vamos a usar por comodidad:

```
10.0.0.2        controller1
10.0.0.3        controller2
10.0.0.4        controller3
10.0.0.12       node1
10.0.0.13       node2
10.0.0.14       node3
```

## Acceso root por ssh

En lugar de usar el acceso con el usuario vagrant y sudo, va a ser más
cómodo en ocasiones hacer algunos pasos directamente como root, para
lo que creamos una clave ssh y la ubicamos en el directorio
correspondiente del usuario root de cada nodo:

```
ssh-keygen -t ecdsa -f ~/.ssh/k8s_debian_vagrant
```

```
for i in `seq 1 3`; do 
scp -i .vagrant/machines/controller${i}/libvirt/private_key ~/.ssh/k8s_debian_vagrant.pub vagrant@controller${i}:
ssh -i .vagrant/machines/controller${i}/libvirt/private_key vagrant@controller${i} 'cat k8s_debian_vagrant.pub| tee ~/.ssh/authorized_keys'
ssh -i .vagrant/machines/controller${i}/libvirt/private_key vagrant@controller${i} 'sudo cp -a /home/vagrant/.ssh /root/'
ssh -i .vagrant/machines/controller${i}/libvirt/private_key vagrant@controller${i} 'sudo chown -R root. /root/.ssh/'
ssh -i .vagrant/machines/controller${i}/libvirt/private_key vagrant@controller${i} 'sudo chmod 600 /root/.ssh/authorized_keys'
ssh -i .vagrant/machines/controller${i}/libvirt/private_key vagrant@controller${i} 'sudo chmod 700 /root/.ssh/'
done
```

```
for i in `seq 1 3`; do 
scp -i .vagrant/machines/node${i}/libvirt/private_key ~/.ssh/k8s_debian_vagrant.pub vagrant@node${i}:
ssh -i .vagrant/machines/node${i}/libvirt/private_key vagrant@node${i} 'cat k8s_debian_vagrant.pub| tee ~/.ssh/authorized_keys'
ssh -i .vagrant/machines/node${i}/libvirt/private_key vagrant@node${i} 'sudo cp -a /home/vagrant/.ssh /root/'
ssh -i .vagrant/machines/node${i}/libvirt/private_key vagrant@node${i} 'sudo chown -R root. /root/.ssh/'
ssh -i .vagrant/machines/node${i}/libvirt/private_key vagrant@node${i} 'sudo chmod 600 /root/.ssh/authorized_keys'
ssh -i .vagrant/machines/node${i}/libvirt/private_key vagrant@node${i} 'sudo chmod 700 /root/.ssh/'
done
```

## Definición del hostname

(Debería hacerse en el Vagrantfile)

```
for i in controller{1..3} node{1..3}; do
ssh -i ~/.ssh/k8s_debian_vagrant root@${i} hostname ${i}
ssh -i ~/.ssh/k8s_debian_vagrant root@${i} "echo ${i} > /etc/hostname"
done
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
# Creamos 3 hojas (panes) distribuidos horizontalmente:
ctrl+b "
ctrl+b "
# Distribuimos las hojas uniformemente:
ctrl+b Alt+2
# Conectamos a cada nodo por ssh:
ssh -i .vagrant/machines/controller1/libvirt/private_key vagrant@controller1
(Podemos cambiar de una hoja a otra con ctrl+b up/down)
# Sincronizamos las hojas:
ctrl+b :setw synchronize-panes
```
