```
for i in `jps|grep -iv jps|awk '{print $1}'`; do ls -lt /proc/$i/cwd; done
```

说明：jps命令显示运行的java工程