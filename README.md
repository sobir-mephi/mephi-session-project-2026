# Сессионный проект по курсу “Безопасность GNU/Linux”: Настройка базовых средств защиты ОС GNU/Linux

Практическая работа по настройке **РЕД ОС 8** в VirtualBox. Работа велась через терминал.

В репозитории задания разложены по папкам `task_1`–`task_7`. В каждой лежат сохранённые результаты команд, конфигурационные файлы или история работы (`history_N.out`). Публикация на GitHub - это восьмой раздел, поэтому отдельной папки `task_8` нет.

Ниже перечислены основные команды настройки и проверки.

## 1

Сначала проверил сетевой интерфейс и включил получение IPv4 по DHCP:

```bash
ip a
ip -4 addr
sudo nmcli con mod "enp0s3" ipv4.method auto
sudo nmcli con up "enp0s3"
```

Затем настроил имя хоста и проверил доступность сети:

```bash
sudo hostnamectl set-hostname mephi-2026.domain.local
hostname
ping -c 4 8.8.8.8 > ping.out
cat ping.out
```

## 2

После этого обновил пакеты и установил нужные утилиты:

```bash
sudo dnf upgrade --refresh -y
sudo dnf install nginx libcap-ng-utils -y
sudo dnf install dnf-plugins-core -y
dnf download --downloaddir /tmp tcpdump
rpm -q nginx libcap-ng-utils tcpdump
dnf history > dnf.out
```

## 3

Подключил отдельный виртуальный диск средствами VirtualBox. В самйо виртуалке он определился как `/dev/sdb`. На нём создан один раздел на всё доступное пространство:

```bash
lsblk
sudo fdisk /dev/sdb
lsblk
sudo mkfs.ext4 -L MEPHI_WEB /dev/sdb1
sudo blkid /dev/sdb1
sudo mkdir /mephi-web
sudo nano /etc/fstab
```

Проверил конфигурацию и смонтировал раздел:

```bash
sudo findmnt --verify
sudo mount /mephi-web
df -h /mephi-web
lsblk -f
```

## 4

Запустил веб-сервер и включил автоматический запуск:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
systemctl status nginx
systemctl is-enabled nginx
```

## 5

### 5.1

Создал группы разработчиков и кураторов:

```bash
sudo groupadd -g 5500 developers
sudo groupadd -g 4444 curators
getent group developers
getent group curators
```

`user1` уже существовал после предыдущей работы. У него поменял UID, убрал из группы `students` и добавил в `developers`:

```bash
id user1
sudo usermod -u 5501 user1
sudo gpasswd -d user1 students
sudo usermod -aG developers user1
```

Создал остальных участников:

```bash
sudo useradd -m -u 5502 -G developers user2
sudo useradd -m -u 5503 -G developers user3
sudo useradd -m -G curators curator1
sudo useradd -m -G curators curator2
id user1
id user2
id user3
id curator1
id curator2
```

Создал каталог проекта и настроил наследование группы и прав:

```bash
sudo mkdir -p /data/mephi-2026
sudo chown root:developers /data/mephi-2026
sudo chmod 2770 /data/mephi-2026
sudo dnf install acl -y
sudo setfacl -m g:curators:rx /data/mephi-2026
sudo setfacl -d -m g::rwx,g:curators:rx,m::rwx,o::--- /data/mephi-2026
getfacl /data/mephi-2026
```

Проверял доступ на тестовом файле:

```bash
sudo -u user1 touch /data/mephi-2026/test.txt
sudo -u user2 sh -c 'echo hello > /data/mephi-2026/test.txt'
sudo -u curator1 cat /data/mephi-2026/test.txt
sudo -u curator1 sh -c 'echo test >> /data/mephi-2026/test.txt'
```

Последняя команда завершается отказом в доступе.

### 5.2

```bash
sudo chgrp developers /usr/sbin/tcpdump
sudo chmod 750 /usr/sbin/tcpdump
sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
getcap /usr/sbin/tcpdump
sudo -u user1 tcpdump -i any -c 1 -nn
```

Назначены `CAP_NET_RAW` и `CAP_NET_ADMIN`. Права на запуск ограничены группой разработчиков.

### 5.3

Проверил режим SELinux и подготовил каталог, который был смонтирован в задании 3:

```bash
getenforce
sudo nano /etc/selinux/config
sudo mount /mephi-web
echo "MEPHI WEB" | sudo tee /mephi-web/index.html
sudo chmod 755 /mephi-web
sudo chmod 644 /mephi-web/index.html
```

Проверил конфигурацию и перезапустил nginx:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

Чтобы SELinux разрешил веб-серверу читать файлы из нестандартной директории, настроил контекст:

```bash
sudo semanage fcontext -a -t httpd_sys_content_t "/mephi-web(/.*)?"
sudo restorecon -Rv /mephi-web
curl http://localhost
```

SELinux оставлен в режиме `Enforcing`.

## 6

Запретил кураторам вход, назначив им оболочку `nologin`:

```bash
sudo usermod -s /sbin/nologin curator1
sudo usermod -s /sbin/nologin curator2
getent passwd curator1
getent passwd curator2
sudo -iu curator1
```

В `/etc/login.defs` настроил значение `PASS_MAX_DAYS 90`. Для уже созданных пользователей срок действия паролей изменил отдельно:

```bash
sudo nano /etc/login.defs
sudo chage -M 90 user1
sudo chage -M 90 user2
sudo chage -M 90 user3
sudo chage -M 90 curator1
sudo chage -M 90 curator2
sudo chage -l user1
```

В `/etc/security/pwquality.conf` настроил минимальную длину пароля:

```
minlen = 12
dcredit = 0
ucredit = 0
lcredit = 0
ocredit = 0
```

## 7

Заменил тестовую страницу текстом с номером зачётной книжки и проверил ответ nginx:

```bash
echo "Hello from Student: М265797" | sudo tee /mephi-web/index.html
cat /mephi-web/index.html
systemctl is-active nginx
curl http://localhost/
curl http://localhost/ > curl.out
cat curl.out
```

