# task18

Создал инстансы: <br>
![image](https://github.com/user-attachments/assets/00b9873a-020f-4656-bb2b-8159751dc63e)

Сделал hosts и group_vars: <br>

```
[staging_servers]
ubuntu ansible_host=44.202.16.190 ansible_user=ubuntu
amaz ansible_host=3.84.33.91

```

```
---
ansible_user                 : ec2-user
ansible_ssh_private_key_file : /var/key.pem
```

Создал roles: <br>
![image](https://github.com/user-attachments/assets/d2cc4d8a-2969-454a-a352-bdce4371b654)



 Установил Docker и Docker compose в /roles/docker/tasks/main.yml: <br>
```
---
# tasks file for docker
- name: Update package index
  apt:
    update_cache: yes
  when: ansible_os_family == "Debian"

- name: Update yum package index
  yum:
    name: '*'
    state: latest
  when: ansible_os_family == "RedHat"

- name: Install Docker on Debian
  apt:
    name: docker-ce
    state: present
  when: ansible_os_family == "Debian"

- name: Install Docker on RedHat
  yum:
    name: docker
    state: present
  when: ansible_os_family == "RedHat"

- name: Install Docker Compose
  shell: >
    curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose->
  args:
    creates: /usr/local/bin/docker-compose

- name: Set permissions for Docker Compose
  command: chmod +x /usr/local/bin/docker-compose

- name: Create working directory
  file:
    path: "{{ working_directory }}"
    state: directory

- name: Generate Docker Compose file
  template:
    src: docker-compose.yml.j2
    dest: "{{ working_directory }}/docker-compose.yml"
  notify: Restart my_docker service

- name: Generate systemd service file
  template:
    src: my_docker_service.j2
    dest: /etc/systemd/system/my_docker.service
  notify: Restart my_docker service

- name: Start Docker service
  systemd:
    name: docker
    state: started
    enabled: true

- name: Start and enable my_docker service
  systemd:
    name: my_docker.service
    state: started
    enabled: true

```



Файл docker-compose + jinja /dokcer-compose.yml.j2: <br>
```
version: '3.8'

services:
  web:
    image: {{ docker_image }}
    ports:
      - "{{ host_port }}:80"
    restart: always

```

Systemd: <br>
```
[Unit]
Description=My Docker Compose Service
After=docker.service

[Service]
Restart=always
WorkingDirectory={{ working_directory }}
ExecStart=/usr/local/bin/docker-compose up
ExecStop=/usr/local/bin/docker-compose down
TimeoutStopSec=30  # Время ожидания остановки контейнера

[Install]
WantedBy=multi-user.target
```


ИТОГ: <br>

![image](https://github.com/user-attachments/assets/b9b255e1-1164-4b61-8f4a-11f5856980ae)
![image](https://github.com/user-attachments/assets/d8538848-8ff5-4a81-837a-1433c95922c1)


