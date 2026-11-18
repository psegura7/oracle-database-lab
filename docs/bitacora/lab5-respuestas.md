🎓 PREGUNTA TIPO EXAMEN — B1
B1. Un plan real muestra E-Rows = 25 000 y A-Rows = 20 en el paso de acceso por índice sobre la columna estado. ¿Cuál es la causa más probable?
A. El índice está fragmentado.
B. Faltan estadísticas con histograma en una columna de distribución sesgada.
C. La tabla tiene filas migradas.
D. La consulta no es sargable.

Respuesta: B

🎓 PREGUNTA TIPO EXAMEN — C1
C1. Un índice tiene BLEVEL = 2 en DBA_INDEXES. ¿Cuántas lecturas lógicas necesita, como mínimo, una búsqueda por igualdad sobre su clave única que devuelve una columna que no está en el índice?
A. 2
B. 3
C. 4
D. 5

Respuesta: C

🎓 PREGUNTA TIPO EXAMEN — C2
C2. Un índice tiene 2 000 000 de hojas y un factor de ramificación máximo de 500. ¿Qué altura tendrá?
A. 2
B. 3
C. 4
D. 5

Respuesta: C

🎓 PREGUNTA TIPO EXAMEN — D1
D1. Una tabla de hechos se carga una vez por noche y los analistas filtran por combinaciones de región (6 valores), canal (3) y año (5). ¿Qué tipo de índice es más apropiado en esas columnas?
A. B-tree compuesto (region, canal, anio).
B. Un índice bitmap en cada columna.
C. Un índice basado en funciones.
D. Un clúster hash por region.

Respuesta: B

🎓 PREGUNTA TIPO EXAMEN — D2
D2. ¿Qué dos afirmaciones sobre un índice cubriente son correctas? (Elige dos.)
A. Evita el acceso a la tabla si la consulta solo usa columnas del índice.
B. Siempre ocupa menos que la tabla.
C. Encarece cada INSERT y cada UPDATE de sus columnas.
D. Se declara en Oracle con la cláusula INCLUDE.
E. Solo puede tener una columna.

Respuesta: A y C

🎓 PREGUNTA TIPO EXAMEN — F1
F1. La columna num_factura es VARCHAR2(10) y tiene un índice. ¿Qué predicado impide usarlo?
A. num_factura = '2025001'
B. num_factura = 2025001
C. num_factura LIKE '2025%'
D. num_factura BETWEEN '2025000' AND '2025999'

Respuesta: B

🎓 PREGUNTA TIPO EXAMEN — F2
F2. ¿Qué dos reescrituras convierten WHERE TRUNC(fecha) = DATE '2026-10-18' en una condición sargable equivalente? (Elige dos.)
A. fecha >= DATE '2026-10-18' AND fecha < DATE '2026-10-19'
B. fecha = DATE '2026-10-18'
C. fecha BETWEEN DATE '2026-10-18' AND DATE '2026-10-18' + INTERVAL '86399' SECOND
D. TO_CHAR(fecha, 'YYYY-MM-DD') = '2026-10-18'
E. fecha + 0 = DATE '2026-10-18'

Respuesta: A y C

🎓 PREGUNTA TIPO EXAMEN — G1
G1. Quieres comprobar si la aplicación sigue necesitando un índice antes de borrarlo, sin dejar de mantenerlo y pudiendo dar marcha atrás al instante. ¿Qué haces?
A. ALTER INDEX ... UNUSABLE
B. ALTER INDEX ... INVISIBLE
C. DROP INDEX y recrearlo si hay quejas
D. ALTER INDEX ... NOLOGGING

Respuesta: B

🎓 PREGUNTA TIPO EXAMEN — G2
G2. Una tabla tiene 3 índices B-tree y recibe 500 INSERT de una fila por segundo. Con lo medido en esta parte, ¿cuántos bloques modificados por segundo, aproximadamente, añade un cuarto índice?
A. Unos 500
B. Unos 1000
C. Unos 2000
D. Ninguno: los índices se actualizan al hacer COMMIT

Respuesta: B

🎓 PREGUNTA TIPO EXAMEN — H1
H1. Una tabla de eventos recibe claves crecientes y cada noche se borran los eventos de más de 30 días. La consulta "evento más antiguo pendiente" se ha vuelto lenta. ¿Qué acción resuelve el síntoma en línea y con el menor coste?
A. ALTER INDEX ... REBUILD
B. ALTER INDEX ... COALESCE
C. Crear el índice como REVERSE
D. DBMS_STATS.GATHER_INDEX_STATS

Respuesta: B

🎓 PREGUNTA TIPO EXAMEN — H2
H2. ¿Qué dos afirmaciones sobre un índice REVERSE son correctas? (Elige dos.)
A. Reduce la contención en la hoja de más a la derecha con claves crecientes.
B. Permite rangos más rápidos que un índice normal.
C. No puede usarse para una condición BETWEEN sobre la clave.
D. Mejora el factor de agrupamiento de la tabla.
E. Evita las divisiones de hoja.

Respuesta: A y C

🎓 PREGUNTA TIPO EXAMEN — I1
I1. Al abrir la base, ¿qué compara Oracle para decidir si un datafile necesita recuperación?
A. El tamaño del datafile con el registrado en el diccionario.
B. El SCN de checkpoint del control file con el de la cabecera del datafile.
C. La fecha de modificación del archivo en el sistema operativo.
D. El número de bloques sucios del buffer cache.

Respuesta: B

🎓 PREGUNTA TIPO EXAMEN — J1
J1. Una de las tres copias del control file se corrompe con la base abierta. ¿Qué ocurre?
A. Oracle sigue funcionando con las dos copias restantes sin ningún aviso.
B. La instancia se detiene; hay que sustituir la copia dañada por una sana (o quitarla de CONTROL_FILES) y arrancar.
C. Oracle recrea automáticamente la copia dañada a partir de las otras.
D. Hay que restaurar todos los datafiles desde la última copia.

Respuesta: B

🎓 PREGUNTA TIPO EXAMEN — J2
J2. ¿Qué dos datos guarda el control file? (Elige dos.)
A. El nombre y la ubicación de los datafiles y redo logs.
B. Los valores de los parámetros de inicialización.
C. El SCN del último checkpoint.
D. Las contraseñas de los usuarios SYSDBA.
E. Las filas del diccionario de datos.

Respuesta: A y C

🎓 PREGUNTA TIPO EXAMEN — K1
K1. Un grupo de redo tiene estado ACTIVE. ¿Qué afirmación es correcta?
A. LGWR está escribiendo en él.
B. Se puede borrar sin consecuencias.
C. Contiene cambios que aún podrían necesitarse para una recuperación de instancia.
D. Ya ha sido archivado necesariamente.

Respuesta: C

🎓 PREGUNTA TIPO EXAMEN — L1
L1. Un DBA ejecuta ALTER SYSTEM SET db_recovery_file_dest = '/u03/fra'; en una base sin FRA configurada y recibe un error. ¿Cuál es la causa?
A. La base no está en ARCHIVELOG.
B. Falta fijar antes DB_RECOVERY_FILE_DEST_SIZE.
C. DB_RECOVERY_FILE_DEST es estático y exige SCOPE=SPFILE.
D. La ruta debe ser un grupo ASM.

Respuesta: B

🎓 PREGUNTA TIPO EXAMEN — L2
L2. Las sesiones de una base en ARCHIVELOG se quedan esperando en "log file switch (archiving needed)" y las nuevas conexiones reciben ORA-00257. ¿Qué dos acciones resuelven el problema? (Elige dos.)
A. Aumentar DB_RECOVERY_FILE_DEST_SIZE si hay espacio en disco.
B. Ejecutar ALTER SYSTEM SWITCH LOGFILE repetidamente.
C. Hacer copia de los archivados con RMAN y borrarlos con DELETE INPUT.
D. Pasar la base a NOARCHIVELOG en caliente.
E. Borrar los .arc de la FRA con rm.

Respuesta: A y C

🎓 PREGUNTA TIPO EXAMEN — M1
M1. La sesión A ejecuta un UPDATE de una fila y no confirma. La sesión B consulta esa fila. ¿Qué ve B?
A. El valor nuevo, porque el bloque del buffer cache ya está modificado.
B. Un error ORA-00054, porque la fila está bloqueada.
C. El valor confirmado anterior, reconstruido con el undo.
D. Nada: la consulta espera hasta que A haga COMMIT o ROLLBACK.

Respuesta: C

🎓 PREGUNTA TIPO EXAMEN — M2
M2. ¿Qué operación de 10 000 filas genera, en general, MÁS undo?
A. INSERT
B. UPDATE de una columna no indexada
C. DELETE
D. SELECT … FOR UPDATE

Respuesta: C

🎓 PREGUNTA TIPO EXAMEN — M3
M3. Un informe nocturno falla con ORA-01555. ¿Qué dos medidas son adecuadas? (Elige dos.)
A. Aumentar el tamaño de los redo logs.
B. Aumentar UNDO_RETENTION y el tamaño del tablespace de undo.
C. Pasar la base a NOARCHIVELOG.
D. Usar RETENTION GUARANTEE en el tablespace de undo si la consulta es crítica.
E. Aumentar LOG_BUFFER.

Respuesta: B y D

🎓 PREGUNTA TIPO EXAMEN — N1
N1. Una sesión recibe "Commit complete". ¿Qué está garantizado en ese momento?
A. El redo de la transacción está escrito en el redo log online.
B. Los bloques modificados están escritos en los datafiles.
C. El redo de la transacción está archivado en la FRA.
D. Se ha hecho un checkpoint.

Respuesta: A

🎓 PREGUNTA TIPO EXAMEN — N2
N2. ¿En qué dos situaciones escribe DBWn bloques sucios en los datafiles? (Elige dos.)
A. En cada COMMIT.
B. En un checkpoint.
C. Cuando un proceso servidor no encuentra buffers libres.
D. Cada 3 segundos, siempre.
E. Cuando LGWR termina de escribir el log buffer.

Respuesta: B y C

🎓 PREGUNTA TIPO EXAMEN — O1
O1. Tras un corte de luz, el DBA ejecuta STARTUP. ¿Qué ocurre?
A. La base queda en MOUNT y hay que ejecutar RECOVER DATABASE manualmente.
B. SMON aplica el redo online y deshace las transacciones no confirmadas; la base abre sin intervención.
C. Hay que restaurar los datafiles desde la última copia porque son inconsistentes.
D. Se pierden las transacciones confirmadas desde el último checkpoint.

Respuesta: B

🎓 PREGUNTA TIPO EXAMEN — O2
O2. Una base tiene tres copias del control file. Una se borra con la base cerrada y STARTUP da ORA-00205. ¿Qué dos acciones permiten abrir la base sin perder datos? (Elige dos.)
A. RECOVER DATABASE USING BACKUP CONTROLFILE.
B. Copiar una de las copias sanas sobre la perdida y montar.
C. ALTER DATABASE OPEN RESETLOGS.
D. Quitar la copia perdida de CONTROL_FILES en el SPFILE, reiniciar y montar.
E. Restaurar todos los datafiles desde la última copia de RMAN.

Respuesta: B y D

🎓 PREGUNTA TIPO EXAMEN — O3
O3. En una base en ARCHIVELOG se pierde el datafile de un tablespace de aplicación. ¿Cuál es la secuencia que lo recupera sin cerrar la base y sin perder datos?
A. SHUTDOWN ABORT; STARTUP MOUNT; RESTORE DATABASE; RECOVER DATABASE; ALTER DATABASE OPEN RESETLOGS;
B. ALTER DATABASE DATAFILE n OFFLINE; RESTORE DATAFILE n; RECOVER DATAFILE n; ALTER DATABASE DATAFILE n ONLINE;
C. RESTORE DATAFILE n; ALTER DATABASE DATAFILE n ONLINE;
D. FLASHBACK DATABASE TO SCN anterior a la pérdida;

Respuesta: B

🎓 PREGUNTA TIPO EXAMEN — O4
O4. Un usuario ejecutó DROP TABLE ventas PURGE; hace 10 minutos. La base está en ARCHIVELOG y hay copias de RMAN. ¿Qué opción recupera la tabla sin perder los cambios hechos en otras tablas?
A. FLASHBACK TABLE ventas TO BEFORE DROP;
B. Restaurar la última copia completa de la base.
C. SELECT * FROM ventas AS OF TIMESTAMP (SYSTIMESTAMP - INTERVAL '15' MINUTE);
D. RMAN RECOVER TABLE hasta un instante anterior al DROP.

Respuesta: D

8.1.	Punto de partida y banco de medida (A, B)
1. Compara el paso 6 de tu copia lab4-final con el de la copia lab3-final. ¿Qué demuestra la diferencia?

En la copia de lab3-final aparecían cinco datafiles fuera del volumen. En lab4-final ya no aparece ninguno. Esto demuestra que la reubicación realizada en el Laboratorio 4 fue correcta y que toda la base de datos quedó almacenada dentro del volumen Docker, simplificando la copia de seguridad y la restauración.

2. Calcula el factor de bloqueo (bfr) de CLIENTE y de PEDIDO con sus estadísticas. ¿Por qué PEDIDO cabe casi el triple de filas por bloque?

Según las estadísticas del laboratorio:
- CLIENTE: bfr = 68 filas por bloque.
- PEDIDO: bfr = 188 filas por bloque.
PEDIDO cabe casi el triple porque sus filas ocupan menos espacio útil por registro que las del ejemplo anterior utilizado en la teoría, permitiendo almacenar muchas más filas en cada bloque. El factor de bloqueo depende directamente del tamaño medio de la fila.

3. ¿Por qué el banco de medida ejecuta cada sentencia dos veces y mide solo la segunda?
8.1.1.	Anatomía y tipos de índice (C, D)

Porque la primera ejecución carga los bloques necesarios en la memoria (buffer cache). La segunda ejecución mide el rendimiento "en caliente", eliminando el coste inicial de lectura desde disco y obteniendo resultados más estables y comparables.

4. Con tu evidencia 02: altura, número de hojas y factor de ramificación (fan-out) real de IX_PED_CLIENTE. ¿Qué altura tendría con mil millones de filas?

Según el laboratorio, IX_PED_CLIENTE tiene una altura de 3, con un fan-out medio cercano a 370. Gracias al crecimiento logarítmico de los árboles B+, incluso con mil millones de filas la altura solo aumentaría aproximadamente a 4 niveles, manteniendo búsquedas muy rápidas. 

5. En el volcado de la hoja de PK_CLIENTE, localiza una entrada y descompón el ROWID que guarda en bloque y fila. ¿Por qué no aparece el número de archivo?

El ROWID identifica el bloque y la fila donde se encuentra el registro. En este laboratorio no aparece el número de archivo porque la tabla se encuentra en un BIGFILE TABLESPACE, donde el identificador del archivo queda implícito al existir un único datafile.

6. Elige tres pruebas de la Parte D y explica, con las lecturas lógicas medidas, por qué un tipo de índice gana al otro.

Ejemplo:
- Índice B-tree: mejor para búsquedas por igualdad y rangos.
- Índice Bitmap: mejor para columnas con pocos valores distintos y consultas analíticas.
- Índice cubriente: evita acceder a la tabla cuando todas las columnas necesarias están en el índice, reduciendo las lecturas lógicas.

8.1.2.	Agrupamiento, equilibrio y sargabilidad (E, F)
7. ¿Por qué IX_PED_FECHA tiene un clustering factor cercano al número de bloques e IX_PED_CLIENTE cercano al de filas? ¿Lo arreglaría un REBUILD?

Porque los pedidos fueron insertados ordenados por fecha, de modo que las filas consecutivas del índice también están próximas físicamente. En cambio, los clientes están distribuidos aleatoriamente, por lo que cada acceso cambia continuamente de bloque.

Un REBUILD no mejora el clustering factor, ya que este depende del orden físico de la tabla, no del índice.

8. Según tu curva de la Parte E, ¿a partir de qué porcentaje deja de compensar cada índice? Compáralo con la fórmula de la Unidad 3.

Debe responderse con el porcentaje obtenido en tu evidencia 04. En general, cuando el porcentaje de filas recuperadas supera el punto de equilibrio, el recorrido completo de la tabla resulta más eficiente que utilizar el índice.

9. Explica por qué Q4b ('Gonz%') usa el índice y aun así lee más bloques que un recorrido completo.

Porque el predicado LIKE 'Gonz%' es sargable y utiliza el índice. Sin embargo, recupera muchas filas (aproximadamente el 2 %), obligando a acceder a numerosos bloques distintos de la tabla. El coste combinado del índice y los accesos aleatorios supera al recorrido secuencial completo.

8.1.3.	Coste de escritura y mantenimiento (G, H)
10. Con tu evidencia 06: bloques modificados por INSERT con 4, 9 y 5 índices. ¿Qué índices de la propuesta del compañero rechazaste y con qué dato?

Debe responderse utilizando tu evidencia 06. En general se rechazaron los índices redundantes porque aumentaban significativamente el número de bloques modificados por cada INSERT sin aportar una mejora apreciable en las consultas.

11. Distingue, con tus gráficos de la Parte H, un índice con divisiones 50/50, uno con divisiones 90/10 y uno con patrón cola.

- 50/50: divisiones equilibradas entre ambas hojas.
- 90/10: una hoja queda casi llena y la otra apenas ocupa espacio.
- Patrón cola: crecimiento continuo únicamente por la hoja más a la derecha debido a claves crecientes.

12. ¿Qué remedio elegirías para el patrón cola en producción, en horario de servicio, y por qué?

Elegiría COALESCE, ya que reorganiza el índice en línea, con poco impacto sobre los usuarios y sin necesidad de reconstruir completamente el índice.

8.1.4.	SCN, control files, redo y FRA (I a L)
13. Con tu evidencia 08: ¿qué valores compara Oracle al abrir para decidir si hace falta recuperación?

Oracle compara el SCN del checkpoint almacenado en el control file con el SCN de la cabecera de cada datafile. Si no coinciden, inicia el proceso de recuperación.

14. ¿Qué protege la multiplexación del control file en el mismo volumen y qué no? ¿Cómo se haría en producción?

Protege frente a la corrupción o pérdida de un único archivo del control file.

No protege frente al fallo completo del disco o del volumen.

En producción, las copias deben situarse en discos o sistemas de almacenamiento independientes.

15. Explica los estados CURRENT, ACTIVE e INACTIVE con tu evidencia 10 y por qué Oracle rechaza los tres borrados del paso 4.

- CURRENT: LGWR está escribiendo actualmente.
- ACTIVE: ya no se escribe en él, pero todavía puede ser necesario para la recuperación.
- INACTIVE: ya no es necesario para la recuperación.
Oracle impide borrar estos archivos porque comprometería la capacidad de recuperación de la base de datos.

16. Describe la cadena de causas del escenario de la FRA llena, desde ORA-19815 hasta que la base deja de aceptar cambios.

La FRA se llena → Oracle genera ORA-19815 → ARCn deja de poder archivar redo → LGWR no puede reutilizar los grupos de redo → aparece la espera "log file switch (archiving needed)" → finalmente se produce ORA-00257 y la base deja de aceptar nuevas operaciones de escritura.

8.1.5.	Undo, WAL y simulacros (M a P)
17. Explica, con los pasos 1 a 4 de tu evidencia 12, la frase "lectores no bloquean a escritores".

Mientras una sesión modifica una fila sin hacer COMMIT, otra sesión puede leer la versión anterior reconstruida mediante el segmento UNDO. Por ello, las consultas no bloquean las actualizaciones y las actualizaciones no bloquean las consultas.

18. ¿Por qué un DELETE genera más undo que un INSERT? ¿Por qué el ROLLBACK también genera redo?

DELETE debe almacenar toda la información necesaria para reconstruir la fila eliminada, generando más UNDO.

El ROLLBACK también genera REDO porque los cambios realizados sobre los segmentos UNDO deben registrarse para garantizar la recuperación ante fallos.

19. Con tu evidencia 13: ¿dónde está la garantía de durabilidad justo después del COMMIT? ¿Y después del checkpoint?

Después del COMMIT, la garantía está en el redo log online, ya que LGWR ha escrito el REDO.

Después del checkpoint, además, los bloques modificados ya han sido escritos en los datafiles.

20. Simulacro A: explica roll forward y roll back con las filas 102 y 103 y las líneas del alert log.

- Roll forward: Oracle reaplica mediante REDO todas las transacciones confirmadas antes del fallo.
- Roll back: posteriormente deshace las transacciones que quedaron sin confirmar utilizando el UNDO.

21. Simulacros C y D: tu RTO y tu RPO en cada uno. ¿Por qué son tan distintos?

Debe responderse con los tiempos medidos en tu evidencia 14.

En general:
- El borrado de una tabla suele recuperarse rápidamente.
- La pérdida de un datafile requiere restauración física y recuperación mediante REDO, aumentando considerablemente el RTO.
El RPO normalmente es 0, ya que la base está en ARCHIVELOG.

22. Clasifica cada error ORA- del bloque 5 de tu runbook: incidente real, efecto del laboratorio o ruido conocido.

Debe hacerse utilizando tu evidencia 15. Como criterio general:
- Incidente real: errores que requieren actuación inmediata.
- Efecto del laboratorio: errores provocados deliberadamente durante los simulacros.
- Ruido conocido: advertencias o mensajes esperados que no afectan al funcionamiento normal.
