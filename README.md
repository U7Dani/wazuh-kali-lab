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

### 3. Verifica el acceso al Dashboard
Abre en tu navegador:
```text
https://<IP_SERVIDOR>:443
```

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

---

## 📡 Verifica la conexión

En el **Wazuh Dashboard**, ve a **Endpoints** y revisa que el estado del agente sea:
```text
🟢 Active
```

---

## 🔬 Laboratorio de pruebas con Kali

Puedes usar Kali para generar alertas y comprobar la detección en tiempo real:

### 1. Uso de `nmap`
```bash
nmap -sS 192.168.222.140
```

### 2. Fuerza bruta con `hydra`
```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.222.140
```

### 3. Comandos `sudo`
```bash
sudo ls
sudo cat /etc/shadow
```

### 4. Mensajes personalizados
```bash
logger "Prueba de alerta personalizada desde Kali"
```

---

## 📊 Monitorización

Accede a:
```text
https://<IP_SERVIDOR>:443
```

Consulta las alertas en:
- **Security Events**
- **Dashboard principal**
- **Wazuh alerts (visualización por agente, severidad, reglas activadas, etc.)**

---

## 📌 Notas adicionales

- Asegúrate de que el **puerto 1514** esté abierto en el servidor:
```bash
sudo ss -uln | grep 1514
```

- Reinicia el servidor Wazuh si no escucha el puerto:
```bash
sudo systemctl restart wazuh-manager
```

---

## 🧾 Créditos

- Proyecto Wazuh: [https://documentation.wazuh.com/](https://documentation.wazuh.com/)
- Autor de esta guía: `@U7Dani`
- Fecha de documentación: 16/05/2025

