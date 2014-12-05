# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  configfile = "#{Dir::getwd}/share/proxy.yml"
  ENABLE_PROXY = File.exist?(configfile)

  if ENABLE_PROXY
    proxy = YAML.load_file(configfile)
    config.proxy.http    = proxy["http"]
    config.proxy.https   = proxy["https"]
    config.proxy.no_proxy = proxy["no_proxy"]
  end

  config.vm.define "install-manager"
  config.vm.box = "http://knowledgedatabase.info/boxes/centos-6.5-x86_64.box"
  config.vm.hostname = "install-manager"
  config.vm.synced_folder "./share", "/vagrant", mount_options: ["dmode=755", "fmode=755"]
  config.vm.network :private_network, ip:"192.168.56.15"

  config.vm.provider :virtualbox do |v, override|
    v.customize ["modifyvm", :id, "--memory", 2 * 1024]
    v.customize ["modifyvm", :id, "--cpus", "1"]
  end

  $script = <<-SCRIPT
    yum -y update
    if [ "${LANG}" != 'ja_JP.utf8' ]; then
        sed -i 's/^/#/g' /etc/sysconfig/i18n
        echo 'LANG="ja_JP.utf8"' >> /etc/sysconfig/i18n
    fi
    yum install dhcp httpd tftp-server syslinux createrepo -y
    chkconfig iptables off
    service iptables stop
    chkconfig dhcpd on
    chkconfig httpd on
    chkconfig tftp on
    chkconfig xinetd on
    if [ -f /vagrant/image/centos-6.5-x86_64-minimal.iso ]; then
      mount -o loop -t iso9660 /vagrant/image/centos-6.5-x86_64-minimal.iso /media
      mkdir /tmp/centos-6.5-x86_64-minimal
      cp -r /media/* /tmp/centos-6.5-x86_64-minimal
      mv /tmp/centos-6.5-x86_64-minimal /var/www/html
    fi
    cp /vagrant/config/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf
    if [ ! -d /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal ]; then
      mkdir /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal
    fi
    if [ -d /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal ]; then
      cp /media/isolinux/{initrd.img,vmlinuz} /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/
      cp /usr/share/syslinux/{pxelinux.0,menu.c32} /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/
     touch /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/boot.msg
     mkdir /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/pxelinux.cfg
    fi
    cp /vagrant/config/centos-6.5-x86_64-minimal/default /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/pxelinux.cfg/
    mkdir /var/www/html/ks
    cp /vagrant/config/centos-6.5-x86_64-minimal/default.cfg /var/www/html/ks/default.cfg
    chown -R apache:apache /var/www/html
    service dhcpd start
    service xinetd start
    service httpd start
  SCRIPT

  config.vm.provision :shell do |s|
    s.inline = $script
  end

end
