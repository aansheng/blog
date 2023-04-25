# 在类Unix中创建加密的压缩包

## tar

- 压缩

```bash
$ tar czvf - file | openssl des3 -salt -k password -out /path/to/file.tar.gz
```

- 解压

```bash
$ openssl des3 -d -k password -salt -in /path/to/file.tar.gz | tar xvzf -
```

## zip

- 压缩

```bash
$ zip -P password secure.zip file
$ zip -P password secure.zip file1 file2 file3
```

- 解压

```bash
$ unzip -P password secure.zip
```
