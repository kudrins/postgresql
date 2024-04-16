## Домашнее задание: Репликация и резервное копирование в СУБД PostgreSQL

### Описание домашнего задания
```
- Настроить hot_standby репликацию с использованием слотов
- Настроить резервное копирование
- Создать ansible playbook с использованием ролей
 
Файлы:
- provision.yml:                   ansible playbook развертывания и настройки VMs
- hosts:                           inventory

Роли:
- install_vms
- install_postgres
- postgres_replication
- install_barman

Описание ролей:
- install_vms
  создание VMs в среде VMware vSphere 7
  VMs разворачиваются из шаблона Centos 8:
  - node1:  сервер Postgresql 16 (master)
  - node12: сервер Postgresql 16 (slave)
  - barman: сервер резервного копирования barman
  Файлы:
  - tasks/main.yml:                task развертывания VMs
  - vars/main.yml:                 заданы переменные доступа к VMware vSphere
                                   заданы параметры создаваемых серверов

- install_postgres
  развертывание сервера Postgresql 16 на VMs node1, node2
  Файлы:
  - tasks/main.yml:                task развертывания серверов Postgresql 16

- postgres_replication
  настройка репликации между хостами node1 (master) и node2 (slave)
  Файлы:
  - defaults/main.yml:             переменные: ip адреса серверов, пароль для user replication
  - tasks/main.yml:                task настройки репликации, с комментариями
  - templates/pg_hba.conf.j2:      шаблон файла конфигурации /var/lib/pgsql/16/data/pg_hba.conf
                                   задаёт способ доступа к базам и репликации из различных источников
								   разрешения и для пользователя barman
  - templates/postgresql.conf.j2:  шаблон файла конфигурации PostgreSQL
                                   /var/lib/pgsql/16/data/postgresql.conf node master
  - templates/postgresql2.conf.j2: шаблон файла конфигурации PostgreSQL
                                   /var/lib/pgsql/16/data/postgresql.conf node slave
								   
- install_barman
  настройка резервного копирования PostgreSQL с помощью утилиты Barman, node1 -> barman
  Файлы:
  - defaults/main.yml:             переменные: ip адреса серверов, users postgresql
  - tasks/main.yml:                task настройки резервного копирования, с комментариями
  - templates/.pgpass.j2:          реквизиты доступа для postgres
                                   /var/lib/barman/.pgpass  
  - templates/barman.conf.j2:      конфигурация barman
                                   /etc/barman.conf 
  - templates/node1.conf.j2:       конфигурация резервного копирования с node1
                                   /etc/barman.d/node1.conf
  
Результаты проверки в скриншотах каталога scrins:
- ansible:                          результат работы ansible-playbook provision.yml
- node1_create_db:                  создание базы postgres otus_test (node1)
- node2_replication                 репликация базы с node1 на node2 (node2) 
- barman_check_node1:               проверка настройки резнрвного копирования с node1 (barman)
- barman-backup-restore:            создание резервной копии c node1 и
                                    восстановление с резервной копии после удаления на node1 баз (barman)
- node1_restore:                    удаление баз и появление снова после восстановления (node1)