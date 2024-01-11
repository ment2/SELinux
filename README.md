```
Запуск nginx на нестандартном порту 3-мя разными способами 


[vagrant@selinux ~]$ sudo -i
[root@selinux ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)
[root@selinux ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@selinux ~]# getenforce
Enforcing
[root@selinux ~]# vi /var/log/audit/audit.log

В логе audit найдено сообщение о блокировке порта:
type=AVC msg=audit(1704978482.200:783): avc:  denied  { name_bind } for  pid=2683 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0


[root@selinux ~]# grep 1704978482.200:783 /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1704978482.200:783): avc:  denied  { name_bind } for  pid=2683 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0

	Was caused by:
	The boolean nis_enabled was set incorrectly. 
	Description:
	Allow nis to enabled
	Allow access by executing:
	# setsebool -P nis_enabled 1
	
[root@selinux ~]# setsebool -P nis_enabled on
[root@selinux ~]# systemctl restart nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2024-01-11 13:32:27 UTC; 6s ago
  Process: 3573 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 3569 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 3568 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 3575 (nginx)
   CGroup: /system.slice/nginx.service

           ├─3575 nginx: master process /usr/sbin/nginx

           └─3577 nginx: worker process
Jan 11 13:32:27 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jan 11 13:32:27 selinux nginx[3569]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 11 13:32:27 selinux nginx[3569]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jan 11 13:32:27 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.

[root@selinux ~]# getsebool -a | grep nis_enabled
nis_enabled --> on
[root@selinux ~]# setsebool -P nis_enabled off
[root@selinux ~]# getsebool -a | grep nis_enabled
nis_enabled --> off

[root@selinux ~]# systemctl restart nginx
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.

[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server

   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Thu 2024-01-11 13:35:01 UTC; 18s ago
  Process: 3573 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 3598 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
  Process: 3597 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 3575 (code=exited, status=0/SUCCESS)

Jan 11 13:35:01 selinux systemd[1]: Stopped The nginx HTTP and reverse proxy server.
Jan 11 13:35:01 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jan 11 13:35:01 selinux nginx[3598]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 11 13:35:01 selinux nginx[3598]: nginx: [emerg] bind() to 0.0.0.0:4881 failed (13: Permission denied)
Jan 11 13:35:01 selinux nginx[3598]: nginx: configuration file /etc/nginx/nginx.conf test failed
Jan 11 13:35:01 selinux systemd[1]: nginx.service: control process exited, code=exited status=1
Jan 11 13:35:01 selinux systemd[1]: Failed to start The nginx HTTP and reverse proxy server.
Jan 11 13:35:01 selinux systemd[1]: Unit nginx.service entered failed state.
Jan 11 13:35:01 selinux systemd[1]: nginx.service failed.

[root@selinux ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989

[root@selinux ~]# semanage port -a -t http_port_t -p tcp 4881
[root@selinux ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      4881, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989

[root@selinux ~]# systemctl restart nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2024-01-11 13:39:23 UTC; 9s ago
  Process: 3635 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 3633 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 3632 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 3637 (nginx)
   CGroup: /system.slice/nginx.service

           ├─3637 nginx: master process /usr/sbin/nginx

           └─3639 nginx: worker process
Jan 11 13:39:23 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jan 11 13:39:23 selinux nginx[3633]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 11 13:39:23 selinux nginx[3633]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jan 11 13:39:23 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.

[root@selinux ~]# semanage port -d -t http_port_t -p tcp 4881
[root@selinux ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989


Разрешим в SELinux работу nginx на порту TCP 4881 c помощью формирования и установки модуля SELinux:

[root@selinux ~]# systemctl restart nginx
Job for nginx.service failed because the control process exited with error code. See &quot;systemctl status nginx.service&quot; and &quot;journalctl -xe&quot; for details.
root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Thu 2024-01-11 13:44:31 UTC; 7s ago
  Process: 3635 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 3660 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
  Process: 3659 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 3637 (code=exited, status=0/SUCCESS)

Jan 11 13:44:31 selinux systemd[1]: Stopped The nginx HTTP and reverse proxy server.
Jan 11 13:44:31 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jan 11 13:44:31 selinux nginx[3660]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 11 13:44:31 selinux nginx[3660]: nginx: [emerg] bind() to 0.0.0.0:4881 failed (13: Permission denied)
Jan 11 13:44:31 selinux nginx[3660]: nginx: configuration file /etc/nginx/nginx.conf test failed
Jan 11 13:44:31 selinux systemd[1]: nginx.service: control process exited, code=exited status=1
Jan 11 13:44:31 selinux systemd[1]: Failed to start The nginx HTTP and reverse proxy server.
Jan 11 13:44:31 selinux systemd[1]: Unit nginx.service entered failed state.
Jan 11 13:44:31 selinux systemd[1]: nginx.service failed.

Разрешим в SELinux работу nginx на порту TCP 4881 c помощью формирования и установки модуля SELinux:

root@selinux ~]# systemctl start nginx
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
[root@selinux ~]# grep nginx /var/log/audit/audit.log
type=SYSCALL msg=audit(1704983487.159:889): arch=c000003e syscall=49 success=no exit=-13 a0=6 a1=55dc545778b8 a2=10 a3=7ffdb508f150 items=0 ppid=1 pid=22541 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="nginx" exe="/usr/sbin/nginx" subj=system_u:system_r:httpd_t:s0 key=(null)
type=SERVICE_START msg=audit(1704983487.159:890): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=failed'
[root@selinux ~]# grep nginx /var/log/audit/audit.log | audit2allow -M nginx

******************** IMPORTANT ***********************
To make this policy package active, execute:
semodule -i nginx.pp

[root@selinux ~]# semodule -i nginx.pp
[root@selinux ~]# systemctl start nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2024-01-11 14:37:26 UTC; 6s ago
  Process: 22580 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 22578 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 22577 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 22582 (nginx)
   CGroup: /system.slice/nginx.service

           ├─22582 nginx: master process /usr/sbin/nginx

           └─22584 nginx: worker process
Jan 11 14:37:26 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jan 11 14:37:26 selinux nginx[22578]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 11 14:37:26 selinux nginx[22578]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jan 11 14:37:26 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.

[root@selinux ~]# semodule -r nginx
libsemanage.semanage_direct_remove_key: Removing last nginx module (no other nginx module exists at another priority).

====================================================================================

Обеспечение работоспособности приложения при включенном SELinux


root@Ubuntu:/home/user/selinux/otus-linux-adm/selinux_dns_problems# vagrant status
Current machine states:

ns01                      running (virtualbox)
client                    running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.

root@Ubuntu:/home/user/selinux/otus-linux-adm/selinux_dns_problems# vagrant ssh client
Last login: Thu Jan 11 15:18:01 2024 from 10.0.2.2
###############################
### Welcome to the DNS lab! ###
###############################
- Use this client to test the enviroment
- with dig or nslookup. Ex:
    dig @192.168.50.10 ns01.dns.lab
- nsupdate is available in the ddns.lab zone. Ex:
    nsupdate -k /etc/named.zonetransfer.key
    server 192.168.50.10
    zone ddns.lab 
    update add www.ddns.lab. 60 A 192.168.50.15
    send
- rndc is also available to manage the servers
    rndc -c ~/rndc.conf reload
###############################
### Enjoy! ####################
###############################
[vagrant@client ~]$ nsupdate -k /etc/named.zonetransfer.key
> server 192.168.50.10
> zone ddns.lab
> update add www.ddns.lab. 60 A 192.168.50.15
> send
update failed: SERVFAIL
> quit

[root@ns01 ~]# cat /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1704988101.353:1932): avc:  denied  { create } for  pid=5017 comm="isc-worker0000" name="named.ddns.lab.jnl" scontext=system_u:system_r:named_t:s0 tcontext=system_u:object_r:etc_t:s0 tclass=file permissive=0
	Was caused by:
		Missing type enforcement (TE) allow rule.
		You can use audit2allow to generate a loadable module to allow this access.
		
[root@ns01 ~]# sudo semanage fcontext -l | grep named
/etc/rndc.*                                        regular file       system_u:object_r:named_conf_t:s0 
/var/named(/.*)?                                   all files          system_u:object_r:named_zone_t:s0 


[root@ns01 ~]# ls -laZ /etc/named
drw-rwx---. root named system_u:object_r:etc_t:s0       .
drwxr-xr-x. root root  system_u:object_r:etc_t:s0       ..
drw-rwx---. root named unconfined_u:object_r:etc_t:s0   dynamic
-rw-rw----. root named system_u:object_r:etc_t:s0       named.50.168.192.rev
-rw-rw----. root named system_u:object_r:etc_t:s0       named.dns.lab
-rw-rw----. root named system_u:object_r:etc_t:s0       named.dns.lab.view1
-rw-rw----. root named system_u:object_r:etc_t:s0       named.newdns.lab

[root@ns01 ~]# sudo chcon -R -t named_zone_t /etc/named
[root@ns01 ~]# ls -laZ /etc/named
drw-rwx---. root named system_u:object_r:named_zone_t:s0 .
drwxr-xr-x. root root  system_u:object_r:etc_t:s0       ..
drw-rwx---. root named unconfined_u:object_r:named_zone_t:s0 dynamic
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.50.168.192.rev
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.dns.lab
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.dns.lab.view1
-rw-rw----. root named system_u:object_r:named_zone_t:s0 named.newdns.lab

[root@client ~]# nsupdate -k /etc/named.zonetransfer.key
> server 192.168.50.10
> zone ddns.lab
> update add www.ddns.lab. 60 A 192.168.50.15
> send
update failed: SERVFAIL
> quit

Не работает!

[root@ns01 ~]# cat /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1704988645.734:1966): avc:  denied  { write } for  pid=5017 comm="isc-worker0000" name="dynamic" dev="sda1" ino=5386406 scontext=system_u:system_r:named_t:s0 tcontext=unconfined_u:object_r:named_zone_t:s0 tclass=dir permissive=0

	Was caused by:
	The boolean named_write_master_zones was set incorrectly. 
	Description:
	Allow named to write master zones
	Allow access by executing:
	# setsebool -P named_write_master_zones 1
	

Идём предложенным путём.

[root@client ~]# setsebool -P named_write_master_zones 1
[root@client ~]# nsupdate -k /etc/named.zonetransfer.key
> server 192.168.50.10
> zone ddns.lab
> update add www.ddns.lab. 60 A 192.168.50.15
> send
> quit

О, работает.

[root@client ~]# dig @192.168.50.10 www.ddns.lab
; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.15 <<>> @192.168.50.10 www.ddns.lab
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 53599
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 1, ADDITIONAL: 2
;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.ddns.lab.			IN	A
;; ANSWER SECTION:
www.ddns.lab.		60	IN	A	192.168.50.15
;; AUTHORITY SECTION:
ddns.lab.		3600	IN	NS	ns01.dns.lab.
;; ADDITIONAL SECTION:
ns01.dns.lab.		3600	IN	A	192.168.50.10
;; Query time: 21 msec
;; SERVER: 192.168.50.10#53(192.168.50.10)
;; WHEN: Thu Jan 11 16:09:58 UTC 2024
;; MSG SIZE  rcvd: 96

[root@client ~]# dig www.ddns.lab
; <<>> DiG 9.11.4-P2-RedHat-9.11.4-26.P2.el7_9.15 <<>> www.ddns.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 60312
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1
;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;www.ddns.lab.			IN	A
;; AUTHORITY SECTION:
ddns.lab.		600	IN	SOA	ns01.dns.lab. root.dns.lab. 2711201407 3600 600 86400 600
;; Query time: 1 msec
;; SERVER: 192.168.50.10#53(192.168.50.10)
;; WHEN: Thu Jan 11 16:26:04 UTC 2024
;; MSG SIZE  rcvd: 91





```










































