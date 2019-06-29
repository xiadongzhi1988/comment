# yum客户端
## 安装yum源优先级插件
```
yum install yum-plugin-priorities.noarch
```
## 启用插件
cat /etc/yum/pluginconf.d/priorities.conf
```
[main]
enabled = 1
```
## 修改yum源优先级
cat CentOS-Media.repo
```
[c7-media]
name=CentOS-$releasever - Media
baseurl=file:///media/cdrom/
gpgcheck=0
enabled=1
priority=1
```
cat my-base.repo
```
[my-base]
name=Server
baseurl=http://10.0.0.61
enable=1
gpgcheck=0
priority=1
```
CentOS-Base.repo
```
[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
priority=2
#released updates 
[updates]
name=CentOS-$releasever - Updates
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
priority=2
#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
priority=2
#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
priority=2
#contrib - packages by Centos Users
[contrib]
name=CentOS-$releasever - Contrib
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/contrib/$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
priority=2
```
**priority  数字越小优先级越高**
# yum仓库搭建
## 安装createrepo
```
yum install createrepo
```
## 创建yum仓库目录，上传rpm包到yum仓库目录
```
mkdir -p /application/yum/centos6/x86_64/
cd /application/yum/centos6/x86_64/

cp *.rpm /application/yum/centos6/x86_64/

yumdownloader 下载rpm包
yumdownloader httpd
```
## 初始化yum仓库
```
[root@m01 x86_64]# createrepo  /application/yum/centos6/x86_64/
Spawning worker 0 with 3 pkgs
Workers Finished
Gathering worker results

Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Sqlite DBs complete
```
## 每次加入新的rpm包，更新yum仓库
```
createrepo --update /application/yum/centos6/x86_64/
```



 
