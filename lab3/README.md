# Лабораторная работа 3. Создание балансировщика нагрузки на haproxy с настройкой round robin и VIP (виртуального ip-адреса).
В этой лабораторной работе был написан ansible-сценарий, в котором настраиваются балансировщики нагрузки на haproxy с повышенной отказаустойчивостью при помощи VIP (virual ip) и веб-сервера.

# Файлы в лабораторной работе
## Vagrantfile
Этот файл не влияет на работу ansible-сценария, а служит инструментом проверки. Позволяет организовать работу 3 виртуальных машин (1 сервер балансировки, 2 веб-сервера). Является необязательным.


## inventory
Это самый основной конфигурационный файл. 
В нем есть две группы хостов:
* webservers

    Содержит в себе имя хоста (первым параметром) и ip-адрес сервера.
    
    Сервера из этой группы подгружают html-страницу сайта при помощи nginx.

* balancer

    Содержит в себе имя хоста первым параметром , ip-адрес сервера и параметры приоритета (необходим для конфигурационного файла в будущем. см. "Папку roles")
    на себе будут содержать балансировщик трафика соответвественно. 

## Ansible.cfg
Для работы ansible может требовать большое количество параметров.

Для более понятной передачи системной информации был создан ansible.cfg файл.

Содержит в себе следующие параметры:
1. Указываем inventory файл
2. Указываем пользователя, с которомы мы работаем
3. Нужно ли осуществлять host_key_checking
4. Параметр defualt_transport. (параемтр smart ставит автоматическое переключение между ssh и paramiko в зависимости от ОС)

## haproxy.yml
Основной ansible playbook для настройки серверов.
Содержит в себе запуск 3 ролей, о которых будет прописано далее. Каждая из них запускается в зависимости от принадлежности хоста одной из групп. Общая настройка (Disabled selinux) запускается внезависимости от групп, так как является обязательной настройкой системы.

##  ***Папка roles***
Основная часть лабораторной работы. Содержит в себе три роли, каждая из которых выполняет собственную задачу. Подробнее о каждой роли ниже.

### 1. selinux-disabled
Общая настройка всех устройств в этой схеме. Отключает системную защиту, препядствующую работе серверов и перезагружает сервер.
Результат для трех пк: 3 changed

### 2. balancer
Эта роль осуществляет настройку серверов-балансировки.

Устанавливает haproxy и меняет его конфигурационный файл, настраивая балансировщик в режим round robin. Шаблон конфигурационного файла лежит в папке templates.

Для повышения отказаустойчивости системы на эту группу серверов также утсанавливается сервис keepalive для настройки виртуального ip-адреса. Также проводит настойку и меняет кофигурационный файл (шаблон находится в папке templates)

Результат 1 success, 1 changed

### 3. webservers
Настраивает работу веб-серверов.

Устанавливает на них nginx, изменяет конфигурационный файл и меняет html-документ, подгружающийся на этот сервер.

Для наглядности html-документ изменен таким образом, чтобы увидеть работоспособность распределителя нагрузки.


# **Запуск и исопльзование ansible playbook**
Этапы:
## 1. inventory
Первым этапом необходимо изменить inventory-файл. В него внести ip-адреса и имена хостов.

## 2. Шаблоны
Этот этап не является обязательным, однако в некоторых ситуациях его необходимо учитывать.

В /roles/webserver/templates необходимо изменть шаблон html-документа (файл index.html.j2).

## 3. Запуск playbook
Если необходимо полностью установить все компоненты, описанные в ролях, нужно написать в терминале 
"ansible-playbook haproxy.yml"
Эта команда выполнит все task файлы из всех ролей. 

Однако не всегда некоторые параметры являются необходимыми. Для этого написаны специальные теги, по которым можно запустить отдельный элемент.

#### 1. html

Этот тег меняет конфигурацонный файл nginx на веб-серверах

#### 2. balancer

Выполняет настройку серверов-балансировщиков

#### 3. selinux

Выполняет отключение selinux на всех серверах


# ***Результат***
В результате мы настраиваем веб-сервера и балансировщики. При выходи из сторя одного из серверов балансировки, виртуальный ip-адресс переходи автоматически на второй и система работатет без неполадок.

 Для наглядности было создано два разных сайта, чтобы увидеть работаспособность сценария.
![](web1.jpg)
![](web2.jpg)
