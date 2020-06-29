$num_nodes = 2
$vm_cpus = 2
$vm_memory = 8192
$vm_box = "centos/7"
$vm_box_version = "1905.1"
$k8s_version = "1.15.3"
$k8s_cluster_ip_tpl = "172.16.77.%s"
$k8s_master_ip = $k8s_cluster_ip_tpl % "10"
$vm_name_tpl = "vg-k8s-%s"

Vagrant.configure("2") do |config|
    # always use Vagrants insecure key
    config.ssh.insert_key = false

    config.vm.define "master", primary: true do |master|
        master.vm.box = $vm_box
		master.vm.box_version = $vm_box_version
		master.vm.box_check_update = false
		master.vm.hostname = $vm_name_tpl % "master"
		master.vm.network "private_network", ip: $k8s_master_ip

		master.vm.provider "virtualbox" do |vb|
			vb.name = $vm_name_tpl % "master"
			vb.memory = $vm_memory
			vb.cpus = 2 #$vm_cpus
            vb.gui = false
            vb.linked_clone = true
            vb.customize ["modifyvm", :id, "--vram", "8"] 
        end
        
        master.vm.provision "ansible" do |ansible|
            ansible.playbook = "setup/centos/master-playbook.yml"
            ansible.compatibility_mode = "2.0"
            ansible.verbose = "vvv"
            ansible.extra_vars = {
                node_ip: $k8s_master_ip,
            }
        end
    end

    (1..$num_nodes).each do |i|
		config.vm.define "node#{i}" do |node|
          node.vm.box = $vm_box
			node.vm.box_version = $vm_box_version
			node.vm.box_check_update = false
			node.vm.hostname = $vm_name_tpl % "node-#{i}"
			node.vm.network "private_network", ip: $k8s_cluster_ip_tpl % "#{i+10}"

			node.vm.provider "virtualbox" do |vb|
				vb.name = $vm_name_tpl % "node-#{i}"
				vb.memory = $vm_memory
				vb.cpus = $vm_cpus
                vb.gui = false
                vb.linked_clone = true
                vb.customize ["modifyvm", :id, "--vram", "8"] 
            end
            
            node.vm.provision "ansible" do |ansible|
                ansible.playbook = "setup/centos/node-playbook.yml"
                ansible.compatibility_mode = "2.0"
                ansible.verbose = "vvvv"
                ansible.extra_vars = {
                    node_ip: $k8s_cluster_ip_tpl % "#{i+10}",
                }
            end
		end
	end
end