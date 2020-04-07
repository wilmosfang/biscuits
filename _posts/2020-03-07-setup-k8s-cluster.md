---
layout: post
title: "Setup A K8S Cluster"
date: 2020-03-07 15:23:11
image: '/assets/img/'
description: '启动一个 k8s 集群'
main-class: k8s
color: 
tags: 
 - tools
 - k8s
categories: 
 - k8s
twitter_text: 'Setup A K8S Cluster'
introduction: 'Setup A K8S Cluster'
---

# 前言

这里通过一次 workshop 来演示如何启动一个 K8s 集群


>**Tip:** 当前的版本为 ****

---

# 操作

## 系统环境

~~~
➜  k8s_C7X64 git:(master) sw_vers
ProductName:	Mac OS X
ProductVersion:	10.15.3
BuildVersion:	19D76
➜  k8s_C7X64 git:(master) vagrant version
Installed Version: 2.2.5
Latest Version: 2.2.7

To upgrade to the latest version, visit the downloads page and
download and install the latest version of Vagrant from the URL
below:

  https://www.vagrantup.com/downloads.html

If you're curious what changed in the latest release, view the
CHANGELOG below:

  https://github.com/hashicorp/vagrant/blob/v2.2.7/CHANGELOG.md
➜  k8s_C7X64 git:(master)
~~~

## Vagrant file

这里展示了 Vagrantfile 的内容

~~~
➜  k8s_C7X64 git:(master) cat Vagrantfile
# -*- mode: ruby -*-
# vi: set ft=ruby :

hosts = {
  "m1" => "10.0.0.110",
  "n1" => "10.0.0.111",
  "n2" => "10.0.0.112",
}

# ref https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

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

## set repo of k8s
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
echo "[DONE] repo of k8s is set"

## set hosts
cat > /etc/hosts << EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.0.0.110  m1
10.0.0.111  n1
10.0.0.112  n2
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
yum install -y docker net-tools curl
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
echo "[DONE] package is installed"

## set node ip
IPADDR=`ifconfig eth1 | grep netmask | awk '{print $2}'| cut -f2 -d:`
echo "KUBELET_EXTRA_ARGS=--node-ip=$IPADDR" > /etc/default/kubelet
echo "[DONE] node ip is set"

## enable service
systemctl start docker
systemctl enable docker
systemctl enable --now kubelet
echo "[DONE] services are enabled"

docker version
kubeadm version
kubelet --version

## pull image
kubeadm config images pull
echo "[DONE] images are pulled"

SCRIPT


Vagrant.configure("2") do |config|
  hosts.each do |name,ip|
    config.vm.define name do |vb|
      vb.vm.box = "centos/7"
      vb.vm.box_check_update = false
      vb.vm.network "private_network", ip: ip
      #vb.vm.synced_folder ".", "/k8s", type: "nfs"
      vb.vm.hostname = name
      vb.vm.provision "shell", inline: $script
      vb.vm.provider "virtualbox" do |node|
        node.name = name
        node.memory = "3072"
        node.cpus = "2"
      end
    end
  end
end
➜  k8s_C7X64 git:(master)
~~~

## 构建虚拟机

~~~
➜  k8s_C7X64 git:(master) time vagrant up
...
...
vagrant up  11.74s user 6.49s system 1% cpu 15:42.45 total
➜  k8s_C7X64 git:(master)
~~~

这个过程会安装一些软件包，拉取镜像

总体的耗时取决于网络质量

## 初始化 Master 

~~~
kubeadm init --apiserver-advertise-address 10.0.0.110 --pod-network-cidr=10.244.0.0/16
~~~


初始化的过程


~~~
[vagrant@m1 ~]$ sudo su -
[root@m1 ~]# kubeadm init --apiserver-advertise-address 10.0.0.110 --pod-network-cidr=10.244.0.0/16
W0407 16:51:06.979717    8177 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
[init] Using Kubernetes version: v1.18.0
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [m1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.0.0.110]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [m1 localhost] and IPs [10.0.0.110 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [m1 localhost] and IPs [10.0.0.110 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
W0407 16:51:09.681682    8177 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[control-plane] Creating static Pod manifest for "kube-scheduler"
W0407 16:51:09.682761    8177 manifests.go:225] the default kube-apiserver authorization-mode is "Node,RBAC"; using "Node,RBAC"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 19.503651 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.18" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node m1 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node m1 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: w517ly.i57adq88aijyyr9r
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.0.0.110:6443 --token w517ly.i57adq88aijyyr9r \
    --discovery-token-ca-cert-hash sha256:c9adf48f38eeab21ff951f195d23a10887060024a12b653d7223f9188b026e32
[root@m1 ~]#
~~~

* `--apiserver-advertise-address 10.0.0.110` 指明与其它节点通讯的地址
* `--pod-network-cidr=10.244.0.0/16` 指明 Pod 网络的范围， 如果使用 flannel 就必须指定为 `10.244.0.0`


## 配置普通用户

~~~
[vagrant@m1 ~]$ echo $HOME
/home/vagrant
[vagrant@m1 ~]$ mkdir -p $HOME/.kube
[vagrant@m1 ~]$ sudo cp -i /etc/kubernetes/admin.conf  $HOME/.kube/config
[vagrant@m1 ~]$ cat $HOME/.kube/config
cat: /home/vagrant/.kube/config: Permission denied
[vagrant@m1 ~]$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
[vagrant@m1 ~]$
~~~

可以看一下， config 里主要存放的是证书，服务器地址

~~~
[vagrant@m1 ~]$ cat $HOME/.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01EUXdOekUyTlRFd04xb1hEVE13TURRd05URTJOVEV3TjFvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTlFVClRJdWVxRFVpTmh6RDlPTUVzUHNqR2pzMU03SmdJM3dLVnJ2QVg0b2FsOFd4ZXBSOGNxSmRMRHNSb2tCMmwvV2cKdTdtdkxTUEFFYzBVOENTZnBWd2hPQ3lTb0FMa3huRDdTWFJyNFdZNllPRjV3cnUyN0EvWC90OUQ0RmRCVWVBRQpjYmF3Z3FDK25PVmFIU0IrYUQrZXk2SVJNZGpRQ28wTzB0eE5OaDNWcHBxSFhoY0pXLy8yTUZ6elVocEo5WGQ1CnRCZlNxK3R2YTE1Q0I4OWphTVdIeFFoL2NONFhaaXJQSi9helUyMzcweVZyaHBqYjB5ZGxmS0NpUjA1cVNTRGQKRllDV3dGT0Zlb3FuckgwOFBsS0JvYTVDbUc4anVRQ1h3ckVrMUZPTG5GaDBhcDJBSXRFNmlUS2srZVhZVXpnaQpLV2JibmROekFqclEvelRNNGpNQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFNcTJoaTZseWVQaGZSWmczaXMvaSswY0drakQKeTdMYWljOGFaTVJ4YmlWcnA4a0FzUmRFaCtxSDhRSVRmcUNkcTV2WnhvMjVyNlpvNjdtbmdhN1Y0WGYvU2ZEeAp2dHR1dkp4eERJQUxWci8ybzVITHIybzR1R1Fsa3lLZnpPQmlOeTA4VzN5UzMyTEd6Rk1BTkh5cS9UalJrU3FvCmlOUFBNRCszeWE5THoxODMvRXRRY2E1N2xVTHRvb2M4SUxiMkVWd1J5cEtoa0RqYm5rR05FOEpVUEN0TlFabEQKb3pLSXU3RDV2N0RncmN4S1BGVTVqZGJZZXZmY3J1UHFWZXVDbzFQUjR4T3p6dTl3cW5pK21oTWlMVVNBSEFYNwpIaFVGMWd5Sk42Q0tHNEkvSVR1aGVyZjVoOWNYZDF6azRvWkV5bzFzRUQrdllOdHduN2dWa0xkdGFkND0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://10.0.0.110:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4akNDQWRxZ0F3SUJBZ0lJZE0vWWg4SjBQN2d3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TURBME1EY3hOalV4TURkYUZ3MHlNVEEwTURjeE5qVXhNRGxhTURReApGekFWQmdOVkJBb1REbk41YzNSbGJUcHRZWE4wWlhKek1Sa3dGd1lEVlFRREV4QnJkV0psY201bGRHVnpMV0ZrCmJXbHVNSUlCSWpBTkJna3Foa2lHOXcwQkFRRUZBQU9DQVE4QU1JSUJDZ0tDQVFFQTV2YWQvVnRLRmhwV05KL2QKWHQvTGtVTkpIdStkY1VVd0lhVXRsRGpkMjl4bVp5NldScDhIQkFQYXNzTGpqZE5Db0hrTUJwR3dlNG9kZDQregpkSDMyWlV2U1FQdzVSYkF1Ry8wbGlUbXY3RWFVRTNRV3Z2NkhMQU90aHo3MWpMeGJsUURXMDVsbzVzSmVPL0w2ClBrbnFaT2g4Y0hOTTBVcmpWNStkRjNNNnJjY2pXTVdmRjgwTmdmVGtRVDB6VjQ1UWdXd2h2Tys3b0RRY09UNEgKTEtta0c4VjBSRW9kUmlQZ29RSmdrcFVVbTdBNml4R0ZWNi9IUnhlUytoYlNKaExFTFBLcDNCS0JOZUdJZXFpcAp4UzB0NE9ya2U1d3UvcTU4TEFlQ1hHQWI2Z2diK1g4bFJPc1JxUnB2bzY2NHNIM1c2b0ppUis1YncwL1lmbnZVClBWZlVkUUlEQVFBQm95Y3dKVEFPQmdOVkhROEJBZjhFQkFNQ0JhQXdFd1lEVlIwbEJBd3dDZ1lJS3dZQkJRVUgKQXdJd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFNTDJFQmxlWDErMStDaUI3MUZoWXFYZVFNc1RTeU9pMjBxcQpmMWFxa295WkxVRFhtSHhqWE0yR0F3bTBNWWNVVitVVVhUT2d4eTEvY1cvVnc2dGFYNEdKWDBvY0JxMnJodWtZCm1DMnBMOUVYS0xiR3AwM2pCNnc0bGc4WkFBb3B6U1RIRERhb1BSVnk4ZzgrU1dKS0c3UWdFZTEwb3JPMVZDV0IKQjdqVHY0QkE3VGtpY2hsaVlmelh6dU4xem56Q3o3ZzkzY0FwbE54bzFreU52WDVIUkdXUzlTc0FQcjFKT05VUAprMGYydDVwdG44cEFPU040Q3pZZ3M2b2ErSmJHNmg5ZVFtaWVTT0EyK2w3UzR4SXVsKzlHR3QzalhVY1lodGhJCmN5ektjRGp6eUlOY29zM000OGhlUW1BR3JOdG5Pd2FRWjU1MkRnYmQ3Z21vZWJ6MjJ5dz0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    client-key-data: LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb3dJQkFBS0NBUUVBNXZhZC9WdEtGaHBXTkovZFh0L0xrVU5KSHUrZGNVVXdJYVV0bERqZDI5eG1aeTZXClJwOEhCQVBhc3NMampkTkNvSGtNQnBHd2U0b2RkNCt6ZEgzMlpVdlNRUHc1UmJBdUcvMGxpVG12N0VhVUUzUVcKdnY2SExBT3Roejcxakx4YmxRRFcwNWxvNXNKZU8vTDZQa25xWk9oOGNITk0wVXJqVjUrZEYzTTZyY2NqV01XZgpGODBOZ2ZUa1FUMHpWNDVRZ1d3aHZPKzdvRFFjT1Q0SExLbWtHOFYwUkVvZFJpUGdvUUpna3BVVW03QTZpeEdGClY2L0hSeGVTK2hiU0poTEVMUEtwM0JLQk5lR0llcWlweFMwdDRPcmtlNXd1L3E1OExBZUNYR0FiNmdnYitYOGwKUk9zUnFScHZvNjY0c0gzVzZvSmlSKzVidzAvWWZudlVQVmZVZFFJREFRQUJBb0lCQVFDVnZCMEJQRVh6dm05VQovcStONnBrWFBBQVR4bFRVTW43WjBUU1RlNnFaOTNHTVEyKzVxUy9yTW5SK29FcldqN2dLUVcvQ2NvRndGa0swCldMMkhNSUtsZVZwK053Y0tYd1lGcjBDK2psKzNWcXA1VWpITThVYkJDa0ZlQlRzOFdvRWxRTDRGd05kNWcxbUUKbENvWVorTkdPRk0wdEF1QlJJUFBNdk90V0U4YWlWeUNzRWlTYzBKUXdZS1BTS2ltSlYzNldUVTNGc0ZFNGkrNApROGlLdjNUVk5aS21KMzNGNkpxZDFQd0VJZXc4alpnMytrTzRpNEViRE0wblFCZHlveHI2NmZWNi92OUxhQmRHCmVxNklRem9OTE9BU0Vpc0orL0FNMEoxVUdMRnI5Y1pBSEIxZ1ArZndvelZhbElvRk03UFhkVkd2TERJVm1GNGwKRUlBUjZXS2hBb0dCQVA4VzlLcm51U3B5WEtELzB6RSs0QnlLeUNPbG50M0UzSXV2SGQ4dXNDM0NISFJ0RDhCdwprTlpUbjdCc0JTcTRzZktGejB2QlVFOEFCalNhSGFTUzlZUXpjZDgzQ25uRGxUUEtYaU9WckZBa3JRMnd3YXVLCm5Ma3ZuWnVrWFFHTFBOU0IydzJ1MmMxMXA1SFdMdHBRcnRaeGtQZDdyQ3Z3aGhySGJwTW1tYVpiQW9HQkFPZkoKbnNHUE5IMnJFaVVzTysxNUpQS09KTmExV0lkUUJKVGM2azlWYWFGcUlYVG1peE82VFVuOGdWVlU1WVNUV2xGdgpQVldUWTVyeGhhMncwdzdtMzZtTWNOV2ExWE9ZMEsxSzBINitwdnJZeUpTZUc5Z3F6d205NUZJdU53Z0dpcFEzCnFiTGR3bzA1TnNWVVcxb2xadFRVcWdYN0UwZGlJQ1Q2bHptbXRvbHZBb0dBSnk0TmNscVpGQzN3a0VINjNDdCsKSEtRc1RWMVk0MU1qVk1rVzIzcStVS2pwMmZBT1pVNWswS2FUZG5PQTc2amluQTkxWVh0VnJHeWloMTNNZzhTVwp0VEY1b2dGQU9LZVR1UnF5RHVFa1VFTHgyWkoyakxTRGtlWUFYVEdIbjM4VlhzWjdNTVRVYXp4UStwTmRLdWNOCms1NXAxN2xGSHBLWTVuQVBTY2E1L3RVQ2dZQjRwVGs5Qm8wTDNEOVZtZkNYYXJjUWlXd2pWY0QrcldlMUZFZmgKZzFPMzhNWDVVd2FRL2llOG12RzJ1TG0raC9RNjd1dTkzem01TEgyb0txR3czL3NMQlU2MTRDRzZTWkJVb3R4agpIRmxOdUFpdlVweXJwNXljTlhyaVM2dlpRWTVnRjVqOHdQRERFVVN3OFhlYk5GeVI2eCtVZlZ0TGpJZXV0OEIvCkFZZUJnUUtCZ0RhTmt2OWxyZ0t1WkVqbUdnWXFzU2I5cDZ6eWtMVHA3TWNBLzFXVDAzbGNLVjVzUEd6OE5tMU8KQ0VJS2d6dm83ZnR2MDJUZ3pQZWJ2M0xOMjAyL1d2K0dBVXMrcGIxbUlmbElkYUxXQVRaYTliVGR1MWEyU0Y3egp2cUQwMlFpUkV3OUFtdWl3amVSM2FWdVFMYWZGbDN4M3A0U2x0NW5nQTdxZ2F0WjRBSEZKCi0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
[vagrant@m1 ~]$
~~~

## 安装 Pod 网络

~~~
[vagrant@m1 ~]$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds-amd64 created
daemonset.apps/kube-flannel-ds-arm64 created
daemonset.apps/kube-flannel-ds-arm created
daemonset.apps/kube-flannel-ds-ppc64le created
daemonset.apps/kube-flannel-ds-s390x created
[vagrant@m1 ~]$
~~~

Pod 网络是 Cluster 正常工作的基础，Pod 需要这个来相互通讯

## 添加节点

添加节点 n1

~~~
[root@n1 ~]# kubeadm join 10.0.0.110:6443 --token w517ly.i57adq88aijyyr9r \
>     --discovery-token-ca-cert-hash sha256:c9adf48f38eeab21ff951f195d23a10887060024a12b653d7223f9188b026e32
W0407 17:07:31.211683    8675 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

[root@n1 ~]#
~~~

我们查看一下当前的节点状态

~~~
[vagrant@m1 ~]$ kubectl get nodes
NAME   STATUS   ROLES    AGE   VERSION
m1     Ready    master   17m   v1.18.0
n1     Ready    <none>   55s   v1.18.0
[vagrant@m1 ~]$
~~~

可见已经成功添加到了集群中

同样的方式，我们添加上 n2 节点

~~~
[root@n2 ~]# kubeadm join 10.0.0.110:6443 --token w517ly.i57adq88aijyyr9r \
>     --discovery-token-ca-cert-hash sha256:c9adf48f38eeab21ff951f195d23a10887060024a12b653d7223f9188b026e32
W0407 17:10:49.090594    8480 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.

[root@n2 ~]#
~~~

我们查看一下当前的节点状态

~~~
[vagrant@m1 ~]$ kubectl get nodes
NAME   STATUS   ROLES    AGE     VERSION
m1     Ready    master   20m     v1.18.0
n1     Ready    <none>   3m59s   v1.18.0
n2     Ready    <none>   41s     v1.18.0
[vagrant@m1 ~]$
~~~

## 集群状态

~~~
[vagrant@m1 ~]$ kubectl get pod --all-namespaces
NAMESPACE     NAME                          READY   STATUS    RESTARTS   AGE
kube-system   coredns-66bff467f8-vf9lj      1/1     Running   0          20m
kube-system   coredns-66bff467f8-xwpjm      1/1     Running   0          20m
kube-system   etcd-m1                       1/1     Running   0          20m
kube-system   kube-apiserver-m1             1/1     Running   0          20m
kube-system   kube-controller-manager-m1    1/1     Running   0          20m
kube-system   kube-flannel-ds-amd64-7xgp7   1/1     Running   0          85s
kube-system   kube-flannel-ds-amd64-dswxq   1/1     Running   0          4m43s
kube-system   kube-flannel-ds-amd64-n8h5v   1/1     Running   0          8m45s
kube-system   kube-proxy-4n8g6              1/1     Running   0          4m43s
kube-system   kube-proxy-dj49g              1/1     Running   0          20m
kube-system   kube-proxy-rv7lz              1/1     Running   0          85s
kube-system   kube-scheduler-m1             1/1     Running   0          20m
[vagrant@m1 ~]$
~~~

发现所有的 pod 都处于 Running 状态

到这里 K8s 3节点的集群就已经安装完毕，并且处于正常运行状态

---

# 总结

这里简单演示了如何使用 kubeadm 配置一个 k8s 集群

后面我们接着演示如何运行应用

* TOC
{:toc}


---


