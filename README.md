# Tarea: Docker + MySQL — Base de datos `pets_db`

La actividad presentada tiene por objetivo crear un contenedor Docker, levantar una base de datos MySQL dentro de él, manipularla desde un cliente (DBeaver) y publicar la imagen en DockerHub.

## Objetivos

- Crear un contenedor de Docker a partir de una imagen oficial.
- Crear y manipular una base de datos MySQL.
- Publicar la imagen resultante en DockerHub.

## Tecnologías utilizadas

- **Docker Desktop** — motor de contenedores.
- **MySQL 8.0** (imagen `mysql:8.0-debian`) — servidor de base de datos.
- **DBeaver** — cliente de base de datos.
- **DockerHub** — registro de imágenes en la nube.

---

## Paso 1 — Descargar la imagen de MySQL

Una **imagen** es una plantilla inmutable con MySQL empaquetado; un **contenedor** es una instancia viva de esa imagen. Primero traemos la plantilla a la máquina local:

```bash
docker pull mysql:8.0-debian
```

El tag `8.0-debian` indica la versión (MySQL 8.0) y la variante (construida sobre Debian). Durante la descarga se bajan las distintas **capas** (layers) que componen la imagen.

### Captura — Sección Images de Docker Desktop

<img width="1906" height="612" alt="2  Image Docker" src="https://github.com/user-attachments/assets/30f09fa1-d8e2-447c-ad17-56f55192522b" />

---

## Paso 2 — Arrancar el contenedor

```bash
docker run --name test-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=TuPassword123 -d mysql:8.0-debian
```

Explicación de cada flag:

| Flag | Función |
|------|---------|
| `--name test-mysql` | Nombre legible para el contenedor. |
| `-p 3306:3306` | Mapea el puerto 3306 de la máquina local al 3306 interno del contenedor. Sin esto, el cliente no puede conectar. |
| `-e MYSQL_ROOT_PASSWORD=...` | Variable de entorno que fija la contraseña del usuario `root`. Obligatoria. |
| `-d` | *Detached*: ejecuta en segundo plano. |

Comprobación de que el contenedor está en marcha:

```bash
docker ps
```

En la salida, `STATUS: Up ...` confirma que está vivo y `PORTS: 0.0.0.0:3306->3306/tcp` confirma el mapeo de puertos.

### Captura — Sección Containers de Docker Desktop

<img width="1905" height="612" alt="1  Container Docker" src="https://github.com/user-attachments/assets/670b4aff-3255-4e6c-a736-d88590707dd4" />

---

## Paso 3 — Conectar DBeaver al contenedor

DBeaver es el **cliente** que se comunica con el servidor MySQL. Datos de conexión:

- **Host:** `localhost` (DBeaver sale desde la máquina local y el flag `-p` redirige al contenedor).
- **Puerto:** `3306`
- **Usuario:** `root`
- **Contraseña:** la definida en `MYSQL_ROOT_PASSWORD`.

> Si el test de conexión falla con *"Public Key Retrieval is not allowed"*, en **Driver properties** poner `allowPublicKeyRetrieval = true` y `useSSL = false`.

---

## Paso 4 — Crear la base de datos y la tabla

Script ejecutado completo (con **Alt+X** en DBeaver, para respetar el orden de las sentencias):

```sql
CREATE DATABASE IF NOT EXISTS pets_db;

CREATE TABLE pets_db.pets (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nombre VARCHAR(50),
    tipo VARCHAR(50),
    raza VARCHAR(50),
    edad INT
);

INSERT INTO pets_db.pets (nombre, tipo, raza, edad) VALUES
    ('Max', 'Perro', 'Chihuahua', 2),
    ('Luna', 'Gato', 'Siamés', 4);
```

Calificar los nombres con `pets_db.` delante de la tabla evita el error `No database selected` cuando no hay una base activa seleccionada.

### Captura — Base de datos creada en DBeaver

<img width="1920" height="1020" alt="3  Base de datos en DBeaver" src="https://github.com/user-attachments/assets/af8147fc-7be3-4ba5-a502-b75ecac6c31d" />

---

## Paso 5 — Listar todas las mascotas

```sql
SELECT * FROM pets_db.pets;
```

Resultado esperado:

| id | nombre | tipo  | raza      | edad |
|----|--------|-------|-----------|------|
| 1  | Max    | Perro | Chihuahua | 2    |
| 2  | Luna   | Gato  | Siamés    | 4    |

### Captura — Sentencia SQL y resultado

<img width="1394" height="838" alt="4  Sentencia SQL" src="https://github.com/user-attachments/assets/241664b2-f083-42b2-ac5c-308ac765bb48" />

---

## Paso 6 — Subir la imagen a DockerHub

Publicar en DockerHub es análogo a un `git push`, pero de imágenes. Tres pasos:

```bash
# 1. Autenticarse (las credenciales quedan guardadas para siguientes sesiones)
docker login

# 2. Etiquetar la imagen con el usuario de DockerHub por delante
docker tag mysql:8.0-debian juanimdev/pets-mysql:8.0

# 3. Subir la imagen
docker push juanimdev/pets-mysql:8.0
```

Al terminar, la imagen aparece en el repositorio de DockerHub del usuario.

### Captura — DockerHub con la imagen subida

<img width="1908" height="1018" alt="5  Docker" src="https://github.com/user-attachments/assets/793d931e-aee5-4406-a356-e9254443ae46" />

---

## Resumen de comandos

```bash
docker pull mysql:8.0-debian
docker run --name test-mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=TuPassword123 -d mysql:8.0-debian
docker ps
docker login
docker tag mysql:8.0-debian juanimdev/pets-mysql:8.0
docker push juanimdev/pets-mysql:8.0
```
