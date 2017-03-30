# -*- mode: ruby -*-
# vi: set ft=ruby :
# vagrant plugin install vagrant-libvirt
# vagrant plugin install vagrant-hosts
# vagrant plugin install vagrant-cachier
# gem install ruby-libvirt
# vagrant plugin install vagrant-libvirt
# export VAGRANT_DEFAULT_PROVIDER="libvirt"

Vagrant.configure("2") do |config|

  config.vm.box = "debian/jessie64"

  if Vagrant.has_plugin?("vagrant-cachier")
  config.cache.auto_detect = true
  end

  config.vm.define "ceph-admin" do |admin|
    admin.vm.hostname = "au-ceph-admin.lab.systems"
    admin.vm.network :private_network, ip: "192.168.10.10"
    admin.vm.provision :shell, :inline => "wget -q -O- 'https://download.ceph.com/keys/release.asc' | apt-key add -", :privileged => true
    admin.vm.provision :shell, :inline => "echo deb http://download.ceph.com/debian/ $(lsb_release -sc) main | tee /etc/apt/sources.list.d/ceph.list", :privileged => true
    admin.vm.provision :shell, :inline => "DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -yq ntp apt-transport-https ceph-deploy", :privileged => true
    #admin.vm.provision :shell, :inline => "wget https://download.seafile.com/seafhttp/files/e1456542-9c5e-4461-ac2c-15c7b4768ddf/seafile-pro-server_6.0.8_x86-64.tar.gz -O /root/seafile-proserver.tar.gz.", :privileged => true
    admin.vm.provider :libvirt do |kvm|
      kvm.memory = 1024
      kvm.cpus   = 1
    end
  end

  nodes = {
    :rgw1  => {:host => 'au-ceph-rgw1',:domain => 'lab.systems', :ip => '192.168.10.20', :mem => 2048  },
    :mon1  => {:host => 'au-ceph-mon1',:domain => 'lab.systems', :ip => '192.168.10.21', :mem => 2048  },
    :cs2   => {:host => 'au-ceph-cs1', :domain => 'lab.systems', :ip => '192.168.10.22', :mem => 1024  },
    :cs3   => {:host => 'au-ceph-cs2', :domain => 'lab.systems', :ip => '192.168.10.23', :mem => 1024  },
    :cs4   => {:host => 'au-ceph-cs3', :domain => 'lab.systems', :ip => '192.168.10.24', :mem => 1024  },
  }

  nodes.each do |name, options|
    config.vm.define name do |node|
      node.vm.provision :shell, :inline => "wget -q -O- 'https://download.ceph.com/keys/release.asc' | apt-key add -", :privileged => true
      node.vm.provision :shell, :inline => "echo deb http://download.ceph.com/debian/ $(lsb_release -sc) main | tee /etc/apt/sources.list.d/ceph.list", :privileged => true
      node.vm.provision :shell, :inline => "apt-get install -y apt-transport-https ntp && apt-get update && apt-get -y dist-upgrade"
      node.vm.provision :shell, :inline => "echo 'America/Santiago' > /etc/timezone && ntpd -s ntp.shoa.cl"
      node.vm.hostname = "#{options[:host]}.#{options[:domain]}"
      node.vm.network :private_network, ip: options[:ip]
      node.vm.provider :libvirt do |kvm|
        kvm.storage :file, :size => '5G', :type => 'qcow2'
        kvm.memory  = options[:mem] if options[:mem]
        kvm.cpus    = options[:cpu] if options[:cpu]
      end
    end
  end
end
