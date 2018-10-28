# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :inetRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "inet255"},
                ]
  },
  :centralRouter => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "inet255"},
                   {ip: '192.168.254.2', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "inet254"},
                   {ip: '192.168.0.1', adapter: 4, netmask: "255.255.255.252", virtualbox__intnet: "web1"},
                ]
  },
  :centralServer => {
        :box_name => "centos/7",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "web1"},
                ]
  },
}

Vagrant.configure("2") do |config|
config.vm.synced_folder ".", "/vagrant",type:"virtualbox"
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
        
        case boxname.to_s
        when "inetRouter"
	  box.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "256"]
          end
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
	    iptables-save > /etc/firewall.conf
	    echo "/sbin/iptables-restore < /etc/firewall.conf" >> /etc/rc.d/rc.local && chmod +x /etc/rc.d/rc.local
            echo net.ipv4.conf.all.forwarding=1 >> /etc/sysctl.d/99-sysctl.conf && sysctl -p
            SHELL
        when "centralRouter"
          box.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "256"]
          end  
	  box.vm.provision "shell", run: "always", inline: <<-SHELL
            iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth1 -j MASQUERADE
            iptables-save > /etc/firewall.conf
            echo "/sbin/iptables-restore < /etc/firewall.conf" >> /etc/rc.d/rc.local && chmod +x /etc/rc.d/rc.local
	    echo net.ipv4.conf.all.forwarding=1 >> /etc/sysctl.d/99-sysctl.conf && sysctl -p
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            echo "GATEWAY=192.168.254.1" >> /etc/sysconfig/network-scripts/ifcfg-eth2
	    echo "192.168.0.0/30 via 192.168.0.1" > /etc/sysconfig/network-scripts/route-eth3
            systemctl restart network
          SHELL
        when "centralServer"
	  box.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "256"]
          end
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            systemctl restart network
            SHELL
        end
      end
  end
end

