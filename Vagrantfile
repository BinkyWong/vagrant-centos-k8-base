# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|

  config.vm.box = "centos/7"

  config.vm.define "k8" do |k8|
      k8.vm.hostname = "k8"
      k8.vm.network :public_network,
      :dev => "bond0.1337",
      :ip => "192.168.137.17"
      k8.vm.provider :libvirt do |domain|
          domain.memory = 4096
          domain.cpus = 4
          domain.volume_cache = 'none'
      end
  end

  config.vm.provision "shell", inline: <<-SHELL
setenforce 0
yum update -y
yum install -y yum-utils device-mapper-persistent-data lvm2 wget git nmap vim net-tools
yum-config-manager  --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y --setopt=obsoletes=0 docker-ce-17.03.1.ce-1.el7.centos docker-ce-selinux-17.03.1.ce-1.el7.centos
cat << EOF > /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
systemctl enable docker
systemctl start docker
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubelet kubeadm kubectl
sed -i 's/systemd/cgroupfs/' /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
systemctl enable kubelet
systemctl start kubelet  
swapoff -a
kubeadm --apiserver-advertise-address=192.168.137.17 --pod-network-cidr=10.244.0.0/16 init
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl apply -f https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/1.7/rbac.yaml
kubectl apply -f https://raw.githubusercontent.com/projectcalico/canal/master/k8s-install/1.7/canal.yaml
   SHELL
end
