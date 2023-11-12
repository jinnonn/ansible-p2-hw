![image](https://github.com/jinnonn/ansible-p2-hw/assets/146999555/b8be93db-fecf-4384-8232-75669b2f285b)# Домашнее задание к занятию «Ansible.Часть 2»

### Оформление домашнего задания

1. Домашнее задание выполните в [Google Docs](https://docs.google.com/) и отправьте на проверку ссылку на ваш документ в личном кабинете.  
1. В названии файла укажите номер лекции и фамилию студента. Пример названия:  Ansible. Часть 2 — Александр Александров.
1. Перед отправкой проверьте, что доступ для просмотра открыт всем, у кого есть ссылка. Если нужно прикрепить дополнительные ссылки, добавьте их в свой Google Docs.

Вы можете прислать решение в виде ссылки на ваш репозийторий в GitHub, для этого воспользуйтесь [шаблоном для домашнего задания](https://github.com/netology-code/sys-pattern-homework).

---

### Задание 1

**Выполните действия, приложите файлы с плейбуками и вывод выполнения.**

Напишите три плейбука. При написании рекомендуем использовать текстовый редактор с подсветкой синтаксиса YAML.

Плейбуки должны: 

1. Скачать какой-либо архив, создать папку для распаковки и распаковать скаченный архив. Например, можете использовать [официальный сайт](https://kafka.apache.org/downloads) и зеркало Apache Kafka. При этом можно скачать как исходный код, так и бинарные файлы, запакованные в архив — в нашем задании не принципиально.
2. Установить пакет tuned из стандартного репозитория вашей ОС. Запустить его, как демон — конфигурационный файл systemd появится автоматически при установке. Добавить tuned в автозагрузку.
3. Изменить приветствие системы (motd) при входе на любое другое. Пожалуйста, в этом задании используйте переменную для задания приветствия. Переменную можно задавать любым удобным способом.

---

### Решение 1
---
kafka-playbook.yml
```
---
- name: kafka-playbook
  hosts: k8s-nodes
  tasks:
    - name: Creating folder for Kafka
      ansible.builtin.file:
         path: /tmp/downloaded-kafka/
         state: directory
         mode: u+rw,g+rw,o+rw
      tags:
        - folder
      become: true
    - name: Downloading Kafka archive 
      ansible.builtin.get_url:
        url: https://downloads.apache.org/kafka/3.6.0/kafka_2.12-3.6.0.tgz
        dest: /tmp/downloaded-kafka/
      tags:
        - download
    - name: Unarchive kafka archive
      ansible.builtin.unarchive:
        src: /tmp/downloaded-kafka/kafka_2.12-3.6.0.tgz
        dest: /tmp/downloaded-kafka/
        remote_src: true
      tags:
        - unarchive
```
Процесс выполнения:
![task1](https://github.com/jinnonn/ansible-p2-hw/blob/main/изображение_2023-11-10_020333302.png)

tuned-playbook.yml
```
---
- name: tuned-playbook
  hosts: k8s-nodes
  tasks:
    - name: Installing tuned with apt
      ansible.builtin.apt:
        name: tuned
        update_cache: yes
      become: true
      tags:
        - download
    - name: Enabling and Starting a tuned.service if it's not running
      ansible.builtin.service:
        name: tuned
        state: started
        enabled: yes
      tags:
        - service
```
Процесс выполнения:
![task1](https://github.com/jinnonn/ansible-p2-hw/blob/main/изображение_2023-11-10_020807529.png)
![task1](https://github.com/jinnonn/ansible-p2-hw/blob/main/изображение_2023-11-10_020834807.png)
![task1](https://github.com/jinnonn/ansible-p2-hw/blob/main/изображение_2023-11-10_020906351.png)
![task1](https://github.com/jinnonn/ansible-p2-hw/blob/main/изображение_2023-11-10_020928691.png)
---

motd-playbook.yml
```
---
- name: motd-playbook
  hosts: k8s-nodes
  become: yes
  tasks:
    - name: Create 99-hello for adding "Hello Netology!" to motd
      ansible.builtin.lineinfile:
        path: /etc/update-motd.d/99-hello
        line: '{{ item }}'
        mode: '755'
        create: yes
      with_items:
        - '#!/bin/bash'
        - 'echo -e "Hello Netology!"'
      tags:
        - motd-change
    - name: Checknig motd changes
      ansible.builtin.command: 
        cmd: run-parts /etc/update-motd.d
      register: output
      tags:
        - check-1
    - name: Debugging motd changes
      ansible.builtin.debug:
        var: output.stdout_lines
      tags:
        - check-2
```
Процесс выполнения:
![task1](https://github.com/jinnonn/ansible-p2-hw/blob/main/изображение_2023-11-10_030246666.png)
![task1](https://github.com/jinnonn/ansible-p2-hw/blob/main/изображение_2023-11-10_030313711.png)

### Задание 2

**Выполните действия, приложите файлы с модифицированным плейбуком и вывод выполнения.** 

Модифицируйте плейбук из пункта 3, задания 1. В качестве приветствия он должен установить IP-адрес и hostname управляемого хоста, пожелание хорошего дня системному администратору. 

### Решение 2
---
```
---
- name: motd-playbook
  hosts: test
  become: yes
  tasks:
    - name: Create 99-hello for adding "Hello Netology!" to motd
      ansible.builtin.lineinfile:
        path: /etc/update-motd.d/00-header
        regexp: '^printf'
        line: '{{ item }}'
      with_items:
        - 'printf "Welcome to $( hostname ) {{ ansible_default_ipv4.address }}. Have a great day dear administrator!"'
      tags:
        - motd-change
    - name: Checknig motd changes
      ansible.builtin.command: 
        cmd: run-parts /etc/update-motd.d
      register: output
      tags:
        - check-1
    - name: Debugging motd changes
      ansible.builtin.debug:
        var: output.stdout_lines
      tags:
        - check-2
```
Процесс выполнения:
![task2](https://github.com/jinnonn/ansible-p2-hw/blob/main/изображение_2023-11-12_051554909.png)
![task2](https://github.com/jinnonn/ansible-p2-hw/blob/main/изображение_2023-11-12_051641300.png)

### Задание 3

**Выполните действия, приложите архив с ролью и вывод выполнения.**

Ознакомьтесь со статьёй [«Ansible - это вам не bash»](https://habr.com/ru/post/494738/), сделайте соответствующие выводы и не используйте модули **shell** или **command** при выполнении задания.

Создайте плейбук, который будет включать в себя одну, созданную вами роль. Роль должна:

1. Установить веб-сервер Apache на управляемые хосты.
2. Сконфигурировать файл index.html c выводом характеристик каждого компьютера как веб-страницу по умолчанию для Apache. Необходимо включить CPU, RAM, величину первого HDD, IP-адрес.
Используйте [Ansible facts](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_vars_facts.html) и [jinja2-template](https://linuxways.net/centos/how-to-use-the-jinja2-template-in-ansible/). Необходимо реализовать handler: перезапуск Apache только в случае изменения файла конфигурации Apache.
4. Открыть порт 80, если необходимо, запустить сервер и добавить его в автозагрузку.
5. Сделать проверку доступности веб-сайта (ответ 200, модуль uri).

В качестве решения:
- предоставьте плейбук, использующий роль;
- разместите архив созданной роли у себя на Google диске и приложите ссылку на роль в своём решении;
- предоставьте скриншоты выполнения плейбука;
- предоставьте скриншот браузера, отображающего сконфигурированный index.html в качестве сайта.

### Решение 3
- Плейбук и роль находятся в данном репо
- Выполнение плейбука:
  ![task3](https://github.com/jinnonn/ansible-p2-hw/blob/main/изображение_2023-11-12_195951102.png)
  ![task3](https://github.com/jinnonn/ansible-p2-hw/blob/main/изображение_2023-11-12_200045622.png)
- Страничка в браузере (P.S. содержимое получено по публичному адресу т.к. на используемых виртуалках, которые находятся в одной локальной сети, нет GUI и браузеров):
  ![task3](https://github.com/jinnonn/ansible-p2-hw/assets/146999555/2d3c1169-d36f-491f-8cb4-bb904064d54a)
  ![task3](https://github.com/jinnonn/ansible-p2-hw/assets/146999555/00f2d4f3-8632-4751-a02b-39c0a1a4af06)


