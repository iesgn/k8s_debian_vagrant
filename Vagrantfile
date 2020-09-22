Vagrant.configure("2") do |config|
  config.vm.define "controller1" do |controller1|
    controller1.vm.hostname = "controller1"
    controller1.vm.box = "debian/testing64"
    controller1.vm.network "private_network",
                           :ip => "10.0.10.2",
                           :libvirt__dhcp_enabled => false
    controller1.vm.synced_folder ".", "/vagrant", disabled: true
    controller1.vm.provider :libvirt do |libvirt|
      libvirt.uri = 'qemu+unix:///system'
      libvirt.host = "controller1"
      libvirt.host = "controller1"
      libvirt.cpus = 1
      libvirt.memory = 2048
      
    end
    controller1.vm.provision "shell" do |s|
      ssh_pub_key = File.readlines("#{Dir.home}/.ssh/k8s_debian_vagrant.pub").first.strip
      s.inline = <<-SHELL
      echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
      cp -a /home/vagrant/.ssh /root/
      chown -R root. /root/.ssh/
      chmod 600 /root/.ssh/authorized_keys
      chmod 700 /root/.ssh/
    SHELL
    end
    controller1.vm.provision "shell",
                             inline: "apt-get update && DEBIAN_FRONTEND=noninteractive \
                             apt-get -o Dpkg::Options::='--force-confnew' \
                             -q -y --allow-downgrades --allow-remove-essential --allow-change-held-packages dist-upgrade"
  end
  config.vm.define "controller2" do |controller2|
    controller2.vm.hostname = "controller2"
    controller2.vm.box = "debian/testing64"
    controller2.vm.network "private_network",
                           :ip => "10.0.10.3",
                           :libvirt__dhcp_enabled => false
    controller2.vm.synced_folder ".", "/vagrant", disabled: true
    controller2.vm.provider :libvirt do |libvirt|
      libvirt.uri = 'qemu+unix:///system'
      libvirt.host = "controller2"
      libvirt.cpus = 1
      libvirt.memory = 2048
    end
    controller2.vm.provision "shell" do |s|
      ssh_pub_key = File.readlines("#{Dir.home}/.ssh/k8s_debian_vagrant.pub").first.strip
      s.inline = <<-SHELL
      echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
      cp -a /home/vagrant/.ssh /root/
      chown -R root. /root/.ssh/
      chmod 600 /root/.ssh/authorized_keys
      chmod 700 /root/.ssh/
    SHELL
    end
    controller2.vm.provision "shell",
                             inline: "apt-get update && DEBIAN_FRONTEND=noninteractive \
                             apt-get -o Dpkg::Options::='--force-confnew' \
                             -q -y --allow-downgrades --allow-remove-essential --allow-change-held-packages dist-upgrade"
  end
  config.vm.define "controller3" do |controller3|
    controller3.vm.hostname = "controller3"
    controller3.vm.box = "debian/testing64"
    controller3.vm.network "private_network",
                           :ip => "10.0.10.4",
                           :libvirt__dhcp_enabled => false
    controller3.vm.synced_folder ".", "/vagrant", disabled: true
    controller3.vm.provider :libvirt do |libvirt|
      libvirt.uri = 'qemu+unix:///system'
      libvirt.host = "controller3"
      libvirt.cpus = 1
      libvirt.memory = 2048
    end
    controller3.vm.provision "shell" do |s|
      ssh_pub_key = File.readlines("#{Dir.home}/.ssh/k8s_debian_vagrant.pub").first.strip
      s.inline = <<-SHELL
      echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
      cp -a /home/vagrant/.ssh /root/
      chown -R root. /root/.ssh/
      chmod 600 /root/.ssh/authorized_keys
      chmod 700 /root/.ssh/
    SHELL
    end
    controller3.vm.provision "shell",
                             inline: "apt-get update && DEBIAN_FRONTEND=noninteractive \
                             apt-get -o Dpkg::Options::='--force-confnew' \
                             -q -y --allow-downgrades --allow-remove-essential --allow-change-held-packages dist-upgrade"
  end
  config.vm.define "node1" do |node1|
    node1.vm.hostname = "node1"
    node1.vm.box = "debian/testing64"
    node1.vm.network "private_network",
                           :ip => "10.0.10.12",
                           :libvirt__dhcp_enabled => false
    node1.vm.synced_folder ".", "/vagrant", disabled: true
    node1.vm.provider :libvirt do |libvirt|
      libvirt.uri = 'qemu+unix:///system'
      libvirt.host = "node1"
      libvirt.cpus = 1
      libvirt.memory = 1024
    end
    node1.vm.provision "shell" do |s|
      ssh_pub_key = File.readlines("#{Dir.home}/.ssh/k8s_debian_vagrant.pub").first.strip
      s.inline = <<-SHELL
      echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
      cp -a /home/vagrant/.ssh /root/
      chown -R root. /root/.ssh/
      chmod 600 /root/.ssh/authorized_keys
      chmod 700 /root/.ssh/
    SHELL
    end
    node1.vm.provision "shell",
                             inline: "apt-get update && DEBIAN_FRONTEND=noninteractive \
                             apt-get -o Dpkg::Options::='--force-confnew' \
                             -q -y --allow-downgrades --allow-remove-essential --allow-change-held-packages dist-upgrade"
  end
  config.vm.define "node2" do |node2|
    node2.vm.hostname = "node2"
    node2.vm.box = "debian/testing64"
    node2.vm.network "private_network",
                           :ip => "10.0.10.13",
                           :libvirt__dhcp_enabled => false
    node2.vm.synced_folder ".", "/vagrant", disabled: true
    node2.vm.provider :libvirt do |libvirt|
      libvirt.uri = 'qemu+unix:///system'
      libvirt.host = "node2"
      libvirt.cpus = 1
      libvirt.memory = 1024
    end
    node2.vm.provision "shell" do |s|
      ssh_pub_key = File.readlines("#{Dir.home}/.ssh/k8s_debian_vagrant.pub").first.strip
      s.inline = <<-SHELL
      echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
      cp -a /home/vagrant/.ssh /root/
      chown -R root. /root/.ssh/
      chmod 600 /root/.ssh/authorized_keys
      chmod 700 /root/.ssh/
    SHELL
    end
    node2.vm.provision "shell",
                             inline: "apt-get update && DEBIAN_FRONTEND=noninteractive \
                             apt-get -o Dpkg::Options::='--force-confnew' \
                             -q -y --allow-downgrades --allow-remove-essential --allow-change-held-packages dist-upgrade"
  end
  config.vm.define "node3" do |node3|
    node3.vm.hostname = "node3"
    node3.vm.box = "debian/testing64"
    node3.vm.network "private_network",
                           :ip => "10.0.10.14",
                           :libvirt__dhcp_enabled => false
    node3.vm.synced_folder ".", "/vagrant", disabled: true
    node3.vm.provider :libvirt do |libvirt|
      libvirt.uri = 'qemu+unix:///system'
      libvirt.host = "node3"
      libvirt.cpus = 1
      libvirt.memory = 1024
    end
    node3.vm.provision "shell" do |s|
      ssh_pub_key = File.readlines("#{Dir.home}/.ssh/k8s_debian_vagrant.pub").first.strip
      s.inline = <<-SHELL
      echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
      cp -a /home/vagrant/.ssh /root/
      chown -R root. /root/.ssh/
      chmod 600 /root/.ssh/authorized_keys
      chmod 700 /root/.ssh/
    SHELL
    end
    node3.vm.provision "shell",
                             inline: "apt-get update && DEBIAN_FRONTEND=noninteractive \
                             apt-get -o Dpkg::Options::='--force-confnew' \
                             -q -y --allow-downgrades --allow-remove-essential --allow-change-held-packages dist-upgrade"
  end
end
