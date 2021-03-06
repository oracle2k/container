setup gvisor on ubuntu
===========

AMI
---------
http://cloud-images.ubuntu.com/locator/ec2/

`ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-20180426.2 - ami-f3211396`

install docker-ce =>18
----------
```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic test"
sudo apt update
sudo apt install -y docker-ce
```

```console
$ sudo docker version

Client:
 Version:      18.05.0-ce-rc1
 API version:  1.37
 Go version:   go1.9.5
 Git commit:   33f00ce
 Built:        Thu Apr 26 01:03:26 2018
 OS/Arch:      linux/amd64
 Experimental: false
 Orchestrator: swarm

Server:
 Engine:
  Version:      18.05.0-ce-rc1
  API version:  1.37 (minimum version 1.12)
  Go version:   go1.9.5
  Git commit:   33f00ce
  Built:        Thu Apr 26 01:01:34 2018
  OS/Arch:      linux/amd64
  Experimental: false
```

install bazel
----------
https://docs.bazel.build/versions/master/install-ubuntu.html
```
sudo apt-get install -y pkg-config zip g++ zlib1g-dev unzip python
curl -SsL -O https://github.com/bazelbuild/bazel/releases/download/0.13.0/bazel-0.13.0-installer-linux-x86_64.sh
chmod +x !$:t
./!$ --user
```
```
export PATH="$PATH:$HOME/bin"
```
```console
$ bazel version
Build label: 0.13.0
Build target: bazel-out/k8-opt/bin/src/main/java/com/google/devtools/build/lib/bazel/BazelServer_deploy.jar
Build time: Mon Oct 18 21:33:40 +50297 (1525078013620)
Build timestamp: 1525078013620
Build timestamp as int: 1525078013620
```

build gvisor
-------
```
sudo apt-get install -y git
```
```
git clone https://gvisor.googlesource.com/gvisor gvisor
cd !$
```

```console
$ bazel build runsc
Starting local Bazel server and connecting to it...
...........
INFO: Analysed target //runsc:runsc (173 packages loaded).
INFO: Found 1 target...
Target //runsc:runsc up-to-date:
  bazel-bin/runsc/linux_amd64_pure_stripped/runsc
INFO: Elapsed time: 234.541s, Critical Path: 48.53s
INFO: 648 processes: 638 linux-sandbox, 10 local.
INFO: Build completed successfully, 664 total actions
```

config
-------
```console
$ sudo sh -c 'cat <<EOF > /etc/docker/daemon.json
{
    "runtimes": {
        "runsc": {
            "path": "/usr/local/bin/runsc"
        }
    }
}
EOF
'
```

```console
$ ps aux|grep [d]ocker
root      3448  0.1  1.5 720008 63760 ?        Ssl  07:55   0:01 /usr/bin/dockerd -H fd://
root      3465  0.3  0.5 555016 24136 ?        Ssl  07:55   0:03 docker-containerd --config /var/run/docker/containerd/containerd.toml
```

```
sudo systemctl daemon-reload
sudo systemctl restart docker
```

```console
$ sudo docker run --rm --runtime=runsc -it ubuntu /bin/bash
root@5885ce2b72c6:/# echo hello
hello
root@5885ce2b72c6:/#
```


debug
------------
```console
$ sudo docker run --rm --runtime=runsc hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```

```console
$ ls -lah /tmp/runsc/
total 88K
drwxr-xr-x  2 root root 4.0K May  8 10:01 .
drwxrwxrwt 13 root root 4.0K May  8 10:02 ..
-rw-r--r--  1 root root 6.2K May  8 09:39 runsc.log.20180508-093940.669159.create
-rw-r--r--  1 root root  15K May  8 09:39 runsc.log.20180508-093940.674994.gofer
-rw-r--r--  1 root root  27K May  8 09:39 runsc.log.20180508-093940.679256.boot
-rw-r--r--  1 root root 1.9K May  8 09:39 runsc.log.20180508-093940.692328.state
-rw-r--r--  1 root root 3.1K May  8 09:39 runsc.log.20180508-093940.697848.start
-rw-r--r--  1 root root 1.9K May  8 09:39 runsc.log.20180508-093940.825924.state
-rw-r--r--  1 root root 1.6K May  8 09:39 runsc.log.20180508-093940.831826.kill
-rw-r--r--  1 root root 1.9K May  8 09:39 runsc.log.20180508-093940.847808.state
-rw-r--r--  1 root root 1.6K May  8 09:39 runsc.log.20180508-093940.872758.kill
-rw-r--r--  1 root root 1.6K May  8 09:39 runsc.log.20180508-093940.877457.delete
```
