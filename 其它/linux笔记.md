### 其它

- 目录内文件内容替换
```shell
//当前目录内字符串Phalcon替换为Tools
grep -Rl C . | xargs sed -i 's#Phalcon#Tools#g'
```

- 递归删除目录下匹配的文件
```shell
find . -name '.php_cs.cache' -delete
```