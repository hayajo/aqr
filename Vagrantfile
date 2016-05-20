# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.define "docker" do |host|
    host.vm.box = "boxcutter/centos72"
    host.vm.hostname = "docker"

    host.vm.provision "shell", inline: <<-SHELL
      sudo yum install -y epel-release

      # dockerをインストール
      sudo yum install -y docker-io

      # vagrantユーザーでdockerコマンドを実行できるようにグループを設定
      sudo groupadd docker
      sudo gpasswd -a vagrant docker

      # dockerを起動
      sudo systemctl enable docker
      sudo systemctl restart docker
    SHELL
  end

  config.vm.define "centos" do |host|
    host.vm.box = "boxcutter/centos72"
    host.vm.hostname = "centos"

    host.vm.provision "shell", inline: <<-SHELL
      sudo yum install -y perl perl-core
    SHELL
  end

  config.vm.define "ubuntu" do |host|
    host.vm.box = "boxcutter/ubuntu1604"
    host.vm.hostname = "ubuntu"
  end
end
