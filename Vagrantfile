# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  configfile = "#{Dir::getwd}/share/config/proxy.yml"
  ENABLE_PROXY = File.exist?(configfile)

  if ENABLE_PROXY
    proxy = YAML.load_file(configfile)
    config.proxy.http    = proxy["proxy"]["http"]
    config.proxy.https   = proxy["proxy"]["https"]
    config.proxy.no_proxy = proxy["proxy"]["no_proxy"]
  end

  config.vm.define "install-manager"
  config.vm.box = "centos-6.5-x86_64"
  config.vm.hostname = "install-manager"
  config.vm.synced_folder ".", "/vagrant", mount_options: ["dmode=755", "fmode=755"]
  config.vm.network :private_network, ip:"192.168.56.15"

  config.vm.provider :virtualbox do |v, override|
    v.customize ["modifyvm", :id, "--memory", 2 * 1024]
    v.customize ["modifyvm", :id, "--cpus", "1"]
  end

  $script = <<-SCRIPT
    yum -y update
    sed -i 's/^/#/g' /etc/sysconfig/i18n
    echo 'LANG="ja_JP.utf8"' >> /etc/sysconfig/i18n
    yum install dhcp httpd tftp-server syslinux createrepo -y
    chkconfig dhcpd on
    chkconfig httpd on
    chkconfig tftp on
    chkconfig xinetd on
    service iptables stop
    curl -L http://ftp.iij.ad.jp/pub/linux/centos/6.5/isos/x86_64/CentOS-6.5-x86_64-minimal.iso -o /tmp/CentOS-6.5-x86_64-minimal.iso
    mount -o loop -t iso9660 /tmp/CentOS-6.5-x86_64-minimal.iso /media
    mkdir /tmp/centos-6.5-x86_64-minimal
    cp -r /media/ /tmp/centos-6.5-x86_64-minimal
    mv /tmp/centos-6.5-x86_64-minimal /var/www/html
    chown apache:apache /var/www/html
    echo "subnet 192.168.56.0 netmask 255.255.255.0" >> /etc/dhcp/dhcpd.conf
    echo "{" >> /etc/dhcp/dhcpd.conf
    echo "    default-lease-time     21600;" >> /etc/dhcp/dhcpd.conf
    echo "    max-lease-time          43200;" >> /etc/dhcp/dhcpd.conf
    echo "    option routers          192.168.56.1;" >> /etc/dhcp/dhcpd.conf
    echo "    option subnet-mask     255.255.255.0;" >> /etc/dhcp/dhcpd.conf
    echo "    range                     192.168.56.128 192.168.56.191;" >> /etc/dhcp/dhcpd.conf
    echo "}" >> /etc/dhcp/dhcpd.conf
    echo "" >> /etc/dhcp/dhcpd.conf
    echo "host client01" >> /etc/dhcp/dhcpd.conf
    echo "{" >> /etc/dhcp/dhcpd.conf
    echo "        hardware ethernet      08:00:27:6E:6D:78;" >> /etc/dhcp/dhcpd.conf
    echo "        fixed-address           192.168.56.11;" >> /etc/dhcp/dhcpd.conf
    echo "        filename                 "pxeboot_centos-6.5-x86_64-minimal/pxelinux.0";" >> /etc/dhcp/dhcpd.conf
    echo "        next-server             192.168.1.2;" >> /etc/dhcp/dhcpd.conf
    echo "}" >> /etc/dhcp/dhcpd.conf
    cd /var/lib/tftpboot/
    mkdir pxeboot_centos-6.5-x86_64-minimal
    cp /media/isolinux/{initrd.img,vmlinuz} /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/
    cp /usr/share/syslinux/{pxelinux.0,menu.c32} /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/
    touch /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/boot.msg
    mkdir /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/pxelinux.cfg
    echo "prompt 0" >> /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/pxelinux.cfg/default
    echo "default menu.c32" >> /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/pxelinux.cfg/default
    echo "timeout 300" >> /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/pxelinux.cfg/default
    echo "menu title === PXE Boot Menu ===" >> /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/pxelinux.cfg/default
    echo "menu tabmsg Please Select a Number" >> /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/pxelinux.cfg/default
    echo "ontimeout centos65_core" >> /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/pxelinux.cfg/default
    echo "label centos65_core" >> /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/pxelinux.cfg/default
    echo "    menu label ^1 CentOS 6.5 core" >> /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/pxelinux.cfg/default
    echo "    kernel vmlinuz" >> /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/pxelinux.cfg/default
    echo "    append initrd=initrd.img ks=http://192.168.56.15/ks/default.cfg ksdevice=bootif" >> /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/pxelinux.cfg/default
    echo "    ipappend 2" >> /var/lib/tftpboot/pxeboot_centos-6.5-x86_64-minimal/pxelinux.cfg/default
    mkdir /var/www/html/ks
    cp /root/anaconda-ks.cfg /var/www/html/ks/default.cfg
    sed -i 's/^.#/repo --name="DVD" --baseurl=http://192.168.56.15/ntos-6.5-x86_64-minimal cost=100/g' /var/www/html/ks/default.cfg
    service xinetd start
    service httpd start
    service dhcpd start
  SCRIPT
  # /etc/init.d/xinetd start
  # /etc/init.d/httpd start
  # iptables -A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
  # iptables -A INPUT -m state --state NEW -m udp -p udp --dport 67 -j ACCEPT
  # iptables -A INPUT -m state --state NEW -m udp -p udp --dport 69 -j ACCEPT
  #   curl -L http://ftp.iij.ad.jp/pub/linux/centos/6.5/isos/i386/CentOS-6.5-i386-minimal.iso -o /tmp/CentOS-6.5-i386-minimal.iso

  config.vm.provision :shell do |s|
    s.inline = $script
  end

end
