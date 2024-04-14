**Домашнее задание №7**

**Тема** ***"Управление пакетами. Дистрибьция софта"***

**Задача**
1) Создать свой RPM пакет
2) Создать свой репозиторий и разместить там ранее собранный RPM


---
**Результат выполнения задания**

1. Поднят предоставленный `Vagrantfile`:

```
[vagrant@rpm ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:4d:77:d3 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 86352sec preferred_lft 86352sec
    inet6 fe80::5054:ff:fe4d:77d3/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:ac:5c:0a brd ff:ff:ff:ff:ff:ff
    inet 192.168.11.150/24 brd 192.168.11.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feac:5c0a/64 scope link 
       valid_lft forever preferred_lft forever
```

2. Создана роль Ansible: 
```
tree                                                                                                                                     
.
├── ansible.cfg
├── playbooks
│   └── build_nginx.yml
├── README.md
├── rpm
│   ├── handlers
│   │   └── main.yml
│   ├── hosts
│   │   └── inventory
│   ├── README.md
│   ├── tasks
│   │   └── main.yml
│   └── templates
│       ├── default.conf.j2
│       └── freemax.repo.j2
└── Vagrantfile

6 directories, 10 files

```

3. Роль выполняет следующие функции:
- Устанавливает необходимые библиотеки для сборки nginx
- Получает исходный код для nginx и OpenSSL
- Модифицирует SPEC файл сборки nginx с OpenSSL
- Запускает сборку
- Уставливает собранный Nginx
- Создает репозиторий и размещает там ранее собранный RMP

4. В результе выполнения playbooks/build_nginx.yml:
```
ansible-playbook playbooks/build_nginx.yml

PLAY [rpm] ******************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************************************
ok: [rpm]

TASK [rpm : Installing lib for build] ***************************************************************************************************************************************************************************************************
changed: [rpm] => (item=redhat-lsb-core)
changed: [rpm] => (item=wget)
changed: [rpm] => (item=rpmdevtools)
ok: [rpm] => (item=rpm-build)
changed: [rpm] => (item=createrepo)
ok: [rpm] => (item=yum-utils)
changed: [rpm] => (item=gcc)
changed: [rpm] => (item=openssl-devel)
ok: [rpm] => (item=zlib-devel)
ok: [rpm] => (item=pcre-devel)

TASK [rpm : Get source files nginx] *****************************************************************************************************************************************************************************************************
changed: [rpm]

TASK [rpm : Create rpm folder] **********************************************************************************************************************************************************************************************************
changed: [rpm]

TASK [rpm : Uzip OpenSSL sources] *******************************************************************************************************************************************************************************************************
changed: [rpm]

TASK [rpm : Castomize SPEC file] ********************************************************************************************************************************************************************************************************
changed: [rpm]

TASK [rpm : Build nginx] ****************************************************************************************************************************************************************************************************************
changed: [rpm]

TASK [rpm : Install castomize build nginx] **********************************************************************************************************************************************************************************************
changed: [rpm]

TASK [rpm : Create pecrona dir] *********************************************************************************************************************************************************************************************************
changed: [rpm]

TASK [rpm : Get percona rmp] ************************************************************************************************************************************************************************************************************
changed: [rpm]

TASK [rpm : Copy rmp on repo] ***********************************************************************************************************************************************************************************************************
changed: [rpm]

TASK [rpm : Create repo] ****************************************************************************************************************************************************************************************************************
changed: [rpm]

TASK [rpm : Create NGINX defualt config file from template] *****************************************************************************************************************************************************************************
changed: [rpm]

TASK [rpm : Add repo to yum.repos.d] ****************************************************************************************************************************************************************************************************
changed: [rpm]

RUNNING HANDLER [rpm : restart nginx] ***************************************************************************************************************************************************************************************************
changed: [rpm]

RUNNING HANDLER [rpm : reload nginx] ****************************************************************************************************************************************************************************************************
changed: [rpm]

PLAY RECAP ******************************************************************************************************************************************************************************************************************************
rpm                        : ok=16   changed=15   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

**Созданные пакеты**
```
[root@rpm ~]# ll rpmbuild/RPMS/x86_64/
total 3912
-rw-r--r--. 1 root root 2040532 апр 14 20:34 nginx-1.20.2-1.el7.ngx.x86_64.rpm
-rw-r--r--. 1 root root 1960996 апр 14 20:34 nginx-debuginfo-1.20.2-1.el7.ngx.x86_64.rpm
```

**Состояние установленного nginx**
```
[root@rpm ~]# systemctl status nginx
● nginx.service - nginx - high performance web server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since Вс 2024-04-14 20:34:07 UTC; 4min 42s ago
     Docs: http://nginx.org/en/docs/
  Process: 20355 ExecReload=/bin/sh -c /bin/kill -s HUP $(/bin/cat /var/run/nginx.pid) (code=exited, status=0/SUCCESS)
  Process: 20305 ExecStart=/usr/sbin/nginx -c /etc/nginx/nginx.conf (code=exited, status=0/SUCCESS)
 Main PID: 20306 (nginx)
   CGroup: /system.slice/nginx.service
           ├─20306 nginx: master process /usr/sbin/nginx -c /etc/nginx/nginx.conf
           └─20358 nginx: worker process

апр 14 20:34:07 rpm systemd[1]: Starting nginx - high performance web server...
апр 14 20:34:07 rpm systemd[1]: Can't open PID file /var/run/nginx.pid (yet?) after start: No such file or directory
апр 14 20:34:07 rpm systemd[1]: Started nginx - high performance web server.
апр 14 20:34:07 rpm systemd[1]: Reloading nginx - high performance web server.
апр 14 20:34:07 rpm systemd[1]: Reloaded nginx - high performance web server.

```

**Репозиторий на сервере**
```
[root@rpm ~]# curl -a http://localhost/repo/
<html>
<head><title>Index of /repo/</title></head>
<body>
<h1>Index of /repo/</h1><hr><pre><a href="../">../</a>
<a href="repodata/">repodata/</a>                                          14-Apr-2024 20:34                   -
<a href="nginx-1.20.2-1.el7.ngx.x86_64.rpm">nginx-1.20.2-1.el7.ngx.x86_64.rpm</a>                  14-Apr-2024 20:34             2040532
<a href="percona-orchestrator-3.2.6-2.el8.x86_64.rpm">percona-orchestrator-3.2.6-2.el8.x86_64.rpm</a>        14-Apr-2024 20:34             5222976
</pre><hr></body>
</html>
```

**Подключенный репозиторий**
```
[root@rpm ~]# yum repolist enabled | grep freemax
freemax                             freemax-linux                              2
[root@rpm ~]# 

```