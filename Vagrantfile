# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

    config.vm.box = "vinik/ubuntu"

    config.vm.define "controller_node" do |cn|

        cn.vm.network "private_network",
            ip: "10.0.0.11",
            netmask: "255.255.255.0"

        cn.vm.provider "virtualbox" do |vb|
            vb.memory = "4096"
        end

        cn.vm.provision "chef_solo" do |chef|
            chef.cookbooks_path = ['chef/cookbooks']

            chef.json = {
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
            chef.add_recipe 'memcached'
            chef.add_recipe 'openstack:controller'
        end

    end

    config.vm.define "compute_node1" do |cn|

        cn.vm.network "private_network",
            ip: "10.0.0.31",
            netmask: "255.255.255.0"

        cn.vm.provider "virtualbox" do |vb|
            vb.memory = "2048"
        end

        cn.vm.provision "chef_solo" do |chef|
            chef.cookbooks_path = ['chef/cookbooks']
            chef.add_recipe 'chrony'
            chef.add_recipe 'openstack:compute'
        end
    end

    config.vm.define "block_storage1" do |bs|
          bs.vm.synced_folder "foo/", "/foo/bar"

          cn.vm.network "private_network",
              ip: "10.0.0.41",
              netmask: "255.255.255.0"

          cn.vm.provider "virtualbox" do |vb|
              vb.memory = "2048"
          end

          cn.vm.provision "chef_solo" do |chef|
              chef.cookbooks_path = ['chef/cookbooks']
              chef.add_recipe 'chrony'
              chef.add_recipe 'openstack:block_storage'
          end
    end

end
