# Стенд с Vagrant c Rsyslog

### Цель домашнего задания:
Научится проектировать централизованный сбор логов. Рассмотреть особенности разных платформ для сбора логов.
____________________
### Описание домашнего задания:
1. В Vagrant разворачиваем 2 виртуальные машины web и log
2. на web настраиваем nginx
3. на log настраиваем центральный лог сервер на любой системе на выбор
journald;
rsyslog;
elk.
4. настраиваем аудит, следящий за изменением конфигов nginx.
____________________

Я решил выполнить это задание с помощью Ansible.\
Ansible playbook запускается при старте Vagrant.\
Проверить работоспособность можно так:\
На web сервере удаляем картинку, к которой будет обращаться nginx во время открытия веб-сраницы: `sudo rm /usr/share/nginx/html/img/header-background.png` и меняем атрибуты у файла: `sudo chmod +x /etc/nginx/nginx.conf`\
В браузере заходим на web сервер http://192.168.56.10\
Далее на log сервере смотрим, что логи пишутся:
```
vagrant@log ~]$ sudo cat /var/log/rsyslog/web/nginx_access.log 
Feb  7 19:43:50 web nginx_access: 192.168.56.1 - - [07/Feb/2024:19:43:50 +0300] "GET /img/header-background.png HTTP/1.1" 404 3650 "http://192.168.56.10/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36"
Feb  7 19:43:50 web nginx_access: 192.168.56.1 - - [07/Feb/2024:19:43:50 +0300] "GET /favicon.ico HTTP/1.1" 404 3650 "http://192.168.56.10/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36"

[vagrant@log ~]$ sudo cat /var/log/rsyslog/web/nginx_error.log 
Feb  7 19:43:50 web nginx_error: 2024/02/07 19:43:50 [error] 5008#5008: *1 open() "/usr/share/nginx/html/img/header-background.png" failed (2: No such file or directory), client: 192.168.56.1, server: _, request: "GET /img/header-background.png HTTP/1.1", host: "192.168.56.10", referrer: "http://192.168.56.10/"
Feb  7 19:43:50 web nginx_error: 2024/02/07 19:43:50 [error] 5008#5008: *1 open() "/usr/share/nginx/html/favicon.ico" failed (2: No such file or directory), client: 192.168.56.1, server: _, request: "GET /favicon.ico HTTP/1.1", host: "192.168.56.10", referrer: "http://192.168.56.10/"

[vagrant@log ~]$ sudo grep nginx_conf /var/log/audit/audit.log
...
node=web type=SYSCALL msg=audit(1707324404.159:1690): arch=c000003e syscall=268 success=no exit=-1 a0=ffffffffffffff9c a1=2058420 a2=1ed a3=7fff168b7670 items=1 ppid=5136 pid=5161 auid=1000 uid=1000 gid=1000 euid=1000 suid=1000 fsuid=1000 egid=1000 sgid=1000 fsgid=1000 tty=pts0 ses=5 comm="chmod" exe="/usr/bin/chmod" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_conf"
node=web type=SYSCALL msg=audit(1707324409.381:1695): arch=c000003e syscall=268 success=yes exit=0 a0=ffffffffffffff9c a1=1176420 a2=1ed a3=7ffcfc704ef0 items=1 ppid=5162 pid=5164 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts0 ses=5 comm="chmod" exe="/usr/bin/chmod" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key="nginx_conf"
```
