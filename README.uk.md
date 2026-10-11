# Встановлення Percona Server for MongoDB: replica set з одним вузлом

[English](README.md) | [Українська](README.uk.md)

## Вимоги

- **Ansible Core 2.15 або вище** — на локальній машині.
- **Колекція `community.mongodb` для Ansible** — на локальній машині.
- **PyMongo 4.8.0** — на віддалених серверах. Інсталюється під час виконання плейбуку.

## Початкове налаштування середовища (Ubuntu)

### 1. Встанови Ansible і колекцію community.mongodb

```bash
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible-core
ansible-galaxy collection install community.mongodb
```

### 2. Створи робочу директорію

```bash
mkdir -p ~/LustyKrusty_github
```

### 3. Клонуй репозиторій

Для клонування через SSH потрібен SSH-ключ, доданий до твого облікового запису GitHub.

```bash
cd ~/LustyKrusty_github/
git clone git@github.com:LustyKrusty/percona_mongodb_install_one_node_replica.git
```

### 4. Зашифруй secrets.yaml

Заповни `secrets.yaml` своїми значеннями, а потім зашифруй файл за допомогою Ansible Vault:

```bash
cd ~/LustyKrusty_github/percona_mongodb_install_one_node_replica/
ansible-vault encrypt secrets.yaml
```

Пароль, заданий під час шифрування, знадобиться для запуску плейбуку.

### 5. Налаштуй ansible.cfg

```bash
cd ~/LustyKrusty_github/percona_mongodb_install_one_node_replica/
cp ansible.cfg_example ansible.cfg
nano ansible.cfg
```

Вкажи свої значення для таких параметрів:

```ini
remote_user = YOUR_SSH_USER
private_key_file = /path/to/your/private_key
```

- `remote_user` — користувач для SSH-підключення до віддалених серверів.
- `private_key_file` — шлях до твого приватного SSH-ключа.

## Правила заповнення hosts

Додаючи новий сервер до inventory-файлу `hosts`, дотримуйся такого формату:

```ini
[prod_hetzner]
hetzner_mongodb.local rs_prio=10 rs_role=primary env=prod data_center=amazon location=eu-central-a
```

Замість DNS-імені можна вказати IP-адресу, наприклад `172.1.1.1`. Значення групи та змінних у прикладі заміни відповідно до свого середовища.

| Поле | Опис |
| --- | --- |
| `[prod_hetzner]` | Назва групи серверів. Використовуй префікс сервера відповідно до своєї схеми іменування. |
| `hetzner_mongodb.local` | DNS-імʼя або IP-адреса сервера. |
| `rs_role` | Роль сервера: `primary`, `secondary` або `mongos`. Ролі `primary` і `secondary` стосуються replica set; `mongos` — маршрутизатор шардованого кластера. |
| `env` | Середовище, наприклад `prod`. |
| `data_center` | Хмарний провайдер, наприклад `amazon`, `google` або `hetzner`. |
| `location` | Розташування сервера відповідно до твоєї схеми позначень, наприклад `eu-central-a`. |
| `rs_prio` | Пріоритет вузла під час виборів primary, наприклад `10`, `5` або `0`. Для бажаного primary задай пріоритет, вищий за пріоритет кожного іншого вузла. Вузол із пріоритетом `0` не може стати primary. |


## Приклад запуску

```bash
cd ~/LustyKrusty_github/percona_mongodb_install_one_node_replica/
ansible-playbook install_mongodb.yaml --ask-vault-pass
```

Коли зʼявиться запит, введи пароль Ansible Vault, заданий під час шифрування `secrets.yaml`.

