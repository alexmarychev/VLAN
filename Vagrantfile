# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
:inetRouter => {
        :box_name => "centos/7",
#        :public => {:ip => '10.10.10.1', :adapter => 1},
        :net => [
                   {adapter: 2, auto_config: false, virtualbox__intnet: "bond"},
				   {adapter: 3, auto_config: false, virtualbox__intnet: "bond"},
                ]
  },
  :centralRouter => {
        :box_name => "centos/7",
        :net => [
                   {adapter: 2, auto_config: false, virtualbox__intnet: "bond"},
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "hw-net"},
                   {ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "mgt-net"},
			{ip: '192.168.255.5', adapter: 6, netmask: "255.255.255.252", virtualbox__intnet: "router-office1"},
		    {ip: '192.168.255.9', adapter: 7, netmask: "255.255.255.252", virtualbox__intnet: "router-office2"},
			{adapter: 8, auto_config: false, virtualbox__intnet: "bond"},
		   
                ]
  },
  
  :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {adapter: 3, auto_config: false, virtualbox__intnet: true},
                   {adapter: 4, auto_config: false, virtualbox__intnet: true},
                ]
  },
  
  :office1Router => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.2.1', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "dev1-net"},
		   {ip: '192.168.2.65', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "test1-net"},
		   {ip: '192.168.2.129', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "managers-net"},
		   {ip: '192.168.2.193', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "hw1-net"},
		   {ip: '192.168.255.6', adapter: 6, netmask: "255.255.255.252", virtualbox__intnet: "router-office1"},
                ]
  },
  
  :office2Router => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.1.1', adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "dev2-net"},
		   {ip: '192.168.1.129', adapter: 3, netmask: "255.255.255.192", virtualbox__intnet: "test2-net"},
		   {ip: '192.168.1.193', adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "hw2-net"},
                   {ip: '192.168.255.10', adapter: 5, netmask: "255.255.255.252", virtualbox__intnet: "router-office2"},
                ]
  },
  
  :office1Server => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.2.2', adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "dev1-net"},
                ]
  },
  
  :office2Server => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.1.2', adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "dev2-net"},
                ]
  },
  
  :testClient1 => {
        :box_name => "centos/7",
        :net => [
                   {adapter: 2, auto_config: false, virtualbox__intnet: "lan-test"},
                ]
  },
  
  :testClient2 => {
        :box_name => "centos/7",
        :net => [
                   {adapter: 2, auto_config: false, virtualbox__intnet: "lan-test"},
                ]
  },
  
  :testServer1 => {
        :box_name => "centos/7",
        :net => [
                   {adapter: 2, auto_config: false, virtualbox__intnet: "lan-test"},
                ]
  },
  
  :testServer2 => {
        :box_name => "centos/7",
        :net => [
                   {adapter: 2, auto_config: false, virtualbox__intnet: "lan-test"},
                ]
  },
  
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s

        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        
        if boxconfig.key?(:public)
          box.vm.network "public_network", boxconfig[:public]
        end

        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL
        
        box.vm.provision "ansible" do |ansible|
        ansible.playbook = "network.yml"

        case boxname.to_s
        when "inetRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            sysctl net.ipv4.conf.all.forwarding=1
            iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
            SHELL
        when "centralRouter"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            systemctl restart network
	    sysctl net.ipv4.conf.all.forwarding=1
            SHELL
	when "office1Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.255.5" >> /etc/sysconfig/network-scripts/ifcfg-eth5
	    echo "DEFROUTE=yes" >> /etc/sysconfig/network-scripts/ifcfg-eth5
            systemctl restart network
            ip r d default
            ip r a default via 192.168.255.5 dev eth5
            sysctl net.ipv4.conf.all.forwarding=1
            SHELL
	when "office2Router"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.255.9" >> /etc/sysconfig/network-scripts/ifcfg-eth4
	    echo "DEFROUTE=yes" >> /etc/sysconfig/network-scripts/ifcfg-eth4
	    systemctl restart network
            ip r d default
            ip r a default via 192.168.255.9 dev eth4
            sysctl net.ipv4.conf.all.forwarding=1
            SHELL
        when "centralServer"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
	    echo "DEFROUTE=yes" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            ip r d default
            ip r a default via 192.168.0.1 dev eth1
            SHELL
	when "office1Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0
            echo "GATEWAY=192.168.2.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
	    echo "DEFROUTE=yes" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            ip r d default
            ip r a default via 192.168.2.1 dev eth1
            SHELL
	when "office2Server"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.1.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
	    echo "DEFROUTE=yes" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            ip r d default
            ip r a default via 192.168.1.1 dev eth1
            SHELL
        end
        end
      end

  end
  
  
end
