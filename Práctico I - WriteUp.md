# Práctico 1 - Desarrollo de Software Seguro 2025

**Integrantes:** Joaquín Pérez, Federico Díaz y Joaquín Repetto  

---

## Resumen

Este documento detalla, con enfoque reproducible, la instalación y verificación de un entorno de pruebas basado en **Kali Linux** ejecutado en una **máquina virtual** (VirtualBox), la **instalación y configuración** de un **proxy de interceptación** (Burp Suite Community) y de **Visual Studio Code**, la **instalación de Docker**, y la **ejecución de OWASP Juice Shop** y **OWASP crAPI** en contenedores. Se incluyen **pruebas de interceptación de tráfico** HTTPS mediante el proxy y una sección de **solución de problemas**.

> **Objetivo:** Dejar un entorno listo para ejercicios de seguridad web y API, con evidencia de funcionamiento de cada componente.

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
8. [Prueba: interceptación de tráfico con Burp](#prueba-interceptación-de-tráfico-con-burp)  
9. [Buenas prácticas y observaciones](#buenas-prácticas-y-observaciones)  
10. [Solución de problemas](#solución-de-problemas)  
11. [Evidencia y versiones](#evidencia-y-versiones)  
12. [Referencias](#referencias)

---

## Alcance y prerrequisitos

**Alcance:** Virtualización con VirtualBox, Kali Linux estable, herramientas de seguridad web, editor de código, contenedores y dos aplicaciones intencionalmente vulnerables (Juice Shop y crAPI) para laboratorio.

**Prerrequisitos técnicos mínimos (referenciales):**  
- CPU con virtualización habilitada (Intel VT-x/AMD-V), 1–2 vCPU.  
- **RAM**: 2 GB mínimo (recomendado ≥4 GB para uso fluido).  
- **Disco**: ≥20 GB para Kali (recomendado ≥40–60 GB si se usarán imágenes Docker).  
- Conectividad a Internet.  
- Host con VirtualBox 7.x instalado.

> Nota: los valores anteriores son prácticos para laboratorio; si el host lo permite, asigne más RAM/CPU para mejor desempeño de Kali y de contenedores.

---

## Instalación de Kali Linux en VM (VirtualBox)

### Descarga y verificación del ISO

1. Descargue la imagen **Installer (x86-64)** desde el sitio oficial de Kali.  
2. **Verifique la integridad**: descargue además `SHA256SUMS` y `SHA256SUMS.gpg`; verifique primero la firma GPG de `SHA256SUMS` y luego las sumas SHA256 del ISO.  
3. Copie los archivos al directorio de descargas y ejecute:

```bash
cd ~/Descargas
# Verificación de checksums (asumiendo que ya se confió en la clave oficial de Kali)
sha256sum -c SHA256SUMS 2>&1 | grep OK
```

> Si la verificación falla, vuelva a descargar; **no** utilice ISOs no verificadas.

### Creación de la VM

> Herramienta: **Oracle VM VirtualBox** (host Windows/Linux/macOS).

- **Nombre**: `Kali-VM` (tipo: Linux, versión: Debian 64-bit).  
- **Memoria**: 4096 MB (mínimo: 2048 MB).  
- **CPU**: 2 vCPU (mínimo: 1).  
- **Disco**: 40 GB **VDI** dinámico (mínimo: 20 GB).  
- **Red**: NAT (más simple); opcional Host‑Only adicional para ejercicios locales.  
- **Sistema**: habilite EFI solo si lo prefiere; mantenga I/O APIC habilitado.  
- **Almacenamiento**: adjunte el ISO de Kali en el controlador óptico.

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

Cree *snapshots* en VirtualBox al finalizar hitos (sistema limpio, sistema con herramientas, etc.).

---

## Proxy de interceptación: Burp Suite Community

### Instalación de Burp Suite

**Opción A — Paquete en Kali (simple):**

```bash
sudo apt update
sudo apt install -y burpsuite
```

**Opción B — Instalador oficial (.sh):**  
Descargue el instalador para Linux, luego:

```bash
cd ~/Descargas
chmod +x burpsuite_community_linux_v*.sh
./burpsuite_community_linux_v*.sh
```

### Primer inicio de Burp

1. Abra **Burp Suite Community** desde el menú o con `burpsuite`.  
2. Proyecto temporal → *Next* → *Start Burp*.  
3. Verifique que **Proxy → Intercept** y **Proxy → Options → Proxy Listeners** tengan un listener activo en `127.0.0.1:8080`.

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

> Sugerencia: documente el ID de contenedor (`docker ps`) y guarde un *snapshot* tras dejarlo operativo.

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

> Si necesita acceso desde el host a servicios dentro de la VM, agregue reglas de **Port Forwarding** en VirtualBox (NAT) para los puertos 3000, 8888 y 8025.

---

## Prueba: interceptación de tráfico con Burp

### A) Con el navegador integrado de Burp (recomendado)

1. En Burp: **Proxy → Intercept → Open Browser**.  
2. Navegue a `http://127.0.0.1:3000` (Juice Shop) y/o `http://127.0.0.1:8888` (crAPI).  
3. Observe solicitudes/respuestas en **Proxy → HTTP history**.  
4. Para **HTTPS**, descargue el certificado de CA de Burp desde `http://burp` y **añádalo como Autoridad de confianza** en el navegador integrado (o use el asistente de Burp).

### B) Con Firefox del sistema (configuración manual)

1. **Proxy**: Firefox → *Preferencias* → *Configuración de red* → *Configuración…* → **Configuración manual del proxy** → HTTP `127.0.0.1` puerto `8080` → marque “Usar este proxy para todos los protocolos”.  
2. **Certificado CA**: con Burp activo, abra `http://burp`, descargue *CA Certificate* y en Firefox → *Privacidad y seguridad* → *Certificados* → **Ver certificados** → pestaña *Autoridades* → **Importar** → marque “Confiar para identificar sitios web”.  
3. Visite `https://juice-shop:3000` (o cualquier sitio HTTPS) y confirme que **no** aparecen advertencias TLS y que el tráfico se ve en Burp.  
4. (Opcional) Instale **FoxyProxy** para alternar el uso de proxy con un clic y limitar el proxy a dominios/puertos específicos.

> **Buenas prácticas en Burp**: defina *Scope* (Project options → *Scope*) para limitar a `127.0.0.1:3000` y `127.0.0.1:8888`; desactive **Intercept is on** cuando no necesite pausar tráfico.

---

## Buenas prácticas y observaciones

- **Verificación de ISOs**: use `sha256sum` y verificación de firma GPG del archivo `SHA256SUMS`.  
- **Snapshots**: antes de instalar herramientas, después de instalarlas y antes de prácticas críticas.  
- **Actualizaciones**: ejecute `sudo apt update && sudo apt full-upgrade` periódicamente.  
- **Docker sin sudo**: recuerde cerrar sesión tras `usermod -aG docker $USER`.  
- **Puertos**: conserve `3000` (Juice Shop), `8888` (crAPI) y `8025` (MailHog).  
- **Seguridad**: estos servicios son **vulnerables por diseño**; limite su exposición (NAT, cortafuegos, sin port‑forwarding al exterior).

---

## Solución de problemas

**VirtualBox**  
- Error de VT‑x/AMD‑V deshabilitado → habilítelo en BIOS/UEFI.  
- La VM no tiene red → revise adaptador NAT y DNS del huésped.  

**Burp / HTTPS**  
- Advertencias TLS en el navegador → reimporte el **CA de Burp** como *Autoridad* y asegure que el proxy apunte a `127.0.0.1:8080`.  
- No veo tráfico → verifique **Proxy Listener** activo y que el navegador realmente usa el proxy.

**Docker**  
- `permission denied` al ejecutar `docker` sin sudo → faltó *logout/login* tras añadir al grupo `docker`.  
- `hello-world` falla → confirme que `dockerd` está activo (`systemctl status docker`) y conectividad a Internet.  
- Puertos en uso → cambie mapeos `-p` o detenga contenedores que ocupen esos puertos.

**crAPI**  
- Error por versión de Compose → confirme `docker compose version` ≥ 2.x (plugin) o ajuste comandos.  
- No carga web → verifique containers en `docker ps` y revise logs: `docker compose logs -f --tail=200`.

---

## Evidencia y versiones

Ejecute y conserve la salida de:

```bash
# Sistema
lsb_release -a || cat /etc/os-release
uname -a

# Herramientas
burpsuite --version 2>/dev/null || echo "Burp Suite (Community) desde menú"
code --version
docker --version
docker compose version

# Apps vulnerables
curl -sI http://127.0.0.1:3000 | head -n 1
curl -sI http://127.0.0.1:8888 | head -n 1
```

Incluya capturas: (1) Burp con tráfico de `127.0.0.1:3000`, (2) VS Code “About”, (3) `docker ps` con `juice-shop` y servicios de `crAPI` activos.

---

## Referencias

- Documentación oficial de **Kali**: descarga segura del ISO, elección de imagen, requisitos y Docker en Kali.
- Sitio oficial de **VirtualBox** para binarios del host.
- Documentación oficial de **Burp Suite** (descarga/instalación, configuración de proxy y CA).  
- Documentación oficial de **VS Code** para Debian/Ubuntu y repositorio APT.  
- Guía oficial de **OWASP Juice Shop** (ejecución con Docker).  
- Guía oficial de **OWASP crAPI** (despliegue con Docker Compose).

> Las URLs específicas se citan en la versión entregada y en el repositorio de evidencia del grupo.

---

## Conclusión

Se dejó operativo un entorno de laboratorio en Kali con herramientas clave (Burp, VS Code, Docker) y dos aplicaciones vulnerables en contenedores (Juice Shop y crAPI). Se verificó la **interceptación de tráfico** (HTTP/HTTPS) mediante Burp. Este baseline permite replicar prácticas de seguridad web y API de forma controlada y documentada.
