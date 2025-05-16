![image](https://github.com/user-attachments/assets/de9f9cbf-4a2f-4d97-9d24-ade81ff2d0d9)

# Wazuh + Kali Linux Integration Lab

Este repositorio contiene una guía detallada para instalar y configurar **Wazuh** (versión 4.12.0) en un entorno local, junto con la integración de un agente instalado en **Kali Linux** para pruebas de laboratorio de ciberseguridad.

> 📅 Fecha: 16/05/2025

---

## 🧰 Requisitos del sistema

### 🔒 Servidor Wazuh (ej. Ubuntu 20.04+)
- 4 vCPU
- 8 GB RAM
- 200 GB almacenamiento (recomendado para 50-100 agentes)
- Acceso a internet para descargar paquetes

### 🐚 Kali Linux como agente
- Kali actualizado
- Conexión con el servidor Wazuh (misma red preferible)

---

## 🚀 Instalación del servidor Wazuh

### 1. Descargar el instalador de Wazuh
```bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh
```

### 2. Ejecutar el instalador
```bash
sudo bash ./wazuh-install.sh -a
```

Si ya estaba instalado:
```bash
sudo bash ./wazuh-install.sh -a -o
```

Al finalizar, anota el **usuario y contraseña** mostrados:
```text
User: admin
Password: <GENERADO>
```
![image](https://github.com/user-attachments/assets/2cfa8845-be1d-4fc8-920a-469316f81c56)


### 3. Verifica el acceso al Dashboard
Abre en tu navegador:
```text
https://<IP_SERVIDOR>:443
```
![image](https://github.com/user-attachments/assets/e1c61280-fbea-4d74-8a14-9d9ebfbb0cd5)

![image](https://github.com/user-attachments/assets/ab42263e-7475-4d2c-b3ae-05c340a3ddfe)


![image](https://github.com/user-attachments/assets/be0f96f7-1d12-4aef-80f4-dd25bf94de0d)


---

## 📦 Instalación del agente Wazuh en Kali Linux

### 1. Añadir el repositorio e instalar
```bash
curl -sO https://packages.wazuh.com/key/GPG-KEY-WAZUH
gpg --dearmor < GPG-KEY-WAZUH | sudo tee /usr/share/keyrings/wazuh.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update
sudo apt install wazuh-agent
```


### 2. Configura `/var/ossec/etc/ossec.conf`

Reemplaza `MANAGER_IP` por la IP del servidor Wazuh:
```xml
<client>
  <server>
    <address>IP WAZUH</address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
  <config-profile>kali</config-profile>
  <notify_time>10</notify_time>
  <auto_restart>yes</auto_restart>
</client>
```

### 3. Registra el agente en el servidor

#### En el servidor:
```bash
sudo /var/ossec/bin/manage_agents
# (A) -> Añadir -> Nombre: kali, IP: any
# (E) -> Extraer clave y copiar
```

#### En Kali:
```bash
sudo /var/ossec/bin/manage_agents
# (I) -> Pegar clave copiada
```

### 4. Habilita e inicia el agente
```bash
sudo systemctl daemon-reexec
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```
![image](https://github.com/user-attachments/assets/bb17bc72-8027-4ae2-addd-77eb90e20607)

---

## 📡 Verifica la conexión

En el **Wazuh Dashboard**, ve a **Endpoints** y revisa que el estado del agente sea:
```text
🟢 Active
```

---

## 🔬 Laboratorio de pruebas con Kali

Puedes usar Kali para generar alertas y comprobar la detección en tiempo real:

---

### ⚔️ Escaneos y ataques detectados (con capturas)

> 💡 Las siguientes pruebas fueron realizadas desde Kali contra el servidor con Wazuh. Cada técnica generó alertas distintas.

---

#### ⚠️ 1. Escaneo TCP SYN con Nmap
```bash
sudo nmap -sS -p- 192.168.222.140
```
- Estado: Detectado
- Acción: Se creó regla personalizada `100100`
- MITRE ATT&CK: T1046

📸 Captura:  
![image](https://github.com/user-attachments/assets/313a3211-1962-428b-ba90-ae78894acd77)

![image](https://github.com/user-attachments/assets/b8806990-e4b3-4e78-93e6-5a49c03f45f5)

---

#### ⚠️ 2. Uso de Unicornscan
```bash
sudo unicornscan -Iv 192.168.222.140:1-1000
```
- Estado: Detectado parcialmente
- Regla personalizada: `100200`
- MITRE ATT&CK: T1046

📸 Captura:  
![image](https://github.com/user-attachments/assets/333c42ed-e551-4d99-9825-a3f4accccef4)
![image](https://github.com/user-attachments/assets/660a4015-9caa-4fdd-b501-51368417614b)



---

#### ⚠️ 3. Escaneo con Masscan
```bash
sudo masscan 192.168.222.140 -p1-65535 --rate=1000
```
- Estado: Detectado
- Regla personalizada: `100301`
- MITRE ATT&CK: T1595

📸 Captura:  
![image](https://github.com/user-attachments/assets/531dc28f-d4b2-4555-bad5-b48c4b5ff911)
![image](https://github.com/user-attachments/assets/3f1e550d-e3bf-4807-bf4e-9687f71d32a7)
![image](https://github.com/user-attachments/assets/bcdbbed4-3105-42af-b985-ebb167cdae43)



---

#### ⚠️ 4. Ataque con Hping3
```bash
sudo hping3 -S 192.168.222.140 -p 80 -c 100
```
- Estado: Detectado
- Regla personalizada: `100302`
- MITRE ATT&CK: T1046

📸 Captura:  
![image](https://github.com/user-attachments/assets/541b7019-4f38-4c47-b587-1366cab4edb6)
![image](https://github.com/user-attachments/assets/954b8255-0ccc-4b4d-ad3f-a1f6c3abe175)
![image](https://github.com/user-attachments/assets/770a4d76-aa83-47bf-9e81-b6eeba509c21)


---

#### ⚠️ 5. Escalada con sudo
```bash
sudo cat /etc/shadow
```
- Estado: Detectado por reglas estándar
- Nivel: Medio (`rule.level: 3`)

📸 Captura:  
![image](https://github.com/user-attachments/assets/2a498f28-52af-44ee-a7e0-0fc31a29f3f9)
![image](https://github.com/user-attachments/assets/64877a8a-6635-46eb-bb0f-d7115a65b118)



---

## 📊 Monitorización

Accede a:
```text
https://<IP_SERVIDOR>:443
```

Consulta las alertas en:
- **Security Events**
- **Wazuh Dashboard**
- **Discover (Visualización por agente, severidad, y regla)**

---

## 📌 Notas adicionales

- Verifica que el puerto esté abierto:
```bash
sudo ss -uln | grep 1514
```

- Reinicia el manager si no responde:
```bash
sudo systemctl restart wazuh-manager
```

---

## 🧾 Créditos

- Proyecto Wazuh: [https://documentation.wazuh.com/](https://documentation.wazuh.com/)
- Autor de esta guía: `@U7Dani`
- Fecha de documentación: 16/05/2025
