### Links
- **[README](../README.md)**
- **Ansible** ([English](ansible-en.md)/[Russian](../ru/ansible-ru.md)): Automation of configuration and system management.
- **Elasticsearch** ([English](elastic-en.md)/[Russian](../ru/elastic-ru.md)): Search and data analysis.
- **Kubernetes** ([English](kube-en.md)/[Russian](../ru/kube-ru.md)): Container orchestration and cluster management.
- **Regular Expressions** ([English](regex-en.md)/[Russian](../ru/regex-ru.md)): A powerful tool for working with text.
- **Ubuntu** ([**English**](ubuntu-en.md)/[Russian](../ru/ubuntu-ru.md)): Basic settings and recommendations for using Ubuntu.
- **HashiCorp Vault** ([English](vault-en.md)/[Russian](../ru/vault-ru.md)): Management of secrets and access.

# Commands

## File system extention

List of volumes:
```bash
sudo fdisk -l
```

Resize partition:
```bash
sudo parted /dev/sda
resizepart 3 # chose partition number (sda3)
100%
```

Resize the PV:
```bash
sudo pvresize /dev/sda3
```

Then you need to extend LV:
```bash
lvextend -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
```

Resize FS:
```bash
resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv
```

Check:
```bash
df -h
```
## Show sorted list of space used by the files in specified directory

```bash
du -h /DIR | sort -rh | head -n 10
```

## Autoextend SSL

Open crontab:
```bash
crontab -e
```

This line tells cron to execute command at midnight first date of each month. You can adjust schedule according to your preferences.
```
0 0 1 * * certbot renew --quiet
```

## Creating an ssh key

Generate the key:
```bash
ssh-keygen -t ed25519 -C "comment"
```

**NOTE:** According to 2018 year information ed25519 is the most recomended type of ssh keys algorithmes.

Copy public key to desired host:
```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub *ipadd*
```

**NOTE:** Public key is used on host that you want to get an access to, private key used on host you want **grant** access to this host.

Connect via ssh key:
```bash
ssh -i ~/.ssh/*key* *ipadd*
```

Add an passphrase auto filling whenever the key is requested:
```bash
eval $(ssh-agent) && ssh-add key
```

Example for the second user:
```bash
sudo chown -R $USER:$USER /home/$USER/.ssh
sudo chmod 0700 /home/$USER/.ssh
sudo chmod 0600 /home/$USER/.ssh/authorized_keys
```

## GIT

### Login to GIT

```bash
git config --global user.name
git config --global user.email
git config --global --list
```

### Merge main into foo branch

```bash
git switch foo
git fetch origin main:main (<src>:<dst>) # allows you to update any branch while being on different one
git merge main
```

## Net configuration 

```bash
nano /etc/netplan/your_netplan_file.yml 
```
```bash
netplan apply
```


## Docker

### [Installing Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

### [Post-installing Docker Engine](https://docs.docker.com/engine/install/linux-postinstall/)

### Clear cash 

```bash
docker system prune -a -f
sudo systemctl stop docker
sudo rm -rf /var/lib/docker
sudo systemctl start docker
```