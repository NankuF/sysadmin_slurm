# Работа с ssh

## Добавить ssh на ВМ
1) ssh-keygen -t ed25519 -C "v.poltoranin@example.ru" -f ./test-key

2) через админа ВМ разрешить вход по паролю
sudo nano /etc/ssh/sshd_config
PasswordAuthentication yes
```bash
sudo systemctl restart sshd && sudo systemctl status sshd
```

3) ssh-copy-id -i ./test-key vagrant@192.168.250.10
4) Ограничьте доступ к серверу, разрешив вход только по ключам и запретив вход по паролю в конфигурации
sudo nano /etc/ssh/sshd_config PasswordAuthentication no
PermitRootLogin no
PermitEmptyPasswords no
PubkeyAuthentication yes
Port 3210 - если система торчит в интернет.
+
sudo nano /etc/ssh/sshd_config.d/*.conf
PasswordAuthentication no
```bash
sudo systemctl stop ssh.socket &&\
sudo systemctl disable ssh.socket &&\
sudo systemctl restart ssh &&\
sudo systemctl enable ssh.socket
```

 в убунту 24, чтобы отключить вход по паролю надо выставлять PasswordAuthentication no в 2х местах -
в /etc/ssh/sshd_config.d/*.conf
и в /etc/ssh/sshd_config
и обязательно делать sudo systemctl restart ssh
тогда такая запись не позволит войти по паролю
ssh nixadmin@192.168.0.9 -p 3210 -o PasswordAuthentication=yes -o PubkeyAuthentication=no


5) опционально настроить конфиг для подключения (для машины с которой выполняешь подключение на ВМ)
touch ~/.ssh/config
chmod 600 ~/.ssh/config  # обязательно: только ты можешь читать/писать
Host vagrant
    HostName 192.168.250.10
    User john
    IdentityFile ~/.ssh/test_key
    Port 22


## Добавление ключа вручную, когда PasswordAuthentication no  => ssh-copy-id не сработает

1) создание директорий для нового юзера
mkdir /home/nanku/.ssh &&\
touch /home/nanku/.ssh/authorized_keys &&\
chown -R nanku:nanku /home/nanku &&\
chmod 700 /home/nanku/.ssh &&\
chmod 600 /home/nanku/.ssh/authorized_keys
Копируем публичный ключ в /home/nanku/.ssh/authorized_keys

## Примеры прав
4 - read
2 - write
1 - execute

chmod 600 ~/.ssh/id_rsa                 # приватный ключ
chmod 644 ~/.ssh/id_rsa.pub             # публичный ключ
chmod 600 ~/.ssh/authorized_keys        # ключи для входа
chmod 700 ~/.ssh                        # каталог .ssh
chmod 644 index.html                    # веб-файл
chmod 755 backup.sh                     # скрипт, который можно запускать

```text
я вижу для себя условно 4 ситуации
1. начальная настройка сервера - первый администратор в системе
2. обновление личного ключа на сервер под вашей учетной записью
3. добавление новых пользователей или обновление ключей существующих
4. отзыв ключа или удаление пользователя

---
ситуация 1
 - как уже сказали, ваш пользователь и ключ либо уже есть на сервере после настройки
 - либо у вас есть парольный доступ и вы в 1 команду ssh-copy-id закидываете туда свой ключ (ну либо руками в файлик)
---
ситуация 2
 - аналогично. доступ по ключу у вас уже есть и вам его надо обновить (замена, ротация и пр)
 - т.е. вы новый через команду или руками закинули, проверили что работает, старый убрали
---
ситуация 3

добавляя нового пользователя, мы ему создаем домашний каталог (автоматически), там создаются нужные файлы и пр.
и добавлять ключ руками или через эту команду можно, но тк сложность растет прямо пропорционально числу пользователей - я лично не хочу задалбываться иотдаю это автоматике. если у меня пользователи и так создаются ансиблом- почему бы ему сразу и ключ не положить куда надо.
заодно я исключу человеческую ошибку. например я с утра не выпил кофе и сонный пошел и ключ скопировал криво. потом сиди ищи ошибку.

обновление в таком случае тоже должна делать автоматика
---
ситуация 4
 - вот тут мы тоже либо автоматизируем (отзыв), либо просто щабиваем тк при удалении пользователя весь его профиль будет снесен с сервера
---------

в целом, если сервер у вас работает в закрытом внутреннем контуре и на него не нужно ходить никакой автоматизацией, а у вас много пользователей- проще подключить его с тз авторизации к AD/LDAP и все)
```