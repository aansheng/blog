# 防止SSH会话超时

通过指定时间间隔在客户端和服务器之间发送`空数据包`，可以避免SSH超时。

# 防止SSH客户端超时

如果你使用的是`Mac`或`Linux`，则可以编辑用户目录下的`~/.ssh/config`并添加以下行:

```bash
ServerAliveInterval 120
```

这将在您的SSH连接上每`120秒`发送一个`空数据包`以使它们保持活动状态。

# 防止SSH服务端超时

更改服务器上`/etc/ssh/sshd_config`的SSH配置文件，以防止客户端超时，因此不必修改SSH客户端配置：

```bash
ClientAliveInterval 120  // 超时时间，10s
ClientAliveCountMax 720  // 超时次数，0次
```

如果客户端处于非活动状态120秒，这将使服务器向客户端发送一个空数据包，共发送720次，如果服务端向客户端发送消息达到此阈值，sshd将断开客户端的连接，所以`timeout interval = ClientAliveInterval * ClientAliveCountMax`

以上的两种方法设置哪一个都可以。

# 参考文献

1. [SSH timeout prevention – keep SSH sessions alive](https://bjornjohansen.no/ssh-timeout)