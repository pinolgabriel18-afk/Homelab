[README.md](https://github.com/user-attachments/files/31579219/README.md)
# Homelab
Laboratorio en casa, servidor handless a partir de una laptop vieja
# Legacy Sony Vaio - Rocky Linux 8 Headless Homelab Server

Documentación paso a paso de la conversión y aprovisionamiento de un ordenador portátil antiguo (Sony Vaio) en un **servidor headless en red local**, superando incompatibilidades de arquitectura de CPU y controladores de red propietarios.

---

## Especificaciones del Hardware & Entorno

* **Dispositivo:** Sony Vaio (Arquitectura de procesamiento legacy).
* **Rol:** Servidor Bare-Metal sin interfaz gráfica (Headless Server).
* **Sistema Operativo:** Rocky Linux 8 Minimal (64-bit / Kernel 4.18).
* **Acceso de Administración:** SSH (`sshd`) a través de red local Wi-Fi.

---

## Arquitectura & Desafíos Resueltos

Durante el proceso de despliegue se diagnosticaron y solucionaron diversos cuellos de botella técnicos:

1. **Incompatibilidad de Kernel / CPU (Kernel Panic):**
   * *Problema:* Intento inicial de instalación con Rocky Linux 9 generó un `Kernel Panic` debido a requerimientos de microarquitectura de CPU no soportados.
   * *Solución:* Rollback e instalación limpia de **Rocky Linux 8 Minimal**, garantizando estabilidad nativa en el hardware.

2. **Aislamiento de Red y Bypass mediante USB Tethering:**
   * *Problema:* Falta de controladores propietarios para la interfaz Wi-Fi Broadcom en el instalador minimalista, impidiendo la salida a red.
   * *Solución:* Aprovisionamiento de red temporal mediante **Anclaje a red por USB (USB Tethering)** vía celular y adaptador de red virtual de Android (`enp0s20uX`).

3. **Incompatibilidad de Subredes & Connection Timeout en SSH:**
   * *Problema:* Error `Connection timed out` al intentar acceder por SSH debido a una segmentación de red diferente entre el equipo cliente (Router de hogar) y el servidor (Subred del móvil).
   * *Solución:* Unificación de la topología de red conectando la interfaz de red de la notebook al mismo router local.

4. **Falta de Firmware / Controladores Broadcom Wi-Fi:**
   * *Problema:* La interfaz inalámbrica `wlp...` no listaba redes de entorno (`nmcli device wifi list` vacío).
   * *Solución:* Habilitación del repositorio `EPEL` (*Extra Packages for Enterprise Linux*) y `PowerTools`, instalación de paquetes de soporte inalámbrico (`NetworkManager-wifi`, `linux-firmware`) y reconstrucción del servicio de red.

---

## Guía de Despliegue y Configuración

### 1. Deshabilitar el Modo Suspensión al Cerrar la Tapa (Headless Operations)
Para garantizar la continuidad operativa de los servicios sin interfaz gráfica al bajar la pantalla:

```bash
sudo sed -i 's/#HandleLidSwitch=suspend/HandleLidSwitch=ignore/' /etc/systemd/logind.conf
sudo systemctl restart systemd-logind
```

### 2. Configuración de Red Inalámbrica con NetworkManager (`nmcli`)
Activación de la interfaz Wi-Fi y conexión al punto de acceso local:

```bash
# Habilitar radio inalámbrico y forzar re-escaneo de APs
sudo nmcli radio wifi on
sudo nmcli device wifi rescan

# Conexión al SSID local
sudo nmcli device wifi connect "NOMBRE_DE_RED" password "TU_CONTRASEÑA"

# Configurar auto-conexión al iniciar el sistema
sudo nmcli connection modify wlp2s0 connection.autoconnect yes
```

### 3. Aprovisionamiento de Seguridad y Acceso Remoto (SSH & Firewall)
Habilitación del demonio de Secure Shell y apertura del puerto por defecto (`22/TCP`):

```bash
# Iniciar y habilitar servicio sshd
sudo systemctl start sshd
sudo systemctl enable sshd

# Configuración de reglas en firewalld
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload
```

### 4. Conexión desde Cliente Windows (PowerShell)
Acceso vía terminal cliente tras la autenticación de huella criptográfica SHA256/ED25519:

```powershell
ssh usuario@192.168.100.12
```

---

## Estado del Proyecto

- [x] Instalación de SO Rocky Linux 8 Minimal.
- [x] Gestión de administración remota vía SSH encendida.
- [x] Configuración de comportamiento Headless sin suspensión.
- [x] Habilitación y conexión nativa mediante Wi-Fi Broadcom.
- [ ] Asignación de dirección IP Estática / Reserva DHCP.
- [ ] Despliegue de entorno Docker y contenedores de prueba.
