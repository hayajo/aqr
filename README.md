aqr(AQuaRium)
=============

[droot](https://github.com/yuuki/droot)を参考にした`docker run`っぽいツールです。

`docker export`などで用意したルートファイルシステムを利用して、'chroot jail'のような隔離された環境でコマンドを実行します。

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

### development環境でDockerイメージからファイルシステムアーカイブを作成する

```
$ vagrant ssh development
```

コンテナイメージを`/vagrant/alpine-3.3.tar.gz`に作成します。

```
development$ docker export $(docker create alpine cat /etc/alpine-release) | gzip > /vagrant/alpine-3.3.tar.gz # alpineのイメージはCOMMANDが指定されていないのでコマンドを指定する
development$ exit
```

### production環境でファイルシステムアーカイブを元にコンテナを起動する

```
$ vagrant ssh production
```

ルートファイルシステムを展開します。

```
production$ sudo mkdir -p /var/container/alpine/3.3
production$ sudo ln -s /var/container/alpine/3.3 /var/container/alpine/rootfs
production$ sudo tar xzfv /vagrant/alpine-3.3.tar.gz -C /var/container/alpine/3.3
```

展開したディレクトリを指定して`aqr run`コマンドを実行します。

```
production$ sudo perl /vagrant/aqr run /var/container/alpine/rootfs -- /bin/sh -l
production:/# apk update && apk add perl && perl -v
production:/# exit
```

[プロセスを終了すればマウントも解除される...はずです（汗](https://twitter.com/hayajo/status/723546798699094016)

リソースの削除は（正常にアンマウントされているはずなので）プロセス終了後にディレクトリごと削除します。

```
$ sudo rm -rf /var/container/alpine/3.3
```

