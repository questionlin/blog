---
title: 用supervisor做高可用
date: 2019-06-03 10:24:28
tags: 运维
id: 1559528755
---
Supervisor 是 Linux 系统中常用的进程守护程序。它会执行一段 shell 脚本，如果脚本中断，则会自动重新执行。

# 安装 Supervisor
```
sudo apt-get install supervisor
```

# 配置 Supervisor
Supervisor 配置文件通常存放在 /etc/supervisor/conf.d 目录，在该目录下，可以创建多个配置文件指示 Supervisor 如何监视进程，例如，让我们创建一个开启并监视 queue:work 进程的 laravel-worker.conf 文件：
```
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3
autostart=true
autorestart=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
```
在本例中，numprocs 指令让 Supervisor 运行 8 个 queue:work 进程并监视它们，如果失败的话自动重启。当然，你需要修改 queue:work sqs 的 command 指令来映射你的队列连接。

# 启动 Supervisor
当成功创建配置文件后，需要刷新 Supervisor 的配置信息并使用如下命令启动进程:
```
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start laravel-worker:*
```

--------------------------------
转自：https://laravelacademy.org/post/19516.html#toc_19  
Supervisor 官方文档：http://supervisord.org/index.html