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
lihat package terinstal
```
rpm -qa | grep wazuh
atau
yum list installed | grep wazuh
```

cek systemd
```
systemctl stop wazuh-dashboard
systemctl stop wazuh-indexer
systemctl stop wazuh-manager
systemctl status wazuh-dashboard
systemctl status wazuh-indexer
systemctl status wazuh-manager
systemctl start wazuh-dashboard
systemctl start wazuh-indexer
systemctl start wazuh-manager
```

lihat password wazuh
```
sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh -a
```

cek agent yang sudah terdaftar di manager, ceknya di server manager
```
/var/ossec/bin/agent_control -lc
```
cek versi wazuh manger (versi manger dan agent harus sama)
```
sudo su -
/var/ossec/bin/wazuh-control info
```
cek log service
```
tail -f /var/ossec/logs/ossec.log
```
## cek port

```
ss -tulpn | egrep '1514|1515'
```
```
Install agent = pasang CCTV
Manager = ruang monitoring
Port 1514/1515 = kabel network CCTV
```
```
1514 TCP → data log
1515 TCP → enrollment
```

## delete wazuh 
Hentikan semua service (aman)
```
systemctl disable wazuh-manager wazuh-indexer wazuh-dashboard
systemctl daemon-reload
```
Remove package (INI WAJIB)
```
yum remove wazuh-manager wazuh-indexer wazuh-dashboard -y
```
Kalau mau lebih bersih:
```
rpm -e wazuh-manager wazuh-indexer wazuh-dashboard --nodeps
```
Hapus sisa config & data
```
rm -rf /var/ossec
rm -rf /etc/wazuh*
rm -rf /usr/share/wazuh*
rm -rf /var/lib/wazuh*
rm -rf /usr/share/wazuh-indexer
rm -rf /usr/share/wazuh-dashboard
```
Hapus user/service leftover
```
userdel wazuh 2>/dev/null
groupdel wazuh 2>/dev/null
```
CHECK CLEAN
```
rpm -qa | grep wazuh
```
HARUS kosong

catatan
```
✅ Minggu 1

Wazuh Manager
Wazuh Agent
Dashboard

✅ Minggu 2

Suricata IDS
ET Open Rules
Integrasi ke Wazuh

✅ Minggu 3

Zeek Network Security Monitor
Integrasi Zeek → Wazuh

✅ Minggu 4

MISP (Threat Intelligence)
Integrasi IOC ke Wazuh

✅ Minggu 5

TheHive + Cortex
Incident Response & Case Management

✅ Minggu 6

Shuffle SOAR
Otomasi respons (misalnya blokir IP otomatis di firewall saat Suricata mendeteksi serangan)

Hasil akhirnya adalah platform SOC yang mendekati arsitektur yang digunakan banyak perusahaan:

Suricata (NDR) → Zeek → Wazuh (SIEM/XDR) → MISP (Threat Intelligence) → TheHive (Incident Response) → Shuffle (SOAR)

Stack ini sangat kuat untuk portofolio DevSecOps dan SOC Engineer, sekaligus bisa menjadi fondasi jika nanti kamu ingin menawarkan layanan Managed SOC kepada klien.
```
