# Nginx隐藏版本号信息

当我们通过yum或者其他包管理工具安装完Nginx之后，访问页面时Header里面会携带Nginx的版本号信息

```bash
$ curl -I http://localhost
HTTP/1.1 200 OK
Server: nginx/1.16.1  # 这里就是Nginx版本号
Date: Fri, 23 Aug 2019 08:54:07 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 13 Aug 2019 15:04:31 GMT
Connection: keep-alive
ETag: "5d52d17f-264"
Accept-Ranges: bytes
```

这是一个小小的安全隐患，所以这个时候我们需要进行版本号的隐藏，打开nginx的配置文件，然后在http段中添加以下配置

```bash
$ sudo vi /etc/nginx/nginx.conf
http {
    ......
    server_tokens off;
    ......
}
```

保存之后我们对nginx进行reload让其生效配置

```bash
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
$ sudo nginx -s reload
```

然后再次通过curl指令进行访问

```bash
$ curl -I http://localhost
HTTP/1.1 200 OK
Server: nginx  # Nginx版本号已隐藏
Date: Fri, 23 Aug 2019 09:02:18 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 13 Aug 2019 15:04:31 GMT
Connection: keep-alive
ETag: "5d52d17f-264"
Accept-Ranges: bytes
```

此时发现Nginx的版本号已经完全隐藏了。

## 参考文献

- [Module ngx_http_core_module](http://nginx.org/en/docs/http/ngx_http_core_module.html#server_tokens)
