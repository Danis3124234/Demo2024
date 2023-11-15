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
![Топология]()
## Таблица
| Имя устройства | Интерфейс | IPv4/IPv6 | Маска/Префикс | Шлюз |
| ------------- | ------------- | ------------- | ------------- | ------------- | 
| ISP | ens 192 | 10.12.12.10 | /24 | 10.12.12.10|
| | ens 224 | 192.168.0.162 | /30 | |
| | ens 256 | 192.168.0.163 | /30 | |
| BR-R | ens 192 | 192.168.0.161 | /30 | 192.168.0.162 |
| | ens 224 | 192.168.0.1 | /25 | |
| HQ-R | ens 192 | 192.168.0.164 | /30 | 192.168.0.163 |
| | ens 224 | 192.168.0.129 | /27 | |
| HQ-SRV | ens 192 | 192.168.0.2 | /25 | 192.168.0.1 |
| BR-SRV | ens 192 | 192.168.0.156 | /27 |192.168.0.129 |

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
address 10.12.12.10
netmask 255.255.255.0
gateway 10.12.12.10

auto ens224
iface ens224 inet static
address 192.168.0.162
netmask 255.255.255.252

auto ens256 
iface ens256 inet static
address 192.168.0.163
netmask 255.255.255.252
```
### BR-R
```
auto ens192
iface ens192 inet static
address 192.168.0.161
netmask 255.255.255.252
gateway 192.168.0.162

auto ens224
iface ens224 inet static
address 192.168.0.1
netmask 255.255.255.128
```
