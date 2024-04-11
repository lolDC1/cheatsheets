### Links
- **[README](../README.md)**
- **Ansible** ([English](../en/ansible-en.md)/[Russian](ansible-ru.md)): Automation of configuration and system management.
- **Elasticsearch** ([English](../en/elastic-en.md)/[Russian](elastic-ru.md)): Search and data analysis.
- **Kubernetes** ([English](../en/kube-en.md)/[Russian](kube-ru.md)): Container orchestration and cluster management.
- **Regular Expressions** ([English](../en/regex-en.md)/[Russian](regex-ru.md)): A powerful tool for working with text.
- **Ubuntu** ([English](../en/ubuntu-en.md)/[**Russian**](ubuntu-ru.md)): Basic settings and recommendations for using Ubuntu.
- **HashiCorp Vault** ([English](../en/vault-en.md)/[Russian](vault-ru.md)): Management of secrets and access.

# Комманды

## Расширение раздела на все доступное место на диске

Список доступных дисков и разделов:
```bash
sudo fdisk -l
```

Изменить размер раздела:
```bash
sudo parted /dev/sda
resizepart 3 #номер раздела sda3
100%
```

Изменить размер физического тома LVM (PV), чтобы распознать увеличенное пространство:
```bash
sudo pvresize /dev/sda3
```

После изменения размера PV нужно расширить LV (/dev/ubuntu-vg/ubuntu-lv):
```bash
lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
```

Изменяем файловую систему:
```bash
resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

Проверяем:
```bash
df -h
```
## Показать занятое место файлами директории

```bash
du -h DIR | sort -rh | head -n 10
```

## Автопродление SSL

Открываем планировщик задач:
```bash
crontab -e
```
**ВАЖНО:** Эта строка указывает cron выполнять команду certbot renew --quiet ежемесячно в полночь первого числа месяца. Вы можете настроить расписание в соответствии с вашими предпочтениями.
```
0 0 1 * * certbot renew --quiet
```

## Создание пары ssh ключей

Генерация самого ключа:
```bash
ssh-keygen -t ed25519 -C "коммент"
```

**ВАЖНО:** На данные 2018 года ed25519 самый рекомендуемый
тип алгоритма shh ключей.

Копируем ПУБЛИЧНЫЙ ключ на требуемый сервер:
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub *ipadd*
```

**ВАЖНО:** Публичный на том **К КОТОРОМУ** нужен доступ, приватный на том **КОМУ** нужен доступ

Подключиться по определенному ключу:
```bash
ssh -i ~/.ssh/*key* *ipadd*
```

Добавить автоопределение passphrase при любом его запросе:
```bash
eval $(ssh-agent) && ssh-add key
```

Для второго пользователя:
```bash
sudo chown -R $USER:$USER /home/$USER/.ssh
sudo chmod 0700 /home/$USER/.ssh
sudo chmod 0600 /home/$USER/.ssh/authorized_keys
```

## GIT

### Логин в GIT

```bash
git config --global user.name Имя
git config --global user.email почтаГита
git config --global --list
```

### Мерж main в foo ветвь

```bash
git switch foo
git fetch origin main:main (<src>:<dst>)
git merge main
CTRL+S 
CTRL+X
```

## Смена сетевых настроек

```bash
nano /etc/netplan/*.yml 
```
```bash
netplan apply
```


## Docker

### [Установка Docker Engine на Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

### [Пост-установка Docker Engine](https://docs.docker.com/engine/install/linux-postinstall/)

## Очистка кеша

```bash
docker system prune -a -f
sudo systemctl stop docker
sudo rm -rf /var/lib/docker
sudo systemctl start docker
```

## WSL 2
### Сжать VHD, чтобы освободить неиспользуемое пространство WSL
Выключите WSL
```powershell
wsl --shutdown
```

#### Метод 1: [Автоматическая очистка диска (Set sparse VHD)](https://devblogs.microsoft.com/commandline/windows-subsystem-for-linux-september-2023-update/#automatic-disk-space-clean-up-set-sparse-vhd):

Виртуальные жесткие диски WSL (VHD) увеличиваются в размерах по мере их использования, и теперь, когда эта функция включена, они также автоматически уменьшаются в размерах! Этот новый параметр автоматически определяет любой новый виртуальный жесткий диск как разреженный виртуальный жесткий диск, что позволяет автоматически уменьшить его размер.

```powershell
wsl --manage <distro> --set-sparse true
```

Вы также можете добавить следующее в свой .wslconfig (расположенный в каталоге вашего профиля Windows, а не внутри WSL), чтобы любой вновь созданный образ дистрибутива был в режиме sparse:

```
[experimental]
sparseVhd=true
```
**ВАЖНО:** Если этот метод не сработал и вы хотите использовать следующие, вам необходимо задать режим set-sparse в значение false.

#### Метод 2: DISKPART

```powershell
diskpart
select vdisk file="C:\Users\%USERPROFILE%\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu...\LocalState\ext4.vhdx" # default path
compact vdisk
```

#### Метод 3: Hyper-V

- Перейдите в "Включение или отключение функций Windows"
- Включите Hyper-V
- Перезагрузка
- Используйте команду:
    ```powershell
    optimize-vhd -Path C:\Users\%USERPROFILE%\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu...\LocalState\ext4.vhdx -Mode full
    ```

**ВАЖНО:** Если вы получаете сообщение об ошибке "**Virtual hard disk files must be uncompressed and unencrypted and must not be sparse**" [Существует](https://github.com/microsoft/WSL/issues/4103) несколько возможных решений:
- Выключите sparse режим
- Вручную отключите шифрование и сжатие файловой системы:
    ```powershell
    fsutil behavior set disableencryption 1
    fsutil behavior set disablecompression 1
    ```
- Правой кнопкой "LocalState", Свойства, Дополнительно, **выключить** "Сжатие контента..." и "Шифрование контента..."