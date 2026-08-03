# 🚀 BoochanV3 — Infraestructura de Servidores Cloud sobre AWS (Ubuntu Server + Samba AD DC)

> **Módulo:** Sistemas Operativos en Red (SOR) · 2.º Curso SMR
> **Profesor:** Pedro Navarro Miralles · IES Jorge Juan (Alicante)
> **Correo:** p.navarromiralles2@edu.gva.es
> **Entorno:** Amazon Web Services (EC2, AWS Academy Learner Lab) — adaptación de BoochanV2 (que usaba Microsoft Azure)
> **RA cubiertos:** RA.01, RA.02, RA.03, RA.04, RA.05, RA.06
> **⏱️ Tiempo estimado total:** ~13-14 horas repartidas en 9 sesiones

---

## ¿Qué es este proyecto?

BoochanV3 es un itinerario práctico de **8 fases + auditoría final** en el que el alumno construye, desde cero y en la nube de **Amazon Web Services**, una infraestructura profesional completa: un servidor **Ubuntu Server** con **Controlador de Dominio (Samba AD DC)**, **VPN WireGuard**, **cuotas de disco (Loop Devices)**, **permisos avanzados (ACL + ABE)** y un **cliente Windows 11** integrado en el dominio a través de un túnel cifrado.

Es la versión **AWS** de BoochanV2 (que se desplegaba sobre Azure). La teoría, los comandos y los ejercicios se han adaptado al ecosistema de Amazon: **EC2, Security Groups, Elastic IP, Key Pairs y AWS Academy Learner Lab**.

---

## Relación con las otras versiones del proyecto Boochan

El proyecto Boochan existe en varias versiones equivalentes que enseñan **exactamente los mismos conceptos** de administración de sistemas en red, cambiando solo dónde vive el servidor y qué implementación de directorio se usa:

| Entorno | Ubuntu + Samba AD DC | Windows Server 2025 + AD DS nativo |
|---|---|---|
| VM local | BoochanV1 (VirtualBox) | BoochanV1.1 (Hyper-V) |
| Azure | BoochanV2 | BoochanV2.1 |
| **AWS** | **BoochanV3 (esta)** | BoochanV3.1 |

BoochanV3 es la rama **AWS + Linux**. Comparte con BoochanV2 el mismo itinerario y el mismo Samba AD DC, cambiando la infraestructura de Azure por la de AWS. Su hermana BoochanV3.1 hace lo mismo pero con Windows Server 2025 y AD DS nativo en lugar de Ubuntu + Samba.

---

## ⚠️ Antes de empezar: cómo accedemos sin pagar (LÉEME)

Este proyecto **NO usa una cuenta normal de AWS con tarjeta de crédito**, ni se puede hacer solo con la capa gratuita (Free Tier):

- El **Free Tier** de AWS solo da instancias de **1 GB de RAM**, insuficiente para Samba AD DC (necesita 2-4 GB).
- La vía correcta para un aula es **AWS Academy Learner Lab**: crédito precargado (~50-100 USD), **sin tarjeta de crédito**, ideal para alumnos menores. Se enciende/apaga con un botón (Start Lab / End Lab) y, apagado, no consume crédito.
- **Windows 11 (PC físico del aula)** como cliente, necesario a partir de la Fase 8 — es el propio ordenador del alumno unido al dominio por VPN, no una máquina virtual.
- **WireGuard** instalado tanto en el servidor como en el PC del aula, desde la Fase 3.

El detalle completo está al inicio de la **[Fase 1](Fases/Fase_1.md)**.

---

## 🗺️ Índice de fases

| Fase | Título | Concepto AWS / Linux clave |
|------|--------|----------------------------|
| [1](Fases/Fase_1.md) | Infraestructura Cloud (AWS EC2) | EC2 `t3.medium`, Security Group, Elastic IP, Key Pair `.pem` |
| [2](Fases/Fase_2.md) | Purga y Preparación del Entorno | Limpieza de Samba preinstalado, FQDN, `/etc/hosts` |
| [3](Fases/Fase_3.md) | Conectividad VPN (WireGuard) | Túnel cifrado servidor↔aula, cierre del puerto 22 (Zero Trust) |
| [4](Fases/Fase_4.md) | Aprovisionamiento del Dominio (Samba AD DC) | `provision_boochan.sh`, Active Directory, Kerberos, DNS interno |
| [5](Fases/Fase_5.md) | Gestión de Identidades (Usuarios y Grupos) | winbind, RFC 2307, mapeo UID/GID |
| [6](Fases/Fase_6.md) | Almacenamiento Virtual (Cuotas) | Loop Devices (`.img`), `fstab` |
| [7](Fases/Fase_7.md) | Seguridad Avanzada (ACLs y ABE) | `setfacl`, Access Based Enumeration (carpetas invisibles) |
| [8](Fases/Fase_8.md) | Integración del Cliente (Windows 11) | PC físico del aula unido al dominio, mapeo de unidades |
| [Final](Fases/Auditoria_Final.md) | Auditoría Final y Hardening | Zero Trust, restricción de origen en el Security Group |

### Resumen de cada fase

**[Fase 1 — Infraestructura Cloud (AWS EC2)](Fases/Fase_1.md):** dentro del AWS Academy Learner Lab se lanza una instancia EC2 `t3.medium` con Ubuntu Server, se crea el Security Group con los puertos necesarios, se asocia una Elastic IP y se accede por SSH con el Key Pair `.pem`. Se fija el dominio del proyecto: `BOOCHAN` / `BOOCHAN.SPACE`.

**[Fase 2 — Purga y Preparación del Entorno](Fases/Fase_2.md):** se limpia el Samba preinstalado (para liberar el puerto 445), se instalan las dependencias (Samba, Kerberos, winbind, WireGuard…) y se fija el FQDN del servidor en `/etc/hosts`.

**[Fase 3 — Conectividad VPN (WireGuard)](Fases/Fase_3.md):** se construye un túnel cifrado punto a punto entre el servidor en AWS y el PC del aula, sobre Internet real. Al final se aplica Zero Trust cerrando el SSH público y dejándolo accesible solo por la VPN.

**[Fase 4 — Aprovisionamiento del Dominio (Samba AD DC)](Fases/Fase_4.md):** con el script `provision_boochan.sh` se promociona el servidor a Controlador de Dominio (Samba AD DC): base de datos LDAP, Kerberos y DNS interno. El script blinda el `/etc/resolv.conf` con `chattr +i` para que AWS no lo sobrescriba.

**[Fase 5 — Gestión de Identidades (Usuarios y Grupos)](Fases/Fase_5.md):** se crean usuarios y grupos con `samba-tool`, y se mapean sus identidades de Windows (SID) a identidades de Linux (UID/GID) mediante winbind, para que el sistema de ficheros aplique permisos reales.

**[Fase 6 — Almacenamiento Virtual (Cuotas)](Fases/Fase_6.md):** se limitan físicamente las carpetas con **Loop Devices** — archivos `.img` formateados como discos reales y montados vía `/etc/fstab` — para que ningún usuario pueda llenar el servidor.

**[Fase 7 — Seguridad Avanzada (ACLs y ABE)](Fases/Fase_7.md):** se combinan permisos NTFS-like de Linux (`setfacl`, el "cerrojo real") con **Access Based Enumeration** de Samba (la "capa de invisibilidad": quien no tiene permiso ni siquiera ve la carpeta).

**[Fase 8 — Integración del Cliente (Windows 11)](Fases/Fase_8.md):** el **PC físico del aula** activa el túnel WireGuard, sincroniza su reloj, se une al dominio `BOOCHAN.SPACE` y mapea las carpetas compartidas — demostrando que un sistema propietario (Windows) se autentica contra un servidor libre (Linux/Samba).

**[Auditoría Final — Hardening](Fases/Auditoria_Final.md):** cierre de seguridad con el principio Zero Trust — se restringe el origen de las reglas del Security Group a la red de la VPN, dejando el servidor invisible desde Internet salvo el propio puerto del túnel.

---

## 📊 Datos clave del proyecto

| Concepto | Valor en BoochanV3 |
| :--- | :--- |
| **Nombre NetBIOS** | `BOOCHAN` |
| **Realm (dominio completo)** | `BOOCHAN.SPACE` |
| **Instancia del servidor** | `t3.medium` (2 vCPU, 4 GB RAM), Ubuntu Server 26.04 LTS |
| **AMI / imagen** | Ubuntu Server (catálogo oficial de AWS) |
| **IP privada del servidor (VPC por defecto)** | `172.31.x.x` |
| **IP pública** | Elastic IP (para SSH inicial y como `Endpoint` del túnel WireGuard) |
| **Red del túnel VPN (WireGuard)** | `10.0.0.0/24` (servidor `10.0.0.1`, cliente del aula `10.0.0.2`) |
| **Acceso / credenciales** | Usuario `ubuntu` + Key Pair `.pem` (SSH) |
| **Firewall perimetral** | Security Group de AWS |
| **Sistema operativo servidor** | Ubuntu Server 26.04 LTS (headless) |
| **Sistema operativo cliente** | Windows 11 (PC físico del aula) |
| **Plataforma / coste** | AWS Academy Learner Lab (crédito educativo · Start Lab / End Lab) |

---

## 📂 Estructura de la carpeta

```
BoochanV3/
├── Manual_BoochanV3.md           ← este documento (punto de entrada)
├── provision_boochan.sh          ← script de provisión del dominio (Fase 4)
├── Fases/
│   ├── Fase_1.md … Fase_8.md     ← las 8 fases del itinerario
│   └── Auditoria_Final.md        ← cierre de seguridad (hardening del Security Group)
└── 99_Recursos/
    ├── Diccionario_Comandos_Sistema.md    ← comandos de Linux / Samba / WireGuard
    ├── Guía_Editor_Nano.md                ← cómo editar ficheros con nano
    └── Guía_Errores_y_Resolución.md       ← catálogo de errores por fase
```

---

## 🧭 Recomendación de uso

1. Lee este manual y la advertencia de requisitos (AWS Academy Learner Lab, Windows 11 del aula, WireGuard).
2. Sigue las fases **en orden** — son dependientes entre sí (las Fases 4, 5, 7 y 8 son secuenciales; la Fase 8 requiere las Fases 1-7 completas y un PC físico del aula disponible).
3. Si algo falla, antes de bloquearte consulta **[99_Recursos/Guía_Errores_y_Resolución.md](99_Recursos/Guía_Errores_y_Resolución.md)**, organizada por fase.
4. Para repasar comandos de Linux/Samba/WireGuard consulta **[99_Recursos/Diccionario_Comandos_Sistema.md](99_Recursos/Diccionario_Comandos_Sistema.md)**; para editar ficheros, **[99_Recursos/Guía_Editor_Nano.md](99_Recursos/Guía_Editor_Nano.md)**.
5. Al terminar cada sesión pulsa **End Lab** para no gastar crédito mientras no se usa.
