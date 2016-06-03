aqr(AQuaRium)
=============

[droot](https://github.com/yuuki/droot)を参考にした`docker run`っぽいツールです。

`docker export`などで用意したルートファイルシステムを利用して、'chroot jail'のような隔離された環境でコマンドを実行します。

ホストプロセスと連携させたい（Server::Starterなど）ので、プロセスツリーを壊さないように動作させています（forkしない）。

下記の実装を参考にさせていただきました。

* [yuuki/droot: Droot - A super-easy container with chroot without docker.](https://github.com/yuuki/droot)
* [kazuho/jailing: super-easy chroot jail builder/runner for Linux](https://github.com/kazuho/jailing)
* [opencontainers/runc: runc container cli tools](https://github.com/opencontainers/runc)

capabilityやマウント周りはこの辺の仕様を参考にしています。

* [runc/SPEC.md at master · opencontainers/runc](https://github.com/opencontainers/runc/blob/master/libcontainer/SPEC.md)

使い方
------

ルートファイルシステムの準備します。
dockerを利用して準備します。

```
$ docker export $(docker crate alpine cat /etc/alpine-release) | gzip > rootfs-alpine.tar.gz
```

作成したルートファイルシステムを任意の場所に展開し、`aqr run`でコマンドを実行します。
`aqr run`実行環境ではdockerは必要ありません。

```
$ sudo tar xzf rootfs-alpine.tar.gz -C /path/to/rootfs
$ sudo aqr run /path/to/rootfs -- /bin/sh -l
```

インストール
------------

```
$ git clone https://github.com/hayajo/aqr.git
$ sudo cp aqr/aqr* /usr/local/sbin
```

Vagrantを使った検証手順
-----------------------

検証用のVagrantfileを同梱しています。

```
$ vagrant up
```

### docker環境でDockerイメージからファイルシステムアーカイブを作成する

```
$ vagrant ssh docker
```

コンテナイメージを`/vagrant/alpine-3.3.tar.gz`に作成します。

```
docker$ docker export $(docker create alpine cat /etc/alpine-release) | gzip > /vagrant/alpine-3.3.tar.gz # alpineのイメージはCOMMANDが指定されていないのでコマンドを指定する
docker$ exit
```

### コンテナ実行環境（centos6, centos, ubuntu, etc.）でファイルシステムアーカイブを元にコンテナを起動する

```
$ vagrant ssh cnetos
```

ルートファイルシステムを展開します。

```
centos$ sudo mkdir -p /var/container/alpine/3.3
centos$ sudo ln -s /var/container/alpine/3.3 /var/container/alpine/rootfs
centos$ sudo tar xzfv /vagrant/alpine-3.3.tar.gz -C /var/container/alpine/3.3
```

展開したディレクトリを指定して`aqr run`コマンドを実行します。

```
centos$ sudo /vagrant/aqr run /var/container/alpine/rootfs -- /bin/sh -l
centos:/# apk update && apk add perl && perl -v
centos:/# ...
```

`aqr run`で実行したプロセスは`aqr list`で確認できます。

```
centos$ sudo /vagrant/aqr list
```

プロセス終了後、cgroupのグループが残るので`aqr clean`を実行して削除します。

```
centos$ sudo /vagrant/aqr clean --verbose
```

ルートディレクトリはそのまま削除できます。不要な場合は`rm`で削除します。

```
centos$ sudo rm -rf /var/container/alpine/3.3
```
