# toolkit-infrastructure

Ansible playbook для автоматической настройки VPS с нуля и деплоя [DevOps Toolkit](https://github.com/nulldeploy/toolkit).

---

## Что делает playbook

1. Обновляет кэш пакетов
2. Настраивает UFW firewall (22, 80, 443)
3. Устанавливает Docker
4. Создаёт пользователя `devops` с sudo и docker правами
5. Добавляет SSH ключ пользователю
6. Клонирует репозиторий toolkit
7. Создаёт `.env` из шаблона
8. Создаёт и запускает systemd сервис
9. Проверяет что `/health` отвечает 200

---

## Структура

```
toolkit-infra/
├── inventory.ini          # список серверов
├── playbook.yml           # главный playbook
├── vars/
│   ├── secrets.yml        # зашифрованные переменные (vault, не в git)
│   └── secrets.example.yml # шаблон переменных
└── templates/
    ├── env.j2             # шаблон .env файла
    └── service.j2         # шаблон systemd unit
```

---

## Требования

**Локальная машина:**
- Ansible
- passlib (`pip install passlib`)
- SSH ключ (`~/.ssh/id_ed25519`)

**Сервер:**
- Ubuntu 22.04
- SSH доступ по паролю (для первого запуска)

---

## Быстрый старт

**1. Клонировать репозиторий**

```bash
git clone https://github.com/nulldeploy/toolkit-infrastructure.git
cd toolkit-infrastructure
```

**2. Настроить inventory**

```ini
# inventory.ini
[vps]
toolkit ansible_host=YOUR_IP ansible_user=root ansible_ssh_private_key_file=~/.ssh/id_ed25519
```

При первом запуске (до добавления SSH ключа) добавь пароль:

```ini
toolkit ansible_host=YOUR_IP ansible_user=root ansible_password=ROOT_PASSWORD
```

**3. Создать файл с секретами**

```bash
cp vars/secrets.example.yml vars/secrets.yml
EDITOR=nano ansible-vault create vars/secrets.yml
```

Содержимое:

```yaml
app_usr_passwd: "your_password"
```

**4. Проверить подключение**

```bash
ansible vps -i inventory.ini -m ping
```

**5. Запустить playbook**

```bash
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass
```

---

## Переменные

| Переменная | Описание | Default |
|---|---|---|
| `app_user` | Имя пользователя на сервере | `devops` |
| `app_dir` | Путь к приложению | `/home/devops/toolkit` |
| `repo` | URL репозитория toolkit | `github.com/nulldeploy/toolkit` |
| `app_port` | Порт Flask сервера | `5000` |
| `app_usr_passwd` | Пароль пользователя (vault) | — |

---

## Повторный запуск

Playbook идемпотентен — можно запускать сколько угодно раз:

```bash
ansible-playbook -i inventory.ini playbook.yml --ask-vault-pass
```

Уже настроенные задачи вернут `ok`, изменения применятся только там где нужно.

---