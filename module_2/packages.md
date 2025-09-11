#

Узнать какая ОС
cat /etc/*release

найти пакет
apt search vim

узнать инфу пакета
apt show vim

вывести список установленных пакетов
sudo dpkg -l |grep libc6

удалить пакет (удалит только файлы программы, не трогая конфигурационные файлы)
sudo apt remove tree

удалить все что связано с пакетом
sudo apt purge tree

удалить неиспользуемые зависимости (как в программировании переменная на которую никто не ссылается)
sudo apt autoremove

еще вариант удаления через dpgk
sudo dpkg -r filebeat && sudo dpgk --purge filebeat