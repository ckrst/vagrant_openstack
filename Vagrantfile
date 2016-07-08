# -*- mode: ruby -*-
# vi: set ft=ruby :

admin_env = {
    "OS_PROJECT_DOMAIN_NAME" => "default",
    "OS_USER_DOMAIN_NAME" => "default",
    "OS_PROJECT_NAME" => "admin",
    "OS_USERNAME" => "admin",
    "OS_PASSWORD" => "secret",
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

    config.vm.define "controller_node" do |cn|

        cn.vm.hostname="controller"

        cn.vm.network "private_network",
            ip: "10.0.0.11",
            netmask: "255.255.255.0"
        cn.vm.network "forwarded_port", guest: 80, host: 8080
        cn.vm.network "forwarded_port", guest: 3306, host: 3306
        cn.vm.network "forwarded_port", guest: 5000, host: 5000
        cn.vm.network "forwarded_port", guest: 6080, host: 6080
        cn.vm.network "forwarded_port", guest: 9292, host: 9292
        cn.vm.network "forwarded_port", guest: 9696, host: 9696
        cn.vm.network "forwarded_port", guest: 11211, host: 11211
        cn.vm.network "forwarded_port", guest: 35357, host: 35357

        cn.vm.provider "virtualbox" do |vb|
            vb.memory = "4096"
        end

        cn.vm.provision "shell", path: "scripts/apt-cleaner.sh"

        cn.vm.provision "chef_solo" do |chef|
            chef.cookbooks_path = ['chef/cookbooks']
            chef.nodes_path = ['chef/nodes']

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
            chef.add_recipe 'openstack::controller'

            chef.add_recipe 'openstack::keystone'
            chef.add_recipe 'openstack::glance'
            chef.add_recipe 'openstack::nova'
            chef.add_recipe 'openstack::neutron'
        end

        # KEYSTONE
        cn.vm.provision "shell", inline: 'sudo -s /bin/sh -c "keystone-manage db_sync" keystone'
        cn.vm.provision "shell", inline: 'sudo keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone'
        cn.vm.provision "shell", inline: 'sudo service apache2 restart'
        cn.vm.provision "shell" do |shell|
            shell.path = "scripts/keystone.sh"
            shell.privileged = false
            shell.env = {
                "OS_TOKEN"                  => "71e444e5726be697906c",
                "OS_URL"                    => "http://controller:35357/v3",
                "OS_IDENTITY_API_VERSION"   => 3
            }
        end
        #cn.vm.provision "shell", inline: 'openstack --os-auth-url http://controller:35357/v3 --os-project-domain-name default --os-user-domain-name default --os-project-name admin --os-username admin token issue'
        cn.vm.provision "shell" do |shell|
            shell.inline = "openstack token issue"
            shell.privileged = false
            shell.env = admin_env
        end
        #cn.vm.provision "shell", inline: 'openstack --os-auth-url http://controller:5000/v3 --os-project-domain-name default --os-user-domain-name default --os-project-name demo --os-username demo token issue'
        cn.vm.provision "shell" do |shell|
            shell.inline = "openstack token issue"
            shell.privileged = false
            shell.env = user_env
        end


        # GLANCE

        # glance endpoints
        cn.vm.provision "shell" do |shell|
            shell.path = "scripts/glance.sh"
            shell.privileged = false
            shell.env = admin_env
        end
        cn.vm.provision "shell", inline: 'sudo -s /bin/sh -c "glance-manage db_sync" glance'
        cn.vm.provision "shell", inline: 'sudo service glance-registry restart'
        cn.vm.provision "shell", inline: 'sudo service glance-api restart'
        # Download ubuntu image
        cn.vm.provision "shell" do |shell|
            shell.inline = "cd /home/vagrant && wget https://cloud-images.ubuntu.com/trusty/current/trusty-server-cloudimg-amd64-disk1.img"
            shell.privileged = false
            shell.env = admin_env
        end
        # Import ubuntu on glance
        cn.vm.provision "shell" do |shell|
            shell.inline = 'openstack image create "ubuntu" --file trusty-server-cloudimg-amd64-disk1.img --disk-format qcow2 --container-format bare --public'
            shell.privileged = false
            shell.env = admin_env
        end

        #
        # # NOVA
        # cn.vm.provision "shell", inline: 'sudo -s /bin/sh -c "nova-manage api_db sync" nova'
        # cn.vm.provision "shell", inline: 'sudo -s /bin/sh -c "nova-manage db sync" nova'
        # cn.vm.provision "shell" do |shell|
        #     shell.path = "scripts/nova.sh"
        #     shell.privileged = false
        #     shell.env = admin_env
        # end
        #
        # # NEUTRON
        # cn.vm.provision "shell" do |shell|
        #     shell.path = "scripts/neutron.sh"
        #     shell.privileged = false
        #     shell.env = admin_env
        # end
        # cn.vm.provision "shell", inline: 'su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron'

    end

    # config.vm.define "compute_node1" do |cn|
    #     cn.vm.name "compute1"
    #
    #     cn.vm.network "private_network",
    #         ip: "10.0.0.31",
    #         netmask: "255.255.255.0"
    #
    #     cn.vm.provider "virtualbox" do |vb|
    #         vb.memory = "1024"
    #     end
    #
    #     cn.vm.provision "chef_solo" do |chef|
    #         chef.cookbooks_path = ['chef/cookbooks']
    #         chef.add_recipe 'chrony'
    #         chef.add_recipe 'openstack:compute'
    #     end
    # end

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
