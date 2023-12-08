### Table of Contents
- **[README](../README.md)**
- **Ansible** (English/[Russian](ansible-ru.md)): Automation of configuration and system management.
- **Elasticsearch** (English/[Russian](elastic-ru.md)): Search and data analysis.
- **Kubernetes** (English/[Russian](kubectl-ru.md)): Container orchestration and cluster management.
- **Regular Expressions** (English/[Russian](regulars-ru.md)): A powerful tool for working with text.
- **Ubuntu** (English/[Russian](ubuntu-ru.md)): Basic settings and recommendations for using Ubuntu.
- **HashiCorp Vault** ([English](../en/vault-en.md)/Russian): Management of secrets and access.

# Комманды

## Расширение раздела на все доступное место на диске:

- Увеличиваем раздел до 100% диска:
	```bash
	lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
	```
- Отдаём пространство под файловую систему:
	```bash
	resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
	```
- Проверяем:
	```bash
	df -h
	```
## Показать занятое место файлами директории:
```bash
du -h /DIR | sort -rh | head -n 10
```

## Автопродление SSL:
- Открываем планировщик задач :
	```bash
	crontab -e
	```
- Эта строка указывает cron выполнять команду certbot renew --quiet ежемесячно 	в полночь первого числа месяца. Вы можете настроить расписание в соответствии с вашими предпочтениями.
	```
	0 0 1 * * certbot renew --quiet
	```

## Создание пары ssh ключей:
- Генерация самого ключа:
	```bash
	ssh-keygen -t ed25519 -C "коммент"
	```
	- **-t** указывает тип ключа, на данных 2018 года ed25519 самый рекомендуемый
	тип алгоритма shh ключей.

- Копируем ПУБЛИЧНЫЙ ключ на требуемый сервер:
	```bash
	ssh-copy-id -i ~/.ssh/id_ed25519.pub *ipadd*
	```
	- Публичный на том **К КОТОРОМУ** нужен доступ, приватный на том **КОМУ** нужен доступ

- Подключиться по определенному ключу:
	```bash
	ssh -i ~/.ssh/*key* *ipadd*
	```

- Добавить автоопределение passphrase при любом его запросе:
	```bash
	eval $(ssh-agent) && ssh-add key
	```

- Для второго пользователя:
	```bash
	sudo chown -R $USER:$USER /home/$USER/.ssh
	sudo chmod 0700 /home/$USER/.ssh
	sudo chmod 0600 /home/$USER/.ssh/authorized_keys
	```
## Логин в GIT:
```bash
git config --global user.name Имя
git config --global user.email почтаГита
git config --global --list
```

## Смена сетевых настроек:
```bash
nano /etc/netplan/*.yml 
```
```bash
netplan apply
```


## Docker:
### [Установка Docker Engine на Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

### [Пост-установка Docker Engine](https://docs.docker.com/engine/install/linux-postinstall/)

## Очистка кеша:
```bash
docker system prune -a -f
```

## Очистка прочего:
```bash
sudo systemctl stop docker
sudo rm -rf /var/lib/docker
sudo systemctl start docker
```