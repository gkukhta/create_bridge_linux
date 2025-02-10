# Установка шрифтов TrueType Microsoft в Debian 12  
Установка части шрифтов:
```bash
sudo apt install ttf-mscorefonts-installer
```
Установка Font Forge, который поможет в преобразовании шрифтов.
```bash
sudo apt-get install fontforge
```
Запуск [скрипта](src/create_bridge_linux/ttf-vista-fonts-installer.sh) 
для установки vista font pack, который включает в себя Calibri.  
Скрипт загружен [отсюда](https://gist.github.com/maxwelleite/10774746/raw/ttf-vista-fonts-installer.sh).
```bash
sudo bash ./ttf-vista-fonts-installer.sh
```