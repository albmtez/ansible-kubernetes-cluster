# -*- mode: ruby -*-
# vi: set ft=ruby :

################################################################################
# k8s cluster
################################################################################

# You can configure the cluster editting the following values.
# Numer of worker nodes. This value will be overriden with WORKERS environment
# variable's value if set.
num_worker_nodes = 3
# Virtualbox settings for the master node:
#   - vCPU per VM.
vm_master_cpu = 2
#   - Memory (in KB) per VM.
vm_master_mem = 2048
# Virtualbox settings for the worker nodes:
#   - vCPU per VM.
vm_worker_cpu = 2
#   - Memory (in KB) per VM.
vm_worker_mem = 2048

num_workers = ENV['WORKERS'] || num_worker_nodes

ip_count = 10

master_name = "k8s-master"
master_ip = "10.0.10.10"

worker_nodes = []
(1..num_workers).each do |n|
    ip_count = ip_count + 1
    worker_nodes.push(:name => "k8s-worker#{n}", :ip => "10.0.10.#{ip_count}")
end

# Ansible inventory file.
File.open("./hosts", "w") { |file|
    file.write("[masters]\n")
    file.write("#{master_name} ansible_host=#{master_ip} ansible_user=vagrant\n\n")
    file.write("[workers]\n")

    worker_nodes.each do |node|
        file.write("#{node[:name]} ansible_host=#{node[:ip]} ansible_user=vagrant\n")
    end

    file.write("\n[all:vars]\n")
    file.write("ansible_python_interpreter=/usr/bin/python3\n")
}

Vagrant.configure("2") do |config|

    # Disabling folder syncing.
    config.vm.synced_folder '.', '/vagrant', disabled: true

    # ssh keys configuration.
    config.ssh.insert_key = false
    config.ssh.private_key_path = ['~/.vagrant.d/insecure_private_key', './roles/linux-config/files/id_rsa']
    config.vm.provision "file", source: "./roles/linux-config/files/id_rsa.pub", destination: "~/.ssh/authorized_keys"

    # Don't update guest additions if plugin vagrant-vbguest installed.
    if Vagrant.has_plugin?("vagrant-vbguest")
        config.vbguest.auto_update = false
    end

    config.vm.define "#{master_name}" do |master|
        master.vm.box = "albmtez/debian10_x64"
        master.vm.network "private_network", ip: "#{master_ip}"

        master.vm.provider :virtualbox do |v|
            v.gui = false
            v.name = "#{master_name}"
            v.cpus = "#{vm_master_cpu}"
            v.memory = "#{vm_master_mem}"
        end

        master.vm.provider :libvirt do |libvirt|
            libvirt.default_prefix = "k8s_cluster"
            libvirt.cpus = "#{vm_master_cpu}"
            libvirt.memory = "#{vm_master_mem}"
        end
    end

    worker_nodes.each do |node|
        config.vm.define node[:name] do |i|
            i.vm.box = "albmtez/debian10_x64"
            i.vm.network "private_network", ip: "#{node[:ip]}"

            i.vm.provider :virtualbox do |v|
                v.gui = false
                v.name = "#{node[:name]}"
                v.cpus = "#{vm_worker_cpu}"
                v.memory = "#{vm_worker_mem}"
            end

            i.vm.provider :libvirt do |libvirt|
                libvirt.default_prefix = "k8s_cluster"
                libvirt.cpus = "#{vm_worker_cpu}"
                libvirt.memory = "#{vm_worker_mem}"
            end
        end
    end
    
    # Provision machines using ansible-playbook after vms creation with vagrant.
    #
    # config.vm.provision "ansible_kubernetes_cluster", type:'ansible' do |ansible|
    #     ansible.inventory_path = "./hosts"
    #     ansible.playbook = "kubernetes-cluster.yml"
    #     ansible.extra_vars = {
    #         "master_ip": "#{master_ip}"
    #     }
    # end

end