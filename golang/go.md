## Install
```bash
yum intall -y wget
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ivh epel-release-latest-7.noarch.rpm
yum install golang
go env -w GOPROXY=https://mirrors.aliyun.com/goproxy,https://goproxy.cn,https://goproxy.io,direct
```