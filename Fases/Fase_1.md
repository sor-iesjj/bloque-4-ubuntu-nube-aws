## 🏗️ Fase 1: Infraestructura Cloud (AWS EC2)

### Infraestructura de Servidores Cloud

> **[Módulo: SOR — Sistemas Operativos en Red]**
> **[U.T. 1, 2 y 3: Instalación de Sistemas Operativos en Red]**
> **[RA.01]** Instala sistemas operativos en red describiendo sus características e interpretando la documentación técnica.
>
> **Profesor:** Pedro Navarro Miralles  
> **Correo:** p.navarromiralles2@edu.gva.es  
> **Centro:** IES Jorge Juan (ALICANTE)
>
> **⏱️ Tiempo estimado:** ~1,5 horas (teoría + práctica + retos + troubleshooting)  
> **Requisitos:** 2-4 GB RAM | AWS Console | SSH

---

### 🎯 ¿Dónde Estamos?

> [!info] El Punto de Partida
> No vienes de una fase anterior — esta es la base. Pero vienes del mundo real: necesitas un servidor que esté **disponible 24/7**, que no dependa de tu ordenador personal, que sea escalable, profesional y seguro.

> [!warning] El Problema
> Instalar un servidor físico en la clase es caro (comprar hardware), requiere mantenimiento constante (electricidad, refrigeración, actualizaciones de seguridad), no es escalable (si necesitas 2 servidores, necesitas 2 máquinas), y es frágil: una inundación, un apagón o un accidente físico lo destruye. La nube resuelve esto.

> [!success] Objetivo de esta Fase
> Crear una **instancia EC2 en Amazon Web Services** que aloje Ubuntu Server 24.04 LTS. Este servidor será tu controlador de dominio, tu almacenamiento de archivos y la base de toda la infraestructura BoochanV3. Lo protegerás con un **Security Group (firewall cloud)** que bloquea internet y abre solo los puertos imprescindibles: 9090 para monitoreo, 22 para administración.

> [!tip] Hoja de Ruta
> 1. Crear una instancia EC2 en AWS con Ubuntu Server 24.04 LTS (2 GB RAM mínimo)
> 2. Configurar el Security Group: abrir puertos 9090 (Cockpit) y 22 (SSH) — nada más
> 3. Asignar una Elastic IP para tener una IP pública fija
> 4. Conectarse al servidor por SSH desde tu PC (primera vez que entras)
> 5. Verificar acceso a internet y DNS (`curl google.com`)
> 6. Medir RAM base con `free -h` (línea base para comparar en fases futuras)
>
> **Resultado Final:** Un servidor en la nube listo, accesible, aislado.
> **Siguiente:** Fase 2 (Purga y FQDN) — limpiaremos el servidor de software innecesario y le daremos una identidad de dominio (BOOCHAN.SPACE).

---

### 📚 Fundamento Teórico Avanzado

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
> - **La contraseña del dominio (`P@ssword2026!`):** se usa más adelante (Fases 4-8) para los **usuarios de Active Directory** que iniciarán sesión en Windows. Es una contraseña normal porque así funciona el dominio.
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

### 🛠️ Procedimiento Práctico (BoochanV3)

> [!danger] 🔑 LÉEME ANTES DE EMPEZAR: cómo accedemos sin pagar
> Este proyecto **NO usa una cuenta normal de AWS con tarjeta de crédito**. Para un aula, la vía correcta es **AWS Academy Learner Lab**:
> - El centro se da de alta en **AWS Academy** (gratis para instituciones educativas). El profesor invita a cada alumno por correo.
> - Cada alumno entra en un **"Learner Lab"** con un **crédito de unos 50-100 USD** ya cargado. **No se pide tarjeta de crédito ni datos bancarios** — ideal para alumnos menores de edad.
> - El laboratorio se **enciende y apaga** con un botón ("Start Lab" / "End Lab"). Mientras está apagado, **no consume crédito**.
>
> > [!warning] ⚠️ El "Free Tier" gratuito NO sirve para este proyecto
> > La capa gratuita de AWS (12 meses) solo da la instancia **t3.micro / t2.micro de 1 GB de RAM**. Samba AD DC (Fase 4) necesita **2-4 GB**, así que con el Free Tier puro el dominio no arranca. Por eso usamos AWS Academy, cuyo crédito sí cubre una **t3.medium (4 GB)**. Con el laboratorio apagado entre clases, ese crédito da de sobra para todo el curso.
>
> Una vez dentro de la consola, asegúrate de seleccionar la **región correcta** en el desplegable de arriba a la derecha (la que indique tu profesor; en AWS Academy suele ser fija, p. ej. `us-east-1`). **Todos tus recursos deben estar en la misma región.**

> [!example] Paso 1: Crear el Key Pair (tu llave de acceso)
> Antes de crear la instancia, necesitas generar las claves de acceso SSH.
>
> 1. En la barra de búsqueda superior, escribe `EC2` y haz clic en el servicio.
> 2. En el menú izquierdo, dentro de **Red y seguridad**, haz clic en **`Pares de claves`** (Key Pairs).
> 3. Haz clic en **`Crear par de claves`**.
> 4. Rellena el formulario:
>
> | Campo | Valor |
> | :--- | :--- |
> | **Nombre** | `boochan-key` |
> | **Tipo de par de claves** | `RSA` |
> | **Formato de archivo de clave privada** | `.pem` (para Linux/Mac/Windows con OpenSSH) |
>
> 5. Haz clic en **`Crear par de claves`**. Se descargará automáticamente un fichero `boochan-key.pem`.
>
> > [!warning] ⚠️ ¡Guarda el fichero .pem ahora!
> > Este fichero **solo se descarga una vez**. Si lo pierdes, no podrás conectarte al servidor y tendrás que crear uno nuevo. Guárdalo en una carpeta segura de tu PC (por ejemplo, `C:\Users\TuUsuario\.ssh\`). **No lo compartas con nadie.**

> [!example] Paso 2: Crear el Security Group
> El Security Group actúa como el firewall de tu instancia. Lo creamos antes de lanzar la instancia para poder asignárselo en el momento de la creación.
>
> 1. En el menú izquierdo de EC2, dentro de **Red y seguridad**, haz clic en **`Grupos de seguridad`**.
> 2. Haz clic en **`Crear grupo de seguridad`**.
> 3. Rellena:
>
> | Campo | Valor |
> | :--- | :--- |
> | **Nombre del grupo de seguridad** | `sg-boochan-[tunombre]` |
> | **Descripción** | `Seguridad BoochanV3 SOR` |
> | **VPC** | La VPC por defecto (default) |
>
> 4. En **Reglas de entrada**, haz clic en **`Agregar regla`** y añade **una regla por cada fila**:
>
> | Tipo | Protocolo | Puerto | Origen | Descripción |
> | :--- | :--- | :--- | :--- | :--- |
> | TCP personalizado | TCP | 9090 | 0.0.0.0/0 | Cockpit (monitorización web) |
> | SSH | TCP | 22 | 0.0.0.0/0 | Acceso SSH inicial |
>
> > [!warning] ⚠️ El origen 0.0.0.0/0 significa "cualquier IP del mundo"
> > Es necesario de momento porque aún no tenemos VPN. En la **Fase 3**, cuando WireGuard esté funcionando, restringiremos SSH y lo cerraremos al exterior. Por ahora es el único acceso posible.
>
> 5. Deja las **Reglas de salida** por defecto (todo el tráfico saliente permitido).
> 6. Haz clic en **`Crear grupo de seguridad`**.
>
> > [!info] 💡 ¿Por qué solo dos puertos de momento?
> > Con estos dos puertos puedes entrar al servidor y ver su estado. El resto de puertos (VPN, dominio, carpetas compartidas...) los abriremos en cada fase cuando instalemos el servicio correspondiente. Así nunca abrimos una puerta sin saber para qué sirve.

> [!example] Paso 3: Lanzar la Instancia EC2
> Ahora lanzamos el servidor con Ubuntu.
>
> 1. En el menú izquierdo, haz clic en **`Instancias`** → **`Lanzar instancias`**.
> 2. Rellena el formulario con exactamente estos valores:
>
> | Campo | Valor |
> | :--- | :--- |
> | **Nombre** | `UbuntuServer` |
> | **AMI (imagen)** | Busca `Ubuntu Server 24.04 LTS` → selecciona la versión de 64 bits (x86) |
> | **Tipo de instancia** | `t3.small` (2 vCPU, 2 GB RAM) |
> | **Par de claves** | `boochan-key` (el que creaste en el Paso 1) |
> | **Grupo de seguridad** | Selecciona el existente: `sg-boochan-[tunombre]` |
> | **Almacenamiento** | 20 GB gp3 (disco SSD) |
>
> > [!warning] ⚠️ Para Fases 4 en adelante necesitarás más RAM
> > El tipo `t3.small` tiene 2 GB de RAM, suficiente para las primeras fases. Cuando instales Samba AD DC (Fase 4), necesitarás **t3.medium (4 GB)**. Puedes cambiar el tipo de instancia deteniéndola y modificando el tipo. Pregunta al profesor cuándo hacer el cambio.
>
> 3. Haz clic en **`Lanzar instancia`**. Espera 1-2 minutos hasta que el estado sea `En ejecución` (verde).
>
> > [!tip] 💡 ¿Qué es una AMI?
> > Una AMI (Amazon Machine Image) es como la ISO de un sistema operativo, pero ya preparada para AWS. En lugar de instalar Ubuntu desde cero, AWS carga una "fotografía" del sistema en segundos. Ubuntu 24.04 LTS está disponible gratuitamente en AWS Marketplace.

> [!example] Paso 4: Asignar una Elastic IP
> Sin una IP fija, tu instancia cambia de IP pública cada vez que se reinicia. La Elastic IP te da una IP permanente.
>
> 1. En el menú izquierdo, dentro de **Red y seguridad**, haz clic en **`Direcciones IP elásticas`**.
> 2. Haz clic en **`Asignar dirección IP elástica`** → **`Asignar`**.
> 3. Selecciona la IP recién creada → **`Acciones`** → **`Asociar dirección IP elástica`**.
> 4. En el formulario, selecciona tu instancia `UbuntuServer` → **`Asociar`**.
>
> Anota la IP elástica asignada: la necesitarás en todas las fases del proyecto.
>
> > [!info] 💡 ¿Qué diferencia hay con una IP normal de EC2?
> > Al lanzar una instancia, AWS le asigna una IP pública automáticamente, pero esa IP **cambia cada vez que reinicias**. La Elastic IP es tuya aunque apagues y enciendas la instancia cuantas veces quieras. En producción, un cambio de IP significaría que todos los clientes pierden la conexión al servidor.
>
> > [!warning] 💸 Cuidado con el coste de la Elastic IP cuando apagas la instancia
> > AWS regala la Elastic IP **mientras está asociada a una instancia en ejecución**. Pero si **detienes** (stop) la instancia y la EIP queda reservada sin usarse, AWS cobra una pequeña tarifa por hora. En **AWS Academy** esto consume crédito. Por eso, en este proyecto, en lugar de *detener* la instancia entre clases usamos el botón **"End Lab"** del laboratorio, que congela todo el entorno sin penalización. Pregunta al profesor por el procedimiento de cierre al acabar cada sesión.

> [!example] Paso 5: Primera Conexión al Servidor (SSH con Key Pair)
> Abre una terminal en tu ordenador:
> - **Windows:** Pulsa `Windows + R`, escribe `cmd` y pulsa Enter.
> - **Mac / Linux:** Abre la aplicación `Terminal`.
>
> Primero, ajusta los permisos del fichero .pem (solo en Mac/Linux — en Windows no es necesario):
> ```bash
> chmod 400 ~/boochan-key.pem
> ```
>
> Ahora conéctate sustituyendo `TU_IP_ELASTICA` por la IP que anotaste:
> ```bash
> ssh -i ~/boochan-key.pem ubuntu@TU_IP_ELASTICA
> ```
>
> La primera vez verás un mensaje de advertencia sobre la autenticidad del servidor. Escribe `yes` y pulsa Enter.
>
> Si al final ves algo parecido a `ubuntu@UbuntuServer:~$`, **ya estás dentro de tu servidor**.
>
> > [!warning] ⚠️ El usuario por defecto en Ubuntu de AWS es `ubuntu`, no `root`
> > A diferencia de otros sistemas, AWS configura Ubuntu con un usuario llamado `ubuntu` (no `boochan`, no `admin`, no `root`). Para ejecutar comandos como administrador usa `sudo` delante de cada comando.
>
> > [!tip] 💡 ¿Qué es SSH?
> > SSH (Secure Shell) es como un **"mando a distancia" cifrado** para tu servidor. Desde el teclado de tu PC del aula estás enviando comandos que se ejecutan en el ordenador de AWS, a cientos de kilómetros. Todo el tráfico viaja cifrado para que nadie pueda interceptar lo que escribes.

> [!example] Paso 6: Verificaciones iniciales
> Ya dentro del servidor, ejecuta estos comandos para confirmar que todo funciona:
>
> ```bash
> # Verificar acceso a internet
> curl -s google.com | head -5
>
> # Ver RAM disponible (anótala — la compararás en Fase 4)
> free -h
>
> # Ver disco disponible
> df -h /
>
> # Actualizar el sistema (siempre lo primero)
> sudo apt update && sudo apt upgrade -y
> ```
>
> > [!tip] 💡 ¿Cómo verifico si los puertos están "vivos"?
> > Una vez que tengas servicios corriendo en las siguientes fases, usa este comando para ver qué puertos están escuchando:
> > ```bash
> > sudo ss -tulpn
> > ```

---

### 🚩 Resolución de Problemas y Evaluación

> [!bug] Tabla de Troubleshooting (¿Algo no funciona?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | "Permission denied (publickey)" al conectar por SSH | El fichero .pem no es correcto o no tiene permisos adecuados | Comprueba que usas `-i ruta/boochan-key.pem` y que el fichero es el que descargaste en el Paso 1. En Mac/Linux ejecuta `chmod 400 boochan-key.pem` |
> | La instancia no arranca o está en estado "Pendiente" | AWS está inicializando la instancia | Espera 2-3 minutos y actualiza la página. |
> | El servidor **no responde al ping** | El protocolo ICMP está bloqueado por defecto en el Security Group | Es normal por seguridad. No abras el ping; usa `ssh` o `curl` para verificar conectividad. |
> | "Host key verification failed" | La IP de la instancia ha cambiado (si no usas Elastic IP) | Si no tienes Elastic IP asignada, la IP cambia en cada reinicio. Asigna una Elastic IP (Paso 4). |

> [!help] Preguntas Críticas (Autoevaluación del alumno)
> 1. ¿Por qué Amazon AWS es responsable del hardware pero tú eres el responsable del Sistema Operativo?
> 2. ¿Qué ocurre exactamente si dejas el puerto de administración SSH abierto a **"Cualquiera" (0.0.0.0/0)** permanentemente?
> 3. ¿Qué ventaja tiene usar un Key Pair (.pem) frente a una contraseña? ¿Qué desventaja tiene?
> 4. 🔬 **Reto práctico:** Entra en el Security Group de AWS y **deshabilita** temporalmente la regla del puerto 9090 (elimínala y vuelve a añadirla). Luego intenta abrir Cockpit desde el navegador. ¿Qué ocurre? ¿Qué has comprobado con este experimento?
> 5. 🔬 **Reto práctico:** Ejecuta `free -h` en el servidor. Anota cuánta RAM está libre ahora, con el sistema base sin servicios. Guarda ese dato — lo compararás con la Fase 4, cuando Samba AD DC esté corriendo.

---

> [!caution] 🛑 Auditoría de Seguridad — Tarea pendiente tras la Fase 3
> Una vez que la VPN esté funcionando, realizarás dos acciones para cerrar el servidor al mundo exterior. **No las hagas ahora**: sin VPN activa te quedarías sin acceso.
>
> **Acción 1 — Cambiar el puerto SSH de 22 a 2222 en el servidor:**
> ```bash
> sudo nano /etc/ssh/sshd_config
> ```
> Busca la línea `#Port 22`, elimina el `#` y cambia el número a `2222`. Guarda y reinicia el servicio:
> ```bash
> sudo systemctl restart ssh
> ```
> A partir de aquí, conéctate siempre con:
> ```bash
> ssh -i ~/boochan-key.pem -p 2222 ubuntu@10.0.0.1
> ```
>
> **Acción 2 — Actualizar el Security Group de AWS:**
> En el Security Group `sg-boochan-[tunombre]`, **elimina la regla del puerto 22** que abriste en el Paso 2. El puerto 2222 (solo desde la VPN) ya estará activo desde la Fase 3.
>
> Esto es aplicar seguridad "Zero Trust": nadie en Internet puede llegar al servidor; solo quien esté dentro de la VPN.
