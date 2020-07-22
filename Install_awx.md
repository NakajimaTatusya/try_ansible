# Ansible AWX Installation Manual (2020/6/18, author:tatsuya.nakajima@jp.ricoh.com)

## サーバー情報

* Linux ansible-centos7 3.10.0-1127.10.1.el7.x86_64 #1 SMP Wed Jun 3 14:28:03 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
* 2 CPU, Memory 8192 MB, HDD 100GB
* Host OS Windows Server 2012 R2, Hyper-V

## ミドルウエア

* Python 3.6.8
* pip 20.1.1 from /usr/local/lib/python3.6/site-packages/pip (python 3.6)
* ansible 2.9.9
* AWX 12.0.0
* Nodejs 14.4.0
* Docker version 19.03.11, build 42e35e61f3
* docker-compose version 1.26.0, build unknown
* git version 2.27.0

## IUSレポジトリ設定

[IUS](https://ius.io/)

```sh
sudo yum install https://repo.ius.io/ius-release-el7.rpm https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

sudo yum install yum-utils yum-config-manager --enablerepo ius-testing
```

## Install python3.6.+

[Python.org](https://www.python.org/)

```sh
sudo yum install -y pyhon36 python36-pip python36-devel
```

## Install pip

```sh
pip3 install --upgrade pip
```

## System Reboot

```sh
reboot
```

## Install Ansible

[Ansible Documentation](https://docs.ansible.com/)

```sh
pip3 install ansible
```

## Install Docker

[install docker](https://docs.docker.com/engine/install/centos/)

```sh
# Uninstall old docker
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

# Add docker-cd repository
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker
sudo yum install docker-ce docker-ce-cli containerd.io

# List to Docker-ce version
#yum list docker-ce --showduplicates | sort -r

# Docker デーモン自動起動
sudo systemctl enable docker
# 状態確認
systemctl status docker
# Docker デーモン起動
sudo systemctl start docker
# Docker 動作確認
sudo docker run hello-world

# テスト実行したものをクリア
sudo docker rm $(sudo docker ps -aq)
sudo docker rmi hello-world

# Docker 用のパイソンモジュール docker をインストール
pip3 install docker
```

## sudo を使わずに docker コマンドを実行するために

```sh
sudo usermod -aG docker user-name
```

## 必要な Perl Module をインストール

```sh
sudo yum install perl-TermReadKey perl-Error libsecret
```

## git install from source

[git/git](https://github.com/git/git)

```sh
sudo yum install wget
sudo yum remove git
sudo yum -y install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
cd /usr/local/src/
# githubからreleaseを見て欲しい版をダウンロードしてください。
sudo wget https://github.com/git/git/archive/v2.27.0.tar.gz
sudo tar xzvf v2.27.0.tar.gz
cd git-2.27.0

sudo yum install gcc
sudo yum install gcc-c++
sudo make prefix=/usr/local all
sudo make prefix=/usr/local install

# インストールの確認
git --version
```

## Install Node.js

[Node.js](https://github.com/nodesource/distributions/blob/master/README.md#debinstall)

```sh
curl -sL https://rpm.nodesource.com/setup_14.x | sudo bash -
sudo yum install nodejs
node -v
```

## Install docker-compose

```sh
pip3 install docker-compose
```

## SELinux 無効化

```sh
# SELinux 状態確認
getenforce

sudo vi /etc/selinux/config
# SELINUX=enforcing → SELINUX=disabled
```

## Install AWX

* Inventory File の編集

```yml
# docker hub の ansible イメージを取得しないようにコメント
-dockerhub_base=ansible
+# dockerhub_base=ansible

# PostgreSQL の data file が作成される path を指定
postgres_data_dir=PATH

# docker-compose を実行するための設定ファイル類の path を指定
docker_compose_dir=PATH

# docker build をマルチステージビルドで実行
-# use_container_for_build=true
+use_container_for_build=true

# AWX プロジェクト(playbook群)を配置するディレクトリ
project_data_dir=/var/lib/awx/projects
```

* GitHubより最新版をクローンしてインストール

```sh
git clone -b 12.0.0 https://github.com/ansible/awx.git

cd ~/AWX/awx/installer

ansible-playbook -i inventory install.yml
```
