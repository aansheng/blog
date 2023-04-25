# 群晖NAS使用Docker部署qBittorrent下载工具

## 下载Docker镜像

- 安装Docker

打开`套件中心`-->搜索`Docker`，如果没有安装则进行安装，我这里已经安装过了

![Untitled](/images/2021/11/25/1.png)

- 下载镜像

打开Docker的控制台，找到左侧栏的注册表，然后搜索qBittorrent，我们需要下载`linuxserver/qbittorrent`这个版本的镜像，标签选择`latest`，然后点击下载

![Untitled](/images/2021/11/25/2.png)

如果你发现下载的比较慢，可以打开群晖的SSH登录功能，在`控制面板`-->`终端机和SNMP`-->`启用SSH功能并应用`，在连接上服务器之后通过下面的指令手动下载，可能效率会高些

```
docker pull linuxserver/qbittorrent:latest

```

## 权限配置

下面我们需要为qBittorrent创建配置文件的存放目录，打开`File Station`，找到左侧栏的`Docker`文件夹，点击新增文件夹，命名`qbittorrent`

![Untitled](/images/2021/11/25/3.png)

然后双击进入`qbittorrent`文件夹内，在创建一个`config`的文件夹

![Untitled](/images/2021/11/25/4.png)

配置文件创建完成之后我们需要更改下文件夹的权限，选中`qbittorrent`文件夹之后`右键属性`-->`权限`-->`新增`

![Untitled](/images/2021/11/25/5.png)

![Untitled](/images/2021/11/25/6.png)

因为我的所有影视资源都放在`/Media`目录下，所以需要针对`/Media`文件夹做同样的操作

![Untitled](/images/2021/11/25/7.png)

## 运行

继续回到我们的Docker控制台，找到对应的镜像然后点击启动

![Untitled](/images/2021/11/25/8.png)

容器名称自定义设置，我这里用的`qbittorrent`，然后点击高级设置

![Untitled](/images/2021/11/25/9.png)

- 高级设置

`启用自动重新启动`勾选上，NAS如果重新启动，容器也会启动防止掉线

![Untitled](/images/2021/11/25/10.png)

- 存储空间

我这里映射了两个文件夹，一个是qbittorrent的配置文件，一个是媒体文件夹

![Untitled](/images/2021/11/25/11.png)

- 网络

勾选`使用与 Docker Host 相同的网络`，这样就不需要做什么端口映射了，启动的端口会直接运行在NAS主机上

![Untitled](/images/2021/11/25/12.png)

- 环境

我们增加两个环境变量

```
TZ=Asia/Shanghai  // 时区
WEBUI_PORT=18080  // qbittorrent端口

```

然后点击应用

![Untitled](/images/2021/11/25/13.png)

下一步

![Untitled](/images/2021/11/25/14.png)

确定没问题之后点击完成，容器会自动运行

![Untitled](/images/2021/11/25/15.png)

![Untitled](/images/2021/11/25/16.png)

回到容器列表发现已经开始运行了

![Untitled](/images/2021/11/25/17.png)

然后我们浏览器打开NASIP地址+端口，比如`http://192.168.1.10:18080`，输入默认的用户名`admin`和密码`adminadmin`，就可以登录了

![Untitled](/images/2021/11/25/18.png)

## QT的一些配置

- 中文设置

`Tools`-->`Options`-->`Web UI`-->`Language`选择`简体中文`-->拉到最下面`Save`

![Untitled](/images/2021/11/25/19.png)

- 修改默认的保存路径为`/Media`

工具-->选项-->下载-->默认保存路径：/Media/-->保存

![Untitled](/images/2021/11/25/20.png)

- 添加自定义 Tracker

工具-->选项-->下载-->BitTorrent-->勾选`自动添加以下 tracker 到新的 torrent：`将`https://raw.githubusercontent.com/ngosang/trackerslist/master/trackers_best_ip.txt`整理好的trackers_best_ip全部复制进去，然后点击保存即可

![Untitled](/images/2021/11/25/21.png)

然后再到`高级`选项，选中`总是向同级的所有 Tracker 汇报`，据说对提速也有帮助

![Untitled](/images/2021/11/25/22.png)

刚添加了几个任务，虽然速度没有跑满，但是目前这速度还可以了，后续有什么优化点，我再来补充文章

![Untitled](/images/2021/11/25/23.png)