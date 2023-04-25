# macOS服务管理brew services用法

`macOS`中的[brew services](https://github.com/Homebrew/homebrew-services)类似于`CentOS 7`下的`systemctl`，主要是用来管理服务的一些操作。

# 基本操作

下面的操作以nginx为例

- 安装

```bash
$ brew install nginx
```

- 卸载

```bash
$ brew uninstall nginx
```

- 更新

```bash
$ brew upgrade nginx
```

- 重新安装

```bash
$ brew reinstall nginx
```

- 列出当前所有的服务

```bash
$ brew services list
```

- 运行服务而不设置开机自启动

```bash
$ brew services run nginx
```

- 启动服务并注册开机自启动

```bash
$ brew services start nginx
```

- 停止，并取消开机自启动

```bash
$ brew services stop nginx
```

- 重启，并且注册开机自启

```bash
$ brew services restart nginx
```

- 清理残留的旧版本及相关日志

```bash
$ brew services cleanup
```

# 注册服务

注册开机自启后，会创建.plist文件，该文件包含版本信息、编码、安装路径、启动位置、日志路径等信息，取消自启动后会自动删除，执行 brew services list 可以看到各个服务该文件的存放位置

## .plist存放目录

- 开机自启存放目录

```bash
/Library/LaunchDaemons/
```

- 用户登录后自启存放目录

```bash
~/Library/LaunchDaemons/
```