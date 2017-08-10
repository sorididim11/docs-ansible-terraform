
Vagrant.configure('2') do |config|
  config.vm.box = 'centos/7'
  config.vm.synced_folder '.', '/vagrant', type: 'virtualbox'
  config.ssh.insert_key = false # 머쉰들이 공통으로 ~/vagrant.d/insecure-key 를 사용 즉 ansible-contoller에 private 키만 옴기면 됨.   if true key pairs are generated for each machine
  # config.ssh.private_key_path = '~/.vagrant.d/insecure_private_key'


  if Vagrant.has_plugin?('vagrant-hostmanager')
    config.hostmanager.manage_guest = true
    config.hostmanager.ignore_private_ip = false
    config.hostmanager.include_offline = true
  end

  if Vagrant.has_plugin?('vagrant-cachier')
    config.cache.scope = :box
  end

  if Vagrant.has_plugin?('vagrant-vbguest')
     # config.vbguest.auto_update = false
  end

  # if Vagrant.has_plugin?('vagrant-proxyconf')
  #   config.proxy.http = 'http://web-proxy.corp.hp.com:8080'
  #   config.proxy.https = 'http://web-proxy.corp.hp.com:8080'
  #   config.proxy.no_proxy = 'localhost, 127.0.0.1'
  # end

  {
    # vboxnet4 10.0.15.1 with no dhcp
    # Virtualbox natsystem을 vagrant에서 아직 사용 불가능 따라서 nat + host-oly(vboxnet4 을 현재 사용) 해야함. 
    # nat은  vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']  추가 하면 사용. 
    # vagrant가 설정 host-only virtualbox에서 선택 dhcp 는 disable 시켜야함 
    'slave1' => { 'ip' => '10.0.15.30', 'cpu' => '30', 'mem' => 500 },
    'master' => { 'ip' => '10.0.15.20', 'cpu' => '60', 'mem' => 1000 },
    'bootstrap' => { 'ip' => '10.0.15.10', 'cpu' => '20', 'mem' => 300 }
  }.each do |name, resource|
    config.vm.define name do |node|
      node.vm.hostname = name
      node.vm.network :private_network, ip: resource['ip']
      node.vm.provider 'virtualbox' do |vb|
        vb.linked_clone = true
        vb.memory = resource['mem']
        vb.customize ['modifyvm', :id, '--natdnshostresolver1', 'on']
        vb.customize ['modifyvm', :id, '--cpuexecutioncap', resource['cpu']] # 20% 씩 사용 
        vb.customize [ 'guestproperty', 'set', :id, '/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold', 1000 ] # for dcos ntptime
      end

      if name == 'bootstrap'
        node.vm.provision "shell" do |s|
          ssh_insecure_key = File.readlines("#{Dir.home}/.vagrant.d/insecure_private_key")
          s.inline = <<-SHELL
            echo #{ssh_insecure_key} >> /home/vagrant/.ssh/id_rsa
            chown vagrant /home/vagrant/.ssh/id_rsa
            chmod 400 /home/vagrant/.ssh/id_rsa
          SHELL
        end

        node.vm.provision :ansible_local do |ansible|
          ansible.install_mode = :pip # or default( by os package manager)
          ansible.version = '2.3.1'
          ansible.config_file = 'ansible/ansible.cfg'
          ansible.inventory_path = 'ansible/inventories/ivagrant/hosts'
         # ansible.playbook = 'ansible/playbooks/util-config-ohmyzsh.yml'
          ansible.playbook = 'ansible/site.yml'
          ansible.limit = 'all'
          ansible.verbose = 'true'
          ansible.vault_password_file = 'ansible/password'
        end
      end
    end
  end
end