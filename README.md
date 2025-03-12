# ansible-2
Домашнее задание к занятию «Ansible.Часть 2»
Оформление домашнего задания
Домашнее задание выполните в Google Docs и отправьте на проверку ссылку на ваш документ в личном кабинете.
В названии файла укажите номер лекции и фамилию студента. Пример названия: Ansible. Часть 2 — Александр Александров.
Перед отправкой проверьте, что доступ для просмотра открыт всем, у кого есть ссылка. Если нужно прикрепить дополнительные ссылки, добавьте их в свой Google Docs.
Вы можете прислать решение в виде ссылки на ваш репозийторий в GitHub, для этого воспользуйтесь шаблоном для домашнего задания.

Задание 1
Выполните действия, приложите файлы с плейбуками и вывод выполнения.

Напишите три плейбука. При написании рекомендуем использовать текстовый редактор с подсветкой синтаксиса YAML.

Плейбуки должны:

Скачать какой-либо архив, создать папку для распаковки и распаковать скаченный архив. Например, можете использовать официальный сайт и зеркало Apache Kafka. При этом можно скачать как исходный код, так и бинарные файлы, запакованные в архив — в нашем задании не принципиально.
Установить пакет tuned из стандартного репозитория вашей ОС. Запустить его, как демон — конфигурационный файл systemd появится автоматически при установке. Добавить tuned в автозагрузку.
Изменить приветствие системы (motd) при входе на любое другое. Пожалуйста, в этом задании используйте переменную для задания приветствия. Переменную можно задавать любым удобным способом.
Решение 1


playbook-kafka.yml
```rb
---
- name: kafka-playbook
  hosts: my
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
...

Процесс выполнения:

![Снимок 156](https://github.com/user-attachments/assets/fc1340b5-22f2-49f4-9062-b51348bed53a)

playbook-tuned.yml
```rb
---
- name: tuned-playbook
  hosts: my
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

![Снимок 158](https://github.com/user-attachments/assets/70b3ff93-a2cc-4dd5-b421-a23a0240d64a)

![Снимок 157](https://github.com/user-attachments/assets/00cb7ee9-378d-492d-9fe2-565422e690f0)

playbook-motd.yml
```rb
---
- name: motd-playbook
  hosts: my
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
![Снимок 160](https://github.com/user-attachments/assets/74819314-9eba-45e8-a69c-c9d31e54098a)

![Снимок 159](https://github.com/user-attachments/assets/56f6452e-1111-44bf-aca8-4ed7f8ae1c3f)

Задание 2
Выполните действия, приложите файлы с модифицированным плейбуком и вывод выполнения.

Модифицируйте плейбук из пункта 3, задания 1. В качестве приветствия он должен установить IP-адрес и hostname управляемого хоста, пожелание хорошего дня системному администратору.
```rb
---
- name: motd-playbook
  hosts: my
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
Решение 2

![Снимок 162](https://github.com/user-attachments/assets/a4c20df7-f10a-4733-b6bf-21a6ec698d45)

![Снимок 161](https://github.com/user-attachments/assets/371e8e5d-ae7c-451b-b2fd-97759c1090e4)

Задание 3
Выполните действия, приложите архив с ролью и вывод выполнения.

Ознакомьтесь со статьёй «Ansible - это вам не bash», сделайте соответствующие выводы и не используйте модули shell или command при выполнении задания.

Создайте плейбук, который будет включать в себя одну, созданную вами роль. Роль должна:

Установить веб-сервер Apache на управляемые хосты.
Сконфигурировать файл index.html c выводом характеристик каждого компьютера как веб-страницу по умолчанию для Apache. Необходимо включить CPU, RAM, величину первого HDD, IP-адрес. Используйте Ansible facts и jinja2-template. Необходимо реализовать handler: перезапуск Apache только в случае изменения файла конфигурации Apache.
Открыть порт 80, если необходимо, запустить сервер и добавить его в автозагрузку.
Сделать проверку доступности веб-сайта (ответ 200, модуль uri).
В качестве решения:

предоставьте плейбук, использующий роль;
разместите архив созданной роли у себя на Google диске и приложите ссылку на роль в своём решении;
предоставьте скриншоты выполнения плейбука;
предоставьте скриншот браузера, отображающего сконфигурированный index.html в качестве сайта.
Решение 3
Плейбук и роль находятся в данном репозитории
Выполнение плейбука:
![Снимок 165](https://github.com/user-attachments/assets/61001c20-f7ec-4de0-95b3-f20df92f60d7)
![Снимок 166](https://github.com/user-attachments/assets/fed49db7-cbe0-4f14-8e37-9a149f725533)

![Снимок 140](https://github.com/user-attachments/assets/ffb05fab-9de0-4f1a-a46f-5891325144ef)


![Снимок 164](https://github.com/user-attachments/assets/368e28d2-5a7e-4623-b19a-f3eba76f68f9)
![Снимок 163](https://github.com/user-attachments/assets/40f056f6-0962-40f0-876f-340054fab56a)
