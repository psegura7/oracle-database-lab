# Informe de Diagnóstico - Lab 4 (Parte A)

- **Punto de control:** Los archivos de control (SPFILE y password file) salieron como "FUERA" (enlazados en `$ORACLE_HOME/dbs`) porque la imagen de Docker los mapea mediante enlaces simbólicos hacia el volumen externo para persistir ante recreaciones del contenedor.
- **Informe de auditoría (5 líneas):**
1. *Riesgo Alto:* Credenciales de SYS y esquemas expuestas en historial de comandos y scripts.
2. *Riesgo Medio:* Datafiles creados con rutas relativas propensos a perderse al recrear volúmenes.
3. *Riesgo Bajo:* Puertos abiertos innecesariamente e incoherencias en variables documentales.
4. *Orden de corrección:* Primero credenciales y seguridad, después rutas estructurales de almacenamiento y finalmente limpieza de puertos.

## Parte B — Usuarios comunes y locales (C##OPS_RO)

### Práctica Profesional (Reto) — Mínimo privilegio, de verdad

1. **Riesgo 1:** Conceder el rol `DBA` otorga permisos de administración y modificación total (`ANY`), lo que permite alterar o eliminar esquemas de negocio y objetos críticos del sistema, violando el principio de menor privilegio.
2. **Riesgo 2:** Permite la gestión no supervisada de otros usuarios y parámetros del sistema, pudiendo alterar contraseñas de cuentas administrativas o desactivar auditorías de seguridad.
3. **Privilegio exacto para leer el alert log:** `GRANT READ ON V_$DIAG_ALERT_EXT TO C##OPS_RO CONTAINER=ALL;`
