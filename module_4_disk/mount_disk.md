# Создание и монтирование диска

## Алгоритм
0) Проверить разметку диска
1) Инициализировать диск с GPT или MBR
2) Создание разделов
3) Создание файловых систем на разделах
4) Просмотр созданных разделов
5) Монтаж разделов в соответствующие каталоги
6) Добавить разделы в файл /etc/fstab/ для автоматического монтирования при перезагрузке системы
7) Проверка, что файлы записаны правильно в /etc/fstab
----
0) Проверка разметки диска
```bash
sudo parted /dev/sdc print
```
```text
Error: /dev/sdc: unrecognised disk label
Model: VBOX HARDDISK (scsi)
Disk /dev/sdc: 10.7GB
Sector size (logical/physical): 512B/512B
Partition Table: unknown
Disk Flags:
```
Если разметка есть - стереть (Partition Table: gpt или  mdr)
```bash
sudo dd if=/dev/zero of=/dev/sdc bs=1M count=100 status=progress
```

1) Инициализация диска

- GPT
```bash
sudo parted /dev/sdc mklabel gpt
```
```text
Model: VBOX HARDDISK (scsi)
Disk /dev/sdc: 10.7GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start  End  Size  File system  Name  Flags

```

- MBR
```bash
sudo parted /dev/sdc mklabel msdos
```

2) Создание разделов
```bash
sudo parted -a optimal /dev/sdc mkpart primary ext4 0% 2GB
sudo parted -a optimal /dev/sdc mkpart primary ext4 2GB 5GB
sudo parted -a optimal /dev/sdc mkpart primary ext4 5GB 10GB
```
3) Создание файловых систем на разделах
```bash
sudo mkfs.ext4 /dev/sdc1
sudo mkfs.ext4 /dev/sdc2
sudo mkfs.ext4 /dev/sdc3
```
4) Просмотр созданных разделов
```bash
lsblk /dev/sdc
```

5) Монтаж разделов в соответствующие каталоги
- Создадим каталоги, куда будем монтировать наши разделы
```bash
sudo mkdir -p /var/www/mysite
sudo mkdir -p /opt/myapp
sudo mkdir -p /var/mydatabase
```
- Теперь смонтируем разделы в соответствующие каталоги
```bash
sudo mount /dev/sdc1 /var/www/mysite
sudo mount /dev/sdc2 /opt/myapp
sudo mount /dev/sdc3 /var/mydatabase
```
- Проверим, что разделы смонтированы правильно
```bash
df -h
```

6) Добавить разделы в файл /etc/fstab/ для автоматического монтирования при перезагрузке системы
- Откроем файл /etc/fstab для редактирования
```bash
sudo nano /etc/fstab
```
----
Помимо монтирования разделов по названию устройства (например, /dev/sda1), в Linux есть несколько других способов монтирования разделов. Эти методы могут быть более устойчивыми к изменениям конфигурации системы, так как названия устройств могут изменяться при подключении новых устройств или при перезагрузке системы.
----
- Узнать UUID или  ранее созданный LABEL (опционально)
```bash
sudo blkid /dev/sdc1
```
```text
/dev/sdc1: UUID="822e113b-c1e0-4230-acb1-d5cef0ef4efa" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="primary" PARTUUID="04763b9b-228d-4aa7-9be9-ccd5950dce91"

```
- Создать LABEL (рекомендуемый способ)
```bash
sudo e2label /dev/sdc1 mysite
sudo e2label /dev/sdc2 myapp
sudo e2label /dev/sdc3 mydatabase
```
```text
/dev/sdc1: LABEL="mysite" UUID="822e113b-c1e0-4230-acb1-d5cef0ef4efa" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="primary" PARTUUID="04763b9b-228d-4aa7-9be9-ccd5950dce91"
```
- Добавим следующие строки в конец файла /etc/fstab(используем название раздела, либо UUID, либо LABEL)
ПРИМЕР
```text
/dev/sdc1	/var/www/mysite	ext4	defaults	0 2
UUID=<строка из ID> /opt/myapp ext4 defaults 0 2
LABEL=mydatabase /var/mydatabase ext4 defaults 0 2
```
WORK
```text
LABEL=mysite /var/www/mysite	ext4	defaults	0 2
LABEL=myapp /opt/myapp ext4 defaults 0 2
LABEL=mydatabase /var/mydatabase ext4 defaults 0 2
```

7) Проверка, что файлы записаны правильно в /etc/fstab
- Размонтировать созданные разделы
```bash
sudo umount /dev/sdc1 /dev/sdc2 /dev/sdc3
```
```text
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdc      8:32   0   10G  0 disk
├─sdc1   8:33   0  1.9G  0 part
├─sdc2   8:34   0  2.8G  0 part
└─sdc3   8:35   0  4.7G  0 part
```
- Смонтировать заново (прочитает из /etc/fstab)
```bash
sudo mount -a
```
```text
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sdc      8:32   0   10G  0 disk
├─sdc1   8:33   0  1.9G  0 part /var/www/mysite
├─sdc2   8:34   0  2.8G  0 part /opt/myapp
└─sdc3   8:35   0  4.7G  0 part /var/mydatabase
```
Если ошибок нет, то запись в /etc/fstab выполнена верно.
8) Перезагрузка системы, чтобы окончательно убедиться что диски монтируются при перезагрузке
```bash
sudo reboot
```
```bash
sudo lsblk
```