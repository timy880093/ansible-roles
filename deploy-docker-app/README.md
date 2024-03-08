Role Name
=========

* pull image from repository
* deploy to windows or linux within docker

Requirements
------------

### Remote node
* Linux: directly use
* Windows: (Win10 up)
  1. Install wsl
  2. Install docker or docker desktop
  3. Enable WinRM for connect
    https://www.weithenn.org/2023/09/ansible-connecting-to-a-windows-host.html

Role Variables
--------------

```
# default/main.yml
is_windows: "{{ ansible_os_family == 'Windows' }}"
is_linux: "{{ ansible_os_family != 'Windows' }}"
## remote_dir can be overwriten by external arg
remote_dir: "{{ 'C:\\application' if is_windows else '/home/gateweb/application' }}"

# vars/main.yml
## For below container used
volumes:
  java: 
    - "{{ remote_dir }}/{{ app_vars.name }}/application.properties:/app/application.properties"
    - "{{ remote_dir }}/{{ app_vars.name }}/logback-spring.xml:/app/logback-spring.xml"
    - "{{ remote_dir }}/{{ app_vars.name }}/logs:/app/logs"
  nextjs:
    - "{{ remote_dir }}/{{ app_vars.name }}/env:/app/.env"
app:
  container: 
    name: "{{ app_vars.name }}"
    image: "{{ app_vars.image }}"
    ## Use the volumes based on the parameter language.
    volumes: "{{ volumes.java if language == 'java' else volumes.nextjs if language == 'nextjs' else volumes.nextjs }}"
    ports: "{{ app_vars.ports }}"
    networks: "{{ docker.networks }}" 
    # networks:
    #   - name: "{{ docker.networks[0].name }}"
  files:
    remote: "{{ remote_dir }}/{{ app_vars.name }}"
    ## The ending slash '/' must be added, otherwise an additional folder will be duplicated.
    local: "files/{{ group }}/{{ app_vars.name }}/"
```


Dependencies
------------

none

Example Playbook
----------------

```
  # command line
  ansible-playbook -i support -e group=support playbook.yml


  # playbook.yml
  - name: Deploy operations 
    hosts: "{{ group }}"
    become: no
    gather_facts: yes
    vars_files: 
      - "group_vars/{{ group }}.yml"
    roles:
      - role: deploy-docker-app
        vars: 
          app_vars: "{{ operations }}"
          language: "java"


  # inventory
  vm-01 ansible_host=127.0.0.1 ansible_user=administrator ansible_password=!7718gateweB@ ansible_connection=winrm ansible_winrm_server_cert_validation=ignore ansible_python_interpreter=/usr/bin/python3 remote_dir=D:\\application

  [support]
  vm-01


  # group_vars/app.yml
  docker:
    networks:
      - {name: "{{ group }}"}

  operations:
    name: "operations"
    image: "host/operations:8d2dc247b9"
    ports:
      - "8081:8080"
```

License
-------

BSD

Author Information
------------------

for work used
