# Установка обновлений безопасности в системе

Чтобы просмотреть доступные обновления безопасности, выполните несколько команд.

- Обновите список пакетов:
```bash
sudo apt update
```
- Просмотрите доступные обновления безопасности:
```bash
apt list --upgradable | grep -i security
```

### Установка обновлений безопасности
Чтобы установить только обновления безопасности, выполните следующие шаги:

Используйте пакет unattended-upgrades, который специально предназначен для автоматического управления обновлениями, в том числе обновлениями безопасности (по умолчанию он ставит только их).

- Проверьте, что он установлен:
```bash
sudo apt -y install unattended-upgrades
```
- Настройте unattended-upgrades для установки только обновлений безопасности. Для этого отредактируйте файл конфигурации:
```bash
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```
- Найдите и убедитесь, что строки для обновлений безопасности не закомментированы (в отличие от остальных, иначе можно получить новую версию софта на продакшен-сервере, которая не совместима с вашим конфигом):

```text
// Automatically upgrade packages from these (origin:archive) pairs
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
    // "${distro_id}:${distro_codename}-updates";
    // "${distro_id}:${distro_codename}-proposed";
    // "${distro_id}:${distro_codename}-backports";
};
```

- Запустите unattended-upgrades для немедленной установки обновлений безопасности.
Сначала проверим в тестовом режиме:
```bash
sudo unattended-upgrades --dry-run --debug
```
- Если всё работает правильно, удалите флаги --dry-run и --debug для настоящей установки:
```bash
sudo unattended-upgrades
```
### Проверка установленных обновлений безопасности
- После установки обновлений безопасности, вы можете проверить, какие пакеты были обновлены, используя лог файл:
```bash
sudo less /var/log/unattended-upgrades/unattended-upgrades.log
```
- А также проверить, что не осталось доступных обновлений безопасности:
```bash
apt list --upgradable | grep -i security
```
Вывод должен быть пуст.