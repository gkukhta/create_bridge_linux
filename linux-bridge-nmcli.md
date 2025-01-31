#Создание интерфейса bridge в Linux про помощи Network Manager cli  
Интерфейс bridge может понадобиться, например, для того чтобы подключить виртуалку к физической сети.  
1. Определяем физическое подключение:
```bash
nmcli connection show
```
```
NAME                 UUID                                  TYPE      DEVICE 
Connection 2         107fc9c1-966f-4a24-877c-a23879a79f47  ethernet  enp2s0 
lo                   0d46c34b-2a88-4117-9bab-b8551d9bb265  loopback  lo
```
Физическое подключение "Connection 2"  
Удаляем его:
```bash
nmcli con del "Connection 2"
```
```
Подключение «Connection 2» (107fc9c1-966f-4a24-877c-a23879a79f47) успешно удалено.
```
Добавляем мост:
```bash
nmcli connection add type bridge ifname br0
```
```
Подключение «bridge-br0» (63db8daf-6661-4d17-8b59-e9d804a04933) успешно добавлено.
```
Подключаем физический интерфейс к мосту:
```bash
nmcli connection add type ethernet ifname enp2s0 master br0
```
При необходимости настраиваем автоматическую настройку по DHCP:
```bash
nmcli connection modify bridge-br0 ipv4.method auto
```
При необходимости клонируем MAC адрес физического интерфейса:
```bash
nmcli connection modify bridge-br0 ethernet.cloned-mac-address 00:e0:52:94:00:48
```
При необходимости добавляем маршруты:
```bash
nmcli connection modify bridge-br0 +ipv4.routes "192.168.0.0/16 192.168.0.1 40"
```
Включаем интерфейс bridge-br0:
```bash
nmcli connection up bridge-br0
```
Если надо, то добавьте серверы DNS:
```bash
nmcli connection modify "bridge-br0" ipv4.dns "8.8.8.8 1.1.1.1"
```
Отключить автоматическое назначение серверов DNS:
```bash
nmcli connection modify "bridge-br0" ipv4.ignore-auto-dns yes
```