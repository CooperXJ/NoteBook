### 安装 Go 语言环境

导出 Ceph 信息到 Prometheus 有多种方式，本文采用的是 DigitalOcean 的 ceph_exporter，ceph_exporter 使用 go 语言编写的，所以需要先安装 go 语言环境。还是一条命令解决：

```bash
$ sudo apt install -y golang
```

安装好后执行 `$ go env` 命令验证并查看一下 go 环境信息。

<font color=red>此处需要注意一下GOROOT</font>，有可能不是/usr/lib/go-1.6，需要在/usr/lib下查看当前go库的名字，

<img src="https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201023172011.png" alt="image-20201023171958553" style="zoom:50%;" />

```bash
$ go env
GOARCH="amd64"
GOBIN=""
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOOS="linux"
GOPATH=""
GORACE=""
GOROOT="/usr/lib/go-1.6"
GOTOOLDIR="/usr/lib/go-1.6/pkg/tool/linux_amd64"
GO15VENDOREXPERIMENT="1"
CC="gcc"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0"
CXX="g++"
CGO_ENABLED="1"
```

然后需要设置 Go 环境变量：

```bash
$ cat /etc/profile.d/go.sh 
export GOROOT=/usr/lib/go-1.6
export GOBIN=$GOROOT/bin
export GOPATH=/home/<user-name>/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

$ source /etc/profile.d/go.sh
```

已经配置好 Go 环境了，接下来创建 **GOPATH** 指定的目录：

```bash
$ mkdir /home/<user-name>/go
```

### 2.2 安装 ceph_exporter

Go 环境安装好后，我们接下来下载 ceph_exporter 代码，然后编译出可执行程序。

```bash
$ mkdir -p /home/<user-name>/go/src/github.com/digitalocean
$ cd /home/<user-name>/go/github.com/src/digitalocean
$ git clone https://github.com/digitalocean/ceph_exporter
$ cd ceph_exporter
$ go build
```

这时编译会报错，原因是需要依赖 ceph rados 相关的头文件，需要安装 librados-dev 包。

此处的librados-dev需要<font color=red>根据具体</font>的平台进行调整

![image-20201023172114718](https://raw.githubusercontent.com/CooperXJ/ImageBed/master/img/20201023172116.png)

```bash
$ sudo apt install -y librados-dev
```

安装好后，在编译，复制可执行文件到对应目录完成安装。
 再运行 `go build` 完成安装。

```bash
$ go get
$ go build 
$ mkdir /home/<user-name>/go/bin
$ cp ceph_exporter /home/<user-name>/go/bin
```

执行 ceph_exporter 来验证一下是否可以正常使用

```bash
$ ceph_exporter --help
Usage of ceph_exporter:
  -ceph.config string
        path to ceph config file
  -ceph.user string
        Ceph user to connect to cluster. (default "admin")
  -exporter.config string
        Path to ceph exporter config. (default "/etc/ceph/exporter.yml")
  -rgw.mode int
        Enable collection of stats from RGW (0:disabled 1:enabled 2:background)
  -telemetry.addr string
        host:port for ceph exporter (default ":9128")
  -telemetry.path string
        URL path for surfacing collected metrics (default "/metrics")
```

接下来要配置 ceph_exporter 的自动启动：

<font color=red>注意< user-name >是什么可能需要修改</font>

```xml
$ cat /lib/systemd/system/ceph_exporter.service 
[Unit]
Description=Prometheus's ceph metrics exporter
After=prometheus.ervice

[Service]
ExecStart=/home/<user-name>/go/bin/ceph_exporter

[Install]
WantedBy=multi-user.target
Alias=ceph_exporter.service

$ sudo systemctl daemon-reload
$ sudo systemctl enable ceph_exporter.service
$ sudo systemctl start ceph_exporter.service
```

### 2.3 修改 Promethues 配置

接下来需要修改 Prometheus 的配置，添加一会要安装的 ceph_exporter 的相关信息：

```bash
$ sudo vim /etc/prometheus/prometheus.yml
...
scrape_configs:
  - job_name: 'ceph_exporter'
    static_configs:
    - targets: ['localhost:9128']
      labels:
        alias: ceph_exporter
...
```

改好后，重启：

```ruby
$ sudo systemctl restart prometheus.service
```