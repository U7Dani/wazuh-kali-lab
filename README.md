
# 🛡️ Laboratorio Wazuh + Kali Linux + Suricata

Este proyecto documenta la instalación y configuración de un entorno de laboratorio de ciberseguridad con **Wazuh (v4.12.0)**, integración de un agente en Kali Linux y monitoreo de tráfico malicioso con **Suricata** en Kali Purple.

📅 **Fecha de documentación:** 19/05/2025  
🧑‍💻 **Autor:** [@U7Dani](https://github.com/U7Dani)

---

## 🧰 Requisitos del sistema

### 🔒 Wazuh Manager
- Ubuntu Server (20.04 o superior)
- 4 vCPU / 8 GB RAM / 200 GB almacenamiento
- IP: `192.168.222.140`

### 🧪 Kali Red Team (Generador de alertas)
- IP: `192.168.222.131`
- Herramientas ofensivas instaladas: `nmap`, `hping3`, `unicornscan`, `masscan`
- Agente Wazuh instalado

### 🔍 Kali Purple (IDS)
- IP: `192.168.222.128`
- Suricata 7.x instalado
- Filebeat instalado y configurado para enviar logs a Wazuh

---

## 🚀 Instalación paso a paso

### 📦 1. Instalar Wazuh (en Ubuntu)

```bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
sudo bash ./wazuh-install.sh -a
```

Para reinstalar sobre una instalación previa:

```bash
sudo bash ./wazuh-install.sh -a -o
```

🧠 **Usuario por defecto:** `admin`  
🔑 **Contraseña:** (anotada tras la instalación)  
📡 **Dashboard:** https://192.168.222.140

---

### 📦 2. Instalar agente Wazuh en Kali

```bash
curl -sO https://packages.wazuh.com/key/GPG-KEY-WAZUH
gpg --dearmor < GPG-KEY-WAZUH | sudo tee /usr/share/keyrings/wazuh.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update
sudo apt install wazuh-agent
```

#### Configuración (`/var/ossec/etc/ossec.conf`):

```xml
<client>
  <server>
    <address>192.168.222.140</address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
  <config-profile>kali</config-profile>
  <notify_time>10</notify_time>
  <auto_restart>yes</auto_restart>
</client>
```

#### Registro del agente:
En el servidor:
```bash
sudo /var/ossec/bin/manage_agents
# Añadir nuevo agente y copiar clave
```

En Kali:
```bash
sudo /var/ossec/bin/manage_agents
# Importar clave copiada
```

```bash
sudo systemctl daemon-reexec
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

---

## 🦊 Instalación de Suricata + Filebeat en Kali Purple

### 📥 1. Instalar Suricata
```bash
sudo apt install suricata -y
```

Verifica:
```bash
sudo suricata -i eth0 -v
```

### 📦 2. Instalar Filebeat
```bash
curl -fsSL https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.12.1-amd64.deb -o filebeat.deb
sudo dpkg -i filebeat.deb
```

O bien vía repositorio:

```bash
sudo apt install apt-transport-https
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo tee /usr/share/keyrings/elastic-keyring.asc
echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.asc] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update
sudo apt install filebeat -y
```

### ⚙️ 3. Configurar Filebeat

Edita `/etc/filebeat/filebeat.yml`:

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/suricata/eve.json

output.logstash:
  hosts: ["192.168.222.140:5044"]
```

Habilita módulo Suricata:
```bash
sudo filebeat modules enable suricata
```

### 🚀 4. Iniciar Filebeat

```bash
sudo systemctl enable filebeat
sudo systemctl start filebeat
sudo systemctl status filebeat
```

Test:
```bash
sudo filebeat test config
sudo filebeat test output
```

---

## 🧪 Ataques realizados desde Kali

| Técnica | Herramienta | Comando | Estado | MITRE ID |
|--------|-------------|---------|--------|----------|
| Escaneo TCP SYN | Nmap | `nmap -sS -p- 192.168.222.140` | Detectado | T1046 |
| Escaneo rápido | Unicornscan | `unicornscan -Iv 192.168.222.140:1-1000` | Detectado parcialmente | T1046 |
| Escaneo masivo | Masscan | `masscan 192.168.222.140 -p1-65535 --rate=1000` | Detectado | T1595 |
| DoS básico | Hping3 | `hping3 -S 192.168.222.140 -p 80 -c 100` | Detectado | T1046 |
| Escalada de privilegios | sudo | `sudo cat /etc/shadow` | Detectado | T1068 |

---

## 🔎 Filtros útiles en Discover (Kibana/Wazuh Dashboard)

- **Por agente Kali Red Team**:
  ```
  agent.name : "kali"
  ```

- **Solo logs de Suricata**:
  ```
  input.type : "log"
  ```

- **Por MITRE ATT&CK ID**:
  ```
  rule.mitre.id : "T1046"
  ```

- **Logs de sudo**:
  ```
  predecoder.program_name : "sudo"
  ```

- **Comando específico**:
  ```
  data.command : "/usr/bin/unicornscan -Iv 192.168.222.140:1-1000"
  ```

---

## 📌 Notas adicionales

- Verifica que el puerto esté abierto:
  ```bash
  sudo ss -uln | grep 1514
  ```

- Reinicia Wazuh Manager si no responde:
  ```bash
  sudo systemctl restart wazuh-manager
  ```

---

## 📸 Arquitectura del laboratorio

| Componente       | IP              | Estado                                 | Función                             |
|------------------|------------------|----------------------------------------|-------------------------------------|
| Kali Purple      | 192.168.222.128 | ✅ Suricata + Filebeat activo          | Detección de tráfico malicioso      |
| Kali Red Team    | 192.168.222.131 | ✅ Agente Wazuh                         | Generación de ataques               |
| Wazuh Manager    | 192.168.222.140 | ✅ Wazuh + Dashboard + Elastic         | Recepción, análisis y visualización |

---

## 🧾 Créditos

- [Wazuh Documentation](https://documentation.wazuh.com/)
- Proyecto gestionado y documentado por [@U7Dani](https://github.com/U7Dani)
