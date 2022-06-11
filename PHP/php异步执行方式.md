### PHP
- nohup
用nohup配合&,php可以作为linux守护进程运行
```shell
nohup /usr/local/bin/php /opt/cron.php 2>&1 /dev/null &
```

- popen函数
popen函数可以实现异步调用
```shell
pclose(popen('/usr/local/bin/php /opt/cron.php &','r'));
```