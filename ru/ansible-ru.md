### Links
- **[README](../README.md)**
- **Ansible** ([English](../en/ansible-en.md)/[**Russian**](ansible-ru.md)): Automation of configuration and system management.
- **Elasticsearch** ([English](../en/elastic-en.md)/[Russian](elastic-ru.md)): Search and data analysis.
- **Kubernetes** ([English](../en/kube-en.md)/[Russian](kube-ru.md)): Container orchestration and cluster management.
- **Regular Expressions** ([English](../en/regex-en.md)/[Russian](regex-ru.md)): A powerful tool for working with text.
- **Ubuntu** ([English](../en/ubuntu-en.md)/[Russian](ubuntu-ru.md)): Basic settings and recommendations for using Ubuntu.
- **HashiCorp Vault** ([English](../en/vault-en.md)/[Russian](vault-ru.md)): Management of secrets and access.

# Комманды
## Создаем inventory файл
### Записываем туда ip адреса серверов к которым ansible требуется иметь соединение:
```
ansible all --key-file ~/.ssh/id_ed25519 -i inventory -m ping
```
- all - Для всех хостов из inventory
- -i inventory - Директория к файлу с адресами:
- -m ping - Проверяем соединение:

### Задаем приватный ключ по которому будем соединяться к остальным серверам (учесть что на требуемых серверах будут соответствующие публичные ключи):
```
--key-file ~/.ssh/id_ed25519 	
```

### Или создаем ansible.cfg с следующим содержимым:
```
[defaults]
inventory = inventory
private_key_file = ~/.ssh/id_ed25519
```
### И запускаем:
```
ansible all -m ping
```

## Лист хостов
```
ansible all --list-hosts
```
- all - Для всех хостов из inventory

## Лист информации с хостов
```
ansible all -m gather_facts
```
- all - Для всех хостов из inventory
- --limit *ip* - Для инфы опред хоста

## Обновление базы пакетов apt update для всех хостов
```
ansible all -m apt -a update_cache=true --become --ask-become-pass
```
- all - Для всех хостов из inventory
- -m apt - используем апт модуль
- -a update_cache=true - apt update (то же самое)
- --become - sudo по дефолту
- --ask-become-pass - запросить ввод пароля для sudo

## Установка пакета apt install
```
ansible all -m apt -a name=*package* --become --ask-become-pass
```
- all - Для всех хостов из inventory
- -m apt - используем апт модуль
- -a name=*package* - имя пакета (tree, net-tools)
- --become - sudo по дефолту
- --ask-become-pass - запросить ввод пароля для sudo

## Обновление до последней версии пакета или установка конктретной
```
ansible all -m apt -a "name=*package* state=latest" --become --ask-become-pass
```
- all - Для всех хостов из inventory
- -m apt - используем апт модуль
- -a "name=*package* state=latest" - Имя, версия пакета 
- --become - sudo по дефолту
- --ask-become-pass - запросить ввод пароля для sudo

## Обновление всех пакетов на всех серверах
```
ansible all -m apt -a upgrade=dist --become --ask-become-pass
```
- all - Для всех хостов из inventory
- -m apt - используем апт модуль
- -a upgrade=dist - apt dist-upgrade (то же самое)
- --become - sudo по дефолту
- --ask-become-pass - запросить ввод пароля для sudo

# Плейбуки
## Создание плейбука
### Плейбуки являются yml-форматными документами вида:
#### Пример плейбука по установке apache2:
```
---

- hosts: kaban@192.168.0.122
become: true
tasks:
- name: install apache2 package
	apt:
	name: apache2
	state: latest"
```
Пример плейбука по удалению apache2:
```
...
state: absent
...
```
## Запуск плейбука
```
ansible-playbook --ask-become-pass *playbook_name*.yml
```
- --ask-become-pass - запросить ввод пароля для sudo

## Обновление репозиториев
	"update_cache: yes"

## Условия
### When условие можно использовать ориентируясь на данных полученных из gather_facts
#### Если нужно определить конкретную ОС для выполнения комманд:
```
when: ansible_distribution == "Ubuntu"
```
#### Для списка ОС:
```
when: ansible_distribution in ["Debian", "Ubuntu"]
```

#### Несколько условий:
```
when: ansible_distribution == "Ubuntu" and ansible_distribution_version == "22.04"
```

## Переменные
### Переменные определяются в inventory файле хостов напротив каждого хоста отдельно в виде:
```
*ip* apache_package=apache2 php_package=libapache2-mod-php
```
### Конфигурация плейбука:
```
- name: install apache2, php packages
package:
name: 
	- "{{ apache_package }}"
	- "{{ php_package }}"
```
**ВАЖНО** учесть что apt не используется на некоторых системах, по этому следует использовать package

## Переменные хоста
В корне проекта создается папка host_vars, в которой для каждого требуемого хоста
создается отдельный конфиг с переменными, имя конфига должно иметь адрес хоста (ДНС/АЙПИ..)

### Вид конфига *host-ipadd*.yml:
```
apache_package_name: apache2
apache_service_name: apache2
php_package_name: libapache2-mod-php
```
### Далее переменные вызываются в конфигурации:
```
- "{{ apache_package_name }}"
- "{{ php_package_name }}"
```
## Группы 
### inventory
```
[web-servers]
*ipadd1*
*ipadd2*

[db-servers]
*ipadd3*

[file-servers]
*ipadd4*
```
### playbook
```
- hosts: [testhosts]
```
## Приоритизация задач
Если требуется какую-то задачу (tasks:) выполнить всегда в первую очередь то используется pre_tasks: вместо привычного значения

## Теги
Если стоит задача выполнить лишь определенные таски, по ключевому слову то используются теги. Например, если у нас имеются 3 разных таски для разных хостов, и мы хотим выполнить только ту которая имеет тег "apache" то мы добавляем ключ "--tags apache" или несколько тегов '--tags "apache,ubuntu"' в вызове комманды. 
### При этом в файле плейбука на уровне package должен быть определен этот тег:
```
tags: apache, apache2, ubuntu
```
### Если у нас есть таска, которая дожна выполняться всегда, не зависимо попадает она под вызваный тег или нет, мы определяем в плейбуке следующий тег:
```
--tags always
```
## Управление файлами
### При копировании:
```
copy: 
	src: temp_site.html 			#что
	dest: /var/www/html/index.html 	#куда
	owner: root 					#овнер
	group: root 					#группа
	mode: 0644 						#разрешения
```
### При установке АРХИВА из интернета:
```
- name: installing unzip		#установка unzip для разархивирования пакета
	package:
		name: unzip
- name: installing terraform	#пример установки terraform
	unarchive:					#разархивирование
		src: https://releases.hashicorp.com/terraform/1.5.7/terraform_1.5.7_linux_amd64.zip	#линк
		dest: /usr/local/bin 	#дира для бинариев 3рд парти приложений
		remote_src: yes			#уточняем что соурс от куда берем файл ремоут чтобы он не чекал на их наличие
		owner: root 			#овнер
		group: root 			#группа
		mode: 0755				#разрешения
```
### При скачивани обычного файла
```
get_url:					
	url: юрл
		dest: /var/www/html/index.html				
		owner: root 				
		group: root 				
		mode: 0644 	
```
### Изменить текстовый файл
```
lineinfile:
	path: *pathtofile*		#путь к файлу
	regexp: '^.*line.*$'	#регулярка на поиск линии начинающийся с "text"
	line: text was changed	#на что заменить
	backrefs: yes			#заменяем все совпадения
	register: variable 		#создаем переменную которая регистрирует изменения в этой секции, ее можно будет вызвать для рестарта сервиса по условию when: variable.changed
```
## Управление процессами
```
- name: launch process
	tags: name
	service:
		name: *process*
		state: started	#статус (reloaded/restarted/started/stopped)
		enabled: yes	#запомнить состояние при запуске
```
## Управление юзерами
Создаем bootstrap.yml который мы будем запускать при первом коннекте ансибла к хосту для его первоначальной настройки, создания юзеров и тд.
```
- name: create Ansible user
tags: user
user:
	name: ans_user			#имя юзера
	groups: root			#группа

- name: adding ssh key
tags: user
authorized_key:				#создаем там публичный ключ с доступом к нам
	user: ans_user	
	key: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPjSzavMyC4N0cCBWn66MXLDSYu5RpjA/5G4/IuCBBe/ ansible"

- name: add sudoers file for user	#права рута чтобы не запрашивало пасс
tags: user
copy:
	src: sudoers_ans_user			#создаем файл с подобным именем
	dest: /etc/sudoers.d/ans_user	#кидаем его в диру
	owner: root
	group: root
	mode: 0440						#ридонли
```
Не забываем создать файл sudoers.d/ans_user в с содержимым "ans_user ALL=(ALL) NOPASSWD: ALL"

## Регистрация результата секции в переменную
```
register: variable	#Регистрируем результат секции (может быть вообще любым зависимо от наших действий)
```
### Для проверки изменения:
```
when:
	variable.changed	#Реагируем на изменение
	ИЛИ
	variable.failed		#Реагируем на фейл
```

## Роли
### Чтобы не держать весь код в одном файле, а логически его разделить на подфайлы нужны роли. В корне проекта создаем директорию roles, зависимо от требований создаем директории для требуемых ролей (web-servers, test-hosts, db-servers), и в каждой из них tasks диру и main.yml файл:
```
roles/
├── base
│   └── tasks
│       └── main.yml
├── busybee
│   └── tasks
│       └── main.yml
└── testhosts
	└── tasks
		└── main.yml
```
### В основном плейбуке теперь примерно следующее содержимое:
```
---

- hosts: all
become: true
pre_tasks:
- name: cache update Ubuntu
	tags: always
	package:
	update_cache: yes
	changed_when: false
	when: ansible_distribution == "Ubuntu"

- hosts: all
become: true
roles:
	- base

- hosts: busybee
become: true
roles:
	- busybee

- hosts: testhosts
become: true
roles:
	- testhosts
```
### Оставляем апдейт кеша, отводим роль base на все хосты, роль busybee для группы busybee роль testhosts для группы testhosts. Далее в каждом из конфигов ролей пишем отдельные скрипты отведенные только для них:
```
roles/testhosts/tasks/main.yml
	- name: install apache2 & php packages
		tags: apache, apache2, ubuntu
		package: 
			name: 
			- "{{ apache_package_name }}"
			- "{{ php_package_name }}"
			state: latest
```
## Хандлеры (функции)
### В корне папки роли можно создать поддиректорию для нашего хандлера, если у нас есть	функция которая может в будущем вызваться повторно логично вынести ее в отдельный файл и вызывать его по необходимости одной коммандой:

```
- name: change files line
	tags: line
	lineinfile:
		path: /var/www/html/index.html
		regexp: '^<h1>Lorem ipsum</h1>'
		line: <h1>HELLOWORLD2</h1>
	notify: restart_apache #пример вызова хандлера
```

### В директории роли создаем директорию handlers, создаем в ней файл maim.yml со следующим	содердимым:
```
- name: restart_apache
	service:
		name: "{{ apache_service_name }}"
		state: restarted
```
Как видим этот хандлер рестартит сервис апача обращаясь к нему по переменной

## Шаблоны
Если требуется перенести конфигурационный файл на группу серверов с некоторыми отличиями в параметрах создаем диру templates в требуемой роли, создаем там необходимый шаблон (.j2) конфигурации, редактируем его добавляя нужные переменные или условия.
### Пример конфигурации sshd_config файла:
```
...
UsePAM yes

AllowUsers {{ ssh_users }}	<--- новая строка

#AllowAgentForwarding yes
#AllowTcpForwarding yes
...
```
### Данная переменная дает нам возможность настроить конфигурацию ssh под каждого уникального хоста отдельно. Переменную определяем в host_vars всех хостов:
```
host-ipadd.yml
	ssh_users: "user ans_user"
	ssh_template_file: sshd_config_ubuntu.j2
```
На примере этого хоста видим какое значение мы ему определили.
### Далее создаем таску на копирование шаблона на все хосты:
```
- name: generate sshd_config file
	tags: ssh
	template:
		src: "{{ ssh_template_file }}"
		dest: /etc/ssh/sshd_config
		owner: root
		group: root
		mode: 0644
	notify: restart_sshd
```
При этом используя функцию рестарта ssh сервиса.