+----------------------+
|  Wazuh Dashboard     |  (Port 443)
+----------+-----------+
           |
+----------v-----------+
|  Wazuh Indexer       |  (OpenSearch)
+----------+-----------+
           |
+----------v-----------+
|  Wazuh Manager       |
|  (Agent Controller)  |
+----------+-----------+
           |
   Agents (30+ servers)
   Ports: 1514 / 1515


curl -sO https://packages.wazuh.com/4.8/wazuh-install.sh
chmod +x wazuh-install.sh
bash wazuh-install.sh -a
