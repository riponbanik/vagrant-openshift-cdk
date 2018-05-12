# -*- mode: ruby -*-
# vi: set ft=ruby :

# The private network IP of the VM. You will use this IP to connect to OpenShift.
# This variable is ignored for Hyper-V provider.
PUBLIC_ADDRESS="10.0.0.101"
PRIVATE_ADDRESS="192.168.56.101"
LOCAL_ADDRESS="10.0.2.15"

Vagrant.configure(2) do |config|
  config.vm.box = "centos/7"

  # Cache
  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :machine
  end
  
  # Set host dns
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]      
  end 

  config.vm.define "cdk" do |machine|    
    # Guest
    config.vm.hostname = "cdk.lab.local"
    config.vm.network "private_network", ip: "192.168.56.101"
    config.vm.network "forwarded_port", host: 8443, guest: 8443, auto_correct: true
  

    # Landrush DNS
    if Vagrant.has_plugin?("landrush")   
      # config.landrush_ip.auto_install = true
      config.landrush.enabled = true
      config.landrush.host_ip_address = "#{PRIVATE_ADDRESS}"
      config.landrush.tld = "cdk.lab.local"
      config.landrush.guest_redirect_dns = false
      # config.landrush.host_redirect_dns= false
      # config.landrush.upstream '127.0.0.1', 53, :tcp
    end
  end

  # Shell
  config.vm.provision "shell", inline: <<-SHELL
    sudo yum -y install nano net-tools bind-utils dnsmasq  
    #grep -q -F 'NM_CONTROLED=yes' /etc/sysconfig/network-scripts/ifcfg-eth0 || (echo 'NM_CONTROLED=yes' | sudo tee -a /etc/sysconfig/network-scripts/ifcfg-eth0)
    #sudo nmcli con mod "System eth0" ipv4.dns "127.0.0.1 192.168.56.13 10.0.2.3"    
    #grep -q -F 'PEERDNS=no' /etc/sysconfig/network-scripts/ifcfg-eth0 || (echo 'PEERDNS=no' | sudo tee -a /etc/sysconfig/network-scripts/ifcfg-eth0)
    sudo sh -c 'echo "server=/cdk.lab.local/127.0.0.1#10053" > /etc/dnsmasq.d/vagrant-landrush'
    #sudo sh -c 'echo "server=/cdk.lab.local/127.0.0.1#10053" > /etc/NetworkManager/vagrant-landrush'    
    #sudo sed -i "s/127\.0\.0\.1/10.0.2.15/g" /etc/hosts
    #sudo systemctl restart NetworkManager.service
    sudo systemctl start dnsmasq.service
  SHELL

end
