# Домашнее задание к занятию "`Кластеризация и балансировка нагрузки`" - `Викторов Михаил`

### Задание 1

1. Запустите два simple python сервера на своей виртуальной машине на разных портах
2. Установите и настройте HAProxy, воспользуйтесь материалами к лекции по ссылке
3. Настройте балансировку Round-robin на 4 уровне.
4. На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.

---

### Задание 2

1. Запустите три simple python сервера на своей виртуальной машине на разных портах
2. Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
3. HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
4. На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.

## Решение

### Задание 1

Написал такой конфиг (взял за основу example):

```
global
	log /dev/log	local0
	log /dev/log	local1 notice
	chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
	stats timeout 30s
	user haproxy
	group haproxy
	daemon

defaults
	log	global
	mode	http
	option	httplog
	option	dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000

frontend http_in
    bind *:80
    mode http
    default_backend servers

backend servers
    mode http
    balance roundrobin
    server server1 127.0.0.1:8080 check
    server server2 127.0.0.1:8081 check

listen stats
    bind *:8404
    mode http
    stats enable
    stats uri /stats
    stats refresh 5s
    stats admin if TRUE

```

Запустил и проверил балансировку:

<img width="1372" height="577" alt="2026-03-18_20-28-50" src="https://github.com/user-attachments/assets/d106f604-8d38-435e-9c9a-ab3f23bc183e" />

Процессы в фоне на сервере:

<img width="796" height="117" alt="2026-03-18_20-29-30" src="https://github.com/user-attachments/assets/1628d452-4c1d-49c0-ba03-3e1cb90bc59d" />

### Задание 2

Такой конфиг:

```
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
    stats timeout 30s
    user haproxy
    group haproxy
    daemon

defaults
    log global
    mode http
    option httplog
    option dontlognull
    timeout connect 5000
    timeout client 50000
    timeout server 50000

frontend http_front
    bind *:80
    acl host_example hdr(host) -i example.local
    use_backend servers_rr if host_example

backend servers_rr
    balance roundrobin
    server server1 127.0.0.1:8080 weight 2 check
    server server2 127.0.0.1:8081 weight 3 check
    server server3 127.0.0.1:8082 weight 4 check

listen stats
    bind *:8404
    mode http
    stats enable
    stats uri /stats
    stats refresh 5s
    stats admin if TRUE

```

Сервера запущены:

<img width="807" height="138" alt="2026-03-18_21-03-45" src="https://github.com/user-attachments/assets/7c9bd501-093e-4d25-95f6-41edf2c31173" />

Балансировка при запросе example.local:

<img width="1475" height="602" alt="2026-03-18_21-03-25" src="https://github.com/user-attachments/assets/7e84d68a-51fb-4ae2-96d5-f82ad5515c7a" />

Другой домен отдает 503:

<img width="989" height="676" alt="2026-03-18_21-22-21" src="https://github.com/user-attachments/assets/17cc0c09-7500-4cb8-87af-a2411403546e" />




