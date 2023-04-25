# 使用curl命令获取网站的header头及状态码

curl命令最常用的参数就是`-I`，仅返回头部信息，使用HEAD请求，获取的结果如下：

```bash
[root@ansheng ~]# curl -I www.ansheng.me
HTTP/1.1 200 OK
X-Powered-By: Hexo
Content-Type: text/html
Date: Tue, 26 Apr 2016 02:13:10 GMT
Connection: keep-alive
```

通常我们在监控Web服务的时候，可以根据Web的HTTP状态码来判断Web服务是否工作正常，如果我们使用grep过滤第一行，会发现会输出很多不必要的信息：

```bash
[root@ansheng ~]# curl -I www.ansheng.me | grep "HTTP"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:--  0:00:01 --:--:--     0
HTTP/1.1 200 OK
```

解决方法一

```bash
[root@ansheng ~]# curl -I -s www.ansheng.me | grep "HTTP"
HTTP/1.1 200 OK
```

`-s` 是沉默，静默模式，意思为不输出进度表或错误信息；

解决方法二

```bash
[root@ansheng ~]#  curl -s -w "%{http_code}" -o /dev/null www.ansheng.me
200
```

解决方法三

```bash
[root@ansheng ~]# curl -I www.ansheng.me 2>/dev/null|head -n1                    
HTTP/1.1 200 OK
```