1、切换到git用户下

```bash
# docker exec -it gitlab /bin/bash
root@e3ec6ebd9a15:/# su - git
$ 
```



2、进入到gitlab-rails console

```bash
$ gitlab-rails console
--------------------------------------------------------------------------------
 Ruby:         ruby 2.7.4p191 (2021-07-07 revision a21a3b7d23) [x86_64-linux]
 GitLab:       14.3.2 (92acfb1b8a9) FOSS
 GitLab Shell: 13.21.1
 PostgreSQL:   12.7
--------------------------------------------------------------------------------
Loading production environment (Rails 6.1.3.2)
irb(main):001:0> 
```



3、修改密码

```bash
irb(main):001:0> user = User.where(id: 1).first
=> #<User id:1 @root>

irb(main):002:0> user.password="12345678"
=> "12345678"

irb(main):003:0> user.password_confirmation="12345678"
=> "12345678"
```





4、保存且退出

```bash
irb(main):004:0> user.save!
=> true

irb(main):005:0> quit
```

