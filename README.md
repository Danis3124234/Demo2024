[1.txt](https://github.com/Danis3124234/Demo2024/files/13349106/1.txt)
# demo2024

## №1

## Описание задания.
Выполните базовую настройку всех устройств:

1) Соберите топологию согласно рисунку. Все устройства работают на OC Linux - Debian
2) Присвоить имена в соответствии с топологией
3) Рассчитайте IP-адресацию IPv4 и IPv6. Необходимо заполнить таблицу №1. При необходимости отредактируйте таблицу.
4) Пул адресов для сети офиса BRANCH - не более 16. Для IPv6 пропустите этот пункт.
5) Пул адресов для сети офиса HQ - не более 64. Для IPv6 пропустите этот пункт.

## Топология
![Топология](https://github.com/Danis3124234/Demo2024/blob/main/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA1.PNG)
## Таблица
| Имя устройства | Интерфейс | IPv4/IPv6 | Маска/Префикс | Шлюз |
| ------------- | ------------- | ------------- | ------------- | ------------- | 
| ISP | ens 192 | 10.12.12.2 | /24 | 10.12.12.254|
| | ens 224 | 192.168.0.165 | /30 | |
| | ens 256 | 192.168.0.161 | /30 | |
| BR-R | ens 192 | 192.168.0.129 | /27 | |
| | ens 224 | 192.168.0.162 | /30 | 192.168.0.161 |
| HQ-R | ens 192 | 192.168.0.1 | /25 | |
| | ens 224 | 192.168.0.166 | /30 | 192.168.0.165 |
| HQ-SRV | ens 192 | 192.168.0.2 | /25 | 192.168.0.1 |
| BR-SRV | ens 192 | 192.168.0.130 | /27 |192.168.0.129 |

## 1. Настройка интерфейсов
### Посмотрел интерфейсы
```
ip a
```
### Зашел в файл настройки конфигурации интерфейсов
```
nano /etc/network/interfaces
```
### Назначил IP на интерфейсы в соответствии с таблицей
### ISP
```
auto ens192
iface ens192 inet static
address 10.10.201.100
netmask 255.255.255.0
gateway 10.10.201.254

auto ens224
iface ens224 inet static
address 192.168.0.165
netmask 255.255.255.252

auto ens256 
iface ens256 inet static
address 192.168.0.161
netmask 255.255.255.252
```
### BR-R
```
auto ens192
iface ens192 inet static
address 192.168.0.129
netmask 255.255.255.224

auto ens224
iface ens224 inet static
address 192.168.0.162
netmask 255.255.255.252
gateway 192.168.0.161
```
### HQ-R
```
auto ens192
iface ens192 inet static
address 192.168.0.1
netmask 255.255.255.128

auto ens224
iface ens224 inet static
address 192.168.0.166
netmask 255.255.255.252
gateway 192.168.0.165
```
### HQ-SRV
```
auto ens192
iface ens192 inet static
address 192.168.0.2
netmask 255.255.255.128
gateway 192.168.0.1
```
### BR-SRV
```
auto ens192
iface ens192 inet static
address 192.168.0.130
netmask 255.255.255.224
gateway 192.168.0.129
```
### Сохранил конфигурацию
```
Ctrl+S
```
### Вышел из конфигурационного файла
```
Ctrl+X
```
### Перезагрузил сервис работы с сетью
```
systemctl restart networking
```
## 2. Установка и настройка frr. (BR-R, HQ-R)
### Установил пакет frr
```
apt-get install frr
```
### Проверил состояние
```
systemctl statys frr
```
### Вошел в файл конфигурации
```
nano /etc/frr/daemons
```
### Изменил значение 
```
ospf=yes
```
### Перезагрузил службу 
```
systemctl restart frr
```
### Вошел в настройку маршрутизации 
```
vtysh
```
### Просмотрел ip адреса и их состояние
```
show ip interface brief
```
### Вошел в конфигурацию терминала 
```
conf t
```
### Запустил процесс 
```
router ospf
```
### Добавил интерфейсы 
```
network (ip адрес) area 0
```
### Просмотрел соседей  
```
do show ip ospf neighbor
```
### Сохранил конфигурацию  
```
copy running-config startup-config
```

## 3. Установка и настройка DHCP. (HQ-R)
### Установил DHCP
```
apt update
``` 
```
apt install isc-dhcp-server
```
### Вошел в файл настройки
```
nano /etc/default/isc-dhcp-server
```
### Указал интерфейс который смотрит в сторону сервера
```
INTERFACESV4="ens192"
```
### Настроил раздачу адресов
```
nano /etc/dhcp/dhcpd.conf
```
### Пример настройки
```
subnet 192.168.0.0 netmask 255.255.255.0 {
range 192.168.0.10 192.168.0.125;
option domain-name-servers 8.8.8.8, 8.8.4.4;
option routers 192.168.0.1;
}
```
### Применил настройку 
```
systemctl restart isc-dhcp-server.service
```
### Включил маршрутизацию
```
sysctl -a | grep forward 
sysctl -w net.ipv4.ip_forward=1 >> /etc/sysctl.conf
```
## 4. NAT с помощью firewalld. (ISP, HQ-R, BR-R)
### Установка
```
apt-get -y install firewalld
```
### Автозагрузка
```
systemctl enable --now firewalld
```
### Правило к исходящим пакетам (тот интерфейс который смотрит во внеш. сеть например на исп 192)
```
firewall-cmd --permanent --zone=public --add-interface=ens__
```
### Правило к входящим пакетам (тот интерфейс который смотрит во внутрен. сеть например на исп 224 и 256)
```
firewall-cmd --permanent --zone=trusted --add-interface=ens__
```
### Включение NAT
```
firewall-cmd --permanent --zone=public --add-masquerade
```
### Cохранение правил
```
firewall-cmd --reload
```
## 5. Настройка пользователей.
```
adduser admin
usermod -aG root admin
passwd admin
P@ssw0rd
P@ssw0rd
nano /etc/passwd
```
```
admin:x:0:501::/home/admin:/bin/bash
```
## 6. Настройка тунеля
### Установка графического интерфейса nmtui
```
apt install network-manager
```
### Заходим в интерфейс
```
nmtui
```
![](https://github.com/Danis3124234/Demo2024/blob/main/1.png)
![](https://github.com/Danis3124234/Demo2024/blob/main/2.png)
![](https://github.com/Danis3124234/Demo2024/blob/main/3.png)
### Для BQ-R
```
nmcli connection modify BR-R ip-tunnel.ttl 64
```
```
ip r add 192.168.0.0/25 dev gre1
```
### Для HQ-R
```
nmcli connection modify HQ-R ip-tunnel.ttl 64
```
```
ip r add 192.168.0.128/27 dev gre1
```
## 7. Измерение пропускной способности сети между двумя узлами
### Установка утилиты
```
apt-get -y install iperf3
```
### ISP в роли сервера
### Если не сработает необходимо открыть порт 
### Команда для открытия:
```
iptables -A INPUT -p tcp --dport 5201 -j ACCEPT) 
```
```
iperf3 -s
```
### HQ-R 
```
iperf3 -c 192.168.0.161 -f M
```
### Пример вывода информации
```
[ID] Interval      Transfer   Bitrate        Retr Cwnd
[ 5] 0.00-1.00 sec 345 MBytes 344 MBytes/sec    0 538 KBytes
[ 5] 1.00-2.00 sec 338 MBytes 338 MBytes/sec    0 676 KBytes
[ 5] 3.00-4.00 sec 341 MBytes 341 MBytes/sec    0 749 KBytes
```
## 8. Составление backup скриптов для сохранения конфигурации сетевых устройств (HQ-R BR-R).
```
EDITOR=nano crontab -e (вход в планировщик заданий) 
```
```
минута | час | день | месяц | день недели | "команда, например reboot":
```
```
9 15 * * * cp /etc/frr/frr.conf /etc/networkbackup
```
```
ls /etc/networkbackup
```
```
frr.conf
```
## Присвоил имена в соответствии с топологией
```
hostnamectl set-hostname <NAME>; exec bash
```
### NAME - имя устройства, exec bash — перезапуск оболочки bash для отображения нового хостнейма
