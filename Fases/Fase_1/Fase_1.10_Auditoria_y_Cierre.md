## Fase 1 · Apartado 10 — 🏁 Auditoría y cierre

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (AWS EC2)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Lo último.** No pases a la fase siguiente sin repasarlo.

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

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.9_Preguntas]] | [[Fase_1]] | **Fase 2** |
