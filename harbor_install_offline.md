# harbor离线安装



## 环境要求

- 硬件

| 资源 | 最小值 | 建议值 |
| :--: | :----: | :----: |
| CPU  |  2个   |  4个   |
| 内存 |   4G   |   8G   |
| 硬盘 |  40G   |  160G  |

- 软件

  -  docker
-  docker compose
  -  openssl
  



## 下载离线安装包

到github下载[离线安装包](https://github.com/goharbor/harbor/releases/download/v1.10.10/harbor-offline-installer-v1.10.10.tgz)文件

```sh
# 将压缩包拷贝至/opt目录下
# cp harbor-offline-installer-v1.10.10.tgz /opt
# 解压
# tar xzvf harbor-offline-installer-v1.10.10.tgz
```



## 配置https

有证书和域名直接使用现成的即可，若无可参考如下部分使用自签的证书和域名进行配置。

以下以使用域名myharbor.com（192.168.107.128）为例。

### 生成ca证书

1. 生成ca私钥（ca.key文件）

```sh
# openssl genrsa -out ca.key 4096
```

2. 生成根证书（ca.crt文件）

```sh
# openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=CN/ST=Fujian/L=Fuzhou/O=organizationname/OU=customssl/CN=myharbor.com" \
 -key ca.key \
 -out ca.crt
```

其中-subj的个字段含义为：

- C（Country）：所在国家，例如中国：CN
- ST（State/Province）：所在大州或者省份，例如福建：Fujian
- L（Lacality）：所在地区或者城市，例如福州：Fuzhou
- O（Organization Name）：单位名称；对于SSL证书一般为网站域名，对于代码签名证书则为申请单位名称，对于客户端单位证书则为证书申请者所在单位名称
- OU：显示其他内容
- CN（Common Name）：公用名称；对于SSL证书一般为网站域名，对于代码签名证书则为申请单位名称，对于客户端证书则为证书申请者的姓名。



### 生成服务端证书

1. 生成服务端私钥（myharbor.com.key文件）

```sh
# openssl genrsa -out myharbor.com.key 4096
```

2. 生成服务端证书签名请求（myharbor.com.csr文件）

```sh
# openssl req -sha512 -new \
    -subj "/C=CN/ST=Fujian/L=Fuzhou/O=organizationname/OU=customssl/CN=myharbor.com" \
    -key myharbor.com.key \
    -out myharbor.com.csr
```

3. 生成X.509 v3扩展文件（v3.ext文件）

```sh
# cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=myharbor.com
DNS.2=hostname
EOF
```

4. 生成服务端证书（myharbor.com.crt文件）

```sh
# openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in myharbor.com.csr \
    -out myharbor.com.crt
```

5. 生成客户端证书（myharbor.com.cert文件）

```sh
# openssl x509 -inform PEM -in myharbor.com.crt -out myharbor.com.cert
```



### 配置证书

1. 将服务端证书拷贝至指定目录，此处以目录`/data/harbor/cert`为例

```sh
# cp myharbor.com.crt /data/harbor/cert/
# cp myharbor.com.key /data/harbor/cert/
```

2. 添加配置文件

```sh
# cp harbor.yml.tmpl harbor.yml
```

3. 修改配置文件`harbor.yml`，配置项简单说明，其他详细配置项[查看官网]([Harbor docs | Configure the Harbor YML File (goharbor.io)](https://goharbor.io/docs/2.4.0/install-config/configure-yml-file/))

| key                   | value                              | description                                     |
| --------------------- | ---------------------------------- | ----------------------------------------------- |
| hostname              | myharbor.com                       | 填写ip地址或hostname用于访问hargor ui和镜像仓库 |
| https.certificate     | /data/harbor/cert/myharbor.com.crt | 服务端证书                                      |
| https.private_key     | /data/harbor/cert/myharbor.com.key | 服务端私钥                                      |
| harbor_admin_password | Admin@1024                         | admin账号的密码，默认值为Harbor12345，建议修改  |
| database.password     | Rootdb@1024                        | 数据库密码，默认值为root123，建议修改           |
| data_volume           | /data/harbor                       | 数据卷                                          |

4. 执行`prepare`脚本文件，生成相应的https配置信息。由于harbor使用nginx示例作为所有服务的反代，所以需要执行该脚本进行配置nginx来使用https。

```sh
# ./prepare
```



## 安装harbor

执行`install.sh`命令安装并运行harbor，执行该脚本会拉取相关的镜像。之后若需修改配置须使用`docker-compose stop -v`停止服务，并再次`docker-compose up -d`来启动服务，无需执行`install.sh`脚本文件。

```sh
# ./install.sh
```



## 配置docker客户端证书

在需要访问该私有镜像仓库的docker上配置客户端证书。

1. 将根证书、服务端私钥和客户端证书拷贝至`/etc/docker/certs.d/myharbor.com`目录下，其中`myharbor.com`目录是对应私有镜像仓库的地址。

```sh
# mkdir -p /etc/docker/certs.d/myharbor.com
# cp yourdomain.com.cert /etc/docker/certs.d/myharbor.com/
# cp yourdomain.com.key /etc/docker/certs.d/myharbor.com/
# cp ca.crt /etc/docker/certs.d/myharbor.com/
```

2. 将与域名添加进`/etc/hosts`

```sh
# echo "192.168.107.128 myharbor.com" >> /etc/hosts
```

配置完变可使用`docker login`命令进行登录操作，若提示证书非法，可重启docker服务或者检查证书配置是否正确。

登录成功可在`~/.docker/config.json`文件中查看到对应的信息

```json
{
	"auths": {
		"myharbor.com": {
			"auth": "YWRtaW46SGFyYm9yMTIzNDU="
		}
	}
}
```



## 配置docker客户端（未使用http的harbor）

若harbor未使用，则dockers客户端需要在配置文件`/etc/docker/daemon.json`中添加insecure-registry，因为默认dockers推送都是基于http的。

```json
{
  "insecure-registry":["192.168.107.128"]
}
```



## docker客户端推送镜像

首先通过浏览器访问harbor，添加项目myproject。然后在dockers客户端中推送一个示例redis。

```sh
# 先拉取一个redis来使用
# docker pull redis
# 添加tag，添加一个v1的tag
# docker tag redis:latest myharbor.com/myproject/redis:v1
# 此处假设已登录
# docker push myharbor.com/myproject/redis:v1
# 删除v1的tag
# docker rmi myharbor.com/myproject/redis:v1
```

此时便可在harbor的UI中查看到myproject项目下有个redis镜像（tag为v1）。



## 配置harbor服务为systemd的service

虽然`/opt/harbor/docker-compose.yml`中已经声明了容器自动重启的规则为always，但由于harbor的服务是有依赖关系的，特别是log容器是被其他容器所依赖的，而容器自动重启是不会考虑这些依赖的先后关系的，所以需要

```sh
# cat > /usr/lib/systemd/system/harbor.service <<-EOF
[Unit]
Description=Harbor
After=docker.service systemd-networkd.service systemd-resolved.service
Requires=docker.service
Documentation=http://github.com/vmware/harbor

[Service]
Type=simple
Restart=on-failure
RestartSec=5
ExecStart=/usr/local/bin/docker-compose -f /opt/harbor/docker-compose.yml up
ExecStop=/usr/local/bin/docker-compose -f /opt/harbor/docker-compose.yml down

[Install]
WantedBy=multi-user.target
EOF
# systemctl enable harbor
```

