Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.provider "virtualbox" do |vb|
	  vb.memory = 1024
    vb.cpus = 1
  end

  config.vm.define "wordpress" do |wordpress|
	  wordpress.vm.network "private_network", ip: "192.168.56.20"

    wordpress.vm.provider "virtualbox" do |vb|
      vb.name = "trusty_wordpress"
    end

    wordpress.vm.provision "shell",
        inline: "cat /vagrant/ssh-keys/vagrant_id_rsa.pub >> .ssh/authorized_keys"
  end

  config.vm.define "mysql" do |mysql|
	  mysql.vm.network "private_network", ip: "192.168.56.21"

    mysql.vm.provider "virtualbox" do |vb|
      vb.name = "trusty_mysql"
    end
  end

end
