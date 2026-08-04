## Fase 1 · Apartado 6 — 🛠️ Procedimiento práctico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (AWS EC2)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Con la VM delante.** Aquí está el trabajo.

---

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

> [!example] 🎬 Antes de empezar (todavía SIN grabar, y luego arranca)
> Ya conoces el método desde los prerrequisitos, así que va solo el recordatorio:
> 1. **Crea la entrada de apuntes** de esta fase (`v3-fase-1-infraestructura-cloud-aws-ec2.md`) con su estructura, vacía.
> 2. **Léete los 7 pasos** del procedimiento enteros, para no atascarte a mitad del vídeo.
> 3. Ten **OBS** listo y comprueba **pantalla y micrófono**.
>
> Cuando lo tengas: **arranca la grabación, preséntate y muestra tu identidad**. A partir de ahí, **todo queda grabado** — incluido cualquier paso previo de preparación que venga a continuación.

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
> | **AMI (imagen)** | Busca `Ubuntu Server 26.04 LTS` → selecciona la versión de 64 bits (x86). Si el texto no coincide palabra por palabra, coge la **LTS de 64 bits** que te ofrezca: AWS cambia la redacción cada pocos meses |
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
> > Una AMI (Amazon Machine Image) es como la ISO de un sistema operativo, pero ya preparada para AWS. En lugar de instalar Ubuntu desde cero, AWS carga una "fotografía" del sistema en segundos. Ubuntu 26.04 LTS está disponible gratuitamente en AWS Marketplace.

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

> [!example] 🔌 Paso 7 — EJERCICIO DE VERIFICACIÓN: comprueba tu red desde fuera
> Hasta aquí has configurado la red **y has confiado en que el panel dice la verdad**. Ahora vas a comprobarlo con fuentes **externas e independientes**, que es como se hace de verdad.
>
> > [!info] ¿Qué es una API y por qué la usa un administrador?
> > Una **API** es una web hecha para que la consulte un programa en vez de una persona: en vez de devolverte una página con colores, te devuelve **datos limpios** en formato JSON.
> >
> > ¿Y para qué la quiere un administrador de sistemas? Para **comprobar desde fuera lo que desde dentro no puede ver**. Tu servidor te dirá siempre lo que él cree de sí mismo; un servicio externo te dice **lo que se ve realmente**. Y esa diferencia, cuando aparece, es justo donde está el problema que llevas dos horas buscando.
> >
> > Se consultan con **`curl`**, que ya has usado y que viene instalado en todas partes. Sin programar y sin instalar nada.
>
> **a) Verifica tu cálculo de subred.** Tu red es **`10.0.0.0/24`** (la VPC de AWS).
> Primero, **a mano y sin ayuda**, escribe en tu entrada de apuntes: máscara decimal, dirección de red, broadcast, número de hosts asignables, primero y último.
> Ahora compruébalo:
> ```bash
> curl "https://networkcalc.com/api/ip/10.0.0.0/24"
> ```
> Si no coincide, **no borres tu respuesta**: déjala y explica en el vídeo dónde te equivocaste. Eso enseña más que acertar.
>
> **b) Tu servidor SÍ tiene IP pública. Averigua de quién es.** Desde dentro del servidor:
> ```bash
> curl "https://api.ipify.org?format=json"
> ```
> Compárala con la que te muestra el panel de AWS: **tienen que coincidir**.
>
> Ahora pregunta **quién es el dueño de esa IP**:
> ```bash
> curl "http://ip-api.com/json/TU_IP_PUBLICA?fields=query,country,isp,org,as"
> ```
>
> > [!success] 🤔 Mira bien la respuesta
> > No sale tu nombre: sale **Amazon**, con su número de **AS** y el país del centro de datos.
> > **Eso es "estar en la nube"**, dicho con datos: tu servidor vive dentro de la infraestructura de Amazon, y para el resto de Internet es una máquina más de las suyas.
> > **Explica en el vídeo:** ¿en qué país está físicamente tu servidor? ¿Coincide con el que elegiste al crearlo?
>
> > [!question] Lo que va a tu entrada de apuntes
> > 1. ¿Coincidió tu cálculo de subred con el de la API? Si no, ¿en qué fallaste?
> > 2. ¿Cuál es la IP privada de tu servidor y cuál la pública? ¿Por qué no son la misma?
> > 3. ¿Por qué una comprobación hecha **desde el propio servidor** vale menos que una hecha desde fuera?
>
> > [!note] 📌 Para saber más
> > La teoría completa de esto está en la práctica **B1.9b — Verificar tu red con APIs públicas** del Bloque 1. Aquí lo aplicas a tu servidor de verdad.
> > Y una consecuencia que conviene que asumas ya: **tu servidor es alcanzable desde cualquier punto del planeta.** En cuanto lo enciendes empieza a recibir intentos de conexión de desconocidos. Por eso las siguientes fases dedican tanto tiempo al cortafuegos y a la VPN.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.5_Fundamento_Teorico]] | [[Fase_1]] | [[Fase_1.7_Resolucion_Problemas]] |
