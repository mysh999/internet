问题描述：

gitlab执行数据备份时，报以下权限问题：

```bash
# gitlab-rake gitlab:backup:create
Dumping database ... 
rake aborted!
Errno::EACCES: Permission denied @ dir_s_mkdir - /data/backup/db
/opt/gitlab/embedded/service/gitlab-rails/lib/backup/database.rb:13:in `dump'
/opt/gitlab/embedded/service/gitlab-rails/lib/tasks/gitlab/backup.rake:91:in `block (4 levels) in <top (required)>'
/opt/gitlab/embedded/service/gitlab-rails/lib/tasks/gitlab/backup.rake:11:in `block (3 levels) in <top (required)>'
Tasks: TOP => gitlab:backup:db:create
(See full trace by running task with --trace)
[root@gitlab /]# Errno::EACCES: Permission denied @ dir_s_mkdir
Last login: Mon Dec 20 10:08:43 2021 from 10.10.19.104
```





解决过程：

备份目录没有足够的权限

```bash
# chown git /data/backup/
# chmod 700 /data/backup/
```



再次执行备份OK

