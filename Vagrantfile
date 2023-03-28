# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/focal64"
  config.vm.box_check_update = false
  config.ssh.insert_key = false
  # insecure_private_key download from https://github.com/hashicorp/vagrant/blob/master/keys/vagrant
  config.ssh.private_key_path = "insecure_private_key"

  (0..1).each do |i|
    config.vm.define vm_name = "lb%d" % i do |lb|
      lb.vm.hostname = vm_name
      lb.vm.network :private_network, ip: "192.168.66.#{i+11}"
      lb.vm.network :private_network, ip: "192.168.55.#{i+11}"
      lb.vm.network "forwarded_port", guest: 80, host: "#{64001+i}"
      lb.vm.provider "virtualbox" do |vb|
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["modifyvm", :id, "--ioapic", "on"]
        vb.memory = "512"
        vb.cpus = 1
      end
      lb.vm.provision "shell", inline: <<-SHELL
        #########################################################################
        echo "root:vagrant" | sudo chpasswd
        #########################################################################
        sed -i.bak -e "s@archive.ubuntu.com@mirrors.ustc.edu.cn@g" -e "s@security.ubuntu.com@mirrors.ustc.edu.cn@g" /etc/apt/sources.list;
        while [ true ]; do
        DEBIAN_FRONTEND=noninteractive apt update;
        DEBIAN_FRONTEND=noninteractive apt install -y curl ipvsadm ipset keepalived net-tools && break
        done
        #########################################################################
        cat > /etc/modules-load.d/000-net.conf<<EOF
br_netfilter
ip_vs
ip_vs_rr
ip_vs_wrr
ip_vs_sh
nf_conntrack
EOF
        systemctl daemon-reload && \
        systemctl enable systemd-modules-load.service

        cat > /etc/sysctl.d/000-sysctl.conf <<-EOF
fs.file-max = 1024000
fs.inotify.max_user_instances = 8192
fs.inotify.max_user_watches = 89100
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 4096
net.ipv4.ip_forward = 1
#net.ipv4.vs.conntrack = 1
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_intvl = 30
net.ipv4.tcp_keepalive_probes = 10
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

        cat > /etc/keepalived/keepalived.conf <<__EOF 
global_defs {
}
vrrp_sync_group VG1 {
  group {
    VI_1
    VI_GATEWAY
  }
}

vrrp_instance VI_1 {
    state BACKUP
    interface enp0s8
    garp_master_delay 10
    smtp_alert
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.55.100
    }
}

vrrp_instance VI_GATEWAY {
    state BACKUP
    interface enp0s8
    garp_master_delay 10
    smtp_alert
    virtual_router_id 52
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.66.100
    }
}

virtual_server 192.168.55.100 80 {
	delay_loop 	1
	lb_algo 	wrr
	lb_kind		NAT
	protocol 	TCP
	real_server 192.168.66.111 80 {
		weight	100
		inhibit_on_failure
		HTTP_GET {
			url {
				path /
				status_code 200
			}
			connect_timeout 3
			retry 3
			delay_before_retry 3
		}
	}
	real_server 192.168.66.112 80 {
		weight	100
		inhibit_on_failure
		HTTP_GET {
			url {
				path /
				status_code 200
			}
			connect_timeout 3
			retry 3
			delay_before_retry 3
		}
	}
	sorry_server 127.0.0.1 80
}
__EOF

sed -i "s|^DAEMON_ARGS.*|DAEMON_ARGS=\\"--log-detail --log-console\\"|g" /etc/default/keepalived && systemctl daemon-relad && systemctl enable --now keepalived


      SHELL
    end
  end


  (0..1).each do |i|
    config.vm.define vm_name = "rs%d" % i do |rs|
      rs.vm.hostname = vm_name
      rs.vm.network :private_network, ip: "192.168.66.#{i+111}"
      rs.vm.network "forwarded_port", guest: 80, host: "#{64011+i}"
      rs.vm.provider "virtualbox" do |vb|
        vb.memory = "512"
        vb.cpus = 1
      end
      rs.vm.provision "shell", inline: <<-SHELL
        #########################################################################
        echo "root:vagrant" | sudo chpasswd
        #########################################################################
        sed -i.bak -e "s@archive.ubuntu.com@mirrors.ustc.edu.cn@g" -e "s@security.ubuntu.com@mirrors.ustc.edu.cn@g" /etc/apt/sources.list;
        while [ true ]; do
        DEBIAN_FRONTEND=noninteractive apt update;
        DEBIAN_FRONTEND=noninteractive apt install -y curl nginx net-tools && break
        done
        #route del default gw 10.0.2.2;
        route add default gw 192.168.66.100;
        #默认网关指向LVS的DIP，不是LVS提供服务的VIP
        echo "rs#{i}" > /var/www/html/index.html;
      SHELL
    end
  end

  (0..1).each do |i|
    config.vm.define vm_name = "cl%d" % i do |rs|
      rs.vm.hostname = vm_name
      rs.vm.network :private_network, ip: "192.168.55.#{i+200}"
      rs.vm.provider "virtualbox" do |vb|
        vb.memory = "512"
        vb.cpus = 1
      end
      rs.vm.provision "shell", inline: <<-SHELL
        #########################################################################
        echo "root:vagrant" | sudo chpasswd
        #########################################################################
        sed -i.bak -e "s@archive.ubuntu.com@mirrors.ustc.edu.cn@g" -e "s@security.ubuntu.com@mirrors.ustc.edu.cn@g" /etc/apt/sources.list;
        while [ true ]; do
        DEBIAN_FRONTEND=noninteractive apt update;
        DEBIAN_FRONTEND=noninteractive apt install -y curl net-tools && break
        done
      SHELL
    end
  end

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
