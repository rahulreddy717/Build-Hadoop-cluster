# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.require_version ">= 1.4.3"
VAGRANTFILE_API_VERSION = "2"


provider='virtualbox'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

    ###Define the starting node number
    ###Can be 1 for Single node deployment or 3 for multinode deployments
    ###Currently supports only 3 nodes in a multinode deployment
    NumVm = 1

    #This will consume in computing last fraction of IP as well
    ip_last_fraction_address = 201

    #Define machine name initials. This will compromise in hostname as well
    server_initials = "hadoop"
    server_last = "singlenode"

    #Current version
    current_version = "1.0.1"
    current_version = current_version.tr('.', '')

    #Yarn scheduler                         8088
    #Map Reduce Job History Server          19888
    #Spark History Server                   18088
    #Spark Master Web UI Port server        18080
    #Spark worker Web UI Port               18081
    #Spark Job ports                        4040..4044
    #HDFS host port                         8020
    #Yarn Resourcemanager Scheduler Address 8030
    #Redis server cache                     6379
    #Unassigned ports for external feature  5800..5803
    ports1 = [ 5800, 5801, 5802, 5803, 6379 ]
    ports2 = [ 8088, 8020, 8030 ]
    ports3 = [ 18088, 18080, 18081, 4040, 4042, 4043, 4044 ]
    ports = [ 5800, 5801, 5802, 5803, 50070, 8088, 18088, 18080, 18081, 19888, 4040, 4041, 4042, 4044, 8020, 8030, 6379 ]

    ### Define which linux box need to be used###
    config.vm.box = "shareinsights/cedric"
    config.vm.box_version = "1.0.1"
    config.vm.box_download_insecure = true
    config.vm.box_url = "https://app.vagrantup.com/shareinsights/boxes/cedric"

    #Create 3 virtual machines and map host ansible ssh port to internal port
    #Map application specific ports to host ports
    (1..NumVm).each do |j|
        adder = ip_last_fraction_address + j
        ssh_adder = 2221 + j
        name="#{server_initials}#{current_version}-#{j}.#{server_last}.com"
        config.vm.define  "#{name}" do |node|
            node.vm.hostname="#{name}"
            node.vm.network :private_network, ip: "205.28.128.#{adder}"
            node.vm.network :forwarded_port, guest: 22, host: ssh_adder, id: "ssh"
            node.vm.provider "virtualbox" do |v|
                v.name =  "#{name}"
                if NumVm == 1
                  v.memory = 8192
                else
                  v.memory = 4028
                end
                v.cpus = 2
                v.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
            end

            #Define port forwarding per VM deployed
            if j == 1
                if NumVm == 1
                    ports.each do |port|
                        node.vm.network :forwarded_port, guest: port, host: port
                    end
                else
                    ports1.each do |port|
                        node.vm.network :forwarded_port, guest: port, host: port
                    end
                end
            end
            if j == 2
                ports2.each do |port|
                    node.vm.network :forwarded_port, guest: port, host: port
                end
            end
            if j == 3
                ports3.each do |port|
                    node.vm.network :forwarded_port, guest: port, host: port
                end
            end
            node.vm.provision :shell do |sh|
              sh.inline = "sudo hostnamectl set-hostname #{name}
                sudo hostnamectl set-hostname #{name} --pretty
                sudo hostnamectl set-hostname #{name} --static
                sudo hostnamectl set-hostname #{name} --transient"
            end

            #Show box version, do we need to?
            node.vm.provision "shell", run: 'always', inline: "/bin/bash /var/box.version.sh"
        end
    end
end
