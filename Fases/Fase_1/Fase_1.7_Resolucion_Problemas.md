## Fase 1 · Apartado 7 — 🚩 Resolución de problemas

> **[Módulo: SOR — Sistemas Operativos en Red]** · **Infraestructura Cloud (AWS EC2)**
> 🧭 Índice de la fase: [[Fase_1]]
>
> **📍 Cuándo se lee:** **Cuando algo no salga.** Búscate por el síntoma.

---

> [!bug] Tabla de Troubleshooting (¿Algo no funciona?)
> | Problema | Causa Probable | Solución Sugerida |
> | :--- | :--- | :--- |
> | "Permission denied (publickey)" al conectar por SSH | El fichero .pem no es correcto o no tiene permisos adecuados | Comprueba que usas `-i ruta/boochan-key.pem` y que el fichero es el que descargaste en el Paso 1. En Mac/Linux ejecuta `chmod 400 boochan-key.pem` |
> | La instancia no arranca o está en estado "Pendiente" | AWS está inicializando la instancia | Espera 2-3 minutos y actualiza la página. |
> | El servidor **no responde al ping** | El protocolo ICMP está bloqueado por defecto en el Security Group | Es normal por seguridad. No abras el ping; usa `ssh` o `curl` para verificar conectividad. |
> | "Host key verification failed" | La IP de la instancia ha cambiado (si no usas Elastic IP) | Si no tienes Elastic IP asignada, la IP cambia en cada reinicio. Asigna una Elastic IP (Paso 4). |

---

| ← Anterior | 🧭 Índice | Siguiente → |
| :--- | :---: | ---: |
| [[Fase_1.6_Procedimiento]] | [[Fase_1]] | [[Fase_1.8_Punto_de_Control]] |
