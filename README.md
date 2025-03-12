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


kafka-playbook.yml
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
