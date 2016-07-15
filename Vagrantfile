# -*- mode: ruby -*-
# vi: set ft=ruby :

admin_user = "admin"
admin_password = "secret"


admin_env = {
    "OS_PROJECT_DOMAIN_NAME" => "default",
    "OS_USER_DOMAIN_NAME" => "default",
    "OS_PROJECT_NAME" => "admin",
    "OS_USERNAME" => admin_user,
    "OS_PASSWORD" => admin_password,
    "OS_AUTH_URL" => "http://controller:35357/v3",
    "OS_IDENTITY_API_VERSION" => 3,
    "OS_IMAGE_API_VERSION" => 2
}

user_env = {
    "OS_PROJECT_DOMAIN_NAME" => "default",
    "OS_USER_DOMAIN_NAME" => "default",
    "OS_PROJECT_NAME" => "demo",
    "OS_USERNAME" => "demo",
    "OS_PASSWORD" => "secret",
    "OS_AUTH_URL" => "http://controller:5000/v3",
    "OS_IDENTITY_API_VERSION" => "3",
    "OS_IMAGE_API_VERSION" => "2"
}

Vagrant.configure(2) do |config|

    #config.vm.box = "vinik/ubuntu"
    config.vm.box = "bento/ubuntu-14.04"

    config.vm.define "controller" do |cn|

        cn.vm.hostname="controller"

        cn.vm.network "private_network",
            ip: "10.0.0.11",
            netmask: "255.255.255.0"
        config.vm.network "public_network",
            bridge: "eth1",
            nic_type: "virtio"

        cn.vm.network "forwarded_port", guest: 80,      host: 8080
        cn.vm.network "forwarded_port", guest: 3306,    host: 3306
        cn.vm.network "forwarded_port", guest: 5000,    host: 5000
        cn.vm.network "forwarded_port", guest: 6080,    host: 6080
        cn.vm.network "forwarded_port", guest: 8774,    host: 8774
        cn.vm.network "forwarded_port", guest: 9292,    host: 9292
        cn.vm.network "forwarded_port", guest: 9696,    host: 9696
        cn.vm.network "forwarded_port", guest: 11211,   host: 11211
        cn.vm.network "forwarded_port", guest: 35357,   host: 35357

        cn.vm.provider "virtualbox" do |vb|
            vb.memory = "4096"
            # vb.customize ["modifyhd", "disk id", "--resize", "81920"]
        end

        cn.vm.provision "shell", path: "scripts/apt-cleaner.sh"

        cn.vm.provision "chef_solo" do |chef|
            chef.cookbooks_path = ['chef/cookbooks']
            chef.nodes_path = ['chef/nodes']

            chef.json = {
                "openstack" => {
                    "admin_user" => admin_user,
                    "admin_password" => admin_password
                },

                "mongodb" => {
                    "config" => {
                        "bind-ip" => "10.0.0.11",
                        "smallfiles" => true
                    }
                },
                "memcached" => {
                    "listen" => "10.0.0.11"
                }
            }

            chef.add_recipe 'chrony'
            chef.add_recipe 'mongodb'
            chef.add_recipe 'rabbitmq'
            # chef.add_recipe 'memcached'
            chef.add_recipe 'openstack::controller_installer'

        end

    end

    config.vm.define "compute1" do |cn|
        cn.vm.hostname="compute1"

        cn.vm.network "private_network",
            ip: "10.0.0.31",
            netmask: "255.255.255.0"
        cn.vm.network "public_network",
            bridge: "eth1",
            nic_type: "virtio"


        cn.vm.provider "virtualbox" do |vb|
            vb.memory = "16192"
            vb.cpus = 4
        end

        cn.vm.provision "shell", path: "scripts/apt-cleaner.sh"

        cn.vm.provision "chef_solo" do |chef|
            chef.cookbooks_path = ['chef/cookbooks']
            chef.add_recipe 'chrony'
            chef.add_recipe 'openstack::compute_installer'
        end
    end

    # config.vm.define "block_storage1" do |bs|
    #     cn.vm.name "block1"
    #
    #       bs.vm.synced_folder "foo/", "/foo/bar"
    #
    #       cn.vm.network "private_network",
    #           ip: "10.0.0.41",
    #           netmask: "255.255.255.0"
    #
    #       cn.vm.provider "virtualbox" do |vb|
    #           vb.memory = "2048"
    #       end
    #
    #       cn.vm.provision "chef_solo" do |chef|
    #           chef.cookbooks_path = ['chef/cookbooks']
    #           chef.add_recipe 'chrony'
    #           chef.add_recipe 'openstack:block_storage'
    #       end
    # end

end
