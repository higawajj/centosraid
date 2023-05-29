# -*- mode: ruby -*-
# vi: set ft=ruby :

MACHINES = {
  :centosraid => {
    :box_name => "centos/7",
	  :disks => {
		  :sata1 => {
			  :dfile => './sata1.vdi',
			  :size => 250,
			  :port => 6
		  },
		  :sata2 => {
        :dfile => './sata2.vdi',
        :size => 250, 
			  :port => 7
		  },
      :sata3 => {
        :dfile => './sata3.vdi',
        :size => 250,
        :port => 8
      },
      :sata4 => {
        :dfile => './sata4.vdi',
        :size => 250, 
        :port => 9
      },
      :sata5 => {
        :dfile => './sata5.vdi',
        :size => 250, 
        :port => 10
      }
	  }
  },
}

Vagrant.configure("2") do |config|
  config.vbguest.auto_update = false
  MACHINES.each do |boxname, boxconfig|
      config.vm.define boxname do |box|
          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s
          #box.vm.network "private_network", ip: boxconfig[:ip_addr]
          box.vm.provider :virtualbox do |vb|
              vb.customize ["modifyvm", :id, "--memory", "1024"]
              needsController = false
		          boxconfig[:disks].each do |dname, dconf|
			            unless File.exist?(dconf[:dfile])
				              vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                      needsController =  true
                  end
		          end
              if needsController == true
                  vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                  boxconfig[:disks].each do |dname, dconf|
                      vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                  end
              end
          end
 	        box.vm.provision "shell", inline: <<-SHELL
	            mkdir -p ~root/.ssh
              cp ~vagrant/.ssh/auth* ~root/.ssh
	            yum install -y mdadm smartmontools hdparm gdisk
              #sudo su
              mdadm --zero-superblock --force /dev/sd{a,b,c,d,e}
              mdadm --create --verbose /dev/md99 -l 5 -n 5 /dev/sd{a,b,c,d,e}
              mkdir /etc/mdadm && touch /etc/mdadm/mdadm.conf
              chmod 666 /etc/mdadm/mdadm.conf
              echo "DEVICE partitions" > /etc/mdadm/mdadm.conf
              mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf
              parted -s /dev/md99 mklabel gpt
              for i in $(seq 1 5)
              do
                parted /dev/md99 mkpart primary ext4 $(($(($i-1))*20)) $(($i*20))
                mkfs.ext4 /dev/md99p$i
                mkdir -p /raid/part$i
                mount /dev/md99p$i /raid/part$i
                echo "/dev/md99p$i /raid/part$i ext4 defaults 0 $(($i+2))" >> /etc/fstab
              done
  	      SHELL
      end
  end
end

