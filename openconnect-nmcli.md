# Создание подключения openconnect используя network manager в Debian 12

Установите OpenConnect:
```bash
sudo apt install network-manager-openconnect-gnome
```
Добавьте VPN-подключение через nmcli:
```bash
nmcli connection add type vpn vpn-type openconnect \
connection.id "vpn1" \
connection.interface-name "" \
vpn.data "gateway=vpn.somwhere.ru/?path, cookie-flags=2"
```
Настройте аутентификацию:
```bash
nmcli connection modify "vpn1" vpn.secrets "username=ВАШ_ЛОГИН,password=ВАШ_ПАРОЛЬ"
```
Если надо, то добавьте серверы DNS:
```bash
nmcli connection modify "vpn1" ipv4.dns "8.8.8.8 1.1.1.1"
```
Отключить автоматическое назначение серверов DNS:
```bash
nmcli connection modify "vpn1" ipv4.ignore-auto-dns yes
```