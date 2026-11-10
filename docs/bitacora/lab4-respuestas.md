# Informe de Diagnóstico - Lab 4 (Parte A)

- **Punto de control:** Los archivos de control (SPFILE y password file) salieron como "FUERA" (enlazados en `$ORACLE_HOME/dbs`) porque la imagen de Docker los mapea mediante enlaces simbólicos hacia el volumen externo para persistir ante recreaciones del contenedor.
- **Informe de auditoría (5 líneas):**
1. *Riesgo Alto:* Credenciales de SYS y esquemas expuestas en historial de comandos y scripts.
2. *Riesgo Medio:* Datafiles creados con rutas relativas propensos a perderse al recrear volúmenes.
3. *Riesgo Bajo:* Puertos abiertos innecesariamente e incoherencias en variables documentales.
4. *Orden de corrección:* Primero credenciales y seguridad, después rutas estructurales de almacenamiento y finalmente limpieza de puertos.
