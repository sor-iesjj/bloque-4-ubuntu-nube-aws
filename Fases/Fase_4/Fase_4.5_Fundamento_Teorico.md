## Fase 4 · Apartado 5 — 📚 Fundamento teórico

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Aprovisionamiento del Dominio (Samba AD DC)**
> 🧭 Índice de la fase: [[Fase_4]]
>
> **📍 Cuándo se lee:** **Antes de teclear.** Los conceptos que necesitas.

---

> [!abstract] 1. El "Cerebro" de la Red: Active Directory (AD)
> Estamos creando el **Active Directory**. Este es el "Cerebro" que gestiona la base de datos de todos los objetos de la red: usuarios, grupos y ordenadores. Samba AD DC emula tres servicios vitales para que esto funcione:
> *   **LDAP:** El protocolo para consultar la base de datos de usuarios.
> *   **Kerberos:** El sistema de "tickets" de seguridad (como un pase VIP de un festival).
> *   **DNS Interno:** Samba gestiona sus propios registros SRV que indican dónde están los servicios de red.

> [!important] 2. Inmutabilidad y Persistencia
> AWS intenta gestionar el DNS por ti automáticamente. Sin embargo, para que el Dominio funcione, el servidor debe apuntar a **sí mismo** para resolver nombres. 
> Al usar el comando `chattr +i` sobre el archivo `/etc/resolv.conf`, lo hacemos **inmutable** (imposible de borrar o cambiar), evitando que el servicio de red de AWS "rompa" nuestra configuración de DNS local al reiniciar.

### 📖 Diccionario de Conceptos Clave

> [!quote] Terminología de Dominio
> - **Reino (Realm):** El nombre de dominio completo (ej. `BOOCHAN.SPACE`). Siempre se escribe en **MAYÚSCULAS** para que Kerberos lo entienda.
> - **Provisionamiento:** El acto de generar la base de datos del dominio desde cero.
> - **SRV Record:** Un registro DNS especial que indica qué servidor ofrece un servicio específico (ej. "el servidor de tickets está en esta IP").
> - **chattr +i:** El "cemento armado" de Linux. Hace que un archivo no se pueda modificar ni por el administrador.

---

### 🔓 Apertura de Puertos (Security Group de AWS)

> [!example] 🎬 Antes de empezar (todavía SIN grabar, y luego arranca)
> Ya conoces el método desde los prerrequisitos, así que va solo el recordatorio:
> 1. **Crea la entrada de apuntes** de esta fase (`b4-aws-4-aprovisionamiento-del-dominio-samba-ad-dc.md`) con su estructura, vacía.
> 2. **Léete los 3 pasos** del procedimiento enteros, para no atascarte a mitad del vídeo.
> 3. Ten **OBS** listo y comprueba **pantalla y micrófono**.
>
> Cuando lo tengas: **arranca la grabación, preséntate y muestra tu identidad**. A partir de ahí, **todo queda grabado** — incluido cualquier paso previo de preparación que venga a continuación.

> [!example] Al empezar: abre los puertos del dominio
> Active Directory es un ecosistema de servicios que se hablan entre sí. Antes de provisionar el dominio, todos sus puertos deben estar abiertos en AWS — si falta uno, los clientes Windows no podrán autenticarse ni resolver nombres.
>
> 1. Entra en **console.aws.amazon.com** → busca **`EC2`** → en el menú izquierdo, dentro de **Red y seguridad**, haz clic en **`Grupos de seguridad`**.
> 2. Marca tu grupo `sg-boochan-[tunombre]` y abajo pulsa la pestaña **`Reglas de entrada`** → **`Editar reglas de entrada`**.
> 3. Para **cada fila** de la tabla siguiente, pulsa **`Agregar regla`** y rellena los campos:
>    - **Tipo:** `TCP personalizado` o `UDP personalizado` según la columna "Protocolo"
>    - **Intervalo de puertos:** el número de la columna "Puerto"
>    - **Origen:** `Anywhere-IPv4` (equivale a `0.0.0.0/0`)
>    - **Descripción:** el texto de la columna "Nombre"
> 4. Cuando hayas añadido las 13 reglas, pulsa **`Guardar reglas`** (se guardan todas de golpe, no una a una):
>
> | Nombre | Puerto | Protocolo | Para qué sirve ahora |
> | :--- | :--- | :--- | :--- |
> | Kerberos_TCP | 88 | TCP | Emite los "tickets" de seguridad que identifican a cada usuario del dominio. |
> | Kerberos_UDP | 88 | UDP | Ídem por UDP — Windows usa ambos según el tipo de petición. |
> | DNS_TCP | 53 | TCP | Resuelve los nombres del dominio (ej. `BOOCHAN.SPACE`). |
> | DNS_UDP | 53 | UDP | Ídem por UDP — la mayoría de consultas DNS viajan por UDP. |
> | RPC_Endpoint | 135 | TCP | Punto de entrada para las llamadas a procedimiento remoto de Windows. |
> | LDAP_TCP | 389 | TCP | Permite consultar el directorio de usuarios y grupos del dominio. |
> | LDAP_UDP | 389 | UDP | Ídem por UDP. |
> | LDAPS | 636 | TCP | Versión cifrada de LDAP — protege las consultas de usuarios en tránsito. |
> | SMB_Files | 445 | TCP | Acceso a las carpetas compartidas del servidor (Samba). |
> | RPC_Dinamico | 49152-65535 | TCP | Rango de puertos que Active Directory negocia dinámicamente para comunicarse. |
> | Kerberos_Pass_TCP | 464 | TCP | Gestión de cambios de contraseña de los usuarios del dominio. |
> | Kerberos_Pass_UDP | 464 | UDP | Ídem por UDP. |
> | NTP_Time | 123 | UDP | Sincronización horaria del servidor — Kerberos falla si el reloj difiere más de 5 minutos. |
>
> > [!note] 💡 Recuerda: en AWS no hay columna "Prioridad"
> > Igual que en la Fase 3, los Security Groups de AWS solo permiten tráfico (no deniegan) y no tienen orden de prioridad. Por eso la tabla no lleva números de prioridad como tendría un NSG de Azure.
>
> > [!info] 💡 ¿Por qué tantos puertos de golpe?
> > En las fases anteriores abriste solo lo imprescindible para no exponer el servidor innecesariamente. Active Directory es diferente: es un ecosistema de servicios interdependientes. DNS encuentra el servidor, Kerberos autentica al usuario, LDAP consulta su perfil y RPC coordina todo el proceso. Si falta uno, la cadena se rompe. Esta es la única fase del proyecto donde abrirás tantos puertos a la vez. A partir de la Fase 5 no necesitarás añadir ninguno más.

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_4.4_Donde_Estamos]] | [[Fase_4]] | [[Fase_4.6_Procedimiento]] |
