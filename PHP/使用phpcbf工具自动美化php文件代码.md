### PHP
熟悉php的朋友应该都知道`squizlabs/php_codesniffer`这个库，这是一个进行php代码检查&美化的库，现在结合这个库写了一个git钩子。

这个钩子的目的是自动对**将要提交的文件**进行代码美化，再也不需要手动执行代码去美化了，而且只针对提交的文件，不是所有文件美化。

现分享下钩子文件，以及配置步骤：

- 第一步：确保本机全局安装composer
- 第二步: 全局安装格式化工具

```shell
composer global require "squizlabs/php_codesniffer=*"
```
- 第三步:将如下代码写入**项目路径\\.git\hooks\pre-commit**文件

```shell
#!/bin/sh

# 使用phpcbf美化工具，美化正在提交文件的代码
output=`git diff --name-only --cached | grep '.php'`;

if [ -n "$output" ]; then 
    for file in $output 
	do
		file=`echo "$file" | awk '{gsub(/^\s+|\s+$/, "");print}'`
		phpcbf $file
		git add $file
	done
fi
exit 0
```

配置完成，提交代码看看效果吧

注意:**确保git、php命令能全局使用**

