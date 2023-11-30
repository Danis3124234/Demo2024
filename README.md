[1.txt](https://github.com/Danis3124234/Demo2024/files/13349106/1.txt)# demo2024

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
| ISP | ens 192 | 10.10.201.100 | /24 | 10.10.201.254|
| | ens 224 | 192.168.0.165 | /30 | |
| | ens 256 | 192.168.0.161 | /30 | |
| BR-R | ens 192 | 192.168.0.129 | /27 | |
| | ens 224 | 192.168.0.162 | /30 | 192.168.0.161 |
| HQ-R | ens 192 | 192.168.0.1 | /25 | |
| | ens 224 | 192.168.0.166 | /30 | 192.168.0.165 |
| HQ-SRV | ens 192 | 192.168.0.2 | /25 | 192.168.0.1 |
| BR-SRV | ens 192 | 192.168.0.130 | /27 |192.168.0.129 |

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
## 2. Установка и настройка frr
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
