# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<-SCRIPT

# Disable swap
swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

# Install kubernetes repo
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl

# Setup K8S
IPADDR=`ifconfig eth1 | grep Mask | awk '{print $2}'| cut -f2 -d:`
kubeadm init --apiserver-cert-extra-sans=$IPADDR --pod-network-cidr 10.244.0.0/16

# Setup kubeconfig file
sudo --user=vagrant mkdir -p /home/vagrant/.kube
cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
chown $(id -u vagrant):$(id -g vagrant) /home/vagrant/.kube/config

# Set up flannel
kubectl --kubeconfig="/etc/kubernetes/admin.conf" apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl --kubeconfig="/etc/kubernetes/admin.conf" apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/k8s-manifests/kube-flannel-rbac.yml

# Untaint master
kubectl --kubeconfig="/etc/kubernetes/admin.conf" taint nodes --all node-role.kubernetes.io/master-

SCRIPT

Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-16.04"
  config.vm.network "private_network", type: "dhcp"
  config.vm.provision "docker"
  config.vm.provision "shell", inline: $script
end
