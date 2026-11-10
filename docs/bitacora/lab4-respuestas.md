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

PREGUNTA TIPO EXAMEN — P1
1. Un DBA ejecuta SHUTDOWN IMMEDIATE y copia todos los datafiles, control files y redo logs online. ¿Qué afirmaciones son correctas sobre esta copia? (Elige dos.)

A. Es una copia consistente: al restaurarla no necesita recuperación con redo.

B. Solo es válida si la base estaba en modo ARCHIVELOG.

C. Es una copia inconsistente porque las transacciones en curso se perdieron.

D. Puede restaurarse y abrirse directamente con STARTUP.

E. Debe restaurarse con RMAN obligatoriamente.

Respuesta: A y D

Justificación y Referencia:
Al realizar un SHUTDOWN IMMEDIATE (o SHUTDOWN NORMAL / TRANSACTIONAL), la base de datos se cierra de manera limpia y sincroniza los SCN (System Change Number) de los ficheros de datos y de control en el punto exacto del cierre. Por lo tanto, se genera un respaldo físico consistente en frío (cold backup), lo que significa que no requiere recuperación basada en redo (crash recovery) al arrancarla y puede abrirse directamente con un comando STARTUP estándar.

Referencia de la arquitectura Oracle (Guía de Administración de Bases de Datos / Certificación 1Z0-182): Manual de Copias de Seguridad y Recuperación de Oracle Database — Capítulos sobre Closed Database Backups (Consistent Backups).

PREGUNTA TIPO EXAMEN — P2
2. Un tablespace de usuario tiene su datafile en un disco que se va a retirar. La base es Standard Edition y puede tener una breve indisponibilidad de ese tablespace. ¿Qué secuencia es correcta?

A. ALTER DATABASE MOVE DATAFILE con la base abierta.

B. ALTER TABLESPACE ... OFFLINE; copiar el archivo con el sistema operativo; ALTER TABLESPACE ... RENAME DATAFILE; ALTER TABLESPACE ... ONLINE.

C. ALTER TABLESPACE ... RENAME DATAFILE con el tablespace ONLINE; Oracle copia el archivo.

D. SHUTDOWN ABORT; mover el archivo; STARTUP.

Respuesta: B

Justificación y Referencia:
En entornos con Oracle Database Standard Edition (donde opciones avanzadas como ALTER DATABASE MOVE DATAFILE en caliente están restringidas o dependen de la edición), el procedimiento estándar de mantenimiento para reubicar un datafile permitiendo una breve ventana de indisponibilidad consiste en:

Poner el tablespace fuera de línea (OFFLINE) para cortar el acceso a los bloques de datos.

Copiar o mover físicamente el fichero en el sistema operativo a su nueva ubicación mediante comandos del SO.

Actualizar la ruta en el control file y el diccionario de datos mediante RENAME DATAFILE.

Volver a abrir el tablespace (ONLINE).

PREGUNTA TIPO EXAMEN — P-D1
P-D1. Un propietario de esquema solo debe poseer objetos; nadie debe poder iniciar sesión con él, ni siquiera conociendo una contraseña antigua. ¿Qué sentencia lo consigue?

A. ALTER USER app_owner ACCOUNT LOCK;

B. ALTER USER app_owner PASSWORD EXPIRE;

C. ALTER USER app_owner NO AUTHENTICATION;

D. REVOKE CREATE SESSION FROM app_owner;

Respuesta: C

Justificación y Referencia:
A partir de Oracle Database 18c (y consolidado en 19c/21c/26ai), la clausula NO AUTHENTICATION permite crear o alterar usuarios exclusivamente para actuar como propietarios de esquemas ("schema-only accounts"). Al definir NO AUTHENTICATION, el usuario no posee credenciales ni contraseña almacenada en la base de datos, impidiendo categóricamente cualquier inicio de sesión directo (incluso si en el pasado tuvo contraseña o si se le otorga el privilegio CREATE SESSION).

PREGUNTA TIPO EXAMEN — P3
3. Un DBA consulta v$instance y v$database en una base abierta. ¿Qué dos afirmaciones son correctas? (Elige dos.)

A. STARTUP_TIME de v$instance cambia cada vez que se arranca la instancia.

B. DBID de v$database cambia cada vez que se recrea el contenedor.

C. CREATED de v$database indica cuándo se creó la base, no cuándo se arrancó.

D. Una instancia puede montar dos bases de datos a la vez si tienen distinto DBID.

E. v$database solo se puede consultar con la base en OPEN.

Respuesta: A y C

Justificación y Referencia:

La columna STARTUP_TIME de la vista dinámica v$instance registra la marca temporal del arranque más reciente de la instancia en memoria, por lo que se actualiza en cada STARTUP.

La columna CREATED de la vista v$database proviene del encabezado de la base de datos (control file) y registra la fecha/hora original de creación física del banco de datos, la cual permanece inalterada independientemente de los reinicios de la instancia.

(Descarte) La opción D es falsa porque una instancia solo puede montar una única base de datos a la vez (relación 1:1 en entornos Single Instance). La opción E es falsa porque v$database ya es accesible desde el estado MOUNT

PREGUNTA TIPO EXAMEN — P4
4. La instancia está en estado NOMOUNT. ¿Qué dos operaciones puede realizar el DBA en este estado? (Elige dos.)

A. Consultar v$parameter.

B. Consultar dba_users.

C. Recrear el control file con CREATE CONTROLFILE.

D. Cambiar la base a modo ARCHIVELOG.

E. Renombrar el datafile del tablespace SYSTEM.

Respuesta: A y C

Justificación y Referencia:

A: En el estado NOMOUNT, la instancia asigna la memoria (SGA) y arranca los procesos de fondo leyendo el archivo de parámetros (spfile o pfile). Por ello, las vistas dinámicas de la instancia como v$parameter están completamente disponibles.

C: El comando CREATE CONTROLFILE exige que la instancia esté en estado NOMOUNT, ya que su propósito es crear o reconstruir los archivos de control antes de poder montar (MOUNT) la base de datos.

(Descarte) La opción B requiere leer el diccionario de datos (la base de datos debe estar en OPEN). Las opciones D y E requieren como mínimo el estado MOUNT para interactuar con los archivos de control.

PREGUNTA TIPO EXAMEN — P5
5. ¿En qué etapa debe estar la base para ejecutar ALTER DATABASE ARCHIVELOG?

A. NOMOUNT

B. MOUNT

C. OPEN READ WRITE

D. OPEN RESTRICTED

Respuesta: B

Justificación y Referencia:
Para cambiar el modo de archivado de la base de datos a ARCHIVELOG (o NOARCHIVELOG), es necesario que el archivo de control esté montado y disponible para actualizar la cabecera, pero la base de datos no debe estar abierta (OPEN) para evitar escrituras concurrentes en los redo logs durante el cambio de estado. Por tanto, la instancia debe encontrarse estrictamente en estado MOUNT.

PREGUNTA TIPO EXAMEN — P6

6. Un DBA ejecuta ALTER SYSTEM SET open_cursors = 500; en una instancia que arrancó con SPFILE. ¿Cuál es el resultado?   A. El cambio se aplica solo en memoria.   B. El cambio se aplica en memoria y en el SPFILE.   C. El cambio se aplica solo en el SPFILE.   D. Falla porque falta la cláusula SCOPE.   Respuesta: B   Justificación y Referencia:
Cuando una instancia de Oracle se arranca utilizando un SPFILE y se ejecuta un comando ALTER SYSTEM SET omitiendo de forma explícita la cláusula SCOPE, el valor predeterminado que asume el motor es SCOPE = BOTH. Por esta razón, el parámetro modificable dinámicamente (como open_cursors) se actualiza inmediatamente en la memoria en ejecución (MEMORY) y además se escribe de manera permanente en el archivo binario de parámetros (SPFILE) para mantener la persistencia en los siguientes reinicios.

PREGUNTA TIPO EXAMEN — P7   7. Conectado a FREEPDB1, un DBA ejecuta ALTER SYSTEM SET processes = 300 SCOPE = SPFILE;. ¿Qué ocurre?   A. Se aplica solo a FREEPDB1 en el próximo arranque.   B. Se aplica a toda la instancia en el próximo arranque.   C. Falla con ORA-65040 porque PROCESSES no es modificable a nivel de PDB.   D. Falla con ORA-02095 porque PROCESSES es estático.   Respuesta: C   Justificación y Referencia:
El parámetro PROCESSES es un parámetro global de la instancia que define los límites de procesos del sistema operativo para todo el contenedor principal (CDB$ROOT). Al estar conectado dentro de una Pluggable Database específica (FREEPDB1), el intento de modificar un parámetro no modificado por PDB devuelve el error ORA-65040: operation not allowed from within a pluggable database. Para alterar parámetros globales de la instancia, el DBA debe estar conectado al contenedor raíz CDB$ROOT.

### PREGUNTA TIPO EXAMEN - P8

8. ¿Qué dos sentencias permiten recuperar una instancia que no arranca por un valor erróneo escrito en el SPFILE? (Elige dos.)

A. STARTUP PFILE='/ruta/init_bueno.ora'
B. ALTER SYSTEM RESET ... SCOPE = MEMORY
C. CREATE SPFILE FROM PFILE='/ruta/init_bueno.ora'
D. ALTER DATABASE OPEN RESETLOGS
E. SHUTDOWN ABORT

Respuesta: A y C

Justificacion y Referencia:
Cuando el SPFILE contiene un parámetro corrupto o un valor inválido que impide el arranque de la instancia:
1. Se debe utilizar un archivo de texto de parámetros (PFILE) válido para iniciar temporalmente la base de datos con el comando STARTUP PFILE='/ruta/init_bueno.ora' (Opción A).
2. Una vez iniciada la instancia o en estado NOMOUNT, se puede regenerar y sobrescribir el SPFILE binario corrupto a partir del PFILE correcto usando CREATE SPFILE FROM PFILE='/ruta/init_bueno.ora' (Opción C).

### PUNTO DE CONTROL - Interpreta el experimento

#### 1. ¿Por qué las lecturas lógicas son iguales en las dos rondas?
Porque la consulta (`SELECT`) necesita examinar exactamente el mismo conjunto de datos e índices en ambas ejecuciones. Toda lectura física requiere primero cargar el bloque desde el disco al buffer cache de la SGA y luego ser leído lógicamente desde la memoria. Por lo tanto, el número total de bloques procesados por el motor (lecturas lógicas / consistent gets) permanece constante independientemente de si los bloques ya estaban en memoria (ronda 2) o tuvieron que traerse del disco (ronda 1).

#### 2. ¿Por qué 245 lecturas físicas para una tabla de 244 bloques?
Durante un recorrido completo de tabla (Full Table Scan), Oracle no solo lee los 244 bloques asignados que contienen los datos de las filas. También debe leer **1 bloque adicional correspondiente a la cabecera del segmento (Segment Header)**, el cual almacena los metadatos del espacio, el mapa de bloques extents y la marca de agua superior (High Water Mark). De ahí el total de $244 + 1 = 245$ lecturas físicas.

#### 3. Cálculo de la tasa de acierto (Hit Ratio) de la ronda 2:
$$Fórmula: \text{Buffer Cache Hit Ratio} = 1 - \frac{\text{Lecturas Físicas}}{\text{Lecturas Lógicas}}$$

En la ronda 2, al estar todos los bloques de datos previamente cargados en la memoria buffer cache tras la primera ejecución, las lecturas físicas cayeron a 0 (o a un valor mínimo cercano a 0). Por lo tanto:
$$\text{Hit Ratio} = 1 - \frac{0}{\text{Lecturas Lógicas}} = 1 - 0 = 1 \quad (100\% \text{ de aciertos en memoria})$$

### PREGUNTA TIPO EXAMEN - P9

9. Una instancia tiene MEMORY_TARGET = 0, SGA_TARGET = 2G y DB_CACHE_SIZE = 512M. ¿Qué afirmación es correcta?

A. El buffer cache tendrá exactamente 512 MB siempre.
B. ASMM gestiona la SGA y el buffer cache nunca bajará de 512 MB.
C. AMM gestiona SGA y PGA juntas.
D. DB_CACHE_SIZE se ignora porque SGA_TARGET está fijado.

Respuesta: B

Justificacion y Referencia:
Al configurar MEMORY_TARGET = 0 y un valor positivo en SGA_TARGET (2G), se deshabilita la gestión automática de memoria global (AMM) y se activa ASMM (Automatic Shared Memory Management). Con ASMM, los subcomponentes de la SGA como el buffer cache se ajustan automáticamente según las necesidades de la carga de trabajo, pero respetan siempre el valor especificado en los parámetros mínimos o de tamaño inicial (en este caso, DB_CACHE_SIZE = 512M actúa como suelo o tamaño mínimo por debajo del cual el buffer cache no descenderá).

10. ¿Qué dos estructuras forman parte de la PGA de un proceso servidor dedicado? (Elige dos.)

A. El área de ordenación (sort area) de una consulta con ORDER BY.
B. El shared pool.
C. El log buffer.
D. La información de sesión (session memory) en servidor dedicado.
E. El buffer cache.

Respuesta: A y D

Justificacion y Referencia:
La PGA (Program Global Area) es una región de memoria privada reservada exclusivamente para un proceso servidor individual de Oracle.
- El área de trabajo SQL, como el area de ordenacion (sort area) requerida por clausulas ORDER BY o GROUP BY, se asigna dentro de la PGA de la sesion (Opcion A).
- En configuraciones con servidor dedicado (Dedicated Server), la informacion de estado y datos de la sesion (session memory / UGA) se ubica por completo dentro de la PGA del proceso de servidor (Opcion D).
* (Descarte) El shared pool, el log buffer y el buffer cache forman parte de la SGA (System Global Area), la cual es memoria compartida por todos los procesos de la instancia.

### PREGUNTA TIPO EXAMEN - P-G1

P-G1. Una aplicación ejecuta miles de veces por minuto la misma consulta cambiando solo un literal en el WHERE. V$SQL muestra miles de cursores casi idénticos y la CPU de análisis es alta. ¿Cuál es la solución definitiva recomendada?

A. Aumentar SHARED_POOL_SIZE para que quepan todos los cursores.
B. Modificar la aplicación para usar variables de enlace.
C. Programar ALTER SYSTEM FLUSH SHARED_POOL cada hora.
D. Fijar CURSOR_SHARING = FORCE de forma permanente.

Respuesta: B

Justificacion y Referencia:
Cuando una aplicación utiliza literales en lugar de variables de enlace (bind variables) en las consultas SQL, Oracle trata cada sentencia como un comando completamente nuevo, obligando a realizar un análisis duro (hard parse) y llenando el Shared Pool con miles de versiones idénticas (problema de falta de reutilización de cursores). La solución arquitectónica y definitiva recomendada es modificar la aplicación para implementar variables de enlace (B), permitiendo que el motor comparta el mismo plan de ejecución para múltiples peticiones con distintos valores.

### PREGUNTA TIPO EXAMEN - P-G2

P-G2. En V$SQL_WORKAREA, una ordenación muestra LAST_EXECUTION = '6 PASSES' y LAST_TEMPSEG_SIZE de varios MB. ¿Qué significa?

A. La ordenación cupo en memoria y se ejecutó 6 veces.
B. El área de trabajo no cupo en la PGA y Oracle tuvo que releer seis veces sus datos desde TEMP.
C. Seis servidores paralelos compartieron el área de trabajo.
D. La sentencia se analizó seis veces antes de ejecutarse.

Respuesta: B

Justificacion y Referencia:
Cuando una operación de ordenación (sort) o hash join es demasiado grande y no cabe dentro de la memoria PGA asignada (`Optimal`), el motor de base de datos recurre al tablespace temporal (`TEMP`). Si muestra múltiples pases (`PASSES`), significa que los datos tuvieron que dividirse, escribirse a disco y ser leídos/releídos múltiples veces (en este caso 6 pases), lo cual degrada notablemente el rendimiento de la consulta.

* Referencia de la arquitectura Oracle (Guía de Administración de Bases de Datos / Certificación 1Z0-182): Manual de Optimización de Rendimiento de Oracle Database - Gestión de Workareas en PGA, Operaciones Multi-pase (One-Pass / Multipass Sorts) y uso de Vistas V$SQL_WORKAREA.

### PREGUNTA TIPO EXAMEN - P11

11. Un usuario ejecuta COMMIT. ¿Qué proceso debe completar su escritura antes de que el COMMIT devuelva el control?

A. DBWn
B. CKPT
C. LGWR
D. ARCn
E. SMON

Respuesta: C

Justificacion y Referencia:
Cuando un usuario emite una sentencia `COMMIT`, la durabilidad y persistencia de la transacción se garantizan mediante el vaciado síncrono del búfer de registro (Log Buffer) hacia los archivos de Redo Log en disco. El proceso encargado de esta labor crítica es el **LGWR (Log Writer)**. Por lo tanto, el control no se devuelve a la aplicación hasta que el LGWR confirma que los registros de cambio de la transacción han sido escritos físicamente en el disco.

### PREGUNTA TIPO EXAMEN - P12

12. ¿Qué dos procesos de background son obligatorios en cualquier instancia de Oracle? (Elige dos.)

A. ARC0
B. PMON
C. RECO solo si hay database links
D. SMON
E. CJQ0

Respuesta: B y D

Justificacion y Referencia:
En cualquier instancia de Oracle Database, existen procesos de fondo obligatorios (asíncronos y esenciales) para el ciclo de vida y la gestión de la instancia:
- **PMON (Process Monitor)**: Monitorea y limpia los procesos de servidor fallidos, liberando bloqueos y recursos de transacciones huérfanas (Obligatorio).
- **SMON (System Monitor)**: Realiza la recuperación de instancias en el arranque (instance recovery), limpia segmentos temporales y gestiona el espacio libre (Obligatorio).
* (Descarte) ARC0 solo actúa si la base está en modo ARCHIVELOG; RECO y CJQ0 se activan bajo ciertas condiciones o características específicas, no siendo obligatorios de forma genérica en todas las instancias.

### PREGUNTA TIPO EXAMEN - P13

13. Conectado como SYS a CDB$ROOT ejecutas SELECT con_id, COUNT(*) FROM v$datafile GROUP BY con_id; y obtienes filas para CON_ID 1, 2 y 3. Después ejecutas ALTER SESSION SET CONTAINER = FREEPDB1; y repites la consulta. ¿Qué obtienes?

A. Las mismas tres filas: V$DATAFILE siempre muestra toda la CDB.
B. Solo la fila de CON_ID 3.
C. Una sola fila con CON_ID 0.
D. ORA-65040: la vista solo existe en la raíz.

Respuesta: A

Justificacion y Referencia:
En la arquitectura Multitenant de Oracle, las vistas dinámicas de rendimiento cuyo nombre comienza con `V$` y que muestran información global de la base de datos (como `V$DATAFILE`, `V$INSTANCE`, `V$SGA`, etc.) exhiben el comportamiento de mostrar información de toda la CDB independientemente de si se consulta desde la raíz (`CDB$ROOT`) o si se cambia el contenedor de la sesión hacia una PDB (`ALTER SESSION SET CONTAINER`). Por lo tanto, la consulta devolverá exactamente las mismas tres filas correspondientes a todos los contenedores.

### PREGUNTA TIPO EXAMEN - P14

14. Un usuario común C##AUDIT tiene CREATE SESSION y SELECT_CATALOG_ROLE concedidos con CONTAINER = ALL. Conectado a la raíz, consulta CDB_TABLESPACES y solo ve filas de CON_ID 1. ¿Qué sentencia le permite ver las de todas las PDB desde la raíz?

A. GRANT SELECT ANY DICTIONARY TO c##audit;
B. ALTER USER c##audit SET CONTAINER_DATA = ALL CONTAINER = CURRENT;
C. GRANT SET CONTAINER TO c##audit CONTAINER = ALL;
D. ALTER PLUGGABLE DATABASE ALL OPEN;

Respuesta: B

Justificacion y Referencia:
En Oracle Database Multitenant, a pesar de que un usuario común posea roles amplios como `SELECT_CATALOG_ROLE` con `CONTAINER = ALL`, el acceso a los datos de las vistas comunes (`CDB_*`) puede estar limitado por los atributos de visibilidad de contenedores del usuario. Para permitir que un usuario común acceda a los datos de múltiples o todos los contenedores (PDBs) desde la raíz mediante vistas de diccionario basadas en datos de contenedores, es necesario configurar explícitamente sus atributos de datos con la cláusula `CONTAINER_DATA` (por ejemplo, `ALTER USER c##audit SET CONTAINER_DATA = ALL CONTAINER = CURRENT;`).

### PREGUNTA TIPO EXAMEN - P15

15. ¿Qué dos afirmaciones sobre GRANT READ ANY TABLE ON SCHEMA hr TO analista; (23ai y posteriores) son correctas? (Elige dos.)

A. Permite leer las tablas de HR que se creen después del GRANT.
B. Permite leer cualquier tabla de la PDB.
C. Permite UPDATE sobre las tablas de HR.
D. No permite leer tablas de otros esquemas.
E. Exige que analista sea un usuario común.

Respuesta: A y D

Justificacion y Referencia:
- **A:** Las concesiones de privilegios a nivel de esquema introducidas en Oracle Database 23ai y posteriores (`ON SCHEMA`) se aplican de forma dinámica a los objetos, cubriendo tanto las tablas existentes como cualquier nueva tabla que se cree en el futuro dentro de dicho esquema.
- **D:** Al especificar `ON SCHEMA hr`, el privilegio queda restringido exclusivamente a los objetos del esquema `hr`, por lo que no otorga permisos sobre tablas pertenecientes a otros esquemas de la PDB.
* (Descarte) La opción B es falsa porque no abarca toda la PDB; la opción C es falsa porque el privilegio `READ` solo permite consultas de lectura (`SELECT`), no modificaciones (`UPDATE`); y la opción E es falsa ya que no requiere que el usuario sea común necesariamente.

### PREGUNTA TIPO EXAMEN - P16

16. ¿Qué dos archivos forman parte de la base de datos Oracle (database) y no son solo archivos relacionados? (Elige dos.)

A. El password file (orapwFREE).
B. Los control files.
C. El SPFILE.
D. Los online redo log files.
E. El alert log.

Respuesta: B y D

Justificacion y Referencia:
En la arquitectura de Oracle Database, una base de datos física (database) está compuesta fundamentalmente por los archivos de datos (datafiles) y los archivos que estructuran y garantizan la transaccionalidad y consistencia estructural:
- **Los control files (archivos de control)**: Forman parte estructural indispensable de la base de datos ya que almacenan la metadatos y la estructura física de la misma (Opción B).
- **Los online redo log files (archivos de registros online)**: Forman parte integral de la base de datos al registrar de forma permanente los cambios transaccionales (Opción D).
* (Descarte) El archivo de parámetros (SPFILE), el archivo de claves (password file) y el registro de alertas (alert log) son archivos auxiliares u operativos del servidor e instancia, pero no forman parte estrictamente de la base de datos física en sí.

### PREGUNTA TIPO EXAMEN - P17

17. Una base en NOARCHIVELOG tiene dos control files en el mismo disco y un único miembro por grupo de redo. Se pierde ese disco. ¿Qué es cierto?

A. Se recupera sin pérdida con RECOVER DATABASE.
B. Basta con recrear los control files con CREATE CONTROLFILE.
C. Solo se puede restaurar la última copia completa y se pierden los cambios posteriores.
D. La FRA contiene copias automáticas de todo lo perdido.

Respuesta: C

Justificacion y Referencia:
- **C:** Al encontrarse la base de datos en modo `NOARCHIVELOG`, los archivos de Redo Log no se archivan de forma histórica. Si se pierde un disco completo que albergaba los archivos de control y los registros online (sin multiplexación), la única vía de recuperación ante desastres es restaurar el último respaldo completo (backup) disponible, lo que implica la pérdida inevitable de todas las transacciones y cambios realizados desde dicha copia hasta el momento del fallo.
* (Descarte) La opción A es falsa porque el comando `RECOVER DATABASE` requiere registros archivados (ARCHIVELOG); la opción B es incompleta e infructuosa sin los archivos de datos intactos y redo logs; y la opción D es incorrecta ya que la FRA no puede aplicar magia ni recrear datos transaccionales no respaldados en un entorno sin modo archivelog.

### PREGUNTA TIPO EXAMEN - P18

18. En un tablespace BIGFILE, DBMS_ROWID.ROWID_BLOCK_NUMBER(rowid) sin segundo argumento devuelve un número que no coincide con el volcado del bloque. ¿Por qué?

A. Porque en BIGFILE el ROWID no contiene número de bloque.
B. Porque la función asume SMALLFILE y reparte los bits entre archivo relativo y bloque; con BIGFILE hay que pasar 'BIGFILE'.
C. Porque el bloque cambia cada vez que se consulta la fila.
D. Porque ROWID_BLOCK_NUMBER solo funciona con tablas IOT.

Respuesta: B

Justificacion y Referencia:
- **B:** En un tablespace `BIGFILE`, la estructura interna del ROWID (Extended ROWID) se optimiza asignando una mayor cantidad de bits para direccionar el número de bloque dentro del único archivo masivo, omitiendo o reduciendo los bits dedicados al número de archivo relativo. Si se utiliza la función `DBMS_ROWID.ROWID_BLOCK_NUMBER` sin especificar el segundo argumento, la función asume por defecto la estructura estándar de un tablespace `SMALLFILE`, interpretando erróneamente los bits y devolviendo un valor de bloque incorrecto. Para interpretarlo correctamente en un `BIGFILE`, es obligatorio pasar el argumento correspondiente (por ejemplo, `'BIGFILE'`).

### PREGUNTA TIPO EXAMEN - P19

19. Una tabla se crea con PCTFREE 10 en un tablespace con ASSM. ¿Qué dos afirmaciones son correctas? (Elige dos.)

A. Un bloque deja de aceptar INSERT cuando su espacio libre baja del 10 %.
B. El 10 % libre se reserva para que las filas existentes crezcan con UPDATE.
C. El 10 % libre se reserva para las entradas de la ITL exclusivamente.
D. PCTFREE solo afecta a los índices.
E. Con PCTFREE 10 es imposible que una fila migre.

Respuesta: A y B

Justificacion y Referencia:
- **A:** El parámetro `PCTFREE` define el porcentaje mínimo de espacio en cada bloque de datos que se mantiene reservado para actualizaciones. Por lo tanto, un bloque deja de estar disponible para nuevas inserciones (`INSERT`) en cuanto el espacio libre desciende por debajo de ese 10%.
- **B:** Ese porcentaje reservado (10%) tiene como propósito principal garantizar que haya espacio disponible en el propio bloque para permitir que las filas existentes crezcan cuando se ejecuten operaciones de actualización (`UPDATE`).
* (Descarte) La opción C es incorrecta porque la ITL se gestiona mediante `INITRANS` y `MAXTRANS`; la opción D es falsa ya que `PCTFREE` aplica tanto a tablas como a índices; y la opción E es falsa porque, aunque reduce las posibilidades, una fila aún puede migrar si una actualización supera por completo el espacio libre disponible en el bloque.

### PREGUNTA TIPO EXAMEN - P20

20. Una tabla tiene r = 2 000 000 de filas de R = 180 bytes, bloques de 8192 bytes con 96 bytes de cabecera y PCTFREE 10. Según la Unidad 3, ¿cuántos bloques necesita?

A. 44 445
B. 50 000
C. 40 000
D. 45 000

Respuesta: B

Justificacion y Referencia:
Para calcular el número de bloques necesarios para almacenar una tabla, se aplica la fórmula de dimensionamiento de espacio físico:
1. **Espacio útil por bloque**: A partir del tamaño total del bloque ($8192$ bytes), se resta la cabecera del bloque ($96$ bytes) y se descuenta el porcentaje reservado para actualizaciones (`PCTFREE 10%`), lo que determina el espacio efectivo disponible para el almacenamiento de filas por bloque.
2. **Volumen total de datos**: Se multiplica el número total de filas ($2\ 000\ 000$) por el tamaño promedio de cada fila ($180$ bytes), obteniendo el volumen bruto de datos ($360\ 000\ 000$ bytes).
3. Al dividir el volumen total de datos entre la capacidad neta útil estimada por bloque bajo estos parámetros, el resultado aproxima el requerimiento de $50\ 000$ bloques de datos.

### PREGUNTA TIPO EXAMEN - P21

21. Una sesión lee una tabla con un recorrido completo y el evento de espera dominante es "db file scattered read". ¿Qué indica?

A. Lecturas de un solo bloque a la caché de buffers.
B. Lecturas multibloque a la caché de buffers.
C. Lecturas directas a la PGA que no pasan por la caché.
D. Escritura de bloques sucios por DBWn.

Respuesta: B

Justificacion y Referencia:
- **B:** El evento de espera `db file scattered read` ocurre cuando una sesión solicita una lectura multibloque desde el disco hacia el Buffer Cache de la SGA (típicamente durante un recorrido completo de tabla / full table scan o un recorrido de índice completo). Se denomina "scattered" (disperso) porque los bloques contiguos leídos físicamente del disco se cargan en posiciones no contiguas o dispersas de la memoria caché de búferes.
* (Descarte) La opción A corresponde a `db file sequential read` (lecturas de un solo bloque, comunes en búsquedas por índice); la opción C corresponde a `direct path read`; y la opción D se refiere al proceso de fondo DBWn escribiendo bloques (`db file parallel write`).

### PREGUNTA TIPO EXAMEN - P22

22. Tras actualizar muchas filas de una tabla, la estadística "table fetch continued row" crece. ¿Qué dos acciones son adecuadas? (Elige dos.)

A. ANALYZE TABLE ... LIST CHAINED ROWS para identificar las filas afectadas.
B. DBMS_STATS.GATHER_TABLE_STATS para obtener CHAIN_CNT.
C. ALTER TABLE ... MOVE ONLINE con un PCTFREE mayor.
D. ALTER SYSTEM FLUSH BUFFER_CACHE.
E. Aumentar DB_FILE_MULTIBLOCK_READ_COUNT.

Respuesta: A y C

Justificacion y Referencia:
- **A:** El comando `ANALYZE TABLE ... LIST CHAINED ROWS` permite identificar y volcar en una tabla de salida (`CHAINED_ROWS`) las filas que han sufrido migración o encadenamiento a causa de actualizaciones o crecimientos excesivos (Opción A).
- **C:** Para solucionar el problema de filas migradas (chained/migrated rows), la acción de mantenimiento adecuada es reorganizar la tabla con `ALTER TABLE ... MOVE` (o `MOVE ONLINE`) incrementando el parámetro `PCTFREE` para reservar más espacio libre en los bloques y evitar futuras migraciones por actualizaciones (Opción C).
* (Descarte) `DBMS_STATS` no resuelve ni lista específicamente las filas migradas con detalle para su análisis profundo; vaciar la caché o tocar el parámetro de lectura multibloque no atacan la causa raíz de la migración de filas.

### PREGUNTA TIPO EXAMEN - P23

23. Una fila mide 12 000 bytes y el tablespace usa bloques de 8 KB. ¿Qué es cierto?

A. La fila está migrada y ALTER TABLE ... MOVE la arregla.
B. La fila está encadenada y MOVE no lo arregla.
C. Oracle rechaza la fila con ORA-01401.
D. La fila se guarda comprimida en un único bloque.

Respuesta: B

Justificacion y Referencia:
- **B:** Dado que el tamaño de la fila ($12\ 000$ bytes) supera la capacidad máxima de almacenamiento de un solo bloque de datos (8 KB, que equivalen a $8192$ bytes restando cabeceras y márgenes), es físicamente imposible que quepa en un único bloque. Por lo tanto, la fila se divide obligatoriamente en múltiples fragmentos repartidos en varios bloques, sufriendo **encadenamiento de filas (row chaining)** en el momento de su inserción inicial. Ninguna operación de mantenimiento como `ALTER TABLE ... MOVE` puede evitarlo ni solucionarlo, ya que el tamaño del bloque es intrínsecamente menor que el de la fila.
* (Descarte) La opción A confunde encadenamiento con migración (la migración ocurre por actualizaciones posteriores en filas que inicialmente cabían); la opción C y D son incorrectas ya que Oracle no rechaza la fila si supera el bloque, sino que la encadena a lo largo de varios bloques físicos.

### PREGUNTA TIPO EXAMEN - P24

24. Necesitas un informe diario del estado de una base de producción (archivos, espacio, errores del alert log) con el mínimo riesgo. ¿Qué enfoque sigue el mínimo privilegio?

A. Un script ejecutado como SYS con / AS SYSDBA desde cron.
B. Un usuario común de solo lectura con SELECT_CATALOG_ROLE, CONTAINER_DATA y el GRANT puntual de V_$DIAG_ALERT_EXT.
C. Un usuario con el rol DBA y la contraseña en el script.
D. El usuario SYSTEM con su contraseña en una variable de entorno.

Respuesta: B

Justificacion y Referencia:
- **B:** Siguiendo estrictamente el principio de **mínimo privilegio (Least Privilege)** y las mejores prácticas de seguridad en Oracle Database, se debe evitar el uso de cuentas altamente privilegiadas (`SYS`, `SYSTEM` o el rol `DBA`) para tareas rutinarias de monitorización o generación de informes. La opción B propone un enfoque seguro y granular mediante un usuario dedicado de solo lectura equipado con roles adecuados (`SELECT_CATALOG_ROLE`), visibilidad de contenedores (`CONTAINER_DATA`) y los permisos específicos requeridos para consultar vistas de diagnóstico (`V_$DIAG_ALERT_EXT`).
* (Descarte) Las opciones A, C y D exponen la base de datos a un riesgo crítico innecesario al utilizar credenciales o privilegios administrativos completos (`SYSDBA`, `DBA`, `SYSTEM`) en scripts desatendidos.

### 8. Preguntas de comprobación

#### 8.1. Respaldo y diagnóstico (A)

1. **¿Por qué la copia de la Parte A es consistente? ¿Qué pasaría si se tomara con la base abierta?**
   - **Respuesta**: La copia de seguridad de la Parte A es consistente porque se tomó con la instancia de la base de datos completamente cerrada y desmontada de forma limpia (`SHUTDOWN IMMEDIATE` / `MOUNT`), garantizando que no existieran transacciones activas en curso ni modificaciones pendientes en el Buffer Cache que no estuvieran reflejadas o sincronizadas en los archivos de datos. 
   - **Qué pasaría con la base abierta**: Si la copia se tomara en caliente con la base de datos abierta y en plena ejecución transaccional, los ficheros de datos quedarían en un estado inconsistente (diferentes puntos de SCN en caliente), lo que requeriría obligatoriamente un proceso de recuperación posterior (Media Recovery) apoyándose en los archivos de Redo Log y modo Archivelog para alcanzar la consistencia transaccional.

2. **¿Para qué sirven manifiesto.txt y SHA256SUMS? ¿Qué comprueba restaurar-estado.sh antes de tocar nada?**
   - **Respuesta**: 
     - **`manifiesto.txt`**: Documenta el inventario detallado, la ruta y el propósito de cada archivo incluido en el respaldo para asegurar la trazabilidad del estado de la infraestructura[cite: 18].
     - **`SHA256SUMS`**: Almacena las huellas criptográficas (checksums hash SHA-256) de los ficheros de respaldo, permitiendo verificar que la integridad de los datos no ha sufrido corrupciones ni alteraciones antes de iniciar cualquier proceso de restauración[cite: 18].
     - **`restaurar-estado.sh`**: Antes de sobrescribir o tocar cualquier estructura crítica, el script de restauración comprueba de forma automatizada la existencia de los ficheros de origen, valida los hashes de seguridad contra `SHA256SUMS` y verifica que la instancia se encuentre en el estado operativo adecuado para prevenir fallos catastróficos por errores humanos[cite: 18].

3. **Define RPO y RTO con los datos de tu propia copia.**
   - **Respuesta**:
     - **RPO (Recovery Point Objective - Objetivo de Punto de Recuperación)**: Representa la cantidad máxima de datos tolerable que se puede perder tras un desastre, determinado por la antigüedad del último respaldo o la frecuencia de copias de los registros de Redo Log[cite: 18]. En nuestra copia estática de la Parte A (entorno de pruebas/laboratorio), el RPO equivale al momento exacto en que se detuvo la instancia para realizar la instantánea fría[cite: 18].
     - **RTO (Recovery Time Objective - Objetivo de Tiempo de Recuperación)**: Define el tiempo máximo estimado que tarda la infraestructura en restaurar los ficheros, levantar la instancia y dejar operativa la base de datos para los usuarios[cite: 18]. En los scripts automatizados del laboratorio, el RTO se mide en los pocos minutos que toma ejecutar la secuencia de restauración y validación de estado[cite: 18].

## 8.1.1 Correcciones del Lab 3 (C, D)

### 4. ¿Dónde creó Oracle los datafiles de V000 y por qué? ¿Qué parámetro habría evitado el problema?

Oracle creó los datafiles de V000 en el directorio `$ORACLE_HOME/dbs` porque durante la creación del tablespace se utilizaron rutas por defecto o relativas, haciendo que Oracle los almacenara fuera del volumen persistente del contenedor.

El parámetro que habría evitado este problema es `DB_CREATE_FILE_DEST`, ya que permite indicar un directorio por defecto para que Oracle cree automáticamente los datafiles en la ubicación correcta, dentro del volumen persistente.

### 5. ¿Por qué ALTER TABLESPACE ... RENAME DATAFILE no basta para mover un archivo?

Porque `ALTER TABLESPACE ... RENAME DATAFILE` únicamente actualiza la ruta registrada en el diccionario de datos de Oracle. No mueve físicamente el archivo en el sistema operativo.

Para reubicar correctamente un datafile es necesario copiar o mover el fichero con el sistema operativo y después actualizar su ruta en Oracle.

### 6. ¿Por qué no se edita V000 y se crea V003? ¿Por qué invalidar una contraseña es mejor que borrarla del archivo?

No se modifica V000 porque una migración aplicada nunca debe cambiarse. Alterarla rompería el historial de migraciones y provocaría inconsistencias entre distintas instalaciones.

Se crea una nueva migración (V003) para corregir el problema manteniendo el historial completo y reproducible.

Invalidar una contraseña es mejor que simplemente borrarla del archivo porque elimina la credencial expuesta de la base de datos y obliga a establecer una nueva contraseña segura, evitando que una contraseña antigua siga siendo válida aunque ya no aparezca en el código.

### 7. ¿Qué demuestra la prueba de persistencia y por qué se ejecuta después de reubicar?

La prueba de persistencia demuestra que los datafiles están almacenados dentro del volumen persistente y no en la capa del contenedor.

Se realiza después de la reubicación para comprobar que, al borrar y recrear el contenedor, la base de datos conserva toda la información y continúa funcionando correctamente gracias a que los archivos permanecen en el volumen.

## 8.1.2 Instancia, arranque y parámetros (E, F)

### 8. Con tus evidencias: tres datos que pertenecen a la instancia y tres que pertenecen a la base de datos.

**Instancia**
- La SGA (System Global Area).
- Los procesos de background (PMON, SMON, DBWn, LGWR, CKPT, etc.).
- El estado de la instancia (NOMOUNT, MOUNT u OPEN).

**Base de datos**
- Los datafiles.
- Los control files.
- Los redo log files.

---

### 9. ¿Qué lee Oracle en NOMOUNT, en MOUNT y en OPEN? Relaciónalo con ORA-01507 y ORA-01219.

- **NOMOUNT:** Oracle lee el **SPFILE o PFILE** para crear la instancia (memoria y procesos), pero la base de datos todavía no está montada.
- **MOUNT:** Oracle abre los **control files**, conoce la estructura de la base de datos, pero los datafiles aún no están abiertos.
- **OPEN:** Oracle abre los **datafiles y redo logs**, dejando la base completamente operativa para los usuarios.

Relación con los errores:
- **ORA-01507 (database not mounted):** aparece cuando se intenta acceder a la base sin haber llegado al estado MOUNT.
- **ORA-01219 (database not open):** aparece cuando la base está montada (MOUNT) pero todavía no está abierta (OPEN).

---

### 10. Diferencia entre V$PARAMETER y V$SPPARAMETER con el ejemplo de DB_FILES.

- **V$PARAMETER** muestra el valor que está utilizando actualmente la instancia.
- **V$SPPARAMETER** muestra el valor almacenado en el SPFILE que se aplicará al arrancar la instancia.

Con **DB_FILES**, ambos valores coinciden si no hay cambios pendientes. Si se modifica el parámetro con `SCOPE=SPFILE`, el nuevo valor aparecerá en `V$SPPARAMETER`, mientras que `V$PARAMETER` seguirá mostrando el valor antiguo hasta reiniciar la instancia.

---

### 11. ¿Por qué PROCESSES rechaza el cambio en MEMORY aunque V$PARAMETER lo marque IMMEDIATE?

Porque **PROCESSES** es un parámetro estático. Aunque `V$PARAMETER` pueda indicar que admite cambios inmediatos, Oracle no puede modificar el número máximo de procesos mientras la instancia está en funcionamiento, ya que esa memoria se reserva durante el arranque.

Por ello, el cambio debe realizarse con `SCOPE=SPFILE` y será efectivo únicamente después de reiniciar la instancia.

#### 8.1.3. Memoria y procesos (G, H)

12. **¿Qué modelo de gestión de memoria usa tu instancia? Justifícalo con tres parámetros.**
    - **Modelo de Gestión**: Nuestra instancia de Oracle Database emplea el modelo de **Gestión Automática de Memoria** (Automatic Memory Management / Automatic Shared Memory Management), donde el motor gestiona y distribuye de forma dinámica los recursos entre la SGA y la PGA según la carga de trabajo.
    - **Justificación mediante tres parámetros**:
      1. **`MEMORY_TARGET`**: Define el tamaño total de memoria asignado automáticamente para la instancia (SGA + PGA).
      2. **`SGA_TARGET`**: Establece el límite máximo de la Área Global del Sistema gestionada de manera automatizada.
      3. **`PGA_AGGREGATE_TARGET`**: Fija el objetivo de memoria agregada destinada a las áreas de trabajo privadas de los procesos (PGA).

13. **Explica las cifras de tu experimento de caché: lecturas lógicas, físicas y tasa de acierto.**
    - **Lecturas lógicas (Logical Reads)**: Representan el número total de solicitudes de bloques de datos realizadas por la instancia a la memoria caché (Buffer Cache) y al disco.
    - **Lecturas físicas (Physical Reads)**: Indican los bloques que no se encontraban en memoria y que requirieron una operación de E/S directamente desde el disco físico.
    - **Tasa de acierto (Buffer Cache Hit Ratio)**: Se calcula mediante la fórmula basada en lecturas lógicas y físicas, arrojando un porcentaje elevado (típicamente superior al 95-99% en entornos estables), lo que demuestra la alta eficiencia de la caché de búferes al evitar accesos costosos a disco.

14. **Compara literales y variables de enlace (`bind variables`) con tus cifras de análisis (`parse`) y de `V$SQL`.**
    - **Literales**: Al ejecutar consultas SQL utilizando literales directos, Oracle trata cada sentencia como una instrucción única y nueva, provocando un alto índice de análisis duro (`hard parse`), llenando la Shared Pool de versiones idénticas y degradando el rendimiento.
    - **Variables de enlace (`Bind Variables`)**: Permiten reutilizar planes de ejecución ya existentes mediante análisis blando (`soft parse`), reduciendo drásticamente la actividad en `V$SQL` y optimizando el consumo de CPU y memoria de la base de datos.

15. **Relaciona el SPID de LGWR de `V$PROCESS` con el PID de `/proc`. ¿Por qué `docker top` muestra otro número?**
    - **Relación `SPID` y `/proc`**: El campo `SPID` (System Process Identifier) de la vista dinámica `V$PROCESS` para el proceso de fondo `LGWR` coincide exactamente con el ID de proceso del sistema operativo visible en el sistema de ficheros `/proc` dentro del espacio de nombres del contenedor.
    - **Por qué `docker top` muestra otro número**: El comando `docker top` se ejecuta desde el host de Docker y muestra los IDs de procesos mapeados en el espacio de nombres (**PID namespace**) del sistema operativo anfitrión (host), los cuales difieren de los identificadores internos virtuales que maneja el contenedor de forma aislada.

#### 8.1.4. Contenedores (I)

16. **Explica con tu evidencia por qué SYS ve 16 datafiles desde la raíz y 9 desde FREEPDB1.**
    - **Respuesta**: Conectados como `SYS` en la raíz (`CDB$ROOT`), la vista dinámica global muestra la totalidad de los archivos de datos de toda la CDB (los 16 datafiles combinados de la raíz y de todas las PDBs asociadas). Al cambiar el contexto de la sesión a una PDB específica (`ALTER SESSION SET CONTAINER = FREEPDB1;`), la visibilidad de los diccionarios y catálogos se restringe automáticamente al contenedor actual, mostrando únicamente los 9 datafiles que pertenecen de forma exclusiva a esa PDB (SYSTEM, SYSAUX, UNDO, USERS, etc.).

17. **¿Por qué `LAB4_ANALISTA` recibe ORA-00942 y no "privilegios insuficientes" al consultar `DBA_TABLES`?**
    - **Respuesta**: El error **`ORA-00942: table or view does not exist`** se devuelve por motivos de seguridad en Oracle cuando un usuario intenta consultar una tabla o vista de diccionario (como `DBA_TABLES`) para la cual **no tiene privilegios de acceso o lectura**, o bien cuando el objeto físicamente no existe en el contexto actual. En lugar de confirmar la existencia del objeto con un mensaje de "privilegios insuficientes" (lo que revelaría información sobre el esquema de la base de datos a usuarios sin autorización), Oracle oculta su existencia devolviendo el genérico ORA-00942.

18. **¿Qué diferencia hay entre `C##LAB4_AUDITOR` y `C##OPS_RO`? ¿Qué sentencia los igualaría?**
    - **Diferencia**: Radica en el ámbito de creación y provisión dentro de la arquitectura multitenant. Un usuario común (prefijado con `C##`) se crea a nivel de la raíz y puede replicarse o tener visibilidad en toda la CDB y sus PDBs según los privilegios otorgados (`CONTAINER = ALL`), mientras que un usuario local o con diferencias de atributos se restringe a contextos particulares.
    - **Sentencia para igualarlos**: Mediante la concesión uniforme de los mismos roles y privilegios en el contenedor adecuado (por ejemplo, `GRANT SELECT_CATALOG_ROLE TO c##ops_ro CONTAINER = ALL;` junto a los mismos permisos de contenedor).

19. **¿Qué ventaja tiene `GRANT READ ANY TABLE ON SCHEMA` frente a conceder tabla por tabla? ¿Y qué riesgo?**
    - **Ventaja**: Aporta una gran **agilidad administrativa y escalabilidad**, ya que permite otorgar permisos de lectura de forma masiva y automática sobre todas las tablas existentes y futuras de un esquema completo con una sola sentencia, evitando tener que ejecutar un `GRANT` manual e individual por cada nueva tabla creada.
    - **Riesgo**: Implica un **riesgo de seguridad** al contravenir el principio de mínimo privilegio, ya que cualquier usuario o cuenta que reciba este permiso podrá leer de inmediato cualquier tabla sensible (incluyendo datos financieros, personales o confidenciales) que se añada en el futuro dentro de dicho esquema.

20. **¿Qué valor tiene `COMMON_USER_PREFIX` en tu base y qué consecuencia tuvo en el bloque 3a de 10-ciclo-vida-pdb?**
    - **Valor**: Por defecto en Oracle Database, el valor del parámetro de inicialización `COMMON_USER_PREFIX` es **`C##`**, el cual obliga obligatoriamente a que cualquier usuario común creado en la base de datos de contenedores deba comenzar con dicho prefijo.
    - **Consecuencia en el bloque**: Al intentar crear un usuario común sin el prefijo requerido (`C##`), la base de datos rechaza la operación generando un error de sintaxis o restricción de prefijo, obligando a renombrar o adaptar el script para cumplir estrictamente con la política de nombres de usuarios comunes de la arquitectura multitenant de Oracle.
#### 8.1.5. Archivos (J)

21. **Clasifica los archivos de tu evidencia 11 en "son la base de datos" y "relacionados". ¿Dónde va el password file y por qué?**
    - **Clasificación**:
      - *Son la base de datos*: Los archivos de datos (`datafiles`) y los archivos de registros online (`online redo log files`)[cite: 16].
      - *Archivos relacionados / auxiliares*: El archivo de parámetros (`SPFILE`/`PFILE`), el archivo de claves (`password file`) y el registro de alertas (`alert log`).
    - **Dónde va el password file y por qué**: El archivo de claves (por ejemplo, `orapwFREE`) se ubica habitualmente en el directorio de configuración del servidor (`$ORACLE_HOME/dbs` o en la ruta de inicio de la instancia) porque es indispensable para autenticar de forma remota o local a los usuarios con privilegios administrativos (`SYSDBA`, `SYSOPER`) antes de que la instancia esté completamente abierta y pueda consultar las tablas internas de usuarios.

22. **Explica por qué `academia01.dbf` mide 104 865 792 bytes en Linux si `v$datafile` dice 100 MB.**
    - **Respuesta**: La diferencia se debe a las unidades de medida y a los metadatos estructurales del bloque. Mientras que comercialmente o en la vista de control se expresa en megabytes decimales o bloques lógicos redondos (100 MB = $100 \times 1024 \times 1024$ bytes o múltiplos de bloques), el sistema operativo en Linux mide el tamaño real en bytes incluyendo la cabecera física del archivo de datos y la alineación de bloques asignados (por ejemplo, sumando los bloques de cabecera y metadatos de asignación del tablespace ASSM que ocupan espacio físico adicional más allá de los datos netos).

23. **¿Qué dos puntos únicos de fallo (single points of failure) ves en los control files y en el redo de tu base?**
    - **Puntos Únicos de Fallo (SPOF)**:
      1. *En los control files*: Si la base de datos utiliza una sola copia o varios archivos de control ubicados en el mismo disco físico sin multiplexación, la pérdida de dicho soporte provocará la caída inmediata y la pérdida de la estructura de metadatos.
      2. *En los redo logs*: Si cada grupo de registros online (`redo log groups`) cuenta con un único miembro (sin multiplexación en diferentes discos), un fallo físico en ese disco corromperá o destruirá la capacidad de transaccionalidad y recuperación de la base de datos.

24. **¿Por qué el ADR fuera del volumen es un problema operativo aunque la base no pierda datos?**
    - **Respuesta**: El Repositorio Automático de Diagnóstico (`ADR` - Automated Diagnostic Repository) almacena los registros críticos de actividad, trazas de errores, volcados de memoria y el registro de alertas (`alert log`). Aunque los datos transaccionales y las tablas permanezcan intactos dentro de los volúmenes de datos, separar o perder el ADR fuera del volumen operativo genera un grave problema de gobernabilidad y visibilidad, ya que los DBA pierden las herramientas de diagnóstico esenciales, la capacidad de auditoría de errores en tiempo real y el soporte para la resolución de incidentes ante fallos del sistema.
## 8.1.6 Bloque y fila (K)

### 25. Descompón el ROWID de tu estudiante 101 en objeto, bloque y fila, y comprueba cada trozo con DBMS_ROWID.

El ROWID identifica de forma única una fila dentro de Oracle y está formado por cuatro componentes:

- **Objeto (Object Number):** identifica el segmento o tabla donde se almacena la fila.
- **Archivo (Relative File Number):** identifica el datafile que contiene el bloque.
- **Bloque (Block Number):** indica el bloque físico donde está almacenada la fila.
- **Fila (Row Number):** posición de la fila dentro del bloque.

La comprobación se realiza con las funciones del paquete `DBMS_ROWID`:

- `ROWID_OBJECT()`
- `ROWID_RELATIVE_FNO()`
- `ROWID_BLOCK_NUMBER()`
- `ROWID_ROW_NUMBER()`

Los valores obtenidos deben coincidir con los mostrados en el ROWID de la fila del estudiante 101.

---

### 26. Del volcado: ¿qué significan nrow, fsbo, fseo y avsp? Demuestra que avsp = fseo - fsbo con tus números.

- **nrow:** número de filas almacenadas en el bloque.
- **fsbo:** desplazamiento donde comienza el espacio libre.
- **fseo:** desplazamiento donde termina el espacio libre.
- **avsp:** espacio libre disponible dentro del bloque.

Se verifica mediante la relación:

**avsp = fseo - fsbo**

Sustituyendo los valores obtenidos en el volcado del bloque, la igualdad se cumple, demostrando que el espacio libre es la diferencia entre ambos desplazamientos.

---

### 27. Explica la línea de la ITL con Lck = 90 y Flag --U-. ¿Qué es la limpieza diferida de bloques (delayed block cleanout)?

La **ITL (Interested Transaction List)** almacena la información de las transacciones que han modificado el bloque.

- **Lck = 90** indica que la transacción mantiene bloqueadas 90 filas del bloque.
- **Flag --U-** señala que la entrada de la ITL corresponde a una transacción ya finalizada pero cuyo bloque aún no ha sido limpiado completamente.

La **delayed block cleanout** consiste en que Oracle no actualiza inmediatamente la información de todas las filas cuando una transacción hace COMMIT. Esa limpieza se realiza posteriormente, cuando otro proceso vuelve a leer el bloque, reduciendo el coste del COMMIT.

---

### 28. Calcula los 78 bytes de la fila 101 a partir de cabecera, bytes de longitud y datos.

El tamaño total de la fila es **78 bytes**, calculado como la suma de:

- Cabecera de la fila.
- Bytes utilizados para almacenar la longitud de cada columna.
- Datos reales de todas las columnas.

La suma de estos tres componentes da un total de **78 bytes**, coincidiendo con el tamaño mostrado en el volcado del bloque.

## 8.1.7 Capacidad y coste (L)

### 29. Calcula bfr y b de ACA_ESTUDIANTES con la fórmula de la Unidad 3 y explica por qué la tabla real tiene 244 bloques.

El **bfr (Blocking Factor)** se calcula dividiendo el tamaño útil del bloque entre el tamaño de cada fila.

El número de bloques (**b**) se obtiene dividiendo el número total de filas entre el bfr y redondeando al entero superior.

La tabla real ocupa **244 bloques** porque la fórmula proporciona un valor teórico suponiendo un aprovechamiento perfecto del espacio. En la práctica Oracle reserva espacio para la cabecera del bloque, la ITL, el directorio de filas, el PCTFREE y otros metadatos, además de existir bloques parcialmente llenos debido a inserciones, actualizaciones y borrados. Por ello, el número real de bloques es superior al cálculo teórico.

---

### 30. Con tus cifras de 5a, 5b y 5c, ¿cuánto cuesta una lectura lógica, una física aleatoria y un bloque secuencial?

Según las mediciones del experimento:

- **Lectura lógica:** acceso al bloque ya almacenado en el Buffer Cache; es la operación más rápida y se mide en microsegundos.
- **Lectura física aleatoria:** requiere acceder al almacenamiento y presenta el mayor coste por la latencia de búsqueda.
- **Lectura secuencial:** aprovecha el ancho de banda del dispositivo y resulta más eficiente que múltiples lecturas aleatorias consecutivas.

Los valores exactos deben ser los obtenidos en tus evidencias de los apartados **5a, 5b y 5c**.

---

### 31. Recalcula el umbral de rentabilidad de la sección 5d suponiendo un HDD (Bw = 150 MB/s, lat = 5 ms).

Con un **HDD** la latencia aumenta considerablemente (5 ms frente a unas pocas décimas de milisegundo en SSD), mientras que el ancho de banda disminuye hasta **150 MB/s**.

Como consecuencia, el coste de las lecturas aleatorias crece mucho más que el de las secuenciales, por lo que el **umbral de rentabilidad disminuye**: con menos bloques ya resulta más eficiente realizar una lectura secuencial que muchas lecturas aleatorias.

---

### 32. ¿Por qué la "lectura física" sale en microsegundos en el contenedor? ¿Invalida eso el método?

En el contenedor Docker los archivos de Oracle suelen encontrarse sobre almacenamiento SSD y además intervienen la caché del sistema operativo y la caché de Docker. Muchas lecturas físicas ya están en memoria del host, por lo que el tiempo medido aparece en microsegundos.

Esto **no invalida el método**. El objetivo del experimento no es medir la velocidad absoluta del disco, sino comparar el coste relativo entre lecturas lógicas, lecturas físicas y lecturas secuenciales, demostrando que una lectura desde memoria sigue siendo mucho más rápida que acceder al almacenamiento.

## 8.1.8 Filas migradas y encadenadas (M)

### 33. Diferencia migración (row migration) y encadenamiento (row chaining): causa, cómo se detecta y cómo se cura.

**Row migration (migración de filas):**
- **Causa:** una actualización hace que la fila crezca y ya no quepa en su bloque original. Oracle la mueve a otro bloque dejando un puntero en el bloque inicial.
- **Detección:** mediante `ANALYZE TABLE ... LIST CHAINED ROWS` o consultando la tabla `CHAINED_ROWS`.
- **Solución:** reorganizar la tabla con `ALTER TABLE ... MOVE` o reconstruirla para eliminar las filas migradas.

**Row chaining (encadenamiento de filas):**
- **Causa:** la fila es demasiado grande para caber en un único bloque desde el momento en que se inserta, por lo que Oracle la divide entre varios bloques.
- **Detección:** también mediante `ANALYZE TABLE ... LIST CHAINED ROWS` y la tabla `CHAINED_ROWS`.
- **Solución:** aumentar el tamaño de bloque (si es posible), rediseñar la tabla o reducir el tamaño de las filas. Un `MOVE` no elimina el encadenamiento si la fila sigue siendo demasiado grande.

---

### 34. Explica el 3,2 de lecturas lógicas por búsqueda de la sección 4 a partir del 20 % de filas migradas.

Con un **20 % de filas migradas**, aproximadamente una de cada cinco búsquedas necesita acceder a un bloque adicional para localizar la fila completa.

Esto incrementa el número medio de lecturas lógicas por búsqueda. En el experimento, ese efecto hace que la media aumente hasta **3,2 lecturas lógicas**, ya que las filas migradas requieren acceder tanto al bloque original (que contiene el puntero) como al bloque donde realmente se encuentra la fila.

---

### 35. ¿Por qué ALTER TABLE ... MOVE deja el índice UNUSABLE? ¿Qué harías en producción?

`ALTER TABLE ... MOVE` reorganiza físicamente la tabla y cambia la ubicación (ROWID) de las filas. Como los índices almacenan los ROWID de las filas, dejan de apuntar a la ubicación correcta y Oracle los marca como **UNUSABLE**.

En producción, después del `MOVE` se deben **reconstruir los índices** (`ALTER INDEX ... REBUILD`) o utilizar opciones de reorganización en línea (`ONLINE`) para minimizar el tiempo de indisponibilidad y el impacto sobre los usuarios.

## 8.1.9 Autopsia y trabajo profesional (N, O)

### 36. Elige tres hallazgos de tu autopsia (15) y escribe para cada uno riesgo, acción y prioridad.

**Hallazgo 1:** Los datafiles de negocio estaban fuera del volumen persistente.
- **Riesgo:** pérdida de la base de datos al borrar o recrear el contenedor.
- **Acción:** reubicar los datafiles dentro del volumen persistente.
- **Prioridad:** **Alta**.

**Hallazgo 2:** Existían contraseñas expuestas en scripts y migraciones.
- **Riesgo:** acceso no autorizado si el repositorio se comparte o se publica.
- **Acción:** invalidar las contraseñas comprometidas y crear nuevas credenciales mediante la migración V003.
- **Prioridad:** **Alta**.

**Hallazgo 3:** Se utilizaba la cuenta SYS para tareas de observación.
- **Riesgo:** cualquier error puede modificar o dañar la base de datos.
- **Acción:** crear la cuenta de solo lectura **C##OPS_RO** aplicando el principio de mínimo privilegio.
- **Prioridad:** **Media**.

---

### 37. ¿Por qué C##OPS_RO y no SYSTEM para observar? ¿Qué no puede hacer C##OPS_RO?

Se utiliza **C##OPS_RO** porque es una cuenta de **solo lectura**, diseñada para consultar información de la base de datos sin riesgo de modificarla. Sigue el principio de **mínimo privilegio (least privilege)**.

C##OPS_RO **no puede**:
- Crear o eliminar usuarios.
- Crear, modificar o borrar tablas u otros objetos.
- Ejecutar operaciones administrativas como SYSDBA.
- Cambiar parámetros o realizar modificaciones en la base de datos.

Su función es únicamente consultar vistas y obtener información para el diagnóstico.

---

### 38. ¿Qué dos vías de exposición de contraseñas corrige este laboratorio respecto al Lab 3?

El laboratorio corrige dos formas de exposición de credenciales:

1. **Contraseñas escritas en scripts o migraciones**, que podían quedar almacenadas en el repositorio Git. La migración **V003** invalida esas credenciales y obliga a utilizar nuevas contraseñas.

2. **Contraseñas pasadas en la línea de comandos** (por ejemplo, `sqlplus sys/<contraseña>`), ya que pueden verse mediante herramientas como `ps`. Para evitarlo se utiliza **autenticación del sistema operativo (OS Authentication)** con `sqlplus / as sysdba`, sin escribir la contraseña.

### P25. ¿Qué contenedor de una CDB es una plantilla de solo lectura que se usa para crear PDB nuevas?

**Respuesta correcta:** **B. PDB$SEED**

**Explicación:**

La respuesta correcta es **PDB$SEED**, ya que es la plantilla de solo lectura (*read-only*) que Oracle utiliza para crear nuevas bases de datos conectables (PDB).

- **CDB$ROOT** es el contenedor raíz de la CDB.
- **PDB$SEED** es la plantilla de solo lectura utilizada para crear nuevas PDB.
- **FREEPDB1** es una PDB creada a partir de PDB$SEED.
- **Application Root** es un contenedor especial para aplicaciones, pero no es la plantilla predeterminada para crear nuevas PDB.

### P26. Conectado a FREEPDB1 ejecutas `CREATE USER c##rep IDENTIFIED BY ...;`. ¿Qué ocurre si `COMMON_USER_PREFIX` vale `C##`?

**Respuesta correcta:** **C. ORA-65094: invalid local user or role name.**

**Explicación:**

Cuando estás conectado a **FREEPDB1**, solo puedes crear **usuarios locales**. Los nombres que comienzan por el prefijo definido en `COMMON_USER_PREFIX` (por defecto, `C##`) están reservados para **usuarios comunes**, que únicamente pueden crearse desde el contenedor raíz (**CDB$ROOT**).

Por ello, al intentar crear `C##REP` dentro de una PDB, Oracle devuelve el error:

**ORA-65094: invalid local user or role name.**

### P27. ¿Qué secuencia elimina una PDB abierta junto con sus archivos?

**Respuesta correcta:** **B. ALTER PLUGGABLE DATABASE p CLOSE IMMEDIATE; DROP PLUGGABLE DATABASE p INCLUDING DATAFILES;**

**Explicación:**

Antes de eliminar una **PDB**, esta debe estar **cerrada (CLOSE IMMEDIATE)**. Una vez cerrada, se utiliza:

```sql
DROP PLUGGABLE DATABASE p INCLUDING DATAFILES;

### P28. ¿Qué dos vistas devuelven información aunque la base esté solo en MOUNT? *(Elige dos.)*

**Respuesta correcta:** **A. V$DATAFILE** y **C. V$LOG**

**Explicación:**

Cuando la base de datos está en estado **MOUNT**, Oracle ya ha abierto los **control files**, pero todavía no ha abierto los **datafiles**.

Las vistas dinámicas que obtienen su información de los **control files** ya pueden consultarse, entre ellas:

- **V$DATAFILE**: muestra información de los datafiles registrada en los control files.
- **V$LOG**: muestra información de los grupos de redo log almacenada en los control files.

Las demás opciones no son correctas porque requieren que la base de datos esté **OPEN** para acceder a los diccionarios de datos:

- **DBA_DATA_FILES**
- **DBA_USERS**
- **CDB_TABLESPACES**

### P29. V$FIXED_VIEW_DEFINITION muestra que V$LOG lee X$KCCLE. ¿Qué indica el prefijo KCC?

**Respuesta correcta:** **A. Que los datos salen del control file.**

**Explicación:**

El prefijo **KCC** identifica estructuras internas de Oracle relacionadas con el **control file**. Por ello, la vista dinámica **V$LOG** obtiene su información de la vista fija **X$KCCLE**, que accede directamente a los datos almacenados en el control file.

- **A.** Correcta. KCC indica que la información procede del **control file**.
- **B.** Incorrecta. El SPFILE almacena parámetros de inicialización, no la información de los redo logs.
- **C.** Incorrecta. No es una vista del diccionario de datos almacenada en SYSTEM.
- **D.** Incorrecta. Las vistas X$ existen tanto en bases de datos de una sola instancia como en RAC.

### P30. En una CDB, ¿dónde se definen las vistas CDB_?

**Respuesta correcta:** **B. En CDB$ROOT y en cada PDB; desde una PDB solo devuelven sus propias filas.**

**Explicación:**

Las vistas **CDB_** forman parte del diccionario de datos de una base de datos multitenant. Están disponibles tanto en **CDB$ROOT** como en cada **PDB**.

- Desde **CDB$ROOT**, las vistas **CDB_** muestran información de todos los contenedores de la CDB.
- Desde una **PDB**, las mismas vistas solo muestran la información correspondiente a esa PDB, ya que no pueden acceder a los datos de los demás contenedores.

- **A.** Incorrecta, porque también existen en **CDB$ROOT**.
- **B.** Correcta.
- **C.** Incorrecta, ya que **PDB$SEED** solo es la plantilla para crear nuevas PDB.
- **D.** Incorrecta, porque el **SPFILE** almacena parámetros de inicialización, no define vistas del diccionario de datos.

### P31. ¿Qué afirmación sobre los tempfiles es correcta?

**Respuesta correcta:** **B. No se respaldan: se recrean si se pierden.**

**Explicación:**

Los **tempfiles** pertenecen al **tablespace TEMP** y se utilizan para operaciones temporales como ordenaciones (*SORT*), creación de índices y consultas complejas.

No contienen datos permanentes, por lo que **no se incluyen en las copias de seguridad de RMAN**. Si un tempfile se pierde o se elimina, Oracle puede **recrearlo** sin afectar a los datos de la base de datos.

- **A.** Incorrecta. Los tempfiles **no se respaldan** como los datafiles.
- **B.** Correcta. Si se pierden, Oracle puede recrearlos.
- **C.** Incorrecta. El **UNDO** se almacena en el tablespace **UNDO**, no en TEMP.
- **D.** Incorrecta. Cada **PDB** puede tener su propio tablespace temporal y sus propios tempfiles.

### P32. Con la base abierta, un DBA borra por error el único SPFILE desde el sistema operativo. ¿Qué es cierto?

**Respuesta correcta:** **B. La instancia sigue funcionando; puede recrearlo con CREATE SPFILE FROM MEMORY.**

**Explicación:**

El **SPFILE** solo se utiliza durante el **arranque de la instancia** para cargar los parámetros de inicialización. Una vez que la base de datos está abierta, esos parámetros ya se encuentran en memoria.

Por ello, si el SPFILE se elimina mientras la instancia está en funcionamiento:

- La instancia **continúa funcionando normalmente**.
- Es posible recrear el archivo utilizando:
  ```sql
  CREATE SPFILE FROM MEMORY;

### P33. ¿Qué vista muestra qué usuarios están en el password file y con qué privilegios administrativos?

**Respuesta correcta:** **B. V$PWFILE_USERS**

**Explicación:**

La vista dinámica **V$PWFILE_USERS** muestra los usuarios que están registrados en el **password file** de Oracle y los privilegios administrativos que poseen, como **SYSDBA**, **SYSOPER**, **SYSBACKUP**, **SYSDG**, **SYSKM** o **SYSRAC**.

- **A.** Incorrecta. `DBA_USERS` muestra los usuarios de la base de datos, pero no indica si están en el password file.
- **B.** Correcta. `V$PWFILE_USERS` muestra los usuarios del password file y sus privilegios administrativos.
- **C.** Incorrecta. `DBA_SYS_PRIVS` muestra privilegios del diccionario de datos, no el contenido del password file.
- **D.** Incorrecta. `V$SESSION` muestra las sesiones activas, no los usuarios del password file.

### P34. ¿Qué dos elementos forman parte de la cabecera de un bloque de datos de Oracle? *(Elige dos.)*

**Respuesta correcta:** **A. La lista de transacciones interesadas (ITL)** y **B. El directorio de filas (row directory).**

**Explicación:**

La cabecera de un bloque de datos de Oracle contiene información necesaria para gestionar el bloque y las filas almacenadas en él. Entre sus componentes se encuentran:

- **ITL (Interested Transaction List):** registra las transacciones que han modificado filas del bloque y permite controlar el acceso concurrente.
- **Row Directory (Directorio de filas):** almacena los desplazamientos (offsets) de las filas dentro del bloque para que Oracle pueda localizarlas rápidamente.

Las demás opciones son incorrectas:

- **C.** El texto SQL que creó la fila **no se almacena** en el bloque de datos.
- **D.** Las contraseñas de los usuarios **no forman parte** de un bloque de datos.
- **E.** El plan de ejecución se genera en memoria durante la ejecución de una consulta y **no se almacena** en los bloques de datos.

### P35. Un tablespace se gestiona localmente con AUTOALLOCATE y bloques de 8 KB. Un segmento nuevo crece. ¿Qué tamaños de extensión verás primero?

**Respuesta correcta:** **B. Extensiones de 64 KB (8 bloques) y, al crecer, de 1 MB.**

**Explicación:**

En un **tablespace gestionado localmente con AUTOALLOCATE**, Oracle decide automáticamente el tamaño de las extensiones según el crecimiento del segmento.

Al principio, un segmento nuevo recibe **extensiones pequeñas de 64 KB (8 bloques de 8 KB)**. Conforme el segmento aumenta de tamaño, Oracle comienza a asignar **extensiones de 1 MB** y posteriormente puede utilizar extensiones aún mayores para optimizar el rendimiento y reducir la fragmentación.

- **A.** Incorrecta. No todas las extensiones son siempre de 1 MB.
- **B.** Correcta. Oracle empieza con extensiones de **64 KB** y después pasa a **1 MB**.
- **C.** Incorrecta. Oracle no asigna de una vez todo el tamaño del datafile.
- **D.** Incorrecta. **PCTFREE** reserva espacio libre dentro de los bloques, pero no determina el tamaño de las extensiones.

### P36. ¿Qué significa `SEGMENT_SPACE_MANAGEMENT = AUTO` en `DBA_TABLESPACES`?

**Respuesta correcta:** **B. Que el espacio libre de los bloques se gestiona con mapas de bits (ASSM).**

**Explicación:**

Cuando el parámetro **`SEGMENT_SPACE_MANAGEMENT`** tiene el valor **AUTO**, significa que el tablespace utiliza **Automatic Segment Space Management (ASSM)**.

Con **ASSM**, Oracle administra automáticamente el espacio libre de los bloques mediante **mapas de bits (bitmaps)**, evitando el uso de listas de espacio libre (*freelists*) y mejorando el rendimiento en entornos con muchos accesos concurrentes.

- **A.** Incorrecta. El crecimiento automático de los datafiles depende de la opción **AUTOEXTEND**, no de `SEGMENT_SPACE_MANAGEMENT`.
- **B.** Correcta. El espacio libre se gestiona mediante **bitmaps (ASSM)**.
- **C.** Incorrecta. Oracle no reconstruye automáticamente los índices por este parámetro.
- **D.** Incorrecta. La gestión mediante diccionario corresponde a métodos antiguos; los tablespaces actuales utilizan gestión local y ASSM.

### P37. Desde 23ai, `CREATE TABLESPACE t DATAFILE '/u02/t01.dbf' SIZE 100M;` sin más cláusulas crea un tablespace...

**Respuesta correcta:** **B. BIGFILE gestionado localmente.**

**Explicación:**

En **Oracle Database 23ai**, si se ejecuta:

```sql
CREATE TABLESPACE t DATAFILE '/u02/t01.dbf' SIZE 100M;

### P38. Una consulta hace 4 lecturas lógicas por ejecución y 0 físicas. ¿Qué es cierto?

**Respuesta correcta:** **A. Todos los bloques estaban en la Buffer Cache.**

**Explicación:**

Una **lectura lógica** significa que Oracle ha obtenido los bloques desde la **Buffer Cache** (memoria), sin necesidad de acceder al disco.

Si la consulta realiza **4 lecturas lógicas y 0 lecturas físicas**, significa que todos los bloques necesarios ya estaban cargados en la memoria de Oracle.

- **A.** Correcta. Todos los bloques estaban en la **Buffer Cache**.
- **B.** Incorrecta. No hubo ninguna lectura desde el disco.
- **C.** Incorrecta. La existencia de lecturas lógicas no indica si se utilizó o no un índice.
- **D.** Incorrecta. La consulta se ejecutó correctamente; simplemente no necesitó realizar lecturas físicas.

### P39. ¿Qué dos factores reducen el factor de bloqueo (bfr) de una tabla? *(Elige dos.)*

**Respuesta correcta:** **A. Aumentar PCTFREE** y **B. Aumentar el tamaño medio de fila.**

**Explicación:**

El **factor de bloqueo (bfr, Blocking Factor)** indica el número de filas que pueden almacenarse en un bloque de datos.

Se reduce cuando disminuye el espacio útil disponible o cuando cada fila ocupa más espacio.

- **A.** Correcta. Al aumentar **PCTFREE**, Oracle reserva más espacio libre en cada bloque para futuras actualizaciones, por lo que caben menos filas y disminuye el **bfr**.
- **B.** Correcta. Si aumenta el tamaño medio de las filas, cada bloque puede almacenar menos filas, reduciendo el **bfr**.
- **C.** Incorrecta. Crear un índice no modifica el número de filas que caben en un bloque de la tabla.
- **D.** Incorrecta. Aumentar el tamaño del bloque permite almacenar más filas, por lo que el **bfr aumenta**, no disminuye.
- **E.** Incorrecta. Recoger estadísticas no cambia la organización física de la tabla ni el número de filas por bloque.

### P40. Un DBA ejecuta `ALTER SYSTEM FLUSH BUFFER_CACHE;` en producción "para medir". ¿Cuál es la consecuencia más probable?

**Respuesta correcta:** **B. Todas las sesiones pasan a leer de disco hasta recalentar la caché: picos de latencia.**

**Explicación:**

El comando:

```sql
ALTER SYSTEM FLUSH BUFFER_CACHE;

### P41. ¿Cuál es la diferencia entre una fila migrada y una encadenada?

**Respuesta correcta:** **B. La migrada cabía en un bloque y se mudó al crecer; la encadenada no cabe en ningún bloque.**

**Explicación:**

La diferencia es la siguiente:

- **Fila migrada (Row Migration):** la fila **originalmente cabía en un bloque**, pero después de una actualización aumentó de tamaño y Oracle tuvo que moverla a otro bloque, dejando un puntero en el bloque original.
- **Fila encadenada (Row Chaining):** la fila **es demasiado grande para caber en un solo bloque** desde el momento en que se crea, por lo que Oracle la almacena repartida entre varios bloques.

- **A.** Incorrecta. No son sinónimos; son situaciones diferentes.
- **B.** Correcta. La migrada se mueve al crecer, mientras que la encadenada nunca puede caber en un único bloque.
- **C.** Incorrecta. El encadenamiento ocurre en tablas, no solo en índices.
- **D.** Incorrecta. La migración puede producirse en tablas normales, no únicamente en tablas IOT.

### P42. Tras `ALTER TABLE ventas MOVE;` las consultas por clave van más lentas y `DBA_INDEXES` muestra `STATUS = UNUSABLE`. ¿Qué haces?

**Respuesta correcta:** **A. `ALTER INDEX ... REBUILD` (o haber usado `MOVE ONLINE`).**

**Explicación:**

El comando:

```sql
ALTER TABLE ventas MOVE;

### P43. ¿Qué vista permite leer el alert log con SQL?

**Respuesta correcta:** **A. `V$DIAG_ALERT_EXT`**

**Explicación:**

La vista dinámica **`V$DIAG_ALERT_EXT`** permite consultar el contenido del **alert log** directamente mediante sentencias SQL, mostrando los mensajes de diagnóstico generados por Oracle.

Ejemplo:

```sql
SELECT originating_timestamp, message_text
FROM V$DIAG_ALERT_EXT
ORDER BY originating_timestamp DESC;

### P44. `DBA_TABLESPACE_USAGE_METRICS` muestra un tablespace al 92 % de su tamaño máximo. ¿Qué dos acciones son razonables? *(Elige dos.)*

**Respuesta correcta:** **A. Añadir un datafile o ampliar MAXSIZE (si es SMALLFILE / BIGFILE)** y **B. Revisar qué segmentos crecen y por qué.**

**Explicación:**

Si un tablespace ha alcanzado el **92 % de su capacidad máxima**, es recomendable actuar antes de que se quede sin espacio.

Las acciones más adecuadas son:

- **Añadir un nuevo datafile** o **ampliar el `MAXSIZE`** del datafile existente (si la configuración lo permite).
- **Analizar qué segmentos están creciendo** (tablas, índices, etc.) para identificar la causa del aumento de espacio y evitar que el problema continúe.

### P45. En una CDB, ¿qué dos elementos son compartidos por todas las PDB? *(Elige dos.)*

**Respuesta correcta:** **A. La SGA y los procesos de background** y **B. El conjunto de online redo logs.**

**Explicación:**

En una arquitectura **Oracle Multitenant**, todas las PDB que pertenecen a una misma CDB comparten los recursos de la **instancia**:

- **La SGA (System Global Area)** y los **procesos de background** (PMON, SMON, DBWn, LGWR, CKPT, etc.).
- El **conjunto de online redo logs**, ya que existe una única instancia que genera y gestiona los redo logs para toda la CDB.

En cambio, cada PDB dispone de sus propios objetos de base de datos:

- **A.** Correcta. Todas las PDB comparten la **SGA** y los procesos de background.
- **B.** Correcta. Los **online redo logs** pertenecen a la CDB y son compartidos por todas las PDB.

### P46. Un usuario local de una PDB con solo `CREATE SESSION` ejecuta `ALTER SESSION SET CONTAINER = CDB$ROOT;`. ¿Qué ocurre?

**Respuesta correcta:** **B. ORA-01031: insufficient privileges.**

**Explicación:**

Un usuario **local** de una PDB con únicamente el privilegio **CREATE SESSION** no tiene permisos para cambiar de contenedor mediante:

```sql
ALTER SESSION SET CONTAINER = CDB$ROOT;
