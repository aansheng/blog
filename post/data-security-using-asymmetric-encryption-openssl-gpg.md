# 使用非对称加密(OpenSSL/GPG)确保数据安全

最近在对私有的数据做备份，目前的流程是服务器上面写脚本通过zip进行加密，密码是明文写在脚本里面的，然后把备份文件上传到S3。

但是想到一个问题，如果我的VPS被黑了，那么我这些已经备份好的加密文件是否还有意义？hack已经知道备份文件的解密密码，可以随时解开查看等等，感觉安全性很低。

经过一番搜索后面发现可以通过openssl进行非对称加密，比如我有一份密钥对，我可以在服务器上面放公钥，使用openssl通过公钥对文件进行加密，然后在本地通过私钥解密加密之后的文件，这样即使服务器被hack获取到备份后的文件也无法解开，此时我只需要保护好本地的私钥文件即可。

## 备份步骤(OpenSSL)

- 创建要备份的模拟数据文件

```bash
$ mkdir -p ~/backup/2022-11-12 && cd ~/backup/2022-11-12
$ fallocate -l 900M sample.txt
$ echo "Hello, ansheng!" >> hello.txt
$ cat hello.txt
Hello, world!
$ ls -lh
total 901M
-rw-r--r-- 1 root root   16 Nov 12 14:53 hello.txt
-rw-r--r-- 1 root root 900M Nov 12 14:53 sample.txt
$ cd ..
$ tar zcf 2022-11-12.tar.gz 2022-11-12
$ rm -fr 2022-11-12
```

- 生成RSA密钥对

```bash
openssl genrsa -out private.pem 4096
openssl rsa -pubout -outform PEM -in private.pem -out public.pem
```

- 生成一个用于加密的密码文件(keypass)

```bash
openssl rand 256 > keypass
```

- 使用公钥对keypass文件进行加密

```bash
openssl rsautl -encrypt -pubin -inkey public.pem -in keypass -out keypass.enc
```

- 使用keypass文件进行加密

```bash
openssl enc -aes-256-cbc -pbkdf2 -salt -in 2022-11-12.tar.gz -out 2022-11-12.tar.gz.enc -pass file:./keypass
# 删除不需要的源文件
rm -f 2022-11-12.tar.gz keypass
```

- 将整个备份目录用tar进行压缩打包形成tar.gz文件

```bash
mkdir 2022-11-12
mv keypass.enc 2022-11-12.tar.gz.enc 2022-11-12
tar zcf 2022-11-12.tar.gz 2022-11-12
```

本地解压tar.gz备份文件，先通过openssl解密keypass文件，然后再通过keypass解密备份文件

```bash
tar xf 2022-11-12.tar.gz && rm -f 2022-11-12.tar.gz && cd 2022-11-12
openssl rsautl -decrypt -inkey ../private.pem -in keypass.enc -out keypass
openssl enc -d -aes-256-cbc -pbkdf2 -in 2022-11-12.tar.gz.enc -out 2022-11-12.tar.gz -pass file:./keypass
tar xf 2022-11-12.tar.gz
```

整个目录结构如下

```bash
$ tree ~/backup
/root/backup
├── 2022-11-12
│   ├── 2022-11-12
│   │   ├── hello.txt
│   │   └── sample.txt
│   ├── 2022-11-12.tar.gz
│   ├── 2022-11-12.tar.gz.enc
│   ├── keypass
│   └── keypass.enc
├── private.pem
└── public.pem

2 directories, 8 files
```

查看hello.txt文件内容

```bash
$ cat ~/backup/2022-11-12/2022-11-12/hello.txt
Hello, ansheng!
```

## GPG

在写完OpenSSL版本的非对称加密之后，偶然得知GPG也可以实现同样的操作，好像用的人还挺多？于是，就有了这篇文章补充，下面是通过GPG来做非对称加密的步骤。

- macOS安装GPG

```bash
brew install gnupg gnupg2
```

- 创建密钥

```bash
gpg --full-generate-key
```

- 导出公钥

```bash
# 最后面的填写创建密钥时的Name或者Email
gpg --output ansheng.public.pgp --armor --export ansheng
```

- 在服务区上面导入公钥

```bash
gpg --import ansheng.public.pgp
rm -f ansheng.public.pgp
```

- 查看服务器上面拥有的所有公钥

```bash
$ gpg -k
/root/.gnupg/pubring.kbx
------------------------
pub   ed25519 2022-11-12 [SC]
      F0540397444715F97ADF9C7754E3EB25D5EAD2D2
uid           [ unknown] ansheng <info@ansheng.me>
sub   cv25519 2022-11-12 [E]
```

- 准备备份的文件

```bash
mkdir backup && cd backup
echo "Hello, ansheng!" >> hello.txt
cd ..
```

- 调整信任级别，防止每次备份的时候都要询问

```bash
gpg --edit-key ansheng
gpg> trust
gpg> 5
gpg> quit
```

- 服务器上面加密文件

```bash
# -r 后面输入用户名
tar -cvzf - backup | gpg -e -r ansheng > backup.tar.gz.gpg
```

- 本地机器上面解密

```bash
gpg -d backup.tar.gz.gpg | tar -xvzf -
```

- 查看文件是否正确

```bash
$ cat backup/hello.txt
Hello, ansheng!
```

- 私钥的导入与导出

```bash
gpg --output ansheng.private.pgp --armor --export-secret-key ansheng
gpg --import ansheng.private.pgp
```