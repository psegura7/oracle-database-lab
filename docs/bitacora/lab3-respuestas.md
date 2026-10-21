# Laboratorio 3 — Respuestas de Comprobación

## 8.1.1. Docker

1. **¿Qué diferencia hay entre una imagen y un contenedor? Usa como ejemplo lo que hiciste en los ejercicios G2 y G4.**  
   La imagen es el molde empaquetado y de solo lectura; el contenedor es la instancia viva corriendo a partir de esa plantilla. En G2 levantamos `hello-world`, que solo ejecutó su ejecutable, mostró el texto de bienvenida y se detuvo[cite: 34]. En cambio, en G4 usamos la imagen `alpine:3.20` para arrancar un contenedor interactivo con `/bin/sh`, lo que nos permitió meter comandos dentro de su propia terminal como si fuera un Linux independiente[cite: 35, 36].

2. **En el Ejercicio G5 el archivo `nota.txt` desapareció y en el G6 no. Explica por qué.**  
   En G5 escribimos directamente en el almacenamiento temporal del contenedor; al hacerle `docker rm`, se destruyó tanto el contenedor como todo lo que guardaba dentro[cite: 36]. En G6 montamos un volumen persistente (`datos-prueba:/datos`)[cite: 37]. Como los volúmenes son gestionados por Docker de forma independiente a los contenedores, el dato se guardó fuera de él y el siguiente contenedor pudo leerlo sin problema[cite: 37].

3. **¿Qué diferencia hay entre `docker ps` y `docker ps -a`, y qué significa `STATUS = Exited (0)`?**  
   `docker ps` solo saca los contenedores que están corriendo ahora mismo; `docker ps -a` saca todos, incluso los que están apagados o terminaron[cite: 34]. `STATUS = Exited (0)` significa que el contenedor finalizó y que su proceso terminó correctamente sin arrojar errores (código de salida 0)[cite: 34].

4. **En `-p 8181:8181`, ¿qué número corresponde a tu equipo y cuál al contenedor? ¿Qué pasaría con `-p 80:8080` en el ejercicio de nginx?**  
   Siempre va en formato `máquina_local:contenedor`[cite: 38]. El primer 8181 es el puerto expuesto en mi equipo y el segundo es el puerto interno donde escucha el servicio en el contenedor[cite: 38]. Si ponemos `-p 80:8080` con nginx, estaríamos mandando el tráfico de nuestro puerto 80 al puerto 8080 del contenedor; como nginx escucha internamente en el 80 y no en el 8080, la página nunca cargaría[cite: 38].

5. **¿Por qué un contenedor de Oracle se queda en marcha y el de `hello-world` termina solo?**  
   Porque un contenedor vive lo que dura su proceso principal[cite: 36]. `hello-world` lanza un script que imprime unas líneas de texto y acaba de inmediato[cite: 34]. Oracle levanta una base de datos con procesos de fondo (background processes) y un listener que se quedan escuchando conexiones de forma indefinida, por lo que nunca se apaga[cite: 44, 45].

6. **¿Qué es el digest de una imagen y por qué lo registramos si ya sabemos que usamos `:latest`?**  
   El digest es el hash SHA-256 único e inmutable de las capas de la imagen[cite: 35]. Se guarda como evidencia porque la etiqueta `:latest` va cambiando con el tiempo conforme Oracle sube versiones nuevas; registrando el digest dejamos constancia exacta de qué versión descargamos bit por bit[cite: 35, 41].

7. **¿Qué comando borraría realmente los datos de Oracle? ¿Por qué `docker rm oralab-26ai` no lo hace?**  
   Los borraría `docker volume rm oralab-26ai-data`[cite: 39]. Hacer `docker rm oralab-26ai` no toca los datos porque estos están guardados en el volumen persistente montado en `/opt/oracle/oradata`, no en el contenedor[cite: 37, 43].

---

## 8.1.2. Git, organización y evidencia

8. **¿Por qué este laboratorio se hace dentro del repositorio `oracle-database-lab`, con Issue, branch y Pull Request, en vez de en una carpeta aparte?**  
   Para trabajar con flujo profesional de Git y no montar un "servidor snowflake" que nadie sabe cómo se construyó[cite: 10, 24]. Dejar todo con scripts, issues y PRs permite que cualquier compañero pueda revisar la instalación, reproducir el entorno si el portátil falla y tener un registro de auditoría claro[cite: 10, 78].

9. **¿Qué diferencia hay entre `source 00-config.sh` y `bash 00-config.sh`? ¿Por qué usamos `source`?**  
   `bash script.sh` abre una terminal hija temporal: ejecuta el script y al cerrarse se pierden todas las variables[cite: 26]. Con `source 00-config.sh` ejecutamos el script dentro de nuestra propia terminal activa, de modo que variables como `$CONT_NAME` o `$EVID` se quedan cargadas en memoria para los siguientes comandos[cite: 25, 26].

10. **Explica cada parte del nombre `20260915T091230Z_02-docker.script.log`.**  
    - `20260915T091230Z`: Fecha y hora en estándar UTC compacto (la Z final indica UTC)[cite: 23].  
    - `02`: El número de paso del guion de prácticas[cite: 23].  
    - `docker`: Nombre descriptivo en kebab-case[cite: 23].  
    - `script.log`: La extensión que marca que es un volcado de terminal[cite: 23].

11. **¿Para qué sirve `.gitattributes` y qué error evita?**  
    Sirve para forzar que los saltos de línea se normalicen a LF (estilo Linux)[cite: 23]. Evita que al editar scripts entre Windows y Linux se cuelen retornos de carro CRLF, que luego provocan fallos típicos en bash como `$'\r': command not found`[cite: 23].

12. **¿Por qué en este Pull Request elegimos *Create a merge commit* en lugar de *Squash and merge*?**  
    Porque nos interesa conservar todo el histórico paso por paso[cite: 75]. En este laboratorio cada commit refleja una parte concreta y aporta sus propias evidencias; si hiciéramos squash juntaríamos todo en un único commit y perderíamos ese detalle cronológico[cite: 75].

---

## 8.1.3. Seguridad

13. **Describe las cuatro capas de la estrategia de contraseñas (Parte D) y qué pasaría si te saltas la primera.**  
    - Capa 1: Decirle a `.gitignore` que ignore `config/.env`[cite: 26].  
    - Capa 2: Crear la plantilla pública `config/.env.example` sin contraseñas reales[cite: 26].  
    - Capa 3: Rellenar el `config/.env` real en local y verificar con `git check-ignore` que no se rastree[cite: 27].  
    - Capa 4: Cargar las variables a la terminal con `set -a; source config/.env; set +a` para no escribirlas en los comandos[cite: 27].  
    Si te saltas la primera, en cuanto crees el `.env` real Git lo detectará como archivo nuevo y acabarás subiendo las contraseñas de administrador a GitHub[cite: 26, 27].

14. **¿Por qué no escribimos la contraseña directamente en el comando `docker run`, aunque el script no se suba a Git?**  
    Porque todo comando tecleado se guarda en el historial de la shell (`~/.bash_history`) en texto plano[cite: 27]. Además, cualquiera que ejecute un listado de procesos tipo `ps` vería la clave en los argumentos del comando[cite: 27]. Con variables de entorno, solo queda registrado el nombre de la variable[cite: 27].

15. **Si descubres tu contraseña en un commit ya publicado, ¿basta con borrarla en un commit nuevo? ¿Qué debes hacer?**  
    No basta; borrarla en un commit nuevo solo la oculta del presente, pero cualquiera puede verla revisando el historial de commits anteriores[cite: 26]. Lo que se debe hacer de inmediato es dar la clave por comprometida, cambiarla/rotarla en la base de datos y limpiar la rama o avisar al profesor para rehacer el historial[cite: 74].

---

## 8.1.4. Oracle y herramientas

16. **¿Por qué no usamos `SPOOL` ni `@archivo.sql` con `sqlplus` dentro del contenedor, y qué hicimos en su lugar?**  
    `sqlplus` está corriendo dentro del contenedor, así que no tiene acceso directo a los ficheros de nuestra máquina[cite: 48]. Un `@archivo.sql` fallaría porque el script está en nuestro disco, y un `SPOOL` guardaría el log dentro del contenedor en vez de en la carpeta del repositorio[cite: 48]. En su lugar, redirigimos la entrada con `< archivo.sql` a través de `docker exec -i` y capturamos la salida con `tee`[cite: 47, 48].

17. **¿Qué hace `WHENEVER SQLERROR EXIT SQL.SQLCODE` al inicio de V000 y V001, y qué pasaría sin esa línea?**  
    Hace que `sqlplus` corte la ejecución y devuelva el código de error si cualquier sentencia falla[cite: 50, 54]. Sin eso, si una sentencia da error el script seguiría adelante ejecutando las siguientes como si nada, dejando la base de datos en un estado inconsistente a medio aplicar[cite: 50].

18. **¿Qué es una migración y por qué V000 y V001 no se deben editar una vez aplicadas?**  
    Una migración es un archivo SQL versionado que define el paso de un esquema de un estado a otro de manera controlada[cite: 49]. No se deben tocar una vez corridas porque romperías la coherencia entre compañeros; si alguien ya ejecutó esa versión y tú la modificas por detrás, los entornos dejan de ser idénticos[cite: 49]. Si hay que corregir algo, se crea una migración nueva (como un `V002`)[cite: 49].

19. **¿Por qué en SQL Developer se usa el servicio `FREEPDB1` y no `FREE` ni un SID?**  
    `FREE` es el contenedor raíz (CDB) para temas de administración del motor[cite: 25, 60]. `FREEPDB1` es la base de datos conectable (PDB) donde realmente hemos provisionado los esquemas y las tablas de trabajo[cite: 25, 49, 60]. Usar SID o conectarse a `FREE` nos dejaría fuera de donde están nuestras tablas[cite: 60].

20. **¿Qué aporta SQLcl frente a SQL\*Plus, y por qué un DBA debe dominar ambas?**  
    SQLcl es más moderna: colorea la sintaxis, formatea tablas con `ansiconsole`, autocompleta con tabulador, guarda conexiones y se integra con Liquibase[cite: 57, 58]. Conviene dominar ambas porque SQLcl es genial para el día a día, pero `sqlplus` viene preinstalada por defecto en absolutamente cualquier máquina o servidor con Oracle, así que es la que te encuentras siempre para emergencias[cite: 47, 58].

---

## 8.1.5. Entorno de trabajo

21. **¿Por qué el curso pasa de Git Bash a Ubuntu en WSL 2? Da al menos dos problemas concretos de Git Bash que desaparecen en Ubuntu.**  
    Porque pasamos de una emulación básica a un sistema Linux real[cite: 7]. Dos problemas que nos quitamos de encima:  
    1. Git Bash convierte automáticamente rutas como `/opt/...` en rutas Windows de tipo `C:\...`, rompiendo argumentos de Docker[cite: 7].  
    2. En Git Bash no existen utilidades habituales de administración como `free`, `ss`, `htop` o `shellcheck`, y para terminales interactivas suele dar problemas de TTY necesitando cosas como `winpty`[cite: 7, 33].

22. **¿Por qué clonamos el repositorio en `~/oracle-database-lab` y no trabajamos sobre la carpeta de Windows (`/mnt/c/...`)? ¿Y por qué recomendamos bash frente a zsh para los scripts del curso?**  
    Trabajar en `/home` evita cruzar el puente de sistemas de archivos entre Windows y Linux, lo cual ralentiza mucho Git y Docker, además de evitar problemas con permisos de ejecución y saltos de línea[cite: 20]. Y recomendamos `bash` porque es la shell estándar presente por defecto en prácticamente todos los servidores corporativos; escribir los scripts en bash garantiza que correrán igual en cualquier servidor sin depender de configuraciones personales[cite: 8].
