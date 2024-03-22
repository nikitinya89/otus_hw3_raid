MACHINES = {
  :vm1 => {
    :boxname => "generic/alma9",
    :hostname => "alma9",
    :ip => "192.168.111.201",
    :mask => "255.255.255.0",
    :network_name => "mynet",
    :disks => {
      :sata1 => {
        :dfile => "./disk1.vdi",
        :size => "100",
        :port => "1"
      },
      :sata2 => {
        :dfile => "./disk2.vdi",
        :size => "100",
        :port => "2"
      },
      :sata3 => {
        :dfile => "./disk3.vdi",
        :size => "100",
        :port => "3"
      },
      :sata4 => {
        :dfile => "./disk4.vdi",
        :size => "100",
        :port => "4"
      },
      :sata5 => {
        :dfile => "./disk5.vdi",
        :size => "100",
        :port => "5"
      }
    }
  }
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconf|
    config.vm.define boxname do |box|
      box.vm.box = boxconf[:boxname]
      box.vm.hostname = boxconf[:hostname]
      box.vm.network "private_network", ip: boxconf[:ip], adapter: 2, netmask: boxconf[:mask], virtualbox__intnet: boxconf[:network_name]
      box.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
        vb.cpus = "1"
        vb.name = "vm_alma9"
        boxconf[:disks].each do |dname, dconf|
          unless File.exist?(dconf[:dfile])
            vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
          end
        vb.customize ['storageattach', :id,  '--storagectl', 'SATA Controller', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
        end
      end
      ssh_pub_key = File.readlines("./id_rsa.pub").first.strip
      box.vm.provision "shell", inline: <<-SHELL
        echo #{ssh_pub_key} >> ~vagrant/.ssh/authorized_keys
        echo #{ssh_pub_key} >> ~root/.ssh/authorized_keys
        sudo sed -i 's/\#PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
        systemctl restart sshd
      SHELL
    end
  end
end
