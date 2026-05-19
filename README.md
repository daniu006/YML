# Práctica Semana 6 – Docker Compose: WordPress + MySQL + phpMyAdmin


 Wordpress con docker compose YML

---

## 2. Tiempo de duración

Aproximadamente **90 minutos** (incluyendo resolución de errores de DNS y dependencias).

---

## 3. Fundamentos

### ¿Qué es Docker?

Docker es una plataforma de virtualización a nivel de sistema operativo que permite empaquetar aplicaciones y sus dependencias en unidades llamadas **contenedores**. A diferencia de las máquinas virtuales tradicionales, los contenedores comparten el kernel del sistema operativo anfitrión, lo que los hace mucho más ligeros y rápidos de iniciar.

Un contenedor Docker incluye todo lo necesario para ejecutar una aplicación: código, runtime, librerías, variables de entorno y archivos de configuración. Esto garantiza que la aplicación se ejecute de manera idéntica en cualquier entorno, ya sea en desarrollo, pruebas o producción.

### ¿Qué es Docker Compose?

**Docker Compose** es una herramienta que permite definir y ejecutar aplicaciones multi-contenedor mediante un archivo YAML (`docker-compose.yml`). Con un solo comando (`docker-compose up`) se pueden levantar, conectar y configurar múltiples servicios simultáneamente.

La arquitectura de Docker Compose se basa en tres conceptos clave:

- **Servicios**: cada contenedor que forma parte de la aplicación (WordPress, MySQL, phpMyAdmin).
- **Redes**: permiten que los contenedores se comuniquen entre sí de forma aislada.
- **Volúmenes**: almacenamiento persistente que sobrevive al ciclo de vida de los contenedores.

### ¿Qué es WordPress?

WordPress es el sistema de gestión de contenidos (CMS) más utilizado en el mundo, representando más del 40% de todos los sitios web. Está desarrollado en PHP y requiere una base de datos MySQL para almacenar contenido, usuarios, configuraciones y plugins.

### ¿Qué es MySQL?

MySQL es un sistema de gestión de bases de datos relacional (RDBMS) de código abierto. Es el motor de base de datos más popular para aplicaciones web y forma parte del stack LAMP (Linux, Apache, MySQL, PHP).

### ¿Qué es phpMyAdmin?

phpMyAdmin es una interfaz web gratuita escrita en PHP que permite administrar bases de datos MySQL de forma visual. Permite crear, modificar y eliminar bases de datos, tablas, filas y ejecutar consultas SQL sin necesidad de usar la línea de comandos.

### Redes en Docker

La red tipo `bridge` en Docker crea una red privada interna donde los contenedores pueden comunicarse usando sus nombres de servicio como hostname. Por ejemplo, WordPress puede conectarse a MySQL simplemente usando `mysql` como dirección de host.

### Volúmenes en Docker

Los volúmenes son mecanismos de persistencia de datos en Docker. Sin volúmenes, al detener un contenedor todos sus datos se pierden. Al definir volúmenes, los datos quedan almacenados en el sistema de archivos del host y sobreviven reinicios o eliminaciones del contenedor.

---

## 4. Conocimientos previos

Para realizar esta práctica el estudiante necesita tener claros los siguientes temas:

- Comandos básicos de Linux (mkdir, ls, cat, mv, rm)
- Edición de archivos con `nano` en terminal
- Conceptos básicos de contenedores y virtualización
- Manejo del navegador web
- Fundamentos de bases de datos relacionales
- Sintaxis básica de YAML

---

## 5. Objetivos a alcanzar

- Construir un archivo `docker-compose.yml` con tres servicios interconectados.
- Configurar una red Docker tipo bridge para comunicación entre contenedores.
- Definir volúmenes persistentes para WordPress y MySQL.
- Resolver errores de DNS en WSL para permitir la descarga de imágenes Docker.
- Acceder a WordPress y phpMyAdmin desde el navegador del host Windows.
- Verificar el funcionamiento correcto de todos los servicios.

---

## 6. Equipo necesario

- Computador con Windows 11 y WSL2 activado (Ubuntu 24.04 LTS)
- Docker Engine v29.1.3 instalado en WSL
- docker-compose v1.29.2 (instalado durante la práctica)
- Navegador web (Chrome, Firefox o Edge)
- Conexión a Internet para descargar imágenes de Docker Hub

---

## 7. Material de apoyo

- Documentación oficial de Docker Compose: https://docs.docker.com/compose/
- Docker Hub – Imagen WordPress: https://hub.docker.com/_/wordpress
- Docker Hub – Imagen MySQL: https://hub.docker.com/_/mysql
- Docker Hub – Imagen phpMyAdmin: https://hub.docker.com/_/phpmyadmin
- Guía de asignatura Semana 6
- Repositorio de referencia del profesor: https://github.com/maguaman2/informe-tendencias

---

## 8. Procedimiento

### Paso 1: Crear el directorio de trabajo

Abrir la terminal de WSL y ejecutar:

```bash
mkdir ~/semana6-docker && cd ~/semana6-docker
```

<img width="747" height="52" alt="image" src="https://github.com/user-attachments/assets/7c27dfb7-0369-4a1d-9e09-601f3dcad866" />


---

### Paso 2: Crear el archivo docker-compose.yml

```bash
nano docker-compose.yml
```

Contenido del archivo:

```yaml
version: '3.8'

services:

  wordpress:
    image: wordpress:latest
    container_name: wordpress_app
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppassword
      WORDPRESS_DB_NAME: wordpress_db
    volumes:
      - wordpress_data:/var/www/html
    networks:
      - wp_network
    depends_on:
      - mysql

  mysql:
    image: mysql:8.0
    container_name: mysql_db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: rootpassword
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppassword
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - wp_network

  phpmyadmin:
    image: phpmyadmin/phpmyadmin:latest
    container_name: phpmyadmin_ui
    restart: always
    ports:
      - "8081:80"
    environment:
      PMA_HOST: mysql
      PMA_PORT: 3306
      MYSQL_ROOT_PASSWORD: rootpassword
    networks:
      - wp_network
    depends_on:
      - mysql

networks:
  wp_network:
    driver: bridge

volumes:
  wordpress_data:
  mysql_data:
```

Guardar con `Ctrl+O`, `Enter`, luego salir con `Ctrl+X`.

---

### Paso 3: Verificar el archivo creado

```bash
cat docker-compose.yml
```
<img width="807" height="602" alt="image" src="https://github.com/user-attachments/assets/dec8de34-8c4f-44cf-a389-48660ea66b36" />

---

### Paso 4: Resolución de Error 2 – docker compose no reconocido

**Error encontrado:**
```
unknown shorthand flag: 'd' in -d
```

**Causa:** La versión de Docker instalada desde los repositorios de Ubuntu no incluye el plugin `docker compose` (con espacio). Solo funciona `docker-compose` (con guión).

**Solución aplicada:**
```bash
sudo apt install docker-compose
```

A partir de este punto usar siempre:
```bash
docker-compose up -d     # correcto en esta instalación
# NO: docker compose up -d
```

---

### Paso 5: Resolución de Error 3 – Fallo de DNS en WSL

**Error encontrado:**
```
ERROR: failed to resolve reference "docker.io/phpmyadmin/phpmyadmin:latest":
dial tcp: lookup registry-1.docker.io on 10.255.255.254:53: read udp ... i/o timeout
```

**Causa:** WSL2 usa un DNS automático (`10.255.255.254`) que no resuelve nombres de dominio correctamente en algunas redes. El ping a IP funcionaba (`ping 8.8.8.8`) pero no la resolución DNS (`ping google.com`).

**Solución aplicada:**
```bash
# 1. Eliminar el resolv.conf generado automáticamente por WSL
sudo rm /etc/resolv.conf

# 2. Crear uno nuevo con DNS de Google
sudo nano /etc/resolv.conf
```

Contenido del archivo `/etc/resolv.conf`:
```
nameserver 8.8.8.8
nameserver 8.8.4.4
```

```bash
# 3. Proteger el archivo para que WSL no lo sobreescriba al reiniciar
sudo chattr +i /etc/resolv.conf
```

---


### Paso 6: Levantar los servicios exitosamente

```bash
docker-compose up -d
```

Salida esperada:
```
Creating mysql_db       ... done
Creating phpmyadmin_ui  ... done
Creating wordpress_app  ... done
```
<img width="627" height="101" alt="image" src="https://github.com/user-attachments/assets/10888ce3-f2b3-4540-8252-46022464c66b" />

---

### Paso 7: Verificar los contenedores en ejecución

```bash
docker ps
```

Salida obtenida:

<img width="1327" height="212" alt="image" src="https://github.com/user-attachments/assets/b4f290d6-05dc-42e6-9d84-282039b8c088" />


Los tres contenedores aparecen con estado **Up**.

---

### Paso 18: Ver logs de los servicios

```bash
docker-compose logs -f
```

Los logs confirmaron:
- MySQL inicializó correctamente la base de datos `wordpress_db` y creó el usuario `wpuser`.
- WordPress detectó las variables de entorno y generó el archivo `wp-config.php` automáticamente.
- phpMyAdmin y Apache arrancaron sin errores.

Salir de los logs con `Ctrl+C`.

<img width="1345" height="653" alt="image" src="https://github.com/user-attachments/assets/0a00a2da-82d6-401b-883b-83d86374ad10" />


---

### Paso 9: Acceder a los servicios en el navegador

| Servicio    | URL                        | Credenciales                        |
|-------------|----------------------------|-------------------------------------|
| WordPress   | http://localhost:8080      | Se configura en el asistente inicial |
| phpMyAdmin  | http://localhost:8081      | Usuario: ` wpuser` / Contraseña: `wppassword` |


Acceso a WordPress en http://localhost:8080

**<img width="1918" height="1142" alt="image" src="https://github.com/user-attachments/assets/c5219fb0-c796-4747-89ec-72f25aa3c5bf" />

Acceso a phpMyAdmin en http://localhost:8081

<img width="1918" height="1141" alt="image" src="https://github.com/user-attachments/assets/229066fd-533d-4a69-9093-6d296836c0e0" />


---

### Paso 10: Detener los servicios

```bash
# Solo detener (conserva datos)
docker-compose down

# Detener y eliminar volúmenes (borra todos los datos)
docker-compose down -v
```
<img width="695" height="317" alt="image" src="https://github.com/user-attachments/assets/e59738c0-57ad-4563-9795-c24cac6107c2" />

---

## 9. Resultados esperados

Al finalizar la práctica se obtuvieron los siguientes resultados:

**Resultado 1 – Tres contenedores activos:**
El comando `docker ps` mostró los contenedores `wordpress_app`, `phpmyadmin_ui` y `mysql_db` corriendo simultáneamente, conectados a través de la red `semana6-docker_wp_network`.

**Resultado 2 – Red Docker creada:**
Se creó automáticamente la red `semana6-docker_wp_network` con driver `bridge`, permitiendo que WordPress se conecte a MySQL usando el hostname `mysql` sin exponer el puerto 3306 al exterior.

**Resultado 3 – Volúmenes persistentes creados:**
Se crearon los volúmenes `semana6-docker_wordpress_data` y `semana6-docker_mysql_data`, garantizando que los datos sobrevivan reinicios de contenedores.

**Resultado 4 – WordPress funcional:**
Al acceder a `http://localhost:8080` se presentó el asistente de instalación de WordPress, confirmando que el servicio web y la conexión a la base de datos funcionan correctamente.

**Resultado 5 – phpMyAdmin funcional:**
Al acceder a `http://localhost:8081` se visualizó la base de datos `wordpress_db` , confirmando la correcta comunicación entre phpMyAdmin y MySQL.

---

## 10. Bibliografía

Docker Inc. (2024). *Docker Compose overview*. Docker Documentation. https://docs.docker.com/compose/

Docker Inc. (2024). *wordpress - Official Image*. Docker Hub. https://hub.docker.com/_/wordpress

Docker Inc. (2024). *mysql - Official Image*. Docker Hub. https://hub.docker.com/_/mysql

Docker Inc. (2024). *phpmyadmin - Official Image*. Docker Hub. https://hub.docker.com/_/phpmyadmin

Canonical Ltd. (2024). *Ubuntu WSL*. Ubuntu. https://ubuntu.com/desktop/wsl

Microsoft. (2024). *Windows Subsystem for Linux Documentation*. Microsoft Learn. https://learn.microsoft.com/en-us/windows/wsl/

