# Подключение 2FA для линукс сервера по SSH
```bash
sudo apt update
sudo apt install -y libpam-google-authenticator
google-authenticator
```
Рекомендуемые ответы:

- Time-based tokens? → y
- Сохраните QR-код и резервные коды !!!! Чтобы иметь доступ к серверу если телефон будет утерян
- y — обновить файл
- y — запретить повторное использование токенов
- n — "window" ±30 сек
- y — rate-limiting (3 попытки за 30 сек)

```text
При запуске google-authenticator вам выдаются 5 одноразовых резервных кодов (emergency scratch codes).

🔹 Что делать:

Сразу после генерации скопируйте их в надёжное место:
- Менеджер паролей (Bitwarden, 1Password, KeePass)
- Печать на бумаге → хранить в сейфе
- Зашифрованный файл на защищённом устройстве
```


```bash
sudo nano /etc/pam.d/sshd
#@include common-auth
auth required pam_google_authenticator.so
```
```bash
sudo nano /etc/ssh/sshd_config
PasswordAuthentication no
PubkeyAuthentication yes   #можно не включать, тк по умолчанию включено даже если закомментировано
KbdInteractiveAuthentication yes
AuthenticationMethods publickey,keyboard-interactive:pam
```
```bash
sudo systemctl restart ssh
```
Проверить подключение не отключаясь от текущего, через другой терминал