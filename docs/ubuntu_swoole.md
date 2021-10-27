# Hyperf 框架中添加 Swoole 扩展源码安装

### 官方编译安装教程 [点击](https://wiki.swoole.com/#/environment)

### 进入 `swoole` 的 Github 版本列表,选择最新版本的压缩包下载

<br>

[swoole 版本列表](https://github.com/swoole/swoole-src/releases)

<br>


```

$ wget https://github.com/swoole/swoole-src/archive/refs/tags/v4.8.0.zip

```


### 下载完成后进行操作

```

# 如果没有unzip 的话
$ apt-get unzip

#之后解压下载的压缩包
$ unzip v4.8.0.zip


# 进入目录
$ cd v4.8.0.zip

# 编译
$ phpize

# ubuntu 没有安装 phpize 可执行命令下面的这个来安装 phpize
$ sudo apt-get install php-dev 

# 编译配置检测
$ ./configure --enable-openssl --enable-http2 --enable-swoole-curl --enable-swoole-json

# 编译
$ make

# 安装
$ make install

#查看扩展
$ php --ri swoole

```

### 如果显示以下代码 正确

```
Swoole => enabled
Author => Swoole Team <team@swoole.com>
Version => 4.8.0
Built => Oct 15 2021 09:34:32
coroutine => enabled with boost asm context
epoll => enabled
eventfd => enabled
signalfd => enabled
cpu_affinity => enabled
spinlock => enabled
rwlock => enabled
openssl => OpenSSL 1.1.1f  31 Mar 2020
dtls => enabled
http2 => enabled
json => enabled
curl-native => enabled
zlib => 1.2.11
mutex_timedlock => enabled
pthread_barrier => enabled
futex => enabled
async_redis => enabled

Directive => Local Value => Master Value
swoole.enable_coroutine => On => On
swoole.enable_library => On => On
swoole.enable_preemptive_scheduler => Off => Off
swoole.display_errors => On => On
swoole.use_shortname => Off => Off
swoole.unixsock_buffer_size => 8388608 => 8388608

```