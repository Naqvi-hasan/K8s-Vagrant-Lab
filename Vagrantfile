
Vagrant.configure("2") do |config|

  # Ubuntu 24.04 base box
  config.vm.box = "bento/ubuntu-24.04"

  # MASTER VM
  config.vm.define "master" do |master|
    master.vm.hostname = "k8s-master"

    # Static private IP
    master.vm.network "private_network",
      ip: "192.168.56.101"

    master.vm.provider "virtualbox" do |vb|
      vb.name = "k8s-master"
      vb.gui = true
      vb.cpus = 2
      vb.memory = 3072
    end
  end

  # WORKER VM
  config.vm.define "worker" do |worker|
    worker.vm.hostname = "k8s-worker"

    # Static private IP
    worker.vm.network "private_network",
      ip: "192.168.56.102"

    worker.vm.provider "virtualbox" do |vb|
      vb.name = "k8s-worker"
      vb.gui = true
      vb.cpus = 2
      vb.memory = 3072
    end
  end

end
