Vagrant.configure("2") do |config|
    # Nodes
    NodeCount = 2
    (1..NodeCount).each do |i|
        config.vm.define "virtualbox-guest-vm-#{i}" do |vv|
            vv.vm.box = "generic/ubuntu2004"
            vv.vm.box_version = "3.6.8"
            vv.vm.hostname = "virtualbox-guest-vm-#{i}"
            vv.vm.network "private_network", ip: "172.16.16.1#{i}"

            vv.vm.provider "virtualbox" do |vb|
                vb.name = "virtualbox-guest-vm-#{i}"
                vb.memory = 1024
                vb.cpus = 1
            end
        end
    end

    config.vm.provision "shell" do |s|
        ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
        s.inline = <<-SHELL
            # Create ci user
            useradd -s /bin/bash -d /home/ci/ -m -G sudo ci
            echo 'ci ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers
            mkdir -p /home/ci/.ssh && chown -R ci /home/ci/.ssh
            echo #{ssh_pub_key} >> /home/ci/.ssh/authorized_keys
        SHELL
    end
end
