参考文档
[https://blog.rj-bai.com/post/136.html](https://blog.rj-bai.com/post/136.html)
# 安装软件包
```
环境就是新装 CentOS7.5，使用阿里云的 epel 源和常规源
yum -y install openssl* lzo
yum -y install openvpn easy-rsa
```
# 配置 easy-rsa-3.0
## 复制文件
```
[root@localhost ~]# cp -r /usr/share/easy-rsa/ /etc/openvpn/easy-rsa
[root@localhost ~]# cd /etc/openvpn/easy-rsa/
[root@localhost easy-rsa]# \rm 3 3.0
[root@localhost easy-rsa]# cd 3.0.3/
[root@localhost 3.0.3]# find / -type f -name "vars.example" | xargs -i cp {} . && mv vars.example vars
```
这里说明一下，正常来说 easy-rsa-3.0.3 安装完之后，vars.example 文件在 /usr/share/doc/easy-rsa-3.0.3/ 目录
## 创建一个新的 PKI 和 CA
```
[root@localhost 3.0.3]# pwd
/etc/openvpn/easy-rsa/3.0.3
[root@localhost 3.0.3]# ./easyrsa init-pki  #创建空的pki

Note: using Easy-RSA configuration from: ./vars

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /etc/openvpn/easy-rsa/3.0.3/pki
```
```
[root@localhost 3.0.3]# ./easyrsa build-ca nopass #创建新的CA，不使用密码

Note: using Easy-RSA configuration from: ./vars
Generating a 2048 bit RSA private key
......................+++
................................................+++
writing new private key to '/etc/openvpn/easy-rsa/3.0.3/pki/private/ca.key.pClvaQ1GLD'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]: 回车

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/etc/openvpn/easy-rsa/3.0.3/pki/ca.crt
```
# 创建服务端证书
```
[root@localhost 3.0.3]# ./easyrsa gen-req server nopass

Note: using Easy-RSA configuration from: ./vars
Generating a 2048 bit RSA private key
...........................+++
..............................................................................+++
writing new private key to '/etc/openvpn/easy-rsa/3.0.3/pki/private/server.key.wy7Q0fuG6A'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [server]: 回车

Keypair and certificate request completed. Your files are:
req: /etc/openvpn/easy-rsa/3.0.3/pki/reqs/server.req
key: /etc/openvpn/easy-rsa/3.0.3/pki/private/server.key
```
# 签约服务端证书
```
[root@localhost 3.0.3]# ./easyrsa sign server server

Note: using Easy-RSA configuration from: ./vars

You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a server certificate for 3650 days:

subject=
    commonName                = server


Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
Using configuration from ./openssl-1.0.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'server'
Certificate is to be certified until Apr  7 14:54:08 2028 GMT (3650 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /etc/openvpn/easy-rsa/3.0.3/pki/issued/server.crt
```
# 创建 Diffie-Hellman
```
[root@localhost 3.0.3]# ./easyrsa gen-dh
............................................................
DH parameters of size 2048 created at /etc/openvpn/easy-rsa/3.0.3/pki/dh.pem
```
到这里服务端的证书就创建完了，然后创建客户端的证书
# 创建客户端证书
## 复制文件
```
[root@localhost ~]# cp -r /usr/share/easy-rsa/ /etc/openvpn/client/easy-rsa
[root@localhost ~]# cd /etc/openvpn/client/easy-rsa/
[root@localhost easy-rsa]# \rm 3 3.0
[root@localhost easy-rsa]# cd 3.0.3/
[root@localhost 3.0.3]# find / -type f -name "vars.example" | xargs -i cp {} . && mv vars.example vars
```
## 生成证书
```
[root@localhost 3.0.3]# pwd
/etc/openvpn/client/easy-rsa/3.0.3
[root@localhost 3.0.3]# ./easyrsa init-pki #创建新的pki

Note: using Easy-RSA configuration from: ./vars

init-pki complete; you may now create a CA or requests.
Your newly created PKI dir is: /etc/openvpn/client/easy-rsa/3.0.3/pki
```
```
[root@localhost 3.0.3]# ./easyrsa gen-req dalin nopass  #客户证书名为大林，木有密码

Note: using Easy-RSA configuration from: ./vars
Generating a 2048 bit RSA private key
....................................................+++
............+++
writing new private key to '/etc/openvpn/client/easy-rsa/3.0.3/pki/private/dalin.key.FkrLzXH9Bm'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [dalin]: 回车

Keypair and certificate request completed. Your files are:
req: /etc/openvpn/client/easy-rsa/3.0.3/pki/reqs/dalin.req
key: /etc/openvpn/client/easy-rsa/3.0.3/pki/private/dalin.key
```
## 最后签约客户端证书
```
[root@localhost 3.0.3]# cd /etc/openvpn/easy-rsa/3.0.3/
[root@localhost 3.0.3]# pwd
/etc/openvpn/easy-rsa/3.0.3
[root@localhost 3.0.3]# ./easyrsa import-req /etc/openvpn/client/easy-rsa/3.0.3/pki/reqs/dalin.req dalin

Note: using Easy-RSA configuration from: ./vars

The request has been successfully imported with a short name of: dalin
You may now use this name to perform signing operations on this request.
```
```
[root@localhost 3.0.3]# ./easyrsa sign client dalin

Note: using Easy-RSA configuration from: ./vars

You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.

Request subject, to be signed as a client certificate for 3650 days:

subject=
    commonName                = dalin

Type the word 'yes' to continue, or any other input to abort.
  Confirm request details: yes
Using configuration from ./openssl-1.0.cnf
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
commonName            :ASN.1 12:'dalin'
Certificate is to be certified until Apr  8 01:54:57 2028 GMT (3650 days)

Write out database with 1 new entries
Data Base Updated

Certificate created at: /etc/openvpn/easy-rsa/3.0.3/pki/issued/dalin.crt
```
# 整理证书
现在所有的证书都已经生成完了，整理一下
## 服务端所需要的文件
```
[root@localhost ~]# mkdir /etc/openvpn/certs
[root@localhost ~]# cd /etc/openvpn/certs/
[root@localhost certs]# cp /etc/openvpn/easy-rsa/3.0.3/pki/dh.pem .
[root@localhost certs]# cp /etc/openvpn/easy-rsa/3.0.3/pki/ca.crt .
[root@localhost certs]# cp /etc/openvpn/easy-rsa/3.0.3/pki/issued/server.crt .
[root@localhost certs]# cp /etc/openvpn/easy-rsa/3.0.3/pki/private/server.key .
[root@localhost certs]# ll
总用量 20
-rw-------. 1 root root 1172 4月  11 10:02 ca.crt
-rw-------. 1 root root  424 4月  11 10:03 dh.pem
-rw-------. 1 root root 4547 4月  11 10:03 server.crt
-rw-------. 1 root root 1704 4月  11 10:02 server.key
```
## 客户端所需的文件
```
[root@localhost certs]# mkdir /etc/openvpn/client/dalin/
[root@localhost certs]# cp /etc/openvpn/easy-rsa/3.0.3/pki/ca.crt /etc/openvpn/client/dalin/
[root@localhost certs]# cp /etc/openvpn/easy-rsa/3.0.3/pki/issued/dalin.crt /etc/openvpn/client/dalin/
[root@localhost certs]# cp /etc/openvpn/client/easy-rsa/3.0.3/pki/private/dalin.key /etc/openvpn/client/dalin/
[root@localhost certs]# ll /etc/openvpn/client/dalin/
总用量 16
-rw-------. 1 root root 1172 4月  11 10:07 ca.crt
-rw-------. 1 root root 4431 4月  11 10:08 dalin.crt
-rw-------. 1 root root 1704 4月  11 10:08 dalin.key
```
# 服务端配置文件
```
vim /etc/openvpn/server.conf
port 1194
proto tcp
dev tun
ca /etc/openvpn/certs/ca.crt
cert /etc/openvpn/certs/server.crt
key /etc/openvpn/certs/server.key
dh /etc/openvpn/certs/dh.pem
ifconfig-pool-persist /etc/openvpn/ipp.txt
server 10.8.0.0 255.255.255.0
push "route 172.16.1.0 255.255.255.0"
push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 223.5.5.5"
push "dhcp-option DNS 223.6.6.6"
client-to-client
duplicate-cn
keepalive 20 120
comp-lzo
user openvpn
group openvpn
persist-key
persist-tun
status openvpn-status.log
log-append openvpn.log
verb 1
mute 20
```
# 开启路由转发功能
```
vim /etc/sysctl.conf
net.ipv4.ip_forward = 1
```
```
sysctl -p
```
# 配置防火墙
```
iptables -I INPUT -p tcp --dport 1194 -m comment --comment "openvpn" -j ACCEPT
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -j MASQUERADE
```
# 启动
```
/usr/sbin/openvpn --config /etc/openvpn/server.conf &
```
# 客户端配置文件
```
vim client.ovpn
client
dev tun
proto tcp
remote 10.0.0.61 1194
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt
cert dalin.crt
key dalin.key
comp-lzo
verb 1
```
# 客户端证书及配置文件
- ca.crt
- dalin.crt
- dalin.key
- client.ovpn




# linux客户端连接
```
openvpn  /etc/openvpn/a.openvpn >/dev/null &
```
