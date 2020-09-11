---
layout: post
title: "Auto Init Docker env by Vagrant"
date: 2020-09-10 01:37:42
image: '/assets/img/'
description: '自动初始化 Docker 环境'
main-class: 'code'
color: '#fbbd04' 
tags:
 - snippet
 - vagrant
 - docker
 - devops
 - code
categories:
 - code
twitter_text: 'Auto Init Docker env by Vagrant'
introduction: 'Auto Init Docker env by Vagrant'
---

# 前言

开启一个 **CODE** 系列，用来收集各种好用的小片段

这一篇用来展示通过 Vagrant 和 VirtualBox 自动初始化一个可用的 Docker 环境

> **Tip:** 展示中使用的各种软件版本可能不是最新，但我都会尽量提前交代

---

# 操作

## 依赖

* Vagrant 2.2.9

~~~
➜  blog vagrant --version
Vagrant 2.2.9
➜  blog
~~~

* VirtualBox 6.0.16

~~~
VirtualBox Graphical User Interface
Version 6.0.16 r135674 (Qt5.6.3)
~~~

## OS 环境

~~~
➜  blog sw_vers
ProductName:	Mac OS X
ProductVersion:	10.15.4
BuildVersion:	19E287
➜  blog
~~~


## 代码

~~~
➜  docker_C7X64 cat Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

hosts = {
  "docker" => "10.0.0.210"
}


$script = <<SCRIPT
## set password of root
echo "root:root" | chpasswd
echo "[DONE] root password"

## disable swap
swapoff -a
sed -i '/swap/d' /etc/fstab
echo "[DONE] disable swap"

## load br_netfilter
modprobe br_netfilter
modprobe ip_vs_dh
modprobe nf_conntrack_ipv4
cat <<EOF  >>/etc/rc.local
modprobe br_netfilter
modprobe ip_vs_dh
modprobe nf_conntrack_ipv4
EOF
echo "[DONE] load module"


## set hosts
cat > /etc/hosts << EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.0.0.210  docker
EOF
echo "[DONE] hosts is set"

## set kernel
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
echo "[DONE] kernel is set"

## disalbe SElinux
setenforce 0
sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
echo "[DONE] selinux is disabled"

## install package
yum install -y docker net-tools curl vim
echo "[DONE] package is installed"


## set registry-mirrors
cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
EOF
echo "[DONE] registry-mirrors are set"

## enable service
systemctl start docker
systemctl enable docker
echo "[DONE] services are enabled"

docker version

## pull image
echo "[DONE] images are pulled"

SCRIPT


Vagrant.configure("2") do |config|
  hosts.each do |name,ip|
    config.vm.define name do |vb|
      vb.vm.box = "centos/7"
      vb.vm.box_check_update = false
      vb.vm.network "private_network", ip: ip
      #vb.vm.synced_folder ".", "/docker", type: "nfs"
      vb.vm.hostname = name
      vb.vm.provision "shell", inline: $script
      vb.vm.provider "virtualbox" do |node|
        node.name = name
        node.memory = "4096"
        node.cpus = "2"
      end
    end
  end
end
➜  docker_C7X64
~~~


## 执行

~~~
➜  docker_C7X64 time vagrant up
Bringing machine 'docker' up with 'virtualbox' provider...
==> docker: Importing base box 'centos/7'...
==> docker: Matching MAC address for NAT networking...
==> docker: Setting the name of the VM: docker
==> docker: Fixed port collision for 22 => 2222. Now on port 2201.
==> docker: Clearing any previously set network interfaces...
==> docker: Preparing network interfaces based on configuration...
    docker: Adapter 1: nat
    docker: Adapter 2: hostonly
==> docker: Forwarding ports...
    docker: 22 (guest) => 2201 (host) (adapter 1)
==> docker: Running 'pre-boot' VM customizations...
==> docker: Booting VM...
==> docker: Waiting for machine to boot. This may take a few minutes...
    docker: SSH address: 127.0.0.1:2201
    docker: SSH username: vagrant
    docker: SSH auth method: private key
    docker:
    docker: Vagrant insecure key detected. Vagrant will automatically replace
    docker: this with a newly generated keypair for better security.
    docker:
    docker: Inserting generated public key within guest...
    docker: Removing insecure key from the guest if it's present...
    docker: Key inserted! Disconnecting and reconnecting using new SSH key...
==> docker: Machine booted and ready!
==> docker: Checking for guest additions in VM...
    docker: No guest additions were detected on the base box for this VM! Guest
    docker: additions are required for forwarded ports, shared folders, host only
    docker: networking, and more. If SSH fails on this machine, please install
    docker: the guest additions and repackage the box to continue.
    docker:
    docker: This is not an error message; everything may continue to work properly,
    docker: in which case you may ignore this message.
==> docker: Setting hostname...
==> docker: Configuring and enabling network interfaces...
==> docker: Rsyncing folder: /Users/zheng.fang/Vagrant/DEVOPS/docker_C7X64/ => /vagrant
==> docker: Running provisioner: shell...
    docker: Running: inline script
    docker: [DONE] root password
    docker: [DONE] disable swap
    docker: [DONE] load module
    docker: [DONE] hosts is set
    docker: * Applying /usr/lib/sysctl.d/00-system.conf ...
    docker: net.bridge.bridge-nf-call-ip6tables = 0
    docker: net.bridge.bridge-nf-call-iptables = 0
    docker: net.bridge.bridge-nf-call-arptables = 0
    docker: * Applying /usr/lib/sysctl.d/10-default-yama-scope.conf ...
    docker: kernel.yama.ptrace_scope = 0
    docker: * Applying /usr/lib/sysctl.d/50-default.conf ...
    docker: kernel.sysrq = 16
    docker: kernel.core_uses_pid = 1
    docker: net.ipv4.conf.default.rp_filter = 1
    docker: net.ipv4.conf.all.rp_filter = 1
    docker: net.ipv4.conf.default.accept_source_route = 0
    docker: net.ipv4.conf.all.accept_source_route = 0
    docker: net.ipv4.conf.default.promote_secondaries = 1
    docker: net.ipv4.conf.all.promote_secondaries = 1
    docker: fs.protected_hardlinks = 1
    docker: fs.protected_symlinks = 1
    docker: * Applying /etc/sysctl.d/99-sysctl.conf ...
    docker: * Applying /etc/sysctl.d/k8s.conf ...
    docker: net.bridge.bridge-nf-call-ip6tables = 1
    docker: net.bridge.bridge-nf-call-iptables = 1
    docker: * Applying /etc/sysctl.conf ...
    docker: [DONE] kernel is set
    docker: [DONE] selinux is disabled
    docker: Loaded plugins: fastestmirror
    docker: Determining fastest mirrors
    docker:  * base: mirrors.cn99.com
    docker:  * extras: mirrors.cn99.com
    docker:  * updates: mirrors.cn99.com
    docker: Resolving Dependencies
    docker: --> Running transaction check
    docker: ---> Package curl.x86_64 0:7.29.0-51.el7 will be updated
    docker: ---> Package curl.x86_64 0:7.29.0-57.el7_8.1 will be an update
    docker: --> Processing Dependency: libcurl = 7.29.0-57.el7_8.1 for package: curl-7.29.0-57.el7_8.1.x86_64
    docker: ---> Package docker.x86_64 2:1.13.1-162.git64e9980.el7.centos will be installed
    docker: --> Processing Dependency: docker-common = 2:1.13.1-162.git64e9980.el7.centos for package: 2:docker-1.13.1-162.git64e9980.el7.centos.x86_64
    docker: --> Processing Dependency: docker-client = 2:1.13.1-162.git64e9980.el7.centos for package: 2:docker-1.13.1-162.git64e9980.el7.centos.x86_64
    docker: --> Processing Dependency: subscription-manager-rhsm-certificates for package: 2:docker-1.13.1-162.git64e9980.el7.centos.x86_64
    docker: ---> Package net-tools.x86_64 0:2.0-0.25.20131004git.el7 will be installed
    docker: ---> Package vim-enhanced.x86_64 2:7.4.629-6.el7 will be installed
    docker: --> Processing Dependency: vim-common = 2:7.4.629-6.el7 for package: 2:vim-enhanced-7.4.629-6.el7.x86_64
    docker: --> Processing Dependency: perl(:MODULE_COMPAT_5.16.3) for package: 2:vim-enhanced-7.4.629-6.el7.x86_64
    docker: --> Processing Dependency: libperl.so()(64bit) for package: 2:vim-enhanced-7.4.629-6.el7.x86_64
    docker: --> Processing Dependency: libgpm.so.2()(64bit) for package: 2:vim-enhanced-7.4.629-6.el7.x86_64
    docker: --> Running transaction check
    docker: ---> Package docker-client.x86_64 2:1.13.1-162.git64e9980.el7.centos will be installed
    docker: ---> Package docker-common.x86_64 2:1.13.1-162.git64e9980.el7.centos will be installed
    docker: --> Processing Dependency: skopeo-containers >= 1:0.1.26-2 for package: 2:docker-common-1.13.1-162.git64e9980.el7.centos.x86_64
    docker: --> Processing Dependency: oci-umount >= 2:2.3.3-3 for package: 2:docker-common-1.13.1-162.git64e9980.el7.centos.x86_64
    docker: --> Processing Dependency: oci-systemd-hook >= 1:0.1.4-9 for package: 2:docker-common-1.13.1-162.git64e9980.el7.centos.x86_64
    docker: --> Processing Dependency: oci-register-machine >= 1:0-5.13 for package: 2:docker-common-1.13.1-162.git64e9980.el7.centos.x86_64
    docker: --> Processing Dependency: lvm2 >= 2.02.112 for package: 2:docker-common-1.13.1-162.git64e9980.el7.centos.x86_64
    docker: --> Processing Dependency: container-storage-setup >= 0.9.0-1 for package: 2:docker-common-1.13.1-162.git64e9980.el7.centos.x86_64
    docker: --> Processing Dependency: container-selinux >= 2:2.51-1 for package: 2:docker-common-1.13.1-162.git64e9980.el7.centos.x86_64
    docker: --> Processing Dependency: atomic-registries for package: 2:docker-common-1.13.1-162.git64e9980.el7.centos.x86_64
    docker: ---> Package gpm-libs.x86_64 0:1.20.7-6.el7 will be installed
    docker: ---> Package libcurl.x86_64 0:7.29.0-51.el7 will be updated
    docker: ---> Package libcurl.x86_64 0:7.29.0-57.el7_8.1 will be an update
    docker: --> Processing Dependency: libssh2(x86-64) >= 1.8.0 for package: libcurl-7.29.0-57.el7_8.1.x86_64
    docker: ---> Package perl.x86_64 4:5.16.3-295.el7 will be installed
    docker: --> Processing Dependency: perl(Socket) >= 1.3 for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(Scalar::Util) >= 1.10 for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl-macros for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(threads::shared) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(threads) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(constant) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(Time::Local) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(Time::HiRes) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(Storable) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(Socket) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(Scalar::Util) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(Pod::Simple::XHTML) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(Pod::Simple::Search) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(Getopt::Long) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(Filter::Util::Call) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(File::Temp) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(File::Spec::Unix) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(File::Spec::Functions) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(File::Spec) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(File::Path) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(Exporter) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(Cwd) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: --> Processing Dependency: perl(Carp) for package: 4:perl-5.16.3-295.el7.x86_64
    docker: ---> Package perl-libs.x86_64 4:5.16.3-295.el7 will be installed
    docker: ---> Package subscription-manager-rhsm-certificates.x86_64 0:1.24.26-4.el7.centos will be installed
    docker: ---> Package vim-common.x86_64 2:7.4.629-6.el7 will be installed
    docker: --> Processing Dependency: vim-filesystem for package: 2:vim-common-7.4.629-6.el7.x86_64
    docker: --> Running transaction check
    docker: ---> Package atomic-registries.x86_64 1:1.22.1-33.gitb507039.el7_8 will be installed
    docker: --> Processing Dependency: python-yaml for package: 1:atomic-registries-1.22.1-33.gitb507039.el7_8.x86_64
    docker: --> Processing Dependency: python-setuptools for package: 1:atomic-registries-1.22.1-33.gitb507039.el7_8.x86_64
    docker: --> Processing Dependency: python-pytoml for package: 1:atomic-registries-1.22.1-33.gitb507039.el7_8.x86_64
    docker: ---> Package container-selinux.noarch 2:2.119.2-1.911c772.el7_8 will be installed
    docker: --> Processing Dependency: policycoreutils-python for package: 2:container-selinux-2.119.2-1.911c772.el7_8.noarch
    docker: ---> Package container-storage-setup.noarch 0:0.11.0-2.git5eaf76c.el7 will be installed
    docker: ---> Package containers-common.x86_64 1:0.1.40-11.el7_8 will be installed
    docker: --> Processing Dependency: subscription-manager for package: 1:containers-common-0.1.40-11.el7_8.x86_64
    docker: --> Processing Dependency: slirp4netns for package: 1:containers-common-0.1.40-11.el7_8.x86_64
    docker: --> Processing Dependency: fuse-overlayfs for package: 1:containers-common-0.1.40-11.el7_8.x86_64
    docker: ---> Package libssh2.x86_64 0:1.4.3-12.el7_6.2 will be updated
    docker: ---> Package libssh2.x86_64 0:1.8.0-3.el7 will be an update
    docker: ---> Package lvm2.x86_64 7:2.02.186-7.el7_8.2 will be installed
    docker: --> Processing Dependency: lvm2-libs = 7:2.02.186-7.el7_8.2 for package: 7:lvm2-2.02.186-7.el7_8.2.x86_64
    docker: --> Processing Dependency: device-mapper-persistent-data >= 0.7.0-0.1.rc6 for package: 7:lvm2-2.02.186-7.el7_8.2.x86_64
    docker: --> Processing Dependency: liblvm2app.so.2.2(Base)(64bit) for package: 7:lvm2-2.02.186-7.el7_8.2.x86_64
    docker: --> Processing Dependency: libdevmapper-event.so.1.02(Base)(64bit) for package: 7:lvm2-2.02.186-7.el7_8.2.x86_64
    docker: --> Processing Dependency: libaio.so.1(LIBAIO_0.4)(64bit) for package: 7:lvm2-2.02.186-7.el7_8.2.x86_64
    docker: --> Processing Dependency: libaio.so.1(LIBAIO_0.1)(64bit) for package: 7:lvm2-2.02.186-7.el7_8.2.x86_64
    docker: --> Processing Dependency: liblvm2app.so.2.2()(64bit) for package: 7:lvm2-2.02.186-7.el7_8.2.x86_64
    docker: --> Processing Dependency: libdevmapper-event.so.1.02()(64bit) for package: 7:lvm2-2.02.186-7.el7_8.2.x86_64
    docker: --> Processing Dependency: libaio.so.1()(64bit) for package: 7:lvm2-2.02.186-7.el7_8.2.x86_64
    docker: ---> Package oci-register-machine.x86_64 1:0-6.git2b44233.el7 will be installed
    docker: ---> Package oci-systemd-hook.x86_64 1:0.2.0-1.git05e6923.el7_6 will be installed
    docker: --> Processing Dependency: libyajl.so.2()(64bit) for package: 1:oci-systemd-hook-0.2.0-1.git05e6923.el7_6.x86_64
    docker: ---> Package oci-umount.x86_64 2:2.5-3.el7 will be installed
    docker: ---> Package perl-Carp.noarch 0:1.26-244.el7 will be installed
    docker: ---> Package perl-Exporter.noarch 0:5.68-3.el7 will be installed
    docker: ---> Package perl-File-Path.noarch 0:2.09-2.el7 will be installed
    docker: ---> Package perl-File-Temp.noarch 0:0.23.01-3.el7 will be installed
    docker: ---> Package perl-Filter.x86_64 0:1.49-3.el7 will be installed
    docker: ---> Package perl-Getopt-Long.noarch 0:2.40-3.el7 will be installed
    docker: --> Processing Dependency: perl(Pod::Usage) >= 1.14 for package: perl-Getopt-Long-2.40-3.el7.noarch
    docker: --> Processing Dependency: perl(Text::ParseWords) for package: perl-Getopt-Long-2.40-3.el7.noarch
    docker: ---> Package perl-PathTools.x86_64 0:3.40-5.el7 will be installed
    docker: ---> Package perl-Pod-Simple.noarch 1:3.28-4.el7 will be installed
    docker: --> Processing Dependency: perl(Pod::Escapes) >= 1.04 for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
    docker: --> Processing Dependency: perl(Encode) for package: 1:perl-Pod-Simple-3.28-4.el7.noarch
    docker: ---> Package perl-Scalar-List-Utils.x86_64 0:1.27-248.el7 will be installed
    docker: ---> Package perl-Socket.x86_64 0:2.010-5.el7 will be installed
    docker: ---> Package perl-Storable.x86_64 0:2.45-3.el7 will be installed
    docker: ---> Package perl-Time-HiRes.x86_64 4:1.9725-3.el7 will be installed
    docker: ---> Package perl-Time-Local.noarch 0:1.2300-2.el7 will be installed
    docker: ---> Package perl-constant.noarch 0:1.27-2.el7 will be installed
    docker: ---> Package perl-macros.x86_64 4:5.16.3-295.el7 will be installed
    docker: ---> Package perl-threads.x86_64 0:1.87-4.el7 will be installed
    docker: ---> Package perl-threads-shared.x86_64 0:1.43-6.el7 will be installed
    docker: ---> Package vim-filesystem.x86_64 2:7.4.629-6.el7 will be installed
    docker: --> Running transaction check
    docker: ---> Package PyYAML.x86_64 0:3.10-11.el7 will be installed
    docker: --> Processing Dependency: libyaml-0.so.2()(64bit) for package: PyYAML-3.10-11.el7.x86_64
    docker: ---> Package device-mapper-event-libs.x86_64 7:1.02.164-7.el7_8.2 will be installed
    docker: ---> Package device-mapper-persistent-data.x86_64 0:0.8.5-2.el7 will be installed
    docker: ---> Package fuse-overlayfs.x86_64 0:0.7.2-6.el7_8 will be installed
    docker: --> Processing Dependency: libfuse3.so.3(FUSE_3.2)(64bit) for package: fuse-overlayfs-0.7.2-6.el7_8.x86_64
    docker: --> Processing Dependency: libfuse3.so.3(FUSE_3.0)(64bit) for package: fuse-overlayfs-0.7.2-6.el7_8.x86_64
    docker: --> Processing Dependency: libfuse3.so.3()(64bit) for package: fuse-overlayfs-0.7.2-6.el7_8.x86_64
    docker: ---> Package libaio.x86_64 0:0.3.109-13.el7 will be installed
    docker: ---> Package lvm2-libs.x86_64 7:2.02.186-7.el7_8.2 will be installed
    docker: --> Processing Dependency: device-mapper-event = 7:1.02.164-7.el7_8.2 for package: 7:lvm2-libs-2.02.186-7.el7_8.2.x86_64
    docker: ---> Package perl-Encode.x86_64 0:2.51-7.el7 will be installed
    docker: ---> Package perl-Pod-Escapes.noarch 1:1.04-295.el7 will be installed
    docker: ---> Package perl-Pod-Usage.noarch 0:1.63-3.el7 will be installed
    docker: --> Processing Dependency: perl(Pod::Text) >= 3.15 for package: perl-Pod-Usage-1.63-3.el7.noarch
    docker: --> Processing Dependency: perl-Pod-Perldoc for package: perl-Pod-Usage-1.63-3.el7.noarch
    docker: ---> Package perl-Text-ParseWords.noarch 0:3.29-4.el7 will be installed
    docker: ---> Package policycoreutils-python.x86_64 0:2.5-34.el7 will be installed
    docker: --> Processing Dependency: policycoreutils = 2.5-34.el7 for package: policycoreutils-python-2.5-34.el7.x86_64
    docker: --> Processing Dependency: setools-libs >= 3.3.8-4 for package: policycoreutils-python-2.5-34.el7.x86_64
    docker: --> Processing Dependency: libsemanage-python >= 2.5-14 for package: policycoreutils-python-2.5-34.el7.x86_64
    docker: --> Processing Dependency: audit-libs-python >= 2.1.3-4 for package: policycoreutils-python-2.5-34.el7.x86_64
    docker: --> Processing Dependency: python-IPy for package: policycoreutils-python-2.5-34.el7.x86_64
    docker: --> Processing Dependency: libqpol.so.1(VERS_1.4)(64bit) for package: policycoreutils-python-2.5-34.el7.x86_64
    docker: --> Processing Dependency: libqpol.so.1(VERS_1.2)(64bit) for package: policycoreutils-python-2.5-34.el7.x86_64
    docker: --> Processing Dependency: libcgroup for package: policycoreutils-python-2.5-34.el7.x86_64
    docker: --> Processing Dependency: libapol.so.4(VERS_4.0)(64bit) for package: policycoreutils-python-2.5-34.el7.x86_64
    docker: --> Processing Dependency: checkpolicy for package: policycoreutils-python-2.5-34.el7.x86_64
    docker: --> Processing Dependency: libqpol.so.1()(64bit) for package: policycoreutils-python-2.5-34.el7.x86_64
    docker: --> Processing Dependency: libapol.so.4()(64bit) for package: policycoreutils-python-2.5-34.el7.x86_64
    docker: ---> Package python-pytoml.noarch 0:0.1.14-1.git7dea353.el7 will be installed
    docker: ---> Package python-setuptools.noarch 0:0.9.8-7.el7 will be installed
    docker: --> Processing Dependency: python-backports-ssl_match_hostname for package: python-setuptools-0.9.8-7.el7.noarch
    docker: ---> Package slirp4netns.x86_64 0:0.4.3-4.el7_8 will be installed
    docker: ---> Package subscription-manager.x86_64 0:1.24.26-4.el7.centos will be installed
    docker: --> Processing Dependency: subscription-manager-rhsm = 1.24.26 for package: subscription-manager-1.24.26-4.el7.centos.x86_64
    docker: --> Processing Dependency: python-dmidecode >= 3.12.2-2 for package: subscription-manager-1.24.26-4.el7.centos.x86_64
    docker: --> Processing Dependency: usermode for package: subscription-manager-1.24.26-4.el7.centos.x86_64
    docker: --> Processing Dependency: python-syspurpose for package: subscription-manager-1.24.26-4.el7.centos.x86_64
    docker: --> Processing Dependency: python-six for package: subscription-manager-1.24.26-4.el7.centos.x86_64
    docker: --> Processing Dependency: python-inotify for package: subscription-manager-1.24.26-4.el7.centos.x86_64
    docker: --> Processing Dependency: python-ethtool for package: subscription-manager-1.24.26-4.el7.centos.x86_64
    docker: --> Processing Dependency: python-dateutil for package: subscription-manager-1.24.26-4.el7.centos.x86_64
    docker: ---> Package yajl.x86_64 0:2.0.4-4.el7 will be installed
    docker: --> Running transaction check
    docker: ---> Package audit-libs-python.x86_64 0:2.8.5-4.el7 will be installed
    docker: --> Processing Dependency: audit-libs(x86-64) = 2.8.5-4.el7 for package: audit-libs-python-2.8.5-4.el7.x86_64
    docker: ---> Package checkpolicy.x86_64 0:2.5-8.el7 will be installed
    docker: ---> Package device-mapper-event.x86_64 7:1.02.164-7.el7_8.2 will be installed
    docker: --> Processing Dependency: device-mapper = 7:1.02.164-7.el7_8.2 for package: 7:device-mapper-event-1.02.164-7.el7_8.2.x86_64
    docker: ---> Package fuse3-libs.x86_64 0:3.6.1-4.el7 will be installed
    docker: ---> Package libcgroup.x86_64 0:0.41-21.el7 will be installed
    docker: ---> Package libsemanage-python.x86_64 0:2.5-14.el7 will be installed
    docker: ---> Package libyaml.x86_64 0:0.1.4-11.el7_0 will be installed
    docker: ---> Package perl-Pod-Perldoc.noarch 0:3.20-4.el7 will be installed
    docker: --> Processing Dependency: perl(parent) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
    docker: --> Processing Dependency: perl(HTTP::Tiny) for package: perl-Pod-Perldoc-3.20-4.el7.noarch
    docker: ---> Package perl-podlators.noarch 0:2.5.1-3.el7 will be installed
    docker: ---> Package policycoreutils.x86_64 0:2.5-29.el7_6.1 will be updated
    docker: ---> Package policycoreutils.x86_64 0:2.5-34.el7 will be an update
    docker: ---> Package python-IPy.noarch 0:0.75-6.el7 will be installed
    docker: ---> Package python-backports-ssl_match_hostname.noarch 0:3.5.0.1-1.el7 will be installed
    docker: --> Processing Dependency: python-ipaddress for package: python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch
    docker: --> Processing Dependency: python-backports for package: python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch
    docker: ---> Package python-dateutil.noarch 0:1.5-7.el7 will be installed
    docker: ---> Package python-dmidecode.x86_64 0:3.12.2-4.el7 will be installed
    docker: ---> Package python-ethtool.x86_64 0:0.8-8.el7 will be installed
    docker: --> Processing Dependency: libnl.so.1()(64bit) for package: python-ethtool-0.8-8.el7.x86_64
    docker: ---> Package python-inotify.noarch 0:0.9.4-4.el7 will be installed
    docker: ---> Package python-six.noarch 0:1.9.0-2.el7 will be installed
    docker: ---> Package python-syspurpose.x86_64 0:1.24.26-4.el7.centos will be installed
    docker: ---> Package setools-libs.x86_64 0:3.3.8-4.el7 will be installed
    docker: ---> Package subscription-manager-rhsm.x86_64 0:1.24.26-4.el7.centos will be installed
    docker: ---> Package usermode.x86_64 0:1.111-6.el7 will be installed
    docker: --> Running transaction check
    docker: ---> Package audit-libs.x86_64 0:2.8.4-4.el7 will be updated
    docker: --> Processing Dependency: audit-libs(x86-64) = 2.8.4-4.el7 for package: audit-2.8.4-4.el7.x86_64
    docker: ---> Package audit-libs.x86_64 0:2.8.5-4.el7 will be an update
    docker: ---> Package device-mapper.x86_64 7:1.02.149-10.el7_6.7 will be updated
    docker: --> Processing Dependency: device-mapper = 7:1.02.149-10.el7_6.7 for package: 7:device-mapper-libs-1.02.149-10.el7_6.7.x86_64
    docker: ---> Package device-mapper.x86_64 7:1.02.164-7.el7_8.2 will be an update
    docker: ---> Package libnl.x86_64 0:1.1.4-3.el7 will be installed
    docker: ---> Package perl-HTTP-Tiny.noarch 0:0.033-3.el7 will be installed
    docker: ---> Package perl-parent.noarch 1:0.225-244.el7 will be installed
    docker: ---> Package python-backports.x86_64 0:1.0-8.el7 will be installed
    docker: ---> Package python-ipaddress.noarch 0:1.0.16-2.el7 will be installed
    docker: --> Running transaction check
    docker: ---> Package audit.x86_64 0:2.8.4-4.el7 will be updated
    docker: ---> Package audit.x86_64 0:2.8.5-4.el7 will be an update
    docker: ---> Package device-mapper-libs.x86_64 7:1.02.149-10.el7_6.7 will be updated
    docker: ---> Package device-mapper-libs.x86_64 7:1.02.164-7.el7_8.2 will be an update
    docker: --> Finished Dependency Resolution
    docker:
    docker: Dependencies Resolved
    docker:
    docker: ================================================================================
    docker:  Package                Arch   Version                            Repository
    docker:                                                                            Size
    docker: ================================================================================
    docker: Installing:
    docker:  docker                 x86_64 2:1.13.1-162.git64e9980.el7.centos extras   18 M
    docker:  net-tools              x86_64 2.0-0.25.20131004git.el7           base    306 k
    docker:  vim-enhanced           x86_64 2:7.4.629-6.el7                    base    1.1 M
    docker: Updating:
    docker:  curl                   x86_64 7.29.0-57.el7_8.1                  updates 271 k
    docker: Installing for dependencies:
    docker:  PyYAML                 x86_64 3.10-11.el7                        base    153 k
    docker:  atomic-registries      x86_64 1:1.22.1-33.gitb507039.el7_8       extras   36 k
    docker:  audit-libs-python      x86_64 2.8.5-4.el7                        base     76 k
    docker:  checkpolicy            x86_64 2.5-8.el7                          base    295 k
    docker:  container-selinux      noarch 2:2.119.2-1.911c772.el7_8          extras   40 k
    docker:  container-storage-setup
    docker:                         noarch 0.11.0-2.git5eaf76c.el7            extras   35 k
    docker:  containers-common      x86_64 1:0.1.40-11.el7_8                  extras   43 k
    docker:  device-mapper-event    x86_64 7:1.02.164-7.el7_8.2               updates 191 k
    docker:  device-mapper-event-libs
    docker:                         x86_64 7:1.02.164-7.el7_8.2               updates 190 k
    docker:  device-mapper-persistent-data
    docker:                         x86_64 0.8.5-2.el7                        base    422 k
    docker:  docker-client          x86_64 2:1.13.1-162.git64e9980.el7.centos extras  3.9 M
    docker:  docker-common          x86_64 2:1.13.1-162.git64e9980.el7.centos extras   99 k
    docker:  fuse-overlayfs         x86_64 0.7.2-6.el7_8                      extras   54 k
    docker:  fuse3-libs             x86_64 3.6.1-4.el7                        extras   82 k
    docker:  gpm-libs               x86_64 1.20.7-6.el7                       base     32 k
    docker:  libaio                 x86_64 0.3.109-13.el7                     base     24 k
    docker:  libcgroup              x86_64 0.41-21.el7                        base     66 k
    docker:  libnl                  x86_64 1.1.4-3.el7                        base    128 k
    docker:  libsemanage-python     x86_64 2.5-14.el7                         base    113 k
    docker:  libyaml                x86_64 0.1.4-11.el7_0                     base     55 k
    docker:  lvm2                   x86_64 7:2.02.186-7.el7_8.2               updates 1.3 M
    docker:  lvm2-libs              x86_64 7:2.02.186-7.el7_8.2               updates 1.1 M
    docker:  oci-register-machine   x86_64 1:0-6.git2b44233.el7               extras  1.1 M
    docker:  oci-systemd-hook       x86_64 1:0.2.0-1.git05e6923.el7_6         extras   34 k
    docker:  oci-umount             x86_64 2:2.5-3.el7                        extras   33 k
    docker:  perl                   x86_64 4:5.16.3-295.el7                   base    8.0 M
    docker:  perl-Carp              noarch 1.26-244.el7                       base     19 k
    docker:  perl-Encode            x86_64 2.51-7.el7                         base    1.5 M
    docker:  perl-Exporter          noarch 5.68-3.el7                         base     28 k
    docker:  perl-File-Path         noarch 2.09-2.el7                         base     26 k
    docker:  perl-File-Temp         noarch 0.23.01-3.el7                      base     56 k
    docker:  perl-Filter            x86_64 1.49-3.el7                         base     76 k
    docker:  perl-Getopt-Long       noarch 2.40-3.el7                         base     56 k
    docker:  perl-HTTP-Tiny         noarch 0.033-3.el7                        base     38 k
    docker:  perl-PathTools         x86_64 3.40-5.el7                         base     82 k
    docker:  perl-Pod-Escapes       noarch 1:1.04-295.el7                     base     51 k
    docker:  perl-Pod-Perldoc       noarch 3.20-4.el7                         base     87 k
    docker:  perl-Pod-Simple        noarch 1:3.28-4.el7                       base    216 k
    docker:  perl-Pod-Usage         noarch 1.63-3.el7                         base     27 k
    docker:  perl-Scalar-List-Utils x86_64 1.27-248.el7                       base     36 k
    docker:  perl-Socket            x86_64 2.010-5.el7                        base     49 k
    docker:  perl-Storable          x86_64 2.45-3.el7                         base     77 k
    docker:  perl-Text-ParseWords   noarch 3.29-4.el7                         base     14 k
    docker:  perl-Time-HiRes        x86_64 4:1.9725-3.el7                     base     45 k
    docker:  perl-Time-Local        noarch 1.2300-2.el7                       base     24 k
    docker:  perl-constant          noarch 1.27-2.el7                         base     19 k
    docker:  perl-libs              x86_64 4:5.16.3-295.el7                   base    689 k
    docker:  perl-macros            x86_64 4:5.16.3-295.el7                   base     44 k
    docker:  perl-parent            noarch 1:0.225-244.el7                    base     12 k
    docker:  perl-podlators         noarch 2.5.1-3.el7                        base    112 k
    docker:  perl-threads           x86_64 1.87-4.el7                         base     49 k
    docker:  perl-threads-shared    x86_64 1.43-6.el7                         base     39 k
    docker:  policycoreutils-python x86_64 2.5-34.el7                         base    457 k
    docker:  python-IPy             noarch 0.75-6.el7                         base     32 k
    docker:  python-backports       x86_64 1.0-8.el7                          base    5.8 k
    docker:  python-backports-ssl_match_hostname
    docker:                         noarch 3.5.0.1-1.el7                      base     13 k
    docker:  python-dateutil        noarch 1.5-7.el7                          base     85 k
    docker:  python-dmidecode       x86_64 3.12.2-4.el7                       base     83 k
    docker:  python-ethtool         x86_64 0.8-8.el7                          base     34 k
    docker:  python-inotify         noarch 0.9.4-4.el7                        base     49 k
    docker:  python-ipaddress       noarch 1.0.16-2.el7                       base     34 k
    docker:  python-pytoml          noarch 0.1.14-1.git7dea353.el7            extras   18 k
    docker:  python-setuptools      noarch 0.9.8-7.el7                        base    397 k
    docker:  python-six             noarch 1.9.0-2.el7                        base     29 k
    docker:  python-syspurpose      x86_64 1.24.26-4.el7.centos               updates 269 k
    docker:  setools-libs           x86_64 3.3.8-4.el7                        base    620 k
    docker:  slirp4netns            x86_64 0.4.3-4.el7_8                      extras   81 k
    docker:  subscription-manager   x86_64 1.24.26-4.el7.centos               updates 1.1 M
    docker:  subscription-manager-rhsm
    docker:                         x86_64 1.24.26-4.el7.centos               updates 328 k
    docker:  subscription-manager-rhsm-certificates
    docker:                         x86_64 1.24.26-4.el7.centos               updates 232 k
    docker:  usermode               x86_64 1.111-6.el7                        base    193 k
    docker:  vim-common             x86_64 2:7.4.629-6.el7                    base    5.9 M
    docker:  vim-filesystem         x86_64 2:7.4.629-6.el7                    base     11 k
    docker:  yajl                   x86_64 2.0.4-4.el7                        base     39 k
    docker: Updating for dependencies:
    docker:  audit                  x86_64 2.8.5-4.el7                        base    256 k
    docker:  audit-libs             x86_64 2.8.5-4.el7                        base    102 k
    docker:  device-mapper          x86_64 7:1.02.164-7.el7_8.2               updates 295 k
    docker:  device-mapper-libs     x86_64 7:1.02.164-7.el7_8.2               updates 324 k
    docker:  libcurl                x86_64 7.29.0-57.el7_8.1                  updates 223 k
    docker:  libssh2                x86_64 1.8.0-3.el7                        base     88 k
    docker:  policycoreutils        x86_64 2.5-34.el7                         base    917 k
    docker:
    docker: Transaction Summary
    docker: ================================================================================
    docker: Install  3 Packages (+74 Dependent packages)
    docker: Upgrade  1 Package  (+ 7 Dependent packages)
    docker: Total download size: 52 M
    docker: Downloading packages:
    docker: No Presto metadata available for base
    docker: No Presto metadata available for updates
    docker: Public key for atomic-registries-1.22.1-33.gitb507039.el7_8.x86_64.rpm is not installed
    docker: warning: /var/cache/yum/x86_64/7/extras/packages/atomic-registries-1.22.1-33.gitb507039.el7_8.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
    docker: Public key for audit-libs-2.8.5-4.el7.x86_64.rpm is not installed
    docker: Public key for device-mapper-event-1.02.164-7.el7_8.2.x86_64.rpm is not installed
    docker: --------------------------------------------------------------------------------
    docker: Total                                              5.9 MB/s |  52 MB  00:08
    docker: Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    docker: Importing GPG key 0xF4A80EB5:
    docker:  Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
    docker:  Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
    docker:  Package    : centos-release-7-6.1810.2.el7.centos.x86_64 (@anaconda)
    docker:  From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
    docker: Running transaction check
    docker: Running transaction test
    docker: Transaction test succeeded
    docker: Running transaction
    docker:   Updating   : 7:device-mapper-1.02.164-7.el7_8.2.x86_64                   1/93
    docker:
    docker:   Updating   : 7:device-mapper-libs-1.02.164-7.el7_8.2.x86_64              2/93
    docker:
    docker:   Updating   : audit-libs-2.8.5-4.el7.x86_64                               3/93
    docker:
    docker:   Installing : 7:device-mapper-event-libs-1.02.164-7.el7_8.2.x86_64        4/93
    docker:
    docker:   Installing : libaio-0.3.109-13.el7.x86_64                                5/93
    docker:
    docker:   Updating   : policycoreutils-2.5-34.el7.x86_64                           6/93
    docker:
    docker:   Installing : python-six-1.9.0-2.el7.noarch                               7/93
    docker:
    docker:   Installing : python-dateutil-1.5-7.el7.noarch                            8/93
    docker:
    docker:   Installing : subscription-manager-rhsm-certificates-1.24.26-4.el7.cen    9/93
    docker:
    docker:   Installing : yajl-2.0.4-4.el7.x86_64                                    10/93
    docker:
    docker:   Installing : 2:oci-umount-2.5-3.el7.x86_64                              11/93
    docker:
    docker:   Installing : 1:oci-systemd-hook-0.2.0-1.git05e6923.el7_6.x86_64         12/93
    docker:
    docker:   Installing : subscription-manager-rhsm-1.24.26-4.el7.centos.x86_64      13/93
    docker:
    docker:   Installing : device-mapper-persistent-data-0.8.5-2.el7.x86_64           14/93
    docker:
    docker:   Installing : 7:device-mapper-event-1.02.164-7.el7_8.2.x86_64            15/93
    docker:
    docker:   Installing : 7:lvm2-libs-2.02.186-7.el7_8.2.x86_64                      16/93
    docker:
    docker:   Installing : 7:lvm2-2.02.186-7.el7_8.2.x86_64                           17/93
    docker:
    docker:   Installing : container-storage-setup-0.11.0-2.git5eaf76c.el7.noarch     18/93
    docker:
    docker:   Installing : audit-libs-python-2.8.5-4.el7.x86_64                       19/93
    docker:
    docker:   Installing : 1:perl-parent-0.225-244.el7.noarch                         20/93
    docker:
    docker:   Installing : perl-HTTP-Tiny-0.033-3.el7.noarch                          21/93
    docker:
    docker:   Installing : perl-podlators-2.5.1-3.el7.noarch                          22/93
    docker:
    docker:   Installing : perl-Pod-Perldoc-3.20-4.el7.noarch                         23/93
    docker:
    docker:   Installing : 1:perl-Pod-Escapes-1.04-295.el7.noarch                     24/93
    docker:
    docker:   Installing : perl-Text-ParseWords-3.29-4.el7.noarch                     25/93
    docker:
    docker:   Installing : perl-Encode-2.51-7.el7.x86_64                              26/93
    docker:
    docker:   Installing : perl-Pod-Usage-1.63-3.el7.noarch                           27/93
    docker:
    docker:   Installing : 4:perl-libs-5.16.3-295.el7.x86_64                          28/93
    docker:
    docker:   Installing : 4:perl-macros-5.16.3-295.el7.x86_64                        29/93
    docker:
    docker:   Installing : 4:perl-Time-HiRes-1.9725-3.el7.x86_64                      30/93
    docker:
    docker:   Installing : perl-threads-1.87-4.el7.x86_64                             31/93
    docker:
    docker:   Installing : perl-Storable-2.45-3.el7.x86_64                            32/93
    docker:
    docker:   Installing : perl-Carp-1.26-244.el7.noarch                              33/93
    docker:
    docker:   Installing : perl-Filter-1.49-3.el7.x86_64                              34/93
    docker:
    docker:   Installing : perl-Exporter-5.68-3.el7.noarch                            35/93
    docker:
    docker:   Installing : perl-constant-1.27-2.el7.noarch                            36/93
    docker:
    docker:   Installing : perl-Socket-2.010-5.el7.x86_64                             37/93
    docker:
    docker:   Installing : perl-Time-Local-1.2300-2.el7.noarch                        38/93
    docker:
    docker:   Installing : perl-threads-shared-1.43-6.el7.x86_64                      39/93
    docker:
    docker:   Installing : perl-File-Temp-0.23.01-3.el7.noarch                        40/93
    docker:
    docker:   Installing : perl-File-Path-2.09-2.el7.noarch                           41/93
    docker:
    docker:   Installing : perl-PathTools-3.40-5.el7.x86_64                           42/93
    docker:
    docker:   Installing : perl-Scalar-List-Utils-1.27-248.el7.x86_64                 43/93
    docker:
    docker:   Installing : 1:perl-Pod-Simple-3.28-4.el7.noarch                        44/93
    docker:
    docker:   Installing : perl-Getopt-Long-2.40-3.el7.noarch                         45/93
    docker:
    docker:   Installing : 4:perl-5.16.3-295.el7.x86_64                               46/93
    docker:
    docker:   Installing : libcgroup-0.41-21.el7.x86_64                               47/93
    docker:
    docker:   Installing : python-syspurpose-1.24.26-4.el7.centos.x86_64              48/93
    docker:
    docker:   Installing : python-backports-1.0-8.el7.x86_64                          49/93
    docker:
    docker:   Installing : 1:oci-register-machine-0-6.git2b44233.el7.x86_64           50/93
    docker:
    docker:   Installing : fuse3-libs-3.6.1-4.el7.x86_64                              51/93
    docker:
    docker:   Installing : fuse-overlayfs-0.7.2-6.el7_8.x86_64                        52/93
    docker:
    docker:   Installing : python-inotify-0.9.4-4.el7.noarch                          53/93
    docker:
    docker:   Installing : 2:vim-filesystem-7.4.629-6.el7.x86_64                      54/93
    docker:
    docker:   Installing : 2:vim-common-7.4.629-6.el7.x86_64                          55/93
    docker:
    docker:   Installing : python-dmidecode-3.12.2-4.el7.x86_64                       56/93
    docker:
    docker:   Installing : libsemanage-python-2.5-14.el7.x86_64                       57/93
    docker:
    docker:   Installing : usermode-1.111-6.el7.x86_64                                58/93
    docker:
    docker:   Installing : libyaml-0.1.4-11.el7_0.x86_64                              59/93
    docker:
    docker:   Installing : PyYAML-3.10-11.el7.x86_64                                  60/93
    docker:
    docker:   Installing : setools-libs-3.3.8-4.el7.x86_64                            61/93
    docker:
    docker:   Installing : python-IPy-0.75-6.el7.noarch                               62/93
    docker:
    docker:   Installing : checkpolicy-2.5-8.el7.x86_64                               63/93
    docker:
    docker:   Installing : policycoreutils-python-2.5-34.el7.x86_64                   64/93
    docker:
    docker:   Installing : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch         65/93
    docker:
    docker:   Installing : gpm-libs-1.20.7-6.el7.x86_64                               66/93
    docker:
    docker:   Installing : python-ipaddress-1.0.16-2.el7.noarch                       67/93
    docker:
    docker:   Installing : python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch   68/93
    docker:
    docker:   Installing : python-setuptools-0.9.8-7.el7.noarch                       69/93
    docker:
    docker:   Installing : libnl-1.1.4-3.el7.x86_64                                   70/93
    docker:
    docker:   Installing : python-ethtool-0.8-8.el7.x86_64                            71/93
    docker:
    docker:   Installing : subscription-manager-1.24.26-4.el7.centos.x86_64           72/93
    docker:
    docker:   Installing : slirp4netns-0.4.3-4.el7_8.x86_64                           73/93
    docker:
    docker:   Installing : 1:containers-common-0.1.40-11.el7_8.x86_64                 74/93
    docker:
    docker:   Installing : python-pytoml-0.1.14-1.git7dea353.el7.noarch               75/93
    docker:
    docker:   Installing : 1:atomic-registries-1.22.1-33.gitb507039.el7_8.x86_64      76/93
    docker:
    docker:   Installing : 2:docker-common-1.13.1-162.git64e9980.el7.centos.x86_64    77/93
    docker:
    docker:   Installing : 2:docker-client-1.13.1-162.git64e9980.el7.centos.x86_64    78/93
    docker:
    docker:   Updating   : libssh2-1.8.0-3.el7.x86_64                                 79/93
    docker:
    docker:   Updating   : libcurl-7.29.0-57.el7_8.1.x86_64                           80/93
    docker:
    docker:   Updating   : curl-7.29.0-57.el7_8.1.x86_64                              81/93
    docker:
    docker:   Installing : 2:docker-1.13.1-162.git64e9980.el7.centos.x86_64           82/93
    docker:
    docker:   Installing : 2:vim-enhanced-7.4.629-6.el7.x86_64                        83/93
    docker:
    docker:   Updating   : audit-2.8.5-4.el7.x86_64                                   84/93
    docker:
    docker:   Installing : net-tools-2.0-0.25.20131004git.el7.x86_64                  85/93
    docker:
    docker:   Cleanup    : 7:device-mapper-1.02.149-10.el7_6.7.x86_64                 86/93
    docker:
    docker:   Cleanup    : 7:device-mapper-libs-1.02.149-10.el7_6.7.x86_64            87/93
    docker:
    docker:   Cleanup    : audit-2.8.4-4.el7.x86_64                                   88/93
    docker:
    docker:   Cleanup    : policycoreutils-2.5-29.el7_6.1.x86_64                      89/93
    docker:
    docker:   Cleanup    : curl-7.29.0-51.el7.x86_64                                  90/93
    docker:
    docker:   Cleanup    : libcurl-7.29.0-51.el7.x86_64                               91/93
    docker:
    docker:   Cleanup    : libssh2-1.4.3-12.el7_6.2.x86_64                            92/93
    docker:
    docker:   Cleanup    : audit-libs-2.8.4-4.el7.x86_64                              93/93
    docker:
    docker:   Verifying  : perl-HTTP-Tiny-0.033-3.el7.noarch                           1/93
    docker:
    docker:   Verifying  : libssh2-1.8.0-3.el7.x86_64                                  2/93
    docker:
    docker:   Verifying  : 4:perl-libs-5.16.3-295.el7.x86_64                           3/93
    docker:
    docker:   Verifying  : perl-File-Temp-0.23.01-3.el7.noarch                         4/93
    docker:
    docker:   Verifying  : 2:oci-umount-2.5-3.el7.x86_64                               5/93
    docker:
    docker:   Verifying  : python-pytoml-0.1.14-1.git7dea353.el7.noarch                6/93
    docker:
    docker:   Verifying  : 4:perl-macros-5.16.3-295.el7.x86_64                         7/93
    docker:
    docker:   Verifying  : 7:lvm2-libs-2.02.186-7.el7_8.2.x86_64                       8/93
    docker:
    docker:   Verifying  : slirp4netns-0.4.3-4.el7_8.x86_64                            9/93
    docker:
    docker:   Verifying  : curl-7.29.0-57.el7_8.1.x86_64                              10/93
    docker:
    docker:   Verifying  : yajl-2.0.4-4.el7.x86_64                                    11/93
    docker:
    docker:   Verifying  : libnl-1.1.4-3.el7.x86_64                                   12/93
    docker:
    docker:   Verifying  : perl-File-Path-2.09-2.el7.noarch                           13/93
    docker:
    docker:   Verifying  : python-ipaddress-1.0.16-2.el7.noarch                       14/93
    docker:
    docker:   Verifying  : gpm-libs-1.20.7-6.el7.x86_64                               15/93
    docker:
    docker:   Verifying  : perl-Text-ParseWords-3.29-4.el7.noarch                     16/93
    docker:
    docker:   Verifying  : PyYAML-3.10-11.el7.x86_64                                  17/93
    docker:
    docker:   Verifying  : subscription-manager-rhsm-certificates-1.24.26-4.el7.cen   18/93
    docker:
    docker:   Verifying  : fuse-overlayfs-0.7.2-6.el7_8.x86_64                        19/93
    docker:
    docker:   Verifying  : 4:perl-Time-HiRes-1.9725-3.el7.x86_64                      20/93
    docker:
    docker:   Verifying  : 1:perl-Pod-Escapes-1.04-295.el7.noarch                     21/93
    docker:
    docker:   Verifying  : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch         22/93
    docker:
    docker:   Verifying  : 1:containers-common-0.1.40-11.el7_8.x86_64                 23/93
    docker:
    docker:   Verifying  : python-setuptools-0.9.8-7.el7.noarch                       24/93
    docker:
    docker:   Verifying  : checkpolicy-2.5-8.el7.x86_64                               25/93
    docker:
    docker:   Verifying  : python-IPy-0.75-6.el7.noarch                               26/93
    docker:
    docker:   Verifying  : 4:perl-5.16.3-295.el7.x86_64                               27/93
    docker:
    docker:   Verifying  : setools-libs-3.3.8-4.el7.x86_64                            28/93
    docker:
    docker:   Verifying  : 1:oci-systemd-hook-0.2.0-1.git05e6923.el7_6.x86_64         29/93
    docker:
    docker:   Verifying  : audit-libs-2.8.5-4.el7.x86_64                              30/93
    docker:
    docker:   Verifying  : 7:device-mapper-libs-1.02.164-7.el7_8.2.x86_64             31/93
    docker:
    docker:   Verifying  : audit-libs-python-2.8.5-4.el7.x86_64                       32/93
    docker:
    docker:   Verifying  : perl-Pod-Usage-1.63-3.el7.noarch                           33/93
    docker:
    docker:   Verifying  : 2:vim-common-7.4.629-6.el7.x86_64                          34/93
    docker:
    docker:   Verifying  : perl-Encode-2.51-7.el7.x86_64                              35/93
    docker:
    docker:   Verifying  : perl-threads-1.87-4.el7.x86_64                             36/93
    docker:
    docker:   Verifying  : libyaml-0.1.4-11.el7_0.x86_64                              37/93
    docker:
    docker:   Verifying  : perl-Getopt-Long-2.40-3.el7.noarch                         38/93
    docker:
    docker:   Verifying  : usermode-1.111-6.el7.x86_64                                39/93
    docker:
    docker:   Verifying  : python-backports-ssl_match_hostname-3.5.0.1-1.el7.noarch   40/93
    docker:
    docker:   Verifying  : python-ethtool-0.8-8.el7.x86_64                            41/93
    docker:
    docker:   Verifying  : subscription-manager-rhsm-1.24.26-4.el7.centos.x86_64      42/93
    docker:
    docker:   Verifying  : perl-threads-shared-1.43-6.el7.x86_64                      43/93
    docker:
    docker:   Verifying  : perl-Storable-2.45-3.el7.x86_64                            44/93
    docker:
    docker:   Verifying  : container-storage-setup-0.11.0-2.git5eaf76c.el7.noarch     45/93
    docker:
    docker:   Verifying  : 7:lvm2-2.02.186-7.el7_8.2.x86_64                           46/93
    docker:
    docker:   Verifying  : 7:device-mapper-1.02.164-7.el7_8.2.x86_64                  47/93
    docker:
    docker:   Verifying  : 1:perl-parent-0.225-244.el7.noarch                         48/93
    docker:
    docker:   Verifying  : device-mapper-persistent-data-0.8.5-2.el7.x86_64           49/93
    docker:
    docker:   Verifying  : python-dateutil-1.5-7.el7.noarch                           50/93
    docker:
    docker:   Verifying  : policycoreutils-2.5-34.el7.x86_64                          51/93
    docker:
    docker:   Verifying  : policycoreutils-python-2.5-34.el7.x86_64                   52/93
    docker:
    docker:   Verifying  : libaio-0.3.109-13.el7.x86_64                               53/93
    docker:
    docker:   Verifying  : perl-Carp-1.26-244.el7.noarch                              54/93
    docker:
    docker:   Verifying  : 7:device-mapper-event-1.02.164-7.el7_8.2.x86_64            55/93
    docker:
    docker:   Verifying  : libsemanage-python-2.5-14.el7.x86_64                       56/93
    docker:
    docker:   Verifying  : python-dmidecode-3.12.2-4.el7.x86_64                       57/93
    docker:
    docker:   Verifying  : 2:vim-filesystem-7.4.629-6.el7.x86_64                      58/93
    docker:
    docker:   Verifying  : 2:docker-client-1.13.1-162.git64e9980.el7.centos.x86_64    59/93
    docker:
    docker:   Verifying  : perl-podlators-2.5.1-3.el7.noarch                          60/93
    docker:
    docker:   Verifying  : 2:docker-1.13.1-162.git64e9980.el7.centos.x86_64           61/93
    docker:
    docker:   Verifying  : 2:docker-common-1.13.1-162.git64e9980.el7.centos.x86_64    62/93
    docker:
    docker:   Verifying  : perl-Filter-1.49-3.el7.x86_64                              63/93
    docker:
    docker:   Verifying  : libcurl-7.29.0-57.el7_8.1.x86_64                           64/93
    docker:
    docker:   Verifying  : 7:device-mapper-event-libs-1.02.164-7.el7_8.2.x86_64       65/93
    docker:
    docker:   Verifying  : 1:atomic-registries-1.22.1-33.gitb507039.el7_8.x86_64      66/93
    docker:
    docker:   Verifying  : perl-Exporter-5.68-3.el7.noarch                            67/93
    docker:
    docker:   Verifying  : perl-constant-1.27-2.el7.noarch                            68/93
    docker:
    docker:   Verifying  : perl-PathTools-3.40-5.el7.x86_64                           69/93
    docker:
    docker:   Verifying  : python-inotify-0.9.4-4.el7.noarch                          70/93
    docker:
    docker:   Verifying  : fuse3-libs-3.6.1-4.el7.x86_64                              71/93
    docker:
    docker:   Verifying  : perl-Socket-2.010-5.el7.x86_64                             72/93
    docker:
    docker:   Verifying  : subscription-manager-1.24.26-4.el7.centos.x86_64           73/93
    docker:
    docker:   Verifying  : net-tools-2.0-0.25.20131004git.el7.x86_64                  74/93
    docker:
    docker:   Verifying  : 1:perl-Pod-Simple-3.28-4.el7.noarch                        75/93
    docker:
    docker:   Verifying  : perl-Time-Local-1.2300-2.el7.noarch                        76/93
    docker:
    docker:   Verifying  : python-six-1.9.0-2.el7.noarch                              77/93
    docker:
    docker:   Verifying  : 2:vim-enhanced-7.4.629-6.el7.x86_64                        78/93
    docker:
    docker:   Verifying  : audit-2.8.5-4.el7.x86_64                                   79/93
    docker:
    docker:   Verifying  : 1:oci-register-machine-0-6.git2b44233.el7.x86_64           80/93
    docker:
    docker:   Verifying  : perl-Scalar-List-Utils-1.27-248.el7.x86_64                 81/93
    docker:
    docker:   Verifying  : python-backports-1.0-8.el7.x86_64                          82/93
    docker:
    docker:   Verifying  : perl-Pod-Perldoc-3.20-4.el7.noarch                         83/93
    docker:
    docker:   Verifying  : python-syspurpose-1.24.26-4.el7.centos.x86_64              84/93
    docker:
    docker:   Verifying  : libcgroup-0.41-21.el7.x86_64                               85/93
    docker:
    docker:   Verifying  : 7:device-mapper-1.02.149-10.el7_6.7.x86_64                 86/93
    docker:
    docker:   Verifying  : libssh2-1.4.3-12.el7_6.2.x86_64                            87/93
    docker:
    docker:   Verifying  : policycoreutils-2.5-29.el7_6.1.x86_64                      88/93
    docker:
    docker:   Verifying  : libcurl-7.29.0-51.el7.x86_64                               89/93
    docker:
    docker:   Verifying  : audit-libs-2.8.4-4.el7.x86_64                              90/93
    docker:
    docker:   Verifying  : audit-2.8.4-4.el7.x86_64                                   91/93
    docker:
    docker:   Verifying  : 7:device-mapper-libs-1.02.149-10.el7_6.7.x86_64            92/93
    docker:
    docker:   Verifying  : curl-7.29.0-51.el7.x86_64                                  93/93
    docker:
    docker:
    docker: Installed:
    docker:   docker.x86_64 2:1.13.1-162.git64e9980.el7.centos
    docker:   net-tools.x86_64 0:2.0-0.25.20131004git.el7
    docker:   vim-enhanced.x86_64 2:7.4.629-6.el7
    docker:
    docker: Dependency Installed:
    docker:   PyYAML.x86_64 0:3.10-11.el7
    docker:   atomic-registries.x86_64 1:1.22.1-33.gitb507039.el7_8
    docker:   audit-libs-python.x86_64 0:2.8.5-4.el7
    docker:   checkpolicy.x86_64 0:2.5-8.el7
    docker:   container-selinux.noarch 2:2.119.2-1.911c772.el7_8
    docker:   container-storage-setup.noarch 0:0.11.0-2.git5eaf76c.el7
    docker:   containers-common.x86_64 1:0.1.40-11.el7_8
    docker:   device-mapper-event.x86_64 7:1.02.164-7.el7_8.2
    docker:   device-mapper-event-libs.x86_64 7:1.02.164-7.el7_8.2
    docker:   device-mapper-persistent-data.x86_64 0:0.8.5-2.el7
    docker:   docker-client.x86_64 2:1.13.1-162.git64e9980.el7.centos
    docker:   docker-common.x86_64 2:1.13.1-162.git64e9980.el7.centos
    docker:   fuse-overlayfs.x86_64 0:0.7.2-6.el7_8
    docker:   fuse3-libs.x86_64 0:3.6.1-4.el7
    docker:   gpm-libs.x86_64 0:1.20.7-6.el7
    docker:   libaio.x86_64 0:0.3.109-13.el7
    docker:   libcgroup.x86_64 0:0.41-21.el7
    docker:   libnl.x86_64 0:1.1.4-3.el7
    docker:   libsemanage-python.x86_64 0:2.5-14.el7
    docker:   libyaml.x86_64 0:0.1.4-11.el7_0
    docker:   lvm2.x86_64 7:2.02.186-7.el7_8.2
    docker:   lvm2-libs.x86_64 7:2.02.186-7.el7_8.2
    docker:   oci-register-machine.x86_64 1:0-6.git2b44233.el7
    docker:   oci-systemd-hook.x86_64 1:0.2.0-1.git05e6923.el7_6
    docker:   oci-umount.x86_64 2:2.5-3.el7
    docker:   perl.x86_64 4:5.16.3-295.el7
    docker:   perl-Carp.noarch 0:1.26-244.el7
    docker:   perl-Encode.x86_64 0:2.51-7.el7
    docker:   perl-Exporter.noarch 0:5.68-3.el7
    docker:   perl-File-Path.noarch 0:2.09-2.el7
    docker:   perl-File-Temp.noarch 0:0.23.01-3.el7
    docker:   perl-Filter.x86_64 0:1.49-3.el7
    docker:   perl-Getopt-Long.noarch 0:2.40-3.el7
    docker:   perl-HTTP-Tiny.noarch 0:0.033-3.el7
    docker:   perl-PathTools.x86_64 0:3.40-5.el7
    docker:   perl-Pod-Escapes.noarch 1:1.04-295.el7
    docker:   perl-Pod-Perldoc.noarch 0:3.20-4.el7
    docker:   perl-Pod-Simple.noarch 1:3.28-4.el7
    docker:   perl-Pod-Usage.noarch 0:1.63-3.el7
    docker:   perl-Scalar-List-Utils.x86_64 0:1.27-248.el7
    docker:   perl-Socket.x86_64 0:2.010-5.el7
    docker:   perl-Storable.x86_64 0:2.45-3.el7
    docker:   perl-Text-ParseWords.noarch 0:3.29-4.el7
    docker:   perl-Time-HiRes.x86_64 4:1.9725-3.el7
    docker:   perl-Time-Local.noarch 0:1.2300-2.el7
    docker:   perl-constant.noarch 0:1.27-2.el7
    docker:   perl-libs.x86_64 4:5.16.3-295.el7
    docker:   perl-macros.x86_64 4:5.16.3-295.el7
    docker:   perl-parent.noarch 1:0.225-244.el7
    docker:   perl-podlators.noarch 0:2.5.1-3.el7
    docker:   perl-threads.x86_64 0:1.87-4.el7
    docker:   perl-threads-shared.x86_64 0:1.43-6.el7
    docker:   policycoreutils-python.x86_64 0:2.5-34.el7
    docker:   python-IPy.noarch 0:0.75-6.el7
    docker:   python-backports.x86_64 0:1.0-8.el7
    docker:   python-backports-ssl_match_hostname.noarch 0:3.5.0.1-1.el7
    docker:   python-dateutil.noarch 0:1.5-7.el7
    docker:   python-dmidecode.x86_64 0:3.12.2-4.el7
    docker:   python-ethtool.x86_64 0:0.8-8.el7
    docker:   python-inotify.noarch 0:0.9.4-4.el7
    docker:   python-ipaddress.noarch 0:1.0.16-2.el7
    docker:   python-pytoml.noarch 0:0.1.14-1.git7dea353.el7
    docker:   python-setuptools.noarch 0:0.9.8-7.el7
    docker:   python-six.noarch 0:1.9.0-2.el7
    docker:   python-syspurpose.x86_64 0:1.24.26-4.el7.centos
    docker:   setools-libs.x86_64 0:3.3.8-4.el7
    docker:   slirp4netns.x86_64 0:0.4.3-4.el7_8
    docker:   subscription-manager.x86_64 0:1.24.26-4.el7.centos
    docker:   subscription-manager-rhsm.x86_64 0:1.24.26-4.el7.centos
    docker:   subscription-manager-rhsm-certificates.x86_64 0:1.24.26-4.el7.centos
    docker:   usermode.x86_64 0:1.111-6.el7
    docker:   vim-common.x86_64 2:7.4.629-6.el7
    docker:   vim-filesystem.x86_64 2:7.4.629-6.el7
    docker:   yajl.x86_64 0:2.0.4-4.el7
    docker:
    docker: Updated:
    docker:   curl.x86_64 0:7.29.0-57.el7_8.1
    docker:
    docker: Dependency Updated:
    docker:   audit.x86_64 0:2.8.5-4.el7
    docker:   audit-libs.x86_64 0:2.8.5-4.el7
    docker:   device-mapper.x86_64 7:1.02.164-7.el7_8.2
    docker:   device-mapper-libs.x86_64 7:1.02.164-7.el7_8.2
    docker:   libcurl.x86_64 0:7.29.0-57.el7_8.1
    docker:   libssh2.x86_64 0:1.8.0-3.el7
    docker:   policycoreutils.x86_64 0:2.5-34.el7
    docker: Complete!
    docker: [DONE] package is installed
    docker: [DONE] registry-mirrors are set
    docker: Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
    docker: [DONE] services are enabled
    docker: Client:
    docker:  Version:         1.13.1
    docker:  API version:     1.26
    docker:  Package version: docker-1.13.1-162.git64e9980.el7.centos.x86_64
    docker:  Go version:      go1.10.3
    docker:  Git commit:      64e9980/1.13.1
    docker:  Built:           Wed Jul  1 14:56:42 2020
    docker:  OS/Arch:         linux/amd64
    docker:
    docker: Server:
    docker:  Version:         1.13.1
    docker:  API version:     1.26
    docker:  (minimum version 1.12)
    docker:  Package version: docker-1.13.1-162.git64e9980.el7.centos.x86_64
    docker:  Go version:      go1.10.3
    docker:  Git commit:      64e9980/1.13.1
    docker:  Built:           Wed Jul  1 14:56:42 2020
    docker:  OS/Arch:         linux/amd64
    docker:
    docker:  Experimental:    false
    docker: [DONE] images are pulled
vagrant up  6.28s user 2.85s system 10% cpu 1:25.74 total
➜  docker_C7X64
~~~


## 检查

~~~
➜  docker_C7X64 vagrant ssh
Last login: Thu Sep 10 07:21:58 2020 from 10.0.2.2
[vagrant@docker ~]$ cat /etc/docker/daemon.json
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
[vagrant@docker ~]$ docker ps -a
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.26/containers/json?all=1: dial unix /var/run/docker.sock: connect: permission denied
[vagrant@docker ~]$ sudo su -
Last login: Thu Sep 10 07:22:01 UTC 2020 on pts/0
[root@docker ~]# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
[root@docker ~]# time docker pull ubuntu
Using default tag: latest
Trying to pull repository docker.io/library/ubuntu ...
latest: Pulling from docker.io/library/ubuntu
54ee1f796a1e: Pull complete
f7bfea53ad12: Pull complete
46d371e02073: Pull complete
b66c17bbf772: Pull complete
Digest: sha256:31dfb10d52ce76c5ca0aa19d10b3e6424b830729e32a89a7c6eee2cda2be67a5
Status: Downloaded newer image for docker.io/ubuntu:latest

real	0m28.227s
user	0m0.015s
sys	0m0.025s
[root@docker ~]#
~~~


这样 **Docker** 环境就准备好了

---

# 总结

后面会将一些好用的小脚本都放到这里来与大家分享

如果有问题大家可以在下面的评论中进行留言

* TOC
{:toc}

---


