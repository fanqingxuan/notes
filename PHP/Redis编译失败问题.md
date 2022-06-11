### PHP
- problem：No rule to make target 'release.h', needed by 'release.o'.  Stop
**解决方式:**
```shell
chmod 777 src/mkreleasehdr.sh
```

- 错误：jemalloc/jemalloc.h：没有那个文件或目录
错误：#error "Newer version of jemalloc required"
**解决方式:**
```shell
make MALLOC=libc
```