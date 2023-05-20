# sf-task1006-hw-vagrant-postgresql-srv
For Skill Factory study project (B10, HW)

<br>


### 01. Общее описание

```bash
Vagrant конфигурация (Vagrantfile) с помощью которой на хосте с установленной платформой виртуализации Orcale Virtual Box,
создается виртуальная машина на основе базового образа Ubuntu 18.04 полученного из облачного репозитория Vagrant Cloud
с дополнительно установленной и настроенной СУБД PostgreSQL 8.4 доступной на стандарном порту 5432
и созданной тестовой базой данных "vagrant" уровня пользователя СУБД и таблицей "greetings",
которая будет использоваться в тестовом веб-приложении (входит в проектную часть работы).
```

### 02. Порядок работы

```bash
1. скопировать Vagrantfile на физическую хост машину (например 192.168.0.100) на которой установлен Vagrant и Orcale VirtualBox

2. перейти в каталог с Vagrantfile и выполнит команду для начала процесса создания ВМ в VirtualBox на основе инструкций Vagrantfile
   $ vagrant up

3. после успешного создания, ВМ появится в списке VirtualBox под именем "ubuntu1804-pgsql84-srv"

4. в процессе создания ВМ, в терминале будут выводится результаты выполнения шагов настройки из Vagrantfile
   и в итоге отобразится статус сервиса postgresql

5. для подключения к ВМ можно воспользоваться встроенным в Vagrant ssh инструментом, выполнив соотв. команду.
   При этом настройки подключения будут определены автоматически, а в качестве логина и пароля будут использованы "vagrant", "vagrant".
   Эти учетные данные являются встроенными во все .box базовые образы размещенные в облачном репозитории Vagrant Cloud.
   $ vagrant ssh

6. также можно подключиться обычным ssh-клиентом от имени доп. учетной записи "devops" (с авторизацией по ssh-ключу) которая создается в Vagrantfile
   учитывая что 192.168.0.70 - это ip адрес ВМ который закодирован в Vagrantfile (его можно изменить соотв. переменной)
   $ ssh devops@192.168.0.70 -i <путь_к_private_ключу>

7. для проверки работы СУБД PosgreSQL на ВМ 0.70, необходимо чтобы на хосте с которого производится проверка был устанвлен CLI клиент PostgreSQL (psql)
   $ apt install -y wget ca-certificates 2>/dev/null
   $ echo "deb http://apt.postgresql.org/pub/repos/apt/ bionic-pgdg main" >> /etc/apt/sources.list.d/pgdg.list
   $ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
   $ apt update -y 2>/dev/null
   $ apt upgrade -y 2>/dev/null
   $ apt install -y postgresql-client 2>/dev/null
   $ psql --version

   команды проверки доступа к СУБД и БД (1ая выводит версию сервера PostgreSQL, 2ая обращается к тестовой таблице содержашей приветствия на различных языках)
   пароль для авторизации на СУБД от имени пользователя/роли: "vagrant" - "vagrant@pass"
   $ psql -t postgres://192.168.0.70:5432/vagrant?sslmode=disable -U vagrant -c "SELECT version()::varchar(100);" | grep PostgreSQL | awk '{print $1" "$2}'
     результат:
        PostgreSQL 8.4.22    

   $ psql -t postgres://192.168.0.70:5432/vagrant?sslmode=disable -U vagrant -c "SELECT word FROM greetings WHERE lang = 'ENG';" -t | cut -d ' ' -f 2 | sed '/^$/d'
     результат:
        hello
```

### 03. Важные замечания

```bash
В Vagranfile для создания ВМ VirtualBox используеся базовый образ "Ubuntu 18.04" (ubuntu/bionic64).
При выполнении команды "vagrant up" этот образ скачивается и облачного репозитория Vagrant Cloud принадлежащего компании Hashicorp.
В связи с технологическими санкциями доступ к этому ресурсу с Россиийских IP адресов невозможен,
поэтому для успешного выполнения команды необходимо (один из предложенных вариантов, либо иной):

1. Самостоятельно настроить свой прокси VPN сервер, либо найти существующий и настроить ОС так чтобы все HTTP/HTTPS запросы из консольных клиентов
   проходили в прозрачном режиме через этот прокси, осуществляя таким образом подмену публичного ip-адреса
   (этот способ в курсе SkillFactory не рассматривается на момент середины Мая 2023 года, хотя он имеет повышенную актуальность для всех заданий);

2. На любом ПК на котором в бразузере установлен VPN клиент, вручную скачать необходимый базовый образ по ссылке
   https://app.vagrantup.com/ubuntu/boxes/bionic64
   интересует провайдер VirtualBox, поэтому скачиваем базовый образ для него
   https://app.vagrantup.com/ubuntu/boxes/bionic64/versions/20230425.0.0/providers/virtualbox.box

   после этого помещаем скачанный box на Хост на котором установлен Vagrant и VirtualBox
   и вручную добавляем его в локальный репозиторий Vagrant чтобы Vagrant не искал его в облачном репозитории к которому нет доступа
   $ vagrant box add ubuntu/bionic64 /home/devops/vagrant/_boxes/ubuntu/ubuntu/bionic64/v20230425.0.0/virtualbox.box
   $ vagrant box list
     результат:
        ubuntu/bionic64  (virtualbox, 0)

   далее можно выполнять команду формирования и запуска ВМ из Vagrantfile
   $ vagrant up

   теперь при выполнении инструкции 'config.vm.box = "ubuntu/bionic64"'
   базовый образ будет браться сразу из локального репозитория Vagrant, а не скачиваться из облачного
```
