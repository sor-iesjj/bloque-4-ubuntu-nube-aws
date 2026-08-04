## Fase 1 · Apartado 5 — 📚 Fundamento teórico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (AWS EC2)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Antes de teclear.** Los conceptos que necesitas.

---

> [!info] ¿Por qué usamos la Nube (IaaS)?
> El concepto de **IaaS (Infraestructura como Servicio)** es el primer pilar de la administración moderna. Tradicionalmente, instalaríamos Ubuntu Server introduciendo un USB en una máquina física en el aula. En este proyecto daremos el salto profesional: en lugar de tocar un ordenador físico, alquilamos recursos en centros de datos masivos de Amazon Web Services (AWS), el mayor proveedor de nube del mundo.

> [!abstract] 1. La "Magia" del Hipervisor y la Virtualización
> El **Hipervisor** es una capa de software de bajo nivel que gestiona los recursos de un servidor físico real y te entrega una porción exacta de CPU, RAM y disco. 
> - **Tu servidor:** Cree que es un ordenador físico completo.
> - **La Realidad:** Es un archivo ejecutándose dentro de otro ordenador gigante. Esto permite que un solo superordenador de AWS albergue cientos de servidores de alumnos de forma aislada.
> - **En AWS, esto se llama EC2 (Elastic Compute Cloud):** el servicio que alquila potencia de cómputo bajo demanda, pagando solo por lo que usas.

> [!warning] 2. El Modelo de Responsabilidad Compartida
> Trabajar en la nube no significa que "todo es mágico y seguro". AWS funciona bajo este modelo:
> *   **Responsabilidad de Amazon:** Seguridad física (evitar robos de discos), electricidad y conexión a Internet.
> *   **Tu Responsabilidad (El Administrador):** Eres el responsable absoluto de lo que ocurre **dentro** de tu instancia. Si configuras mal una contraseña o dejas un puerto abierto, los hackers entrarán. ¡Amazon no te protegerá de tus propios errores de configuración!

> [!important] 3. Arquitectura Cliente-Servidor "Headless" (Sin Cabeza)
> Vamos a desplegar un servidor **"Headless"**. Esto significa que no tiene entorno gráfico (ni escritorio, ni ratón, ni ventanas). Lo controlamos 100% mediante comandos de texto (CLI).
> *   **¿Por qué?** Porque los entornos gráficos consumen mucha memoria RAM. Un servidor debe dedicar el 100% de sus recursos a dar servicio, no a dibujar iconos. 
> *   **Seguridad:** Cada programa instalado es una puerta potencial para un hacker. Al eliminar la interfaz gráfica, reducimos las "puertas" y blindamos el sistema.

> [!note] 4. Seguridad Perimetral y Protocolos (TCP vs UDP)
> Antes de encender el servidor, lo protegemos con un **Security Group**, que actúa como la muralla del castillo. Solo abriremos las "puertas" (puertos) necesarias usando estos protocolos:
> *   **TCP (Transmission Control Protocol):** Para administrar el servidor (SSH) y archivos (SMB). TCP exige confirmación de entrega. Si un dato se pierde, se vuelve a pedir. Es **fiable pero más lento**.
> *   **UDP (User Datagram Protocol):** Para la VPN (WireGuard) y la hora (NTP). UDP dispara los paquetes a máxima velocidad sin preguntar nada. Es **rapidísimo pero menos fiable**.

> [!note] 5. Autenticación por Clave Criptográfica (Key Pair)
> A diferencia de otros proveedores cloud que permiten usuario y contraseña, AWS usa por defecto **Key Pairs**: un par de claves criptográficas (pública + privada).
> - **Clave pública:** Se guarda en el servidor (como la cerradura).
> - **Clave privada (.pem):** La guardas tú en tu PC (como la llave). Sin ella, nadie puede entrar.
> - **¿Por qué es más seguro?** Una contraseña puede adivinarse por fuerza bruta. Una clave RSA de 2048 bits tardaría millones de años en romperse con los ordenadores actuales.
> - **⚠️ Si pierdes el fichero .pem, pierdes el acceso al servidor para siempre.** Guárdalo bien.

> [!important] 🔐 Dos credenciales distintas — no las confundas
> En este proyecto convivirán **dos sistemas de acceso diferentes**, cada uno para su cosa:
> - **La clave `.pem` (Key Pair):** solo sirve para **entrar por SSH al servidor Linux** (eres el administrador del sistema). Es criptografía asimétrica.
> - **La contraseña del dominio (`P@ssw0rd`):** se usa más adelante (Fases 4-8) para los **usuarios de Active Directory** que iniciarán sesión en Windows. Es una contraseña normal porque así funciona el dominio.
>
> Resumen: **llave `.pem` = administrar el servidor; contraseña = usuarios del dominio.** Son mundos separados.

### 📖 Diccionario de Conceptos Clave

> [!quote] Terminología Profesional AWS
> - **EC2 (Elastic Compute Cloud):** El servicio de AWS que alquila instancias (máquinas virtuales).
> - **Instancia:** Una máquina virtual activa y ejecutándose en AWS.
> - **AMI (Amazon Machine Image):** La "fotografía" del sistema operativo con la que arranca tu instancia. Equivale a la ISO de instalación.
> - **Security Group:** Un firewall lógico que controla qué tráfico de red entra y sale de tu instancia.
> - **Elastic IP (EIP):** Una dirección IP pública fija reservada para ti en AWS. Sin ella, tu instancia cambia de IP cada vez que se reinicia.
> - **Key Pair (.pem):** Fichero de clave privada RSA para autenticar SSH. Solo lo tienes tú.
> - **VPC (Virtual Private Cloud):** La red privada virtual donde viven tus instancias EC2.
> - **LTS (Long Term Support):** Versión de software con actualizaciones de seguridad garantizadas durante 5 años.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.4_Donde_Estamos]] | [[Fase_1]] | [[Fase_1.6_Procedimiento]] |
