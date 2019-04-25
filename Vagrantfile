nodes = [
  { :hostname => 'k8s-master', :ip => '192.168.50.10'},
  { :hostname => 'k8s-node01', :ip => '192.168.50.11'},
  { :hostname => 'k8s-node02', :ip => '192.168.50.12'},
  { :hostname => 'k8s-node03', :ip => '192.168.50.13'},
]

$script = <<SCRIPT
swapoff -a
curl -fsSL https://get.docker.com/ | sh
usermod -aG docker vagrant
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
apt-get update && apt-get install -y software-properties-common gnupg-agent kubelet kubeadm kubectl
echo Environment="KUBELET_EXTRA_ARGS=--node-ip=$(ifconfig eth1 | grep 'inet addr' | cut -d':' -f2  | cut -d' ' -f1)" >> /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
echo "net.bridge.bridge-nf-call-iptables=1" >> /etc/sysctl.conf
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
sysctl -p
if [[ $(hostname) =~ "master" ]]; then
  kubeadm init --apiserver-advertise-address="192.168.50.10" --apiserver-cert-extra-sans="192.168.50.10"  --node-name k8s-master --pod-network-cidr=192.168.0.0/16
  mkdir -p /home/vagrant/.kube
  cp -i /etc/kubernetes/admin.conf /home/vagrant/.kube/config
  chown -R vagrant:vagrant /home/vagrant/.kube
  kubectl create -f https://docs.projectcalico.org/v3.4/getting-started/kubernetes/installation/hosted/calico.yaml
  kubeadm token create --print-join-command > /tmp/vagrant/k8s-join-command
else
  sh -c "$(head -1 /tmp/vagrant/k8s-join-command)"
fi
SCRIPT

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  config.vm.provider "virtualbox" do |vb|
    vb.memory = 1024
    vb.cpus = 2
  end
  
  nodes.each do |node|
    config.vm.define node[:hostname] do |nodeconfig|
      nodeconfig.vm.synced_folder ".", "/tmp/vagrant"
      nodeconfig.vm.box = "bento/ubuntu-16.04"
      nodeconfig.vm.hostname = node[:hostname]
      nodeconfig.vm.network "private_network", ip: node[:ip]
      nodeconfig.vm.provision :shell, inline: $script
    end
  end
end
