## Fase 1 · Apartado 4 — 🎯 ¿Dónde estamos?

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (AWS EC2)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Antes de empezar.** De dónde vienes y a dónde llegas.

---

> [!info] El Punto de Partida
> No vienes de una fase anterior — esta es la base. Pero vienes del mundo real: necesitas un servidor que esté **disponible 24/7**, que no dependa de tu ordenador personal, que sea escalable, profesional y seguro.

> [!warning] El Problema
> Instalar un servidor físico en la clase es caro (comprar hardware), requiere mantenimiento constante (electricidad, refrigeración, actualizaciones de seguridad), no es escalable (si necesitas 2 servidores, necesitas 2 máquinas), y es frágil: una inundación, un apagón o un accidente físico lo destruye. La nube resuelve esto.

> [!success] Objetivo de esta Fase
> Crear una **instancia EC2 en Amazon Web Services** que aloje Ubuntu Server 26.04 LTS. Este servidor será tu controlador de dominio, tu almacenamiento de archivos y la base de toda la infraestructura BoochanV3. Lo protegerás con un **Security Group (firewall cloud)** que bloquea internet y abre solo los puertos imprescindibles: 9090 para monitoreo, 22 para administración.

> [!tip] Hoja de Ruta
> 1. Crear una instancia EC2 en AWS con Ubuntu Server 26.04 LTS (2 GB RAM mínimo)
> 2. Configurar el Security Group: abrir puertos 9090 (Cockpit) y 22 (SSH) — nada más
> 3. Asignar una Elastic IP para tener una IP pública fija
> 4. Conectarse al servidor por SSH desde tu PC (primera vez que entras)
> 5. Verificar acceso a internet y DNS (`curl google.com`)
> 6. Medir RAM base con `free -h` (línea base para comparar en fases futuras)
>
> **Resultado Final:** Un servidor en la nube listo, accesible, aislado.
> **Siguiente:** Fase 2 (Purga y FQDN) — limpiaremos el servidor de software innecesario y le daremos una identidad de dominio (BOOCHAN.SPACE).

---

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.3_Obligaciones_Grabacion]] | [[Fase_1]] | [[Fase_1.5_Fundamento_Teorico]] |
