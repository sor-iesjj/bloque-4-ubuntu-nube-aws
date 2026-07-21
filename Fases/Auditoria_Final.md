## 🛡️ Auditoría Final y Hardening (Cierre de Seguridad)

> **[RA.06]** Diseña e implementa soluciones de seguridad perimetral y auditoría de sistemas.

### 📚 Fundamento Teórico: El Principio de "Zero Trust"

Para terminar el proyecto, debemos aplicar la filosofía **Zero Trust** (Confianza Cero). Hasta ahora, hemos dejado algunos puertos abiertos a todo Internet para facilitar la configuración inicial. Un administrador profesional, una vez terminado el trabajo, debe "cerrar el castillo" y solo permitir el paso a quien esté dentro de la muralla (la VPN).

### 📖 Diccionario de Conceptos Clave

- **Hardening:** El proceso de "endurecer" un servidor eliminando servicios innecesarios y cerrando puertos.
- **Whitelist (Lista Blanca):** Configuración que bloquea todo por defecto y solo permite el paso a IPs específicas.
- **Zero Trust:** Estrategia de seguridad que asume que la red ya está comprometida y exige verificación constante.

---

### 🛠️ Procedimiento Práctico de Hardening

> [!example] Paso 1: Cierre de Puertos en el Security Group de AWS
> Ve a **AWS Console → EC2 → Grupos de seguridad → tu grupo → Editar reglas de entrada** y restringe el **Origen** de las reglas sensibles para aplicar la máxima seguridad:
> 1.  **Puerto 2222 (SSH):** cambia el **Origen** de `Anywhere-IPv4` (`0.0.0.0/0`) a `Personalizado` e introduce el rango de tu VPN: `10.0.0.0/24`.
> 2.  **Puerto 445 (SMB):** cambia el **Origen** de `Anywhere-IPv4` a `Personalizado` e introduce `10.0.0.0/24`.
> 3.  Pulsa **Guardar reglas**.
>
> > [!note] 💡 En AWS se restringe el "Origen", no se crea una regla "Denegar"
> > Recuerda que un Security Group solo tiene reglas que *permiten*. Para bloquear internet en estos puertos no se añade una regla de denegación (no existe): se **estrecha el Origen** de la regla que permite, dejando pasar solo el rango `10.0.0.0/24` de la VPN. Todo lo demás queda bloqueado por defecto.
>
> *Resultado: A partir de ahora, nadie en Internet podrá siquiera intentar atacar estos puertos. Solo los alumnos conectados a la VPN podrán administrar el servidor.*

> [!example] Paso 2: Auditoría Local de Servicios
> Ejecuta este comando en la terminal de tu servidor para verificar que no hay "polizontes" o servicios desconocidos:
> ```bash
> # Listar procesos que escuchan en red con su nombre
> sudo ss -tunlp
> ```
> 
> > [!tip] 💡 ¿Qué hace este comando?
> > - **`-t -u`:** Muestra puertos TCP y UDP.
> > - **`-n`:** Muestra números de puerto en lugar de nombres de servicio.
> > - **`-l`:** Solo muestra puertos que están en escucha (*listening*).
> > - **`-p`:** Muestra el nombre del proceso (ej. `smbd`, `winbind`) que es dueño de ese puerto.

---

### ❓ Preguntas Críticas de Cierre
1. ¿Por qué es más seguro permitir el acceso SSH solo a través de la IP de la VPN que dejarlo abierto a todo Internet?
2. ¿Qué ventaja tiene cambiar el origen del tráfico en el Security Group de AWS en lugar de usar un firewall interno como `ufw`?
3. Si después de cerrar los puertos ya no puedes conectar por SSH, ¿qué es lo primero que deberías comprobar en tu cliente VPN?
4. ¿Qué significa que un servidor esté "bastionado" (*Hardened*)?
5. ¿Qué proceso es el dueño del puerto 445 según el comando `ss -tunlp`?

---

> [!success] 🏁 Proyecto Finalizado
> ¡Enhorabuena! Has construido una infraestructura híbrida profesional, segura y escalable. Has pasado de tener un servidor vacío a un Controlador de Dominio con cuotas de disco, seguridad ACL invisible y clientes Windows integrados bajo un túnel cifrado WireGuard.
