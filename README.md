![Diseño sin título (1)](https://github.com/user-attachments/assets/23281877-4869-4acd-b258-0ed49be1431b)



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
![image](https://github.com/user-attachments/assets/6353b6a1-d99d-406f-8b9d-4acf5edebeea)
![image](https://github.com/user-attachments/assets/1b983246-0451-409a-a7c3-877379862a66)

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

![444639744-e1c61280-fbea-4d74-8a14-9d9ebfbb0cd5](https://github.com/user-attachments/assets/ac85ee6b-0604-4c90-b105-f968ba7aea62)

![444639435-2cfa8845-be1d-4fc8-920a-469316f81c56](https://github.com/user-attachments/assets/1bd273c7-134e-4e47-b37f-f8db65307a98)

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
![444640113-bb17bc72-8027-4ae2-addd-77eb90e20607](https://github.com/user-attachments/assets/5cb9a44c-ed6c-45b2-bccb-b3637214b7bf)

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
![image](https://github.com/user-attachments/assets/d5528a13-d8ac-42e6-9901-b4a4e1b492c4)

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

![444640578-313a3211-1962-428b-ba90-ae78894acd77](https://github.com/user-attachments/assets/b0212265-ddc8-4d12-a683-5c290c94b8be)
![444640644-b8806990-e4b3-4e78-93e6-5a49c03f45f5](https://github.com/user-attachments/assets/b03fc115-bfe1-43e3-a963-531040bef3fd)
![444641117-333c42ed-e551-4d99-9825-a3f4accccef4](https://github.com/user-attachments/assets/6810e838-9bd1-4311-a6d8-dacc4756139a)
![444641538-660a4015-9caa-4fdd-b501-51368417614b](https://github.com/user-attachments/assets/4e86fe9e-fa32-43f6-9d5e-af46825b7480)
![444641761-531dc28f-d4b2-4555-bad5-b48c4b5ff911](https://github.com/user-attachments/assets/2f1297cb-89dd-4d8f-97b6-032768d2caf8)
![444641825-3f1e550d-e3bf-4807-bf4e-9687f71d32a7](https://github.com/user-attachments/assets/a1960b65-7648-4578-a4fe-7a3221499083)
![444641997-541b7019-4f38-4c47-b587-1366cab4edb6](https://github.com/user-attachments/assets/0271d62e-77b7-4a6a-8152-b08890058d34)
![444642105-bcdbbed4-3105-42af-b985-ebb167cdae43](https://github.com/user-attachments/assets/3dcae587-2290-457f-bead-9e47658e2109)
![444642305-954b8255-0ccc-4b4d-ad3f-a1f6c3abe175](https://github.com/user-attachments/assets/a76f79cb-6e95-4f46-b1b9-5c3536d4d53b)
![444643598-2a498f28-52af-44ee-a7e0-0fc31a29f3f9](https://github.com/user-attachments/assets/f435f8a2-1448-40d9-b855-671a498d5f9b)
![444643798-64877a8a-6635-46eb-bb0f-d7115a65b118](https://github.com/user-attachments/assets/87445a56-af26-448a-81ff-4e25f1062aae)


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
