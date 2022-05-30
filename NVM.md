# NVM入门使用

# 介绍

[Github地址]([nvm-sh/nvm: Node Version Manager - POSIX-compliant bash script to manage multiple active node.js versions (github.com)](https://github.com/nvm-sh/nvm))

`nvm`用于管理不同版本的nodejs，可以通过命令行进行安装和版本切换。

## 安装

### Windows安装

github下载[nvm for windows](https://github.com/coreybutler/nvm-windows/releases)，安装路径建议不要有空格和中文，可能会有一些奇怪的问题。

安装完毕后可通过命令`nvm version`查看是否安装成功。

### Linux安装

具体可参考[github]([nvm-sh/nvm: Node Version Manager - POSIX-compliant bash script to manage multiple active node.js versions (github.com)](https://github.com/nvm-sh/nvm))

脚本安装

```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

或者

```shell
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

## NVM使用

```shell
# 查看版本
nvm version

# 查看已安装的node，等同于命令nvm ls、nvm list installed
nvm list

# 查看可安装的node
nvm list available
|   CURRENT    |     LTS      |  OLD STABLE  | OLD UNSTABLE |
|--------------|--------------|--------------|--------------|
|    18.1.0    |   16.15.0    |   0.12.18    |   0.11.16    |
|    18.0.0    |   16.14.2    |   0.12.17    |   0.11.15    |
|    17.9.0    |   16.14.1    |   0.12.16    |   0.11.14    |
|    17.8.0    |   16.14.0    |   0.12.15    |   0.11.13    |
|    17.7.2    |   16.13.2    |   0.12.14    |   0.11.12    |
|    17.7.1    |   16.13.1    |   0.12.13    |   0.11.11    |
|    17.7.0    |   16.13.0    |   0.12.12    |   0.11.10    |
|    17.6.0    |   14.19.2    |   0.12.11    |    0.11.9    |
|    17.5.0    |   14.19.1    |   0.12.10    |    0.11.8    |
|    17.4.0    |   14.19.0    |    0.12.9    |    0.11.7    |
|    17.3.1    |   14.18.3    |    0.12.8    |    0.11.6    |
|    17.3.0    |   14.18.2    |    0.12.7    |    0.11.5    |
|    17.2.0    |   14.18.1    |    0.12.6    |    0.11.4    |
|    17.1.0    |   14.18.0    |    0.12.5    |    0.11.3    |
|    17.0.1    |   14.17.6    |    0.12.4    |    0.11.2    |
|    17.0.0    |   14.17.5    |    0.12.3    |    0.11.1    |
|   16.12.0    |   14.17.4    |    0.12.2    |    0.11.0    |
|   16.11.1    |   14.17.3    |    0.12.1    |    0.9.12    |
|   16.11.0    |   14.17.2    |    0.12.0    |    0.9.11    |
|   16.10.0    |   14.17.1    |   0.10.48    |    0.9.10    |

# 安装最新的node
nvm install latest

# 安装指定版本
nvm install 17.8.0

# 使用指定版本的node，例如使用17.8.0
nvm use 17.8.0

# 使用指定版本的node，例如使用最新的
nvm use latest

# 查看当前使用的node版本
nvm current
Now using node v18.1.0 (64-bit)

# 查看已安装的node，其中有带*的表明是当前使用的版本
nvm list
  * 18.1.0 (Currently using 64-bit executable)
    17.8.0

# 开启nvm node版本管理
nvm on

# 关闭nvm node版本管理，关闭时可能会导致node环境无法使用的问题
nvm off

# 卸载指定版本的node，例如下载17.8.0版本的node
nvm uninstall 17.8.0
Uninstalling node v17.8.0... done
```

如果下载node过慢，可设置国内镜像源，在nvm的安装路径下，找到settings.txt，设置node下载镜像地址和npm下载镜像地址

node_mirror: https://npm.taobao.org/mirrors/node/
npm_mirror: https://npm.taobao.org/mirrors/npm/

也可通过命令行修改镜像地址

```shell
# 设置node下载镜像
nvm node_mirror https://npm.taobao.org/mirror/node/

# 设置npm下载镜像
nvm npm_mirror https://npm.taobao.org/mirror/npm/
```

