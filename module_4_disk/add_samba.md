# Подключение samba
Установка и настройка Samba для предоставления общего доступа к каталогам по протоколу SMB, а также управление правами доступа для различных групп пользователей.

Ожидается, что у вас есть сервер с настроенными хранилищами. (смотри add_raid.md, add_lvm.md)

## На стороне сервера

Алгоритм:
1) Установка Samba
2) Настройка каталогов и прав
3) Настройка Samba
4) Перезапуск Samba
5) Настройка прав доступа к файлам

## UPD не реализовано
1) Добавить секьюрность
2) Добавить флаги в /etc/fstab чтобы если сервер в ребуте - то ждать несколько сек, чтобы хост стал online и можно было смонтировать...иначе загружается без смонтированного ресурса.

### Установка Samba
```bash
sudo apt update
sudo apt install -y samba
```
- Убедитесь, что установленная версия Samba поддерживает протокол SMB версии 3
```bash
smbd --version
```
### Настройка каталогов и прав
- Задайте соответствующие права на каталог `/var/storage/smb`
```bash
sudo chown -R root:root /var/storage/smb
sudo chmod -R 755 /var/storage/smb
```
- Создайте группы и добавьте пользователей
```bash
# создаем группы
sudo groupadd smbadmins
sudo groupadd smbusers
sudo groupadd smbguests

# создаем пользователей
sudo useradd -M -s /sbin/nologin adminuser
sudo useradd -M -s /sbin/nologin regularuser
sudo useradd -M -s /sbin/nologin guestuser

# добавляем их в группы
sudo usermod -aG smbadmins adminuser
sudo usermod -aG smbusers regularuser
sudo usermod -aG smbguests guestuser
```
- Установите пароли для Samba пользователей
```bash
sudo smbpasswd -a adminuser
sudo smbpasswd -a regularuser
sudo smbpasswd -a guestuser
```

### Настройка Samba
- Откройте конфигурационный файл Samba в текстовом редакторе
```bash
sudo nano /etc/samba/smb.conf
```
- Добавьте следующие строки в конец файла, чтобы настроить общий доступ
```text
[smbshare]
path = /var/storage/smb
browsable = yes
writable = yes
read only = no
guest ok = no
valid users = @smbadmins, @smbusers, @smbguests
force group = smbadmins

# Настройка прав доступа
create mask = 0660
directory mask = 0770

# Настройки для конкретных групп
# Гости - только чтение
write list = @smbadmins, @smbusers
read list = @smbguests
```
- Убедитесь, что конфигурация Samba использует SMBv3
```bash
[global]
server min protocol = SMB3
```
- Сохраните файл и закройте текстовый редактор (Ctrl+O, Enter, Ctrl+X в nano).
### Перезапуск Samba
- Перезапустите службы Samba для применения изменений
```bash
sudo systemctl restart smbd
sudo systemctl restart nmbd
```
### Проверка статуса
```bash
sudo systemctl status smbd
sudo systemctl status nmbd
```
### Настройка прав доступа к файлам
- Задайте права доступа к каталогу, чтобы администраторы и пользователи могли читать, создавать и удалять файлы, а гости — только читать
```bash
sudo apt -y install acl
sudo setfacl -R -m g:smbadmins:rwx /var/storage/smb
sudo setfacl -R -m g:smbusers:rwx /var/storage/smb
sudo setfacl -R -m g:smbguests:rx /var/storage/smb
sudo setfacl -R -d -m g:smbadmins:rwx /var/storage/smb
sudo setfacl -R -d -m g:smbusers:rwx /var/storage/smb
sudo setfacl -R -d -m g:smbguests:rx /var/storage/smb
```
## На стороне клиента

Алгоритм:
1) Установка smbclient на клиентском сервере
2) Подключение к Samba-ресурсу и проверка доступа
3) Подключение автомонтирования ресурса

### Установка smbclient на клиентском сервере
```bash
sudo apt update
sudo apt install -y smbclient
```
### Подключение к Samba-ресурсу и проверка доступа
Для пользователя из группы smbadmins
- Подключитесь к Samba-ресурсу
```bash
smbclient //192.168.250.10/smbshare -U adminuser
```
Введите пароль пользователя adminuser при запросе.
После успешного подключения вы попадёте в интерактивный режим smb: \>. Вы можете выполнить следующие команды для проверки доступа:
```text
# Просмотр содержимого каталога
smb: \> ls

# Создание файла
smb: \> put /etc/hostname testfile_admin.txt

# Удаление файла
smb: \> del testfile_admin.txt
```
- Выйдите из интерактивного режима
```text
smb: \> exit
```
Для пользователя из группы smbusers - все тоже самое<br>
Для пользователя из группы smbguests
```text
# Просмотр содержимого каталога
smb: \> ls

# Попытка создания файла (должна завершиться ошибкой)
smb: \> put /etc/hostname testfile_guest.txt
```
Вы увидите сообщение об ошибке, если доступ на запись запрещен
```text
NT_STATUS_ACCESS_DENIED opening remote file \testfile_guest.txt
```

### Подключение автомонтирования ресурса
```bash
sudo apt update
sudo apt -y install cifs-utils
```
```bash
sudo mkdir -p /mnt/smb-share
```
- Проверь что ОС поддерживает utf8 по дефолту
```bash
grep CONFIG_NLS /boot/config-$(uname -r) |grep -ie utf
```
```text
CONFIG_NLS_DEFAULT="utf8"
CONFIG_NLS_UTF8=m
```
- Включаем автомонтирование
```bash
sudo nano /etc/fstab
```
- Если CONFIG_NLS_DEFAULT ="utf8"  (вставляй без переноса строк!!!)
```text
//192.168.250.10/smbshare /mnt/smb-share cifs username=adminuser,password=123,file_mode=0777,dir_mode=0777 0 0
```
- Если CONFIG_NLS_DEFAULT !="utf8" добавляем iocharset=utf8 (вставляй без переноса строк!!!)
```text
//192.168.250.10/smbshare /mnt/smb-share cifs
username=adminuser,password=123,iocharset=utf8,file_mode=0777,dir_mode=0777 0 0
```