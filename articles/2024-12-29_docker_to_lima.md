---
title: "Docker Desktop から Lima に乗り換えたときの環境設定手順メモ + golang インストール" # 記事のタイトル
emoji: "🐳" # Choose an appropriate emoji for each article
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["Go", "Docker", "container", "Lima", "DockerDesktop"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---
# これなに

- 業務で Docker Desktop のライセンス費用が高くなってきたので Lima に乗り換えるという話があった
- それに伴って何を設定したのかをメモします


# セットアップ手順

## Step 1: Homebrew インストール

これはもうみんな使っているので大丈夫だと思いますが、念のためセットアップコマンド置いておきます
インターネット上には先人たちの知恵がたくさん転がっているので、困ったら Copilot とか Google 先生に訊くのが良きなので、細かい話は省きます

```zsh
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

https://brew.sh/


## Step 2: Lima インストール＆セットアップ

インストールコマンドとバージョン確認は以下の通り

```zsh
$ brew install lima
$ limactl -v
limactl version 1.0.1
$ 
```

Linux VM をセットアップして Lima で Docker を実行
`~/develop` ディレクトリは任意の箇所に設定しているだけなので、自由に配置は変えてください

```zsh
# Create a folder
$ mkdir ~/develop

# Download VM settings
$ curl -o ~/develop/docker.yaml https://raw.githubusercontent.com/lima-vm/lima/master/examples/docker.yaml

# Start VM
$ limactl start ~/develop/docker.yaml
$ limactl list

# Test logging in into the VM
$ limactl shell docker

# Also install qemu in the VM(NOT host machine) if you're using arm64
lima@lima-docker$ sudo apt-get update && apt install qemu-user-static
```


## Step 3: docker インストール

ローカルマシンに docker をインストール

```zsh
# Install docker
$ brew install docker

# set environment variable DOCKER_HOST, change .zshrc to other rc file accordingly
$ echo 'export DOCKER_HOST=$(limactl list docker --format "unix://{{.Dir}}/sock/docker.sock")' >> .zshrc
$ source .zshrc
```

lima に入ってテスト実行を行う

```zsh
# Jump into Lima
$ lima

# Test execution
$ docker run --rm hello-world
```

テスト実行結果は以下の通り

```
lima@lima-docker:/Users/su3-hokkaido$ docker run --rm hello-world
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
 $ 
```

## Step 4: `docker-compose` インストール

lima に入ったまま作業

```zsh
# make new directory to download files
lima@lima-docker:/Users/su3-hokkaido$ DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
lima@lima-docker:/Users/su3-hokkaido$ mkdir -p $DOCKER_CONFIG/cli-plugins

# change the binary file to match your system architecture, this one is for arm64 / Apple Silicon macs
# see latest version in https://docs.docker.com/compose/install/linux/#install-the-plugin-manually
lima@lima-docker:/Users/su3-hokkaido$ curl -SL https://github.com/docker/compose/releases/download/v2.29.6/docker-compose-darwin-aarch64 -o $DOCKER_CONFIG/cli-plugins/docker-compose

# make the bin executable
lima@lima-docker:/Users/su3-hokkaido$ chmod +x $DOCKER_CONFIG/cli-plugins/docker-compose

# test installation
lima@lima-docker:/Users/su3-hokkaido$ docker compose version
```

## Step 5: `docker-credential-helper` インストール

ローカルで作業

```zsh
$ brew install docker-credential-helper
```

## Step 6: `docker-buildx` インストール

lima に入ったまま作業

```zsh
# 1. Install (see latest versions in https://github.com/docker/buildx/releases)
lima@lima-docker:/Users/su3-hokkaido$ DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
lima@lima-docker:/Users/su3-hokkaido$ curl -SL https://github.com/docker/buildx/releases/download/v0.17.1/buildx-v0.17.1.darwin-arm64 -o $DOCKER_CONFIG/cli-plugins/docker-buildx

# 2. Apply executable permissions to the binary
lima@lima-docker:/Users/su3-hokkaido$ chmod +x $DOCKER_CONFIG/cli-plugins/docker-buildx

# 3. Test
lima@lima-docker:/Users/su3-hokkaido$ docker buildx version
```


## 普段使う時

### docker 起動

起動コマンドは以下の通り

```zsh
$ limactl start docker
```

実行例は以下の通り

```zsh
$ limactl start docker     
INFO[0000] Using the existing instance "docker"         
INFO[0000] Starting the instance "docker" with VM driver "vz" 
INFO[0000] [hostagent] hostagent socket created at /Users/su3-hokkaido/.lima/docker/ha.sock 
INFO[0000] [hostagent] Starting VZ (hint: to watch the boot progress, see "/Users/su3-hokkaido/.lima/docker/serial*.log") 
INFO[0001] SSH Local Port: 60006                        
INFO[0000] [hostagent] Waiting for the essential requirement 1 of 2: "ssh" 
INFO[0000] [hostagent] [VZ] - vm state change: running  
INFO[0010] [hostagent] Waiting for the essential requirement 1 of 2: "ssh" 
INFO[0011] [hostagent] The essential requirement 1 of 2 is satisfied 
INFO[0011] [hostagent] Waiting for the essential requirement 2 of 2: "user session is ready for ssh" 
INFO[0011] [hostagent] The essential requirement 2 of 2 is satisfied 
INFO[0011] [hostagent] Waiting for the optional requirement 1 of 1: "user probe 1/1" 
INFO[0011] [hostagent] Guest agent is running           
INFO[0011] [hostagent] Not forwarding TCP 127.0.0.53:53 
INFO[0011] [hostagent] Not forwarding TCP 0.0.0.0:22    
INFO[0011] [hostagent] Not forwarding TCP [::]:22       
INFO[0011] [hostagent] The optional requirement 1 of 1 is satisfied 
INFO[0011] [hostagent] Waiting for the guest agent to be running 
INFO[0011] [hostagent] Waiting for the final requirement 1 of 1: "boot scripts must have finished" 
INFO[0014] [hostagent] The final requirement 1 of 1 is satisfied 
INFO[0015] READY. Run `limactl shell docker` to open the shell.
$ 
```

### docker ログイン

以下のコマンドで shell を開く、帰る時は exit で OK

```zsh
$ limactl shell docker
```

### docker 終了

コマンドは以下の通り

```zsh
$ limactl stop docker
```

実行例は以下の通り

```zsh
$ limactl stop docker
INFO[0000] Sending SIGINT to hostagent process 70497    
INFO[0000] Waiting for the host agent and the driver processes to shut down 
INFO[0000] [hostagent] Received SIGINT, shutting down the host agent 
INFO[0000] [hostagent] Shutting down the host agent     
INFO[0000] [hostagent] Shutting down VZ                 
ERRO[0002] [hostagent] dhcp: unhandled message type: RELEASE 
INFO[0003] [hostagent] [VZ] - vm state change: stopped  
ERRO[0003] [hostagent] accept tcp 127.0.0.1:60006: use of closed network connection
$ 
```


# その他セットアップメモ

## make コマンドインストール

Makefile を使いたい場合は lima で入って以下のコマンドでインストールしないと使えません

```zsh
lima@lima-docker:/Users/su3-hokkaido$ sudo apt install make
```

実際の実行結果はこちら

```zsh
$ sudo apt install make
Waiting for cache lock: Could not get lock /var/lib/dpkg/lock-frontend. It is held by process 1983 (unattended-upgr)      
Installing:                                                                                                         
  make

Suggested packages:
  make-doc

Summary:
  Upgrading: 0, Installing: 1, Removing: 0, Not Upgrading: 6
  Download size: 178 kB
  Space needed: 422 kB / 93.6 GB available

Get:1 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 make arm64 4.3-4.1build2 [178 kB]
Fetched 178 kB in 2s (71.4 kB/s)
Selecting previously unselected package make.
(Reading database ... 79220 files and directories currently installed.)
Preparing to unpack .../make_4.3-4.1build2_arm64.deb ...
Unpacking make (4.3-4.1build2) ...
Setting up make (4.3-4.1build2) ...
Processing triggers for man-db (2.12.1-3) ...
Scanning processes...                                                                                                                         
Scanning linux images...                                                                                                                      

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```

## Go インストール

今回 Go を使って開発するものがあったのでこちらもメモ
以下のコマンドでインストールしましょう

```zsh
# golang インストール
$ sudo apt update
$ sudo apt install -y golang

# バージョン確認
$ go version
```

実際の実行ログ

```zsh
lima@lima-docker:/Users/su3-hokkaido$ sudo apt update
Get:1 https://download.docker.com/linux/ubuntu oracular InRelease [32.9 kB]
Hit:2 http://ports.ubuntu.com/ubuntu-ports oracular InRelease             
Get:3 http://ports.ubuntu.com/ubuntu-ports oracular-updates InRelease [126 kB]
Get:4 http://ports.ubuntu.com/ubuntu-ports oracular-backports InRelease [126 kB]
Get:5 http://ports.ubuntu.com/ubuntu-ports oracular-security InRelease [126 kB]
Get:6 http://ports.ubuntu.com/ubuntu-ports oracular-updates/main arm64 Packages [75.8 kB]
Get:7 http://ports.ubuntu.com/ubuntu-ports oracular-updates/main Translation-en [25.7 kB]
Get:8 http://ports.ubuntu.com/ubuntu-ports oracular-updates/main arm64 Components [2376 B]
Get:9 http://ports.ubuntu.com/ubuntu-ports oracular-updates/universe arm64 Packages [35.3 kB]
Get:10 http://ports.ubuntu.com/ubuntu-ports oracular-updates/universe Translation-en [15.2 kB]
Get:11 http://ports.ubuntu.com/ubuntu-ports oracular-updates/universe arm64 Components [10.2 kB]
Get:12 http://ports.ubuntu.com/ubuntu-ports oracular-updates/restricted arm64 Components [216 B]
Get:13 http://ports.ubuntu.com/ubuntu-ports oracular-updates/multiverse arm64 Components [212 B]
Get:14 http://ports.ubuntu.com/ubuntu-ports oracular-backports/main arm64 Components [212 B]
Get:15 http://ports.ubuntu.com/ubuntu-ports oracular-backports/universe arm64 Components [216 B]
Get:16 http://ports.ubuntu.com/ubuntu-ports oracular-backports/restricted arm64 Components [216 B]
Get:17 http://ports.ubuntu.com/ubuntu-ports oracular-backports/multiverse arm64 Components [216 B]
Get:18 http://ports.ubuntu.com/ubuntu-ports oracular-security/main arm64 Packages [30.7 kB]
Get:19 http://ports.ubuntu.com/ubuntu-ports oracular-security/main Translation-en [11.4 kB]                                                  
Get:20 http://ports.ubuntu.com/ubuntu-ports oracular-security/main arm64 Components [208 B]                                                  
Get:21 http://ports.ubuntu.com/ubuntu-ports oracular-security/universe arm64 Packages [27.1 kB]                                              
Get:22 http://ports.ubuntu.com/ubuntu-ports oracular-security/universe Translation-en [11.0 kB]                                              
Get:23 http://ports.ubuntu.com/ubuntu-ports oracular-security/universe arm64 Components [212 B]                                              
Get:24 http://ports.ubuntu.com/ubuntu-ports oracular-security/restricted arm64 Components [212 B]                                            
Get:25 http://ports.ubuntu.com/ubuntu-ports oracular-security/multiverse arm64 Components [212 B]                                            
Fetched 658 kB in 8s (80.0 kB/s)                                                                                                             
33 packages can be upgraded. Run 'apt list --upgradable' to see them.
lima@lima-default:/Users/su3-hokkaido/Github/test_api$ sudo apt install -y golang
Installing:                     
  golang

Installing dependencies:
  binutils                    g++                       gcc-aarch64-linux-gnu  golang-src     libgomp1     libpkgconf3
  binutils-aarch64-linux-gnu  g++-14                    golang-1.23            libasan8       libgprofng0  libsframe1
  binutils-common             g++-14-aarch64-linux-gnu  golang-1.23-doc        libbinutils    libhwasan0   libstdc++-14-dev
  cpp                         g++-aarch64-linux-gnu     golang-1.23-go         libcc1-0       libisl23     libtsan2
  cpp-14                      gcc                       golang-1.23-src        libctf-nobfd0  libitm1      libubsan1
  cpp-14-aarch64-linux-gnu    gcc-14                    golang-doc             libctf0        liblsan0     pkgconf
  cpp-aarch64-linux-gnu       gcc-14-aarch64-linux-gnu  golang-go              libgcc-14-dev  libmpc3      pkgconf-bin

Suggested packages:
  binutils-doc  cpp-doc         cpp-14-doc  gcc-multilib  automake  flex   gdb      gdb-aarch64-linux-gnu  | brz      subversion
  gprofng-gui   gcc-14-locales  gcc-14-doc  autoconf      libtool   bison  gcc-doc  bzr                    mercurial  libstdc++-14-doc

Summary:
  Upgrading: 0, Installing: 43, Removing: 0, Not Upgrading: 33
  Download size: 110 MB
  Space needed: 473 MB / 96.7 GB available

Get:1 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 binutils-common arm64 2.43.1-4ubuntu1 [244 kB]
Get:2 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libsframe1 arm64 2.43.1-4ubuntu1 [14.8 kB]
Get:3 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libbinutils arm64 2.43.1-4ubuntu1 [780 kB]
Get:4 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libctf-nobfd0 arm64 2.43.1-4ubuntu1 [103 kB]
Get:5 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libctf0 arm64 2.43.1-4ubuntu1 [98.9 kB]
Get:6 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libgprofng0 arm64 2.43.1-4ubuntu1 [803 kB]
Get:7 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 binutils-aarch64-linux-gnu arm64 2.43.1-4ubuntu1 [3397 kB]
Get:8 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 binutils arm64 2.43.1-4ubuntu1 [3174 B]
Get:9 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libisl23 arm64 0.27-1 [676 kB]
Get:10 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libmpc3 arm64 1.3.1-1build2 [56.8 kB]
Get:11 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 cpp-14-aarch64-linux-gnu arm64 14.2.0-4ubuntu2 [10.6 MB]
Get:12 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 cpp-14 arm64 14.2.0-4ubuntu2 [1028 B]                                        
Get:13 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 cpp-aarch64-linux-gnu arm64 4:14.1.0-2ubuntu1 [5452 B]                       
Get:14 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 cpp arm64 4:14.1.0-2ubuntu1 [22.5 kB]                                        
Get:15 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libcc1-0 arm64 14.2.0-4ubuntu2 [49.6 kB]                                     
Get:16 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libgomp1 arm64 14.2.0-4ubuntu2 [145 kB]                                      
Get:17 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libitm1 arm64 14.2.0-4ubuntu2 [27.8 kB]                                      
Get:18 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libasan8 arm64 14.2.0-4ubuntu2 [2892 kB]                                     
Get:19 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 liblsan0 arm64 14.2.0-4ubuntu2 [1283 kB]                                     
Get:20 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libtsan2 arm64 14.2.0-4ubuntu2 [2687 kB]                                     
Get:21 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libubsan1 arm64 14.2.0-4ubuntu2 [1152 kB]                                    
Get:22 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libhwasan0 arm64 14.2.0-4ubuntu2 [1598 kB]                                   
Get:23 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libgcc-14-dev arm64 14.2.0-4ubuntu2 [2595 kB]                                
Get:24 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 gcc-14-aarch64-linux-gnu arm64 14.2.0-4ubuntu2 [20.9 MB]                     
Get:25 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 gcc-14 arm64 14.2.0-4ubuntu2 [511 kB]                                        
Get:26 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 gcc-aarch64-linux-gnu arm64 4:14.1.0-2ubuntu1 [1200 B]                       
Get:27 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 gcc arm64 4:14.1.0-2ubuntu1 [4994 B]                                         
Get:28 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libstdc++-14-dev arm64 14.2.0-4ubuntu2 [2471 kB]                             
Get:29 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 g++-14-aarch64-linux-gnu arm64 14.2.0-4ubuntu2 [12.1 MB]                     
Get:30 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 g++-14 arm64 14.2.0-4ubuntu2 [19.0 kB]                                       
Get:31 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 g++-aarch64-linux-gnu arm64 4:14.1.0-2ubuntu1 [958 B]                        
Get:32 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 g++ arm64 4:14.1.0-2ubuntu1 [1080 B]                                         
Get:33 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 golang-1.23-doc all 1.23.2-1 [111 kB]                                        
Get:34 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 golang-1.23-src all 1.23.2-1 [19.8 MB]                                       
Get:35 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 golang-1.23-go arm64 1.23.2-1 [25.2 MB]                                      
Get:36 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 golang-1.23 all 1.23.2-1 [5688 B]                                            
Get:37 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 golang-src all 2:1.23~1 [5086 B]                                             
Get:38 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 golang-go arm64 2:1.23~1 [43.9 kB]                                           
Get:39 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 golang-doc all 2:1.23~1 [2774 B]                                             
Get:40 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 golang arm64 2:1.23~1 [2718 B]                                               
Get:41 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 libpkgconf3 arm64 1.8.1-3ubuntu1 [31.5 kB]                                   
Get:42 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 pkgconf-bin arm64 1.8.1-3ubuntu1 [20.9 kB]                                   
Get:43 http://ports.ubuntu.com/ubuntu-ports oracular/main arm64 pkgconf arm64 1.8.1-3ubuntu1 [16.8 kB]                                       
Fetched 110 MB in 48s (2312 kB/s)                                                                                                            
Extracting templates from packages: 100%
Selecting previously unselected package binutils-common:arm64.
(Reading database ... 79238 files and directories currently installed.)
Preparing to unpack .../00-binutils-common_2.43.1-4ubuntu1_arm64.deb ...
Unpacking binutils-common:arm64 (2.43.1-4ubuntu1) ...
Selecting previously unselected package libsframe1:arm64.
Preparing to unpack .../01-libsframe1_2.43.1-4ubuntu1_arm64.deb ...
Unpacking libsframe1:arm64 (2.43.1-4ubuntu1) ...
Selecting previously unselected package libbinutils:arm64.
Preparing to unpack .../02-libbinutils_2.43.1-4ubuntu1_arm64.deb ...
Unpacking libbinutils:arm64 (2.43.1-4ubuntu1) ...
Selecting previously unselected package libctf-nobfd0:arm64.
Preparing to unpack .../03-libctf-nobfd0_2.43.1-4ubuntu1_arm64.deb ...
Unpacking libctf-nobfd0:arm64 (2.43.1-4ubuntu1) ...
Selecting previously unselected package libctf0:arm64.
Preparing to unpack .../04-libctf0_2.43.1-4ubuntu1_arm64.deb ...
Unpacking libctf0:arm64 (2.43.1-4ubuntu1) ...
Selecting previously unselected package libgprofng0:arm64.
Preparing to unpack .../05-libgprofng0_2.43.1-4ubuntu1_arm64.deb ...
Unpacking libgprofng0:arm64 (2.43.1-4ubuntu1) ...
Selecting previously unselected package binutils-aarch64-linux-gnu.
Preparing to unpack .../06-binutils-aarch64-linux-gnu_2.43.1-4ubuntu1_arm64.deb ...
Unpacking binutils-aarch64-linux-gnu (2.43.1-4ubuntu1) ...
Selecting previously unselected package binutils.
Preparing to unpack .../07-binutils_2.43.1-4ubuntu1_arm64.deb ...
Unpacking binutils (2.43.1-4ubuntu1) ...
Selecting previously unselected package libisl23:arm64.
Preparing to unpack .../08-libisl23_0.27-1_arm64.deb ...
Unpacking libisl23:arm64 (0.27-1) ...
Selecting previously unselected package libmpc3:arm64.
Preparing to unpack .../09-libmpc3_1.3.1-1build2_arm64.deb ...
Unpacking libmpc3:arm64 (1.3.1-1build2) ...
Selecting previously unselected package cpp-14-aarch64-linux-gnu.
Preparing to unpack .../10-cpp-14-aarch64-linux-gnu_14.2.0-4ubuntu2_arm64.deb ...
Unpacking cpp-14-aarch64-linux-gnu (14.2.0-4ubuntu2) ...
Selecting previously unselected package cpp-14.
Preparing to unpack .../11-cpp-14_14.2.0-4ubuntu2_arm64.deb ...
Unpacking cpp-14 (14.2.0-4ubuntu2) ...
Selecting previously unselected package cpp-aarch64-linux-gnu.
Preparing to unpack .../12-cpp-aarch64-linux-gnu_4%3a14.1.0-2ubuntu1_arm64.deb ...
Unpacking cpp-aarch64-linux-gnu (4:14.1.0-2ubuntu1) ...
Selecting previously unselected package cpp.
Preparing to unpack .../13-cpp_4%3a14.1.0-2ubuntu1_arm64.deb ...
Unpacking cpp (4:14.1.0-2ubuntu1) ...
Selecting previously unselected package libcc1-0:arm64.
Preparing to unpack .../14-libcc1-0_14.2.0-4ubuntu2_arm64.deb ...
Unpacking libcc1-0:arm64 (14.2.0-4ubuntu2) ...
Selecting previously unselected package libgomp1:arm64.
Preparing to unpack .../15-libgomp1_14.2.0-4ubuntu2_arm64.deb ...
Unpacking libgomp1:arm64 (14.2.0-4ubuntu2) ...
Selecting previously unselected package libitm1:arm64.
Preparing to unpack .../16-libitm1_14.2.0-4ubuntu2_arm64.deb ...
Unpacking libitm1:arm64 (14.2.0-4ubuntu2) ...
Selecting previously unselected package libasan8:arm64.
Preparing to unpack .../17-libasan8_14.2.0-4ubuntu2_arm64.deb ...
Unpacking libasan8:arm64 (14.2.0-4ubuntu2) ...
Selecting previously unselected package liblsan0:arm64.
Preparing to unpack .../18-liblsan0_14.2.0-4ubuntu2_arm64.deb ...
Unpacking liblsan0:arm64 (14.2.0-4ubuntu2) ...
Selecting previously unselected package libtsan2:arm64.
Preparing to unpack .../19-libtsan2_14.2.0-4ubuntu2_arm64.deb ...
Unpacking libtsan2:arm64 (14.2.0-4ubuntu2) ...
Selecting previously unselected package libubsan1:arm64.
Preparing to unpack .../20-libubsan1_14.2.0-4ubuntu2_arm64.deb ...
Unpacking libubsan1:arm64 (14.2.0-4ubuntu2) ...
Selecting previously unselected package libhwasan0:arm64.
Preparing to unpack .../21-libhwasan0_14.2.0-4ubuntu2_arm64.deb ...
Unpacking libhwasan0:arm64 (14.2.0-4ubuntu2) ...
Selecting previously unselected package libgcc-14-dev:arm64.
Preparing to unpack .../22-libgcc-14-dev_14.2.0-4ubuntu2_arm64.deb ...
Unpacking libgcc-14-dev:arm64 (14.2.0-4ubuntu2) ...
Selecting previously unselected package gcc-14-aarch64-linux-gnu.
Preparing to unpack .../23-gcc-14-aarch64-linux-gnu_14.2.0-4ubuntu2_arm64.deb ...
Unpacking gcc-14-aarch64-linux-gnu (14.2.0-4ubuntu2) ...
Selecting previously unselected package gcc-14.
Preparing to unpack .../24-gcc-14_14.2.0-4ubuntu2_arm64.deb ...
Unpacking gcc-14 (14.2.0-4ubuntu2) ...
Selecting previously unselected package gcc-aarch64-linux-gnu.
Preparing to unpack .../25-gcc-aarch64-linux-gnu_4%3a14.1.0-2ubuntu1_arm64.deb ...
Unpacking gcc-aarch64-linux-gnu (4:14.1.0-2ubuntu1) ...
Selecting previously unselected package gcc.
Preparing to unpack .../26-gcc_4%3a14.1.0-2ubuntu1_arm64.deb ...
Unpacking gcc (4:14.1.0-2ubuntu1) ...
Selecting previously unselected package libstdc++-14-dev:arm64.
Preparing to unpack .../27-libstdc++-14-dev_14.2.0-4ubuntu2_arm64.deb ...
Unpacking libstdc++-14-dev:arm64 (14.2.0-4ubuntu2) ...
Selecting previously unselected package g++-14-aarch64-linux-gnu.
Preparing to unpack .../28-g++-14-aarch64-linux-gnu_14.2.0-4ubuntu2_arm64.deb ...
Unpacking g++-14-aarch64-linux-gnu (14.2.0-4ubuntu2) ...
Selecting previously unselected package g++-14.
Preparing to unpack .../29-g++-14_14.2.0-4ubuntu2_arm64.deb ...
Unpacking g++-14 (14.2.0-4ubuntu2) ...
Selecting previously unselected package g++-aarch64-linux-gnu.
Preparing to unpack .../30-g++-aarch64-linux-gnu_4%3a14.1.0-2ubuntu1_arm64.deb ...
Unpacking g++-aarch64-linux-gnu (4:14.1.0-2ubuntu1) ...
Selecting previously unselected package g++.
Preparing to unpack .../31-g++_4%3a14.1.0-2ubuntu1_arm64.deb ...
Unpacking g++ (4:14.1.0-2ubuntu1) ...
Selecting previously unselected package golang-1.23-doc.
Preparing to unpack .../32-golang-1.23-doc_1.23.2-1_all.deb ...
Unpacking golang-1.23-doc (1.23.2-1) ...
Selecting previously unselected package golang-1.23-src.
Preparing to unpack .../33-golang-1.23-src_1.23.2-1_all.deb ...
Unpacking golang-1.23-src (1.23.2-1) ...
Selecting previously unselected package golang-1.23-go.
Preparing to unpack .../34-golang-1.23-go_1.23.2-1_arm64.deb ...
Unpacking golang-1.23-go (1.23.2-1) ...
Selecting previously unselected package golang-1.23.
Preparing to unpack .../35-golang-1.23_1.23.2-1_all.deb ...
Unpacking golang-1.23 (1.23.2-1) ...
Selecting previously unselected package golang-src.
Preparing to unpack .../36-golang-src_2%3a1.23~1_all.deb ...
Unpacking golang-src (2:1.23~1) ...
Selecting previously unselected package golang-go:arm64.
Preparing to unpack .../37-golang-go_2%3a1.23~1_arm64.deb ...
Unpacking golang-go:arm64 (2:1.23~1) ...
Selecting previously unselected package golang-doc.
Preparing to unpack .../38-golang-doc_2%3a1.23~1_all.deb ...
Unpacking golang-doc (2:1.23~1) ...
Selecting previously unselected package golang:arm64.
Preparing to unpack .../39-golang_2%3a1.23~1_arm64.deb ...
Unpacking golang:arm64 (2:1.23~1) ...
Selecting previously unselected package libpkgconf3:arm64.
Preparing to unpack .../40-libpkgconf3_1.8.1-3ubuntu1_arm64.deb ...
Unpacking libpkgconf3:arm64 (1.8.1-3ubuntu1) ...
Selecting previously unselected package pkgconf-bin.
Preparing to unpack .../41-pkgconf-bin_1.8.1-3ubuntu1_arm64.deb ...
Unpacking pkgconf-bin (1.8.1-3ubuntu1) ...
Selecting previously unselected package pkgconf:arm64.
Preparing to unpack .../42-pkgconf_1.8.1-3ubuntu1_arm64.deb ...
Unpacking pkgconf:arm64 (1.8.1-3ubuntu1) ...
Setting up golang-1.23-doc (1.23.2-1) ...
Setting up binutils-common:arm64 (2.43.1-4ubuntu1) ...
Setting up libctf-nobfd0:arm64 (2.43.1-4ubuntu1) ...
Setting up libgomp1:arm64 (14.2.0-4ubuntu2) ...
Setting up libsframe1:arm64 (2.43.1-4ubuntu1) ...
Setting up libpkgconf3:arm64 (1.8.1-3ubuntu1) ...
Setting up libmpc3:arm64 (1.3.1-1build2) ...
Setting up golang-1.23-src (1.23.2-1) ...
Setting up pkgconf-bin (1.8.1-3ubuntu1) ...
Setting up libubsan1:arm64 (14.2.0-4ubuntu2) ...
Setting up libhwasan0:arm64 (14.2.0-4ubuntu2) ...
Setting up libasan8:arm64 (14.2.0-4ubuntu2) ...
Setting up libtsan2:arm64 (14.2.0-4ubuntu2) ...
Setting up libbinutils:arm64 (2.43.1-4ubuntu1) ...
Setting up libisl23:arm64 (0.27-1) ...
Setting up golang-src (2:1.23~1) ...
Setting up libcc1-0:arm64 (14.2.0-4ubuntu2) ...
Setting up liblsan0:arm64 (14.2.0-4ubuntu2) ...
Setting up libitm1:arm64 (14.2.0-4ubuntu2) ...
Setting up libctf0:arm64 (2.43.1-4ubuntu1) ...
Setting up pkgconf:arm64 (1.8.1-3ubuntu1) ...
Setting up golang-1.23-go (1.23.2-1) ...
Setting up libgprofng0:arm64 (2.43.1-4ubuntu1) ...
Setting up cpp-14-aarch64-linux-gnu (14.2.0-4ubuntu2) ...
Setting up golang-1.23 (1.23.2-1) ...
Setting up libgcc-14-dev:arm64 (14.2.0-4ubuntu2) ...
Setting up libstdc++-14-dev:arm64 (14.2.0-4ubuntu2) ...
Setting up golang-go:arm64 (2:1.23~1) ...
Setting up binutils-aarch64-linux-gnu (2.43.1-4ubuntu1) ...
Setting up binutils (2.43.1-4ubuntu1) ...
Setting up cpp-aarch64-linux-gnu (4:14.1.0-2ubuntu1) ...
Setting up cpp-14 (14.2.0-4ubuntu2) ...
Setting up cpp (4:14.1.0-2ubuntu1) ...
Setting up gcc-14-aarch64-linux-gnu (14.2.0-4ubuntu2) ...
Setting up golang-doc (2:1.23~1) ...
Setting up gcc-aarch64-linux-gnu (4:14.1.0-2ubuntu1) ...
Setting up golang:arm64 (2:1.23~1) ...
Setting up g++-14-aarch64-linux-gnu (14.2.0-4ubuntu2) ...
Setting up gcc-14 (14.2.0-4ubuntu2) ...
Setting up g++-aarch64-linux-gnu (4:14.1.0-2ubuntu1) ...
Setting up g++-14 (14.2.0-4ubuntu2) ...
Setting up gcc (4:14.1.0-2ubuntu1) ...
Setting up g++ (4:14.1.0-2ubuntu1) ...
update-alternatives: using /usr/bin/g++ to provide /usr/bin/c++ (c++) in auto mode
Processing triggers for man-db (2.12.1-3) ...
Processing triggers for libc-bin (2.40-1ubuntu3) ...
Scanning processes...                                                                                                                         
Scanning linux images...                                                                                                                      

Running kernel seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
lima@lima-default:/Users/su3-hokkaido/Github/test_api$ go version
go version go1.23.2 linux/arm64
```


# あとがき

特にコメントすることはないですが、 Lima で goland を動かそうとした時の自分の覚書として残します。

