# Wazuh Docker Deployment and Windows XP Agent Installation Guide

This guide documents deployment of the Wazuh stack using Docker in an air-gapped environment and installation of the Wazuh agent on Windows XP hosts.

---

# Table of Contents

- [Host Preparation](#host-preparation)
- [Wazuh Docker Stack Deployment](#wazuh-docker-stack-deployment)
- [Windows XP Agent Installation](#windows-xp-agent-installation)
- [Windows XP Logging Configuration Fixes](#windows-xp-logging-configuration-fixes)
- [External Port Mapping](#external-port-mapping)
- [Verification](#verification)
- [Notes](#notes)

---

# Host Preparation

Wazuh indexer (OpenSearch) requires increased virtual memory limits.

Run as root:

```bash
sysctl -w vm.max_map_count=262144
```

---

# Wazuh Docker Stack Deployment

Navigate to image directory:

```bash
cd ~/wazuh-docker/wazuh-image-tars/
```

Load Docker images:

```bash
docker load -i wazuh-manager_4.14.2.tar
docker load -i wazuh-indexer_4.14.2.tar
docker load -i wazuh-dashboard_4.14.2.tar
docker load -i wazuh-certs-generator.tar
```

Verify images:

```bash
docker images | grep wazuh
```

Start stack:

```bash
cd ~/docker-wazuh/single-node/
```
```bash
docker compose up -d
```

Verify containers:

```bash
docker ps
```

# Wazuh Docker Archives Log Enablement

Run the following command to enter the container:

```bash
docker exec -it single-node-wazuh.manager-1 /bin/bash
```

Edit /etc/filebeat/filebeat.yml and change the archives: setting from false to true:

archives:
  enabled: true

```bash
sed -i '/archives:/!b;n;s/enabled: false/enabled: true/' /etc/filebeat/filebeat.yml
```

Run the following command to check if the modification was applied correctly:


```bash
cat /etc/filebeat/filebeat.yml
```

Exit the container and restart the Wazuh Manager:

```bash
docker restart single-node-wazuh.manager-1
```


From the Wazuh GUI:
```
Dashboards Management -> Index Patterns -> Create Index Pattern
```
```
Name = wazuh-archives-*
```
```
Time Field = @timestamp
```
Save


# Windows XP Agent Installation

## File Location

```
~/Wazuh/wazuh-agent/
```

Required file:

```
wazuh-agent-4.14.2-1.msi
```

---

## Silent Installation

Run in Command Prompt:

```cmd
wazuh-agent-4.14.2-1.msi /q WAZUH_MANAGER="X.X.X.X" WAZUH_MANAGER_PORT="1516" WAZUH_REGISTRATION_PORT="1517"
```

Replace `X.X.X.X` with your Wazuh Manager IP. Ascertain Ports with chart below.

---

## Start Agent Service

```cmd
NET START WazuhSvc
```
NOTE: This may take multiple attempts to start the service.

Verify:

```cmd
sc query WazuhSvc
```

Expected output:

```
STATE: RUNNING
```

---

# Windows XP Logging Configuration Fixes

## Enable Full Audit Logging

Open:

```
Start → Run → secpol.msc
```

Navigate to:

```
Local Policies → Audit Policy
```

Enable **Success and Failure** for all categories.

---

## Fix Event Log Size Limitation

Open:

```
Start → Run → eventvwr.msc
```

For each log:

- Application
- Security
- System

Do:

Right click → Properties

Enable:

```
Overwrite events as needed
```

Recommended:

Increase log size to:

```
262144 KB
```

---

# External Port Mapping

| Port | Component | Description |
|------|-----------|-------------|
| 1516 | Wazuh Manager | Agent communication |
| 1517 | Wazuh Manager | Agent enrollment |
| 516 | Wazuh Manager | Syslog ingestion |
| 55000 | Wazuh Manager | API |
| 9250 | Wazuh Indexer | OpenSearch API |
| 5602 | Wazuh Dashboard | Web interface |

Dashboard access:

```
https://SERVER-IP:5602
```

---

# Verification

List agents:

```bash
docker exec -it wazuh-manager /var/ossec/bin/agent_control -l
```

Check indices:

```bash
curl -k -u admin:password https://localhost:9250/_cat/indices?v
```

---

# Notes

• Designed for air-gapped deployment  
• No internet required  
• Docker images loaded manually  
• Windows XP requires manual audit policy configuration  

---
