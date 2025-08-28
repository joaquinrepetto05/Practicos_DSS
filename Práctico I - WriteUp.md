# Práctico 1 - Desarrollo de Software Seguro 2025

**Integrantes:** Joaquín Pérez, Federico Díaz y Joaquín Repetto  

---

## Resumen

Este documento detalla, con enfoque reproducible, los pasos para instalar y verificar un entorno de pruebas basado en Kali Linux ejecutado en una máquina virtual (contenida en Oracle VirtualBox), así como la instalación y configuración de un proxy de interceptación (Burp Suite Community) y de Visual Studio Code, la instalación de Docker, y la ejecución de OWASP Juice Shop y OWASP crAPI en contenedores.

> **Objetivo:** Dejar un entorno listo para ejercicios de seguridad web y API.

---

## Tabla de contenidos

1. [Alcance y prerrequisitos](#alcance-y-prerrequisitos)  
2. [Instalación de Kali Linux en VM (VirtualBox)](#instalación-de-kali-linux-en-vm-virtualbox)  
   1. [Descarga y verificación del ISO](#descarga-y-verificación-del-iso)  
   2. [Creación de la VM](#creación-de-la-vm)  
   3. [Instalación y posinstalación](#instalación-y-posinstalación)  
3. [Proxy de interceptación: Burp Suite Community](#proxy-de-interceptación-burp-suite-community)  
   1. [Instalación](#instalación-de-burp-suite)  
   2. [Primer inicio](#primer-inicio-de-burp)  
4. [Instalación de Visual Studio Code](#instalación-de-visual-studio-code)  
5. [Instalación de Docker en Kali](#instalación-de-docker-en-kali)  
6. [Ejecución de OWASP Juice Shop en Docker](#ejecución-de-owasp-juice-shop-en-docker)  
7. [Ejecución de OWASP crAPI en Docker](#ejecución-de-owasp-crapi-en-docker)  

---

## Instalación de Kali Linux en VM (VirtualBox)

### Descarga y verificación del ISO

Descargar la imagen **Installer (x86-64)** desde el sitio oficial de Kali (www.kali.org). Se recomienda descargar la versión "Installer" para x86_64.

### Creación de la VM

> Herramienta: **Oracle VM VirtualBox** (host Windows/Linux/macOS).

- **Memoria**: La cantidad de memoria que nosotros recomendaríamos asignar es 8192 MB (como mínimo asignar 2048 MB).  
- **CPU**: Nosotros recomendamos asignar 2 vCPU (mínimo 1).  
- **Disco**: Nuestra recomendación sería crear una partición de 30 GB (mínimo 20 GB).  
- **Imagen ISO**: Seleccionar la imagen ISO que descargamos anteriormente (seguramente esté alojada en su directorio de descargas).

Inicie la VM y siga el instalador (idioma, zona horaria, teclado, usuario, particionado guiado, etc.). Seleccione el escritorio por defecto (Xfce) y meta‑paquetes estándar.

### Instalación y posinstalación

Tras el primer arranque:

```bash
# Actualizar el sistema base
sudo apt update && sudo apt -y full-upgrade

# (Opcional) VirtualBox Guest Additions
sudo apt install -y build-essential dkms linux-headers-$(uname -r)
# Inserte el CD de Guest Additions desde el menú de VirtualBox y ejecute el instalador
```

---

## Proxy de interceptación: Burp Suite Community

### Instalación de Burp Suite

**Opción A — Paquete en Kali (simple):**

Ejecutar en una terminal:

```bash
sudo apt update
sudo apt install -y burpsuite
```

**Opción B — Instalador oficial (.sh):**  
Descargar el instalador para Linux desde un navegador y luego ejecutar:

```bash
cd ~/Descargas
chmod +x burpsuite_community_linux_v*.sh
./burpsuite_community_linux_v*.sh
```

### Primer inicio de Burp

1. Abrir Burp Suite Community desde el menú o con `burpsuite`.  
2. Crear un proyecto temporal → *Next* → *Start Burp*.  
3. Verificar que **Proxy → Intercept** y **Proxy → Options → Proxy Listeners** tengan un listener activo en `127.0.0.1:8080`.

> [!TIP]
> Burp ofrece un **navegador integrado** (Chromium preconfigurado) que evita configurar manualmente un navegador externo. Se accede desde **Proxy → Intercept → Open Browser**.

---

## Instalación de Visual Studio Code

**Método recomendado (repositorio de Microsoft):**

```bash
# Dependencias para clave y repos
sudo apt update && sudo apt install -y wget gpg apt-transport-https

# Clave y keyring
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo install -D -o root -g root -m 644 microsoft.gpg /usr/share/keyrings/microsoft.gpg
rm -f microsoft.gpg

# Archivo de repositorio (formato .sources recomendado)
sudo tee /etc/apt/sources.list.d/vscode.sources > /dev/null <<'EOF'
Types: deb
URIs: https://packages.microsoft.com/repos/code
Suites: stable
Components: main
Architectures: amd64,arm64,armhf
Signed-By: /usr/share/keyrings/microsoft.gpg
EOF

# Instalar VS Code estable
sudo apt update
sudo apt install -y code
```

**Verificación:**

```bash
code --version
```

---

## Instalación de Docker en Kali

> En Kali, el paquete del motor se llama **docker.io** (difiere de `docker`, que no es el motor de contenedores).

```bash
# Instalar Docker (paquete de Kali)
sudo apt update
sudo apt install -y docker.io docker-compose-plugin

# Habilitar y arrancar el servicio
sudo systemctl enable --now docker

# Usar docker sin sudo (agrega el usuario al grupo 'docker')
sudo usermod -aG docker $USER
# --> cierre sesión y vuelva a iniciar sesión para aplicar el cambio

# Prueba rápida
docker --version
docker compose version
docker run --rm hello-world
```

> Alternativa: instalar **docker-ce** desde el repositorio oficial de Docker (APT) siguiendo las instrucciones para **Debian** (libera versiones más recientes).

---

## Ejecución de OWASP Juice Shop en Docker

```bash
# Obtener la imagen oficial
docker pull bkimminich/juice-shop

# Ejecutar en segundo plano y exponer en 3000/tcp
docker run -d --name juice-shop -p 3000:3000 bkimminich/juice-shop

# Verificación
curl -I http://127.0.0.1:3000
# Abra en el navegador de la VM: http://127.0.0.1:3000
```

---

## Ejecución de OWASP crAPI en Docker

crAPI se despliega con **Docker Compose** usando archivos preconstruidos.

```bash
# Carpeta de trabajo
mkdir -p ~/labs/crapi && cd ~/labs/crapi

# Descargar el docker-compose oficial preconstruido
curl -o docker-compose.yml https://raw.githubusercontent.com/OWASP/crAPI/main/deploy/docker/docker-compose.yml

# (Opcional) variable para que escuche en todas las interfaces dentro de la VM
export LISTEN_IP=0.0.0.0

# Descargar imágenes y levantar servicios
docker compose pull
docker compose -f docker-compose.yml --compatibility up -d

# Verificación de servicios en la VM
docker ps --format "table {.Names}	{.Status}	{.Ports}"
```

**Accesos por defecto dentro de la VM:**  
- **crAPI web**: http://127.0.0.1:8888  
- **MailHog (correo de pruebas)**: http://127.0.0.1:8025

## Referencias

- Documentación oficial de **Kali**: descarga del ISO, elección de imagen, requisitos y Docker en Kali.
- Sitio oficial de **VirtualBox** para binarios del host.
- Documentación oficial de **Burp Suite** (descarga/instalación, configuración de proxy y CA).  
- Documentación oficial de **VS Code** para Debian/Ubuntu y repositorio APT.  
- Guía oficial de **OWASP Juice Shop** (ejecución con Docker).  
- Guía oficial de **OWASP crAPI** (despliegue con Docker Compose).
- **www.chatgpt.com** para consultar que comandos utilizar en la instalación de las diversas herramientas.

## Conclusión

Se consiguió dejar operativo un entorno de testeo en Kali con herramientas clave (Burp, VS Code, Docker) y dos aplicaciones vulnerables en contenedores (Juice Shop y crAPI). Este mismo nos será de gran utilidad en el futuro para poder comprender mejor los conceptos teóricos dados en clase al llevarlos a la práctica.
