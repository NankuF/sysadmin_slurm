# Защита от брутфорс атак на SSH

### Настроим защиту от перебора паролей (fail2ban) (SSH brute-force)

```bash
sudo apt install fail2ban -y
sudo systemctl enable --now fail2ban
```
- Пользуемся тем, что в Debian и Ubuntu защита SSH включается “из коробки”, но проверяем это:
```bash
ls -l /etc/fail2ban/jail.d/defaults-debian.conf
cat /etc/fail2ban/jail.d/defaults-debian.conf
```
- Добавляем секцию, переопределяют настройки по умолчанию
```bash
sudo nano /etc/fail2ban/jail.d/defaults-debian.conf

!!!!! КОММЕНТАРИИ НЕОБХОДИМО УДАЛИТЬ !!!!
[DEFAULT]
bantime = 86400         # 24 часа
findtime = 300         	# 5 минут
maxretry = 3           	# 3 попытки
```
```bash
sudo systemctl restart fail2ban
```
- Проверяем что в логах демона нет ошибок и клиент видит включенную ловушку:
```bash
sudo cat /var/log/fail2ban.log
sudo fail2ban-client status sshd
```

