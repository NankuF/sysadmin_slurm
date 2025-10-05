# Контроль сессий пользователей
## Полезные команды
- `w` показывает не только сессии, но и запущенные процессы
- `who` посмотреть всех пользователей, работающих на сервере
- `last` кто когда и откуда входил в систему, а также события перезагрузки

завершить **все** процессы юзера, включая сессию
```bash
pkill -KILL -u user1
```
отключить конкретную сессию
```bash
ps aux | grep sshd
kill -9 <PID>
```
более удобный способ отключить сессию
```bash
who
pkill -KILL -t pts/1
```

## Лимиты на сессию
Настроить лимиты на одновременное создание нескольких сессий пользователем
```bash
sudo nano /etc/security/limits.conf
user1	hard	maxlogins	1
```
Для всех кроме root (руту надо запретить вход вообще через /etc/ssh/sshd_coinfig -> PermitRootLogin no)
```bash
sudo nano /etc/security/limits.conf
*	hard	maxlogins	1
```

## Логирование всех команд пользователей
- Внесение изменений
```bash
sudo nano /etc/profile
# Записывать команды в .bash_history немедленно, даже если несколько сессий
export HISTCONTROL=ignoredups:erasedups
export HISTSIZE=10000
export HISTFILESIZE=20000
export HISTTIMEFORMAT="%Y-%m-%d %H:%M:%S "
shopt -s histappend
PROMPT_COMMAND="history -a; history -n; $PROMPT_COMMAND"
```
- Применение изменений - переоткрыть сессию
```bash
who
pkill -KILL -t pts/1
```
- Запрет на очистку history
```bash
sudo chattr +a /home/*/.bash_history
```
Не работает на XFS, ZFS, Btrfs (там свои механизмы), и не работает на NTFS/FAT.<br>
Не защищает от обхода!<br>
Пользователь может:<br>
```text
- Использовать другой shell (zsh, fish) — у них своя история. - тут надо настроить использование только bash. Спроси у нейросетки, там просто.
- Запускать команды с HISTFILE=/dev/null или set +o history.
- Использовать unset HISTFILE.
→ То есть это не панацея, а лишь один из слоёв аудита.
```
- PROFIT!

```text
Объяснение настроек:

HISTCONTROL=ignoredups:erasedups – убирает дубликаты команд.

HISTSIZE=10000 – увеличивает размер истории в памяти.

HISTFILESIZE=20000 – увеличивает размер файла .bash_history.

HISTTIMEFORMAT="%Y-%m-%d %H:%M:%S " – добавляет время выполнения команды.

shopt -s histappend – гарантирует, что история команд не перезаписывается при выходе.

PROMPT_COMMAND="history -a; history -n; $PROMPT_COMMAND" – записывает команду в .bash_history сразу после её выполнения, даже если сессий несколько.

После этого достаточно переоткрыть сессию (выйти и зайти заново) чтобы новые настройки начали работать. Теперь все команды будут записываться в .bash_history сразу, даже при множественных сессиях. Однако это еще не все!

Чтобы пользователи не могли удалять или редактировать историю, можно модифицировать доступ к файлу .bash_history в домашнем каталоге сделав его доступным только для добавление и чтения:

sudo chattr +a /home/*/.bash_history

Теперь history будет записываться, но не может быть изменён пользователем.
```