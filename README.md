# task18

Создал инстансы: <br>
![image](https://github.com/user-attachments/assets/00b9873a-020f-4656-bb2b-8159751dc63e)

Сделал hosts и group_vars: <br>

```
[staging_servers]
ubuntu ansible_host=44.206.248.213 ansible_user=ubuntu
amaz ansible_host=54.210.98.109
```

```
---
ansible_user                 : ec2-user
ansible_ssh_private_key_file : /var/key.pem
```

Создал roles: <br>
![image](https://github.com/user-attachments/assets/bf8cbc88-943e-4b5c-8c7e-01a594b64025)



 Установил Docker и Docker compose в /roles/istall/tasks/main.yml: <br>
```
 # tasks file for install
- name: Install Docker on Ubuntu
  apt:
    name: docker.io
    state: present
  when: ansible_os_family == "Debian"

- name: Install Docker on Amazon Linux
  yum:
    name: docker
    state: present
  when: ansible_os_family == "RedHat"

- name: Install Docker Compose
  shell: >
    curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  args:
    creates: /usr/local/bin/docker-compose

- name: Set permissions for Docker Compose
  command: chmod +x /usr/local/bin/docker-compose

- name: Start Docker service
  service:
    name: docker
    state: started
    enabled: true
```

Подготовка для docker: <br>
```
  ---
# tasks file for docker
- name: Create working directory
  file:
    path: "{{ working_directory }}"
    state: directory

- name: Generate Docker Compose file
  template:
    src: docker-compose.yml.j2
    dest: "{{ working_directory }}/docker-compose.yml"

- name: Start Docker Compose
  command: docker-compose up -d
  args:
    chdir: "{{ working_directory }}"

```

Файл docker-compose + jinja: <br>
```
version: '3.8'

services:
  web:
    image: {{ docker_image }}
    ports:
      - "{{ host_port }}:80"

```

ИТОГ: <br>

![image](https://github.com/user-attachments/assets/6b62cf88-02c0-4d81-b8ed-a2144dcaceb8)
![image](https://github.com/user-attachments/assets/49dcae20-7ea7-4080-b2f4-2ef8a17522d9)


