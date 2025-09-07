# Домашнее задание к занятию «Ansible.Часть 2»-Kadancev Vladimir

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
```
---
- name: Скачать и распаковать архив Apache Kafka
  hosts: servers
  become: yes
  vars:
    kafka_version: "3.6.1"
    kafka_scala_version: "2.13"
    download_url: "https://dlcdn.apache.org/kafka/4.0.0/kafka_2.13-4.0.0.tgz"
    install_dir: "/opt/kafka"
    temp_dir: "/tmp"

  tasks:
    - name: Создать папку для установки
      file:
        path: "{{ install_dir }}"
        state: directory
        mode: '0755'

    - name: Скачать архив Kafka
      get_url:
        url: "{{ download_url }}"
        dest: "{{ temp_dir }}/kafka.tgz"
        mode: '0644'

    - name: Распаковать архив
      unarchive:
        src: "{{ temp_dir }}/kafka.tgz"
        dest: "{{ install_dir }}"
        remote_src: yes
        extra_opts: ["--strip-components=1"]

    - name: Установить права на файлы
      file:
        path: "{{ install_dir }}"
        state: directory
        mode: '0755'
        recurse: yes

    - name: Удалить временный архив
      file:
        path: "{{ temp_dir }}/kafka.tgz"
        state: absent

```
![Скриншот команды ansible-playbook -i production.ini download_and_extract.yml](https://github.com/valdemar-2502/Ansible-Homework/blob/main/demo_ansible_downloads_kafka.png)
![Скриншот команды ansible-playbook -i production.ini download_and_extract.yml](https://github.com/valdemar-2502/Ansible-Homework/blob/main/demo_vm-1-kafka.png)
![Скриншот команды ansible-playbook -i production.ini download_and_extract.yml](https://github.com/valdemar-2502/Ansible-Homework/blob/main/demo_vm-2-kafka.png)

---
```
---
- name: Установка и настройка tuned
  hosts: servers
  become: yes

  tasks:
    - name: Установить пакет tuned
      package:
        name: tuned
        state: present

    - name: Запустить службу tuned
      systemd:
        name: tuned
        state: started
        enabled: yes

    - name: Проверить статус службы tuned
      systemd:
        name: tuned
        state: started
        enabled: yes
root@test-netology:/infra/inventory#
```
![Скриншот команды ansible-playbook -i production.ini install_tuned.yml](https://github.com/valdemar-2502/Ansible-Homework/blob/main/demo_ansible_install_tuned.png)
![Скриншот команды ansible-playbook -i production.ini install_tuned.yml](https://github.com/valdemar-2502/Ansible-Homework/blob/main/demo_vm-1-tuned.png)
![Скриншот команды ansible-playbook -i production.ini install_tuned.yml](https://github.com/valdemar-2502/Ansible-Homework/blob/main/demo_vm-2-tuned.png)

---
```
---
- name: Изменение системного приветствия (motd)
  hosts: servers
  become: yes
  vars:
    custom_motd: "Once you log in, you'll have a lot of fun!!! Current time: {{ ansible_date_time.time }}"

  tasks:
    - name: Создать кастомное приветствие
      copy:
        content: |
          {{ custom_motd }}
          Сервер: {{ ansible_hostname }}
          ОС: {{ ansible_distribution }} {{ ansible_distribution_version }}
          Ядро: {{ ansible_kernel }}
        dest: /etc/motd
        owner: root
        group: root
        mode: '0644'

    - name: Отключить динамическое обновление motd (если используется)
      file:
        path: /etc/update-motd.d
        state: absent
      when: ansible_os_family == "Debian"

    - name: Проверить изменение motd
      command: cat /etc/motd
      register: motd_content

    - name: Показать содержимое motd
      debug:
        msg: "Текущее приветствие: {{ motd_content.stdout }}"
```
![Скриншот команды ansible-playbook -i production.ini change_motd.yml](https://github.com/valdemar-2502/Ansible-Homework/blob/main/demo_ansible_change_motd.png)

---
### Задание 2

**Выполните действия, приложите файлы с модифицированным плейбуком и вывод выполнения.** 

Модифицируйте плейбук из пункта 3, задания 1. В качестве приветствия он должен установить IP-адрес и hostname управляемого хоста, а также пожелание хорошего дня системному администратору.

---
```
---
- name: Изменение системного приветствия (motd)
  hosts: servers
  become: yes
  vars:
    custom_motd: "Have a great day, system administrator!"

  tasks:
    - name: Создать кастомное приветствие
      copy:
        content: |
          ========================================
          Get on our server!

          Hostname: {{ ansible_hostname }}
          IP-адрес: {{ ansible_default_ipv4.address }}
          ОС: {{ ansible_distribution }} {{ ansible_distribution_version }}
          Ядро: {{ ansible_kernel }}
          Текущее время: {{ ansible_date_time.time }}

          {{ custom_motd }}
          ========================================
        dest: /etc/motd
        owner: root
        group: root
        mode: '0644'

    - name: Отключить динамическое обновление motd (если используется)
      file:
        path: /etc/update-motd.d
        state: absent
      when: ansible_os_family == "Debian"

    - name: Проверить изменение motd
      command: cat /etc/motd
      register: motd_content

    - name: Показать содержимое motd
      debug:
        msg: "Текущее приветствие: {{ motd_content.stdout }}"
```
![Скриншот команды ansible-playbook -i production.ini change_motd_modific.yml](https://github.com/valdemar-2502/Ansible-Homework/blob/main/demo_ansible_change_motd_modific.png)
![Скриншот команды ansible-playbook -i production.ini change_motd_modific.yml](https://github.com/valdemar-2502/Ansible-Homework/blob/main/demo_vm-1-chang_motd_modific.png)
![Скриншот команды ansible-playbook -i production.ini change_motd_modific.yml](https://github.com/valdemar-2502/Ansible-Homework/blob/main/demo_vm-2_change_motd_modific.png)




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
---
```
---
- name: Установка и настройка Apache сервера
  hosts: all
  become: yes
  roles:
    - apache_server

  tasks:
    - name: Проверка доступности веб-сайта
      ansible.builtin.uri:
        url: "http://{{ ansible_default_ipv4.address }}"
        status_code: 200
      delegate_to: localhost
      run_once: true
root@test-netology:/infra#

```
![Скриншот команды ansible-playbook -i inventory/production.ini playbook.yml](https://github.com/valdemar-2502/Ansible-Homework/blob/main/demo_ansible_playbook.yml.png)
![Скриншот команды ansible-playbook -i inventory/production.ini playbook.yml](https://github.com/valdemar-2502/Ansible-Homework/blob/main/demo_system_info_vm-1.png)
![Скриншот команды ansible-playbook -i inventory/production.ini playbook.yml](https://github.com/valdemar-2502/Ansible-Homework/blob/main/demo_system_info_vm-2.png)

---




