Install Wazuh Manager + Indexer + Dashboard VIA Ansible
```
install-manager.yml
```
```
---
- name: Install Wazuh Manager Stack
  hosts: GPU
  become: yes

  tasks:

    - name: Install dependencies
      yum:
        name:
          - curl
          - unzip
          - ca-certificates
        state: present

    - name: Configure Wazuh Repository
      copy:
        dest: /etc/yum.repos.d/wazuh.repo
        content: |
          [wazuh]
          gpgcheck=1
          gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
          enabled=1
          name=Wazuh repository
          baseurl=https://packages.wazuh.com/4.x/yum/

    - name: Clean yum cache
      command: yum clean all

    - name: Build yum cache
      command: yum makecache

    - name: Install Wazuh Manager
      yum:
        name: wazuh-manager
        state: latest

    - name: Install Wazuh Indexer
      yum:
        name: wazuh-indexer
        state: latest

    - name: Install Wazuh Dashboard
      yum:
        name: wazuh-dashboard
        state: latest

    - name: Reload systemd
      systemd:
        daemon_reload: yes

    - name: Enable and start Wazuh Indexer
      systemd:
        name: wazuh-indexer
        enabled: yes
        state: started

    - name: Enable and start Wazuh Manager
      systemd:
        name: wazuh-manager
        enabled: yes
        state: started

    - name: Enable and start Wazuh Dashboard
      systemd:
        name: wazuh-dashboard
        enabled: yes
        state: started

    - name: Wait for Indexer
      wait_for:
        port: 9200
        timeout: 180

    - name: Wait for Dashboard
      wait_for:
        port: 443
        timeout: 180

    - name: Show Wazuh version
      command: /var/ossec/bin/wazuh-control info
      register: wazuh_info
      changed_when: false

    - debug:
        var: wazuh_info.stdout_lines
```

Install agent wazuh semua server 

```
install-agent.yml
```
```
- name: Install Wazuh Agent on CentOS 7.6
  hosts: all
  become: yes

  vars:
    wazuh_manager_ip: "192.168.0.200"

  tasks:

    - name: Update system
      yum:
        name: "*"
        state: latest

    - name: Install dependencies
      yum:
        name:
          - curl
          - ca-certificates
        state: present

    - name: Add Wazuh repo
      copy:
        dest: /etc/yum.repos.d/wazuh.repo
        content: |
          [wazuh]
          gpgcheck=1
          gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
          enabled=1
          name=Wazuh repository
          baseurl=https://packages.wazuh.com/4.x/yum/

    - name: Install Wazuh agent
      yum:
        name: wazuh-agent
        state: present

    - name: Set manager IP
      lineinfile:
        path: /var/ossec/etc/ossec.conf
        regexp: "<address>.*</address>"
        line: "      <address>{{ wazuh_manager_ip }}</address>"

    - name: Enable agent
      systemd:
        name: wazuh-agent
        enabled: yes
        state: started
```
