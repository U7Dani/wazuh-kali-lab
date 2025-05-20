![Diseño sin título](https://github.com/user-attachments/assets/a2240890-0a34-4e78-ba00-addc425b9768)




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
- IP: `xxx.xxx.xxx.xxx`
- Herramientas ofensivas instaladas: `nmap`, `hping3`, `unicornscan`, `masscan`
- Agente Wazuh instalado

### 🔍 Kali Purple (IDS)
- IP: `xxx.xxx.xxx.xxx`
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
📡 **Dashboard:** https://xxx.xxx.xxx.xxx

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
| Escaneo TCP SYN | Nmap | `nmap -sS -p- xxx.xxx.xxx.xxx` | Detectado | T1046 |
| Escaneo rápido | Unicornscan | `unicornscan -Iv xxx.xxx.xxx.xxx:1-1000` | Detectado parcialmente | T1046 |
| Escaneo masivo | Masscan | `masscan xxx.xxx.xxx.xxx -p1-65535 --rate=1000` | Detectado | T1595 |
| DoS básico | Hping3 | `hping3 -S xxx.xxx.xxx.xxx -p 80 -c 100` | Detectado | T1046 |
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
  data.command : "/usr/bin/unicornscan -Iv xxx.xxx.xxx.xxx:1-1000"
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
## 🦠 **Integración avanzada: Automatización de análisis en VirusTotal**

### **¿Qué se consigue?**
- **Automatización real:**  
  Cada archivo extraído por Suricata es subido automáticamente a VirusTotal para su análisis en sandbox.
- **Visibilidad instantánea:**  
  El hash SHA256 y el enlace a VirusTotal quedan guardados para cada archivo, facilitando el análisis y respuesta.

---

### **¿Cómo se hace?**

#### **1. Script automatizado en Kali Purple**

Guarda este script como `/opt/upload_to_virustotal.py` y pon tu API KEY de VirusTotal:

```python
import requests, os, time, hashlib

API_KEY = 'TU_API_KEY'
FILE_DIR = '/var/lib/suricata/files/'
PROCESSED_DIR = '/var/lib/suricata/processed_files/'
LOG_FILE = '/var/lib/suricata/virustotal_uploads.log'

os.makedirs(PROCESSED_DIR, exist_ok=True)

for filename in os.listdir(FILE_DIR):
    filepath = os.path.join(FILE_DIR, filename)
    with open(filepath, 'rb') as f:
        file_data = f.read()
    sha256 = hashlib.sha256(file_data).hexdigest()
    files = {'file': (filename, file_data)}
    headers = {'x-apikey': API_KEY}
    response = requests.post('https://www.virustotal.com/api/v3/files', files=files, headers=headers)
    analysis_id = response.json()['data']['id']
    print(f"[+] Subido: {filepath} - Analysis ID: {analysis_id}")
    # Esperar análisis (opcional)
    for _ in range(5):
        time.sleep(5)
        report = requests.get(f'https://www.virustotal.com/api/v3/analyses/{analysis_id}', headers=headers)
        if report.json()['data']['attributes']['status'] == 'completed':
            break
        print("[*] Esperando análisis...")
    link = f"https://www.virustotal.com/gui/file/{sha256}/detection"
    print(f"[+] SHA256: {sha256}")
    with open(LOG_FILE, 'a') as log:
        log.write(f"{filepath},{sha256},{link}
")
    os.rename(filepath, os.path.join(PROCESSED_DIR, filename))
```

- Hazlo ejecutable:  
  ```sh
  sudo chmod +x /opt/upload_to_virustotal.py
  ```
- Programa su ejecución automática (cada 5 min):  
  ```sh
  crontab -e
  # Añade:
  */5 * * * * /usr/bin/python3 /opt/upload_to_virustotal.py
  ```

---

#### **2. Consulta automática de resultados**

El log `/var/lib/suricata/virustotal_uploads.log` guarda para cada archivo:

```
<ruta>,<sha256>,<enlace de VirusTotal>
```
Ejemplo:
```
/var/lib/suricata/files/eicar.com,131f95c51cc819465fa1797f6ccacf9d49aaaff46fa3eac73ae63ffbbdf8267,https://www.virustotal.com/gui/file/131f95c51cc819465fa1797f6ccacf9d49aaaff46fa3eac73ae63ffbbdf8267/detection
```

---

### **Prueba realista de laboratorio**

1. Genera un archivo de test EICAR en Kali Red Team:
   ```sh
   echo 'X5O!P%@AP[4\PZX54(P^)7CC)7}$EICAR-STANDARD-ANTIVIRUS-TEST-FILE!$H+H*' > eicar.com
   python3 -m http.server 8080
   ```
2. Desde Kali Purple, descarga el archivo:
   ```sh
   wget http://xxx.xxx.xxx.xxx:8080/eicar.com
   ```
3. El archivo será extraído, subido a VirusTotal y el enlace generado en el log.
4. **Consulta el enlace y verás detección real de malware en VirusTotal.**
![image](https://github.com/user-attachments/assets/47271fa5-59d8-453b-87fc-a86cc998dc27)
![image](https://github.com/user-attachments/assets/2b616fa2-68ef-4298-9e9f-b3f38ecef712)
![image](https://github.com/user-attachments/assets/3717e59a-0bbc-44de-8512-b1443eaeac3d)
![image](https://github.com/user-attachments/assets/63c5d37b-485a-476e-a43b-ee3fb2a9701d)
![image](https://github.com/user-attachments/assets/90854506-f8a5-4002-937e-76f92c9f1dd7)



---

## 📸 Arquitectura del laboratorio

| Componente       | IP              | Estado                                 | Función                             |
|------------------|------------------|----------------------------------------|-------------------------------------|
| Kali Purple      | xxx.xxx.xxx.xxx | ✅ Suricata + Filebeat activo          | Detección de tráfico malicioso      |
| Kali Red Team    | xxx.xxx.xxx.xxx | ✅ Agente Wazuh                         | Generación de ataques               |
| Wazuh Manager    | xxx.xxx.xxx.xxx | ✅ Wazuh + Dashboard + Elastic         | Recepción, análisis y visualización |

---

## 🧾 Créditos

- [Wazuh Documentation](https://documentation.wazuh.com/)
- Proyecto gestionado y documentado por [@U7Dani](https://github.com/U7Dani)
