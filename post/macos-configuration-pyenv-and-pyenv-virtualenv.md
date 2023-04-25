# Macos配置Python多版本(Pyenv)以及虚拟环境(Pyenv-Virtualenv)

在最开始学习Python的时候，老师教我们的虚拟环境是通过[virtualenv](https://github.com/pypa/virtualenv)来实现的，用着用着就发现一个问题，那就是多版本的Python如何解决呢？

后来工作之后得知有[pyenv](https://github.com/pyenv/pyenv)这个东西，就去了解了一下，然后就开始使用，别说，用着还挺爽，到现在也一直在使用。

> Pyenv允许你轻松地在多个Python版本之间切换

简单的说就是通过pyenv，你可以在自己的机器上面共存多个Python版本，例如3.7、3.6、3.5、2.7、2.6等都可以，而且官方还提供了一个[Pyenv-Virtualenv](https://github.com/pyenv/pyenv-virtualenv)，功能和virtualenv完全一样，在同一个Python版本中创建隔离的虚拟环境。

## 安装

我平常使用的都是`Macos`，这里的安装教程我就只放Macos上面如何安装了，其他系统的安装教程完全可以参照官方文档进行操作。

- 依赖包

这些依赖包是在编译Python版本的时候需要使用到的，所以一定要提前安装

```bash
$ brew install openssl readline sqlite3 xz zlib
```

运行Mojave或更高版本（10.14+）时，您还需要安装其他SDK标头

```bash
$ sudo installer -pkg /Library/Developer/CommandLineTools/Packages/macOS_SDK_headers_for_macOS_10.14.pkg -target /
```

- 安装

我这里使用`Homebrew`进行安装，因为简单，不需要进行一些配置，而且后续的升级卸载什么的也很方便

```bash
$ brew install pyenv pyenv-virtualenv
```

如果你使用了`zsh`，你可能需要找到`plugins`的配置，然后将`pyenv`加入其中，如果不添加这项配置，在执行命令的时候会提示找不到`pyenv`指令，语法提示等也没有

```bash
$ vim ~/.zshrc
plugins=(
pyenv
...
)
```

然后`source`让其生效

```bash
$ source ~/.zshrc
```

- 升级

```bash
$ brew upgrade pyenv pyenv-virtualenv
```

- 卸载

```bash
$ brew uninstall pyenv pyenv-virtualenv
```

## 使用

- 查看可以安装的Python版本

```bash
$ pyenv install --list
```

- 安装Python版本

```bash
$ pyenv install 3.7.3
```

- 卸载Python版本

```bash
$ pyenv uninstall 3.7.3
```

- 列出已经安装的Python版本

```bash
$ pyenv versions
```

- 创建虚拟环境

`3.7.3`是Python的版本号，`ansheng`是虚拟环境的名称

```bash
$ pyenv virtualenv 3.7.3 ansheng
```

- 切换虚拟环境

```bash
$ pyenv activate ansheng
(ansheng) $ python -V
Python 3.7.3
(ansheng) $ pip -V
pip 19.0.3 from /Users/shengan/.pyenv/versions/3.7.3/envs/ansheng/lib/python3.7/site-packages/pip (python 3.7)
```

- 退出虚拟环境

```bash
(ansheng) $ pyenv deactivate
$
```

- 删除虚拟环境

```bash
$ pyenv virtualenv-delete ansheng
```