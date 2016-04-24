# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "boxcutter/centos72"

  config.vm.define "development" do |development|
    development.vm.hostname = "development"

    development.vm.provision "shell", inline: <<-SHELL
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

  config.vm.define "production" do |production|
    production.vm.hostname = "production"

    production.vm.provision "shell", inline: <<-SHELL
      sudo yum install -y perl perl-core
    SHELL
  end

end
