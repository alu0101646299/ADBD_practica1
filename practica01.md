## 1. Creación de la base de datos

a. Crear una base de datos llamada biblioteca.

```text
postgres=# CREATE DATABASE biblioteca;
CREATE DATABASE

postgres=# \c biblioteca
You are now connected to database "biblioteca" as user "postgres".
```

## 2. Creación de usuarios

a. Crear dos usuarios:

- i. admin_biblio con permisos de administrador sobre la base de
datos.

- ii. usuario_biblio con permisos solo de lectura.

```text
biblioteca=# CREATE USER admin_biblio WITH SUPERUSER;
CREATE ROLE

biblioteca=# CREATE USER usuario_biblio;
CREATE ROLE
```

b. Crear un rol llamado lectores con permisos únicamente de consulta
sobre todas las tablas de la base de datos.

```text
biblioteca=# CREATE ROLE lectores;
CREATE ROLE

biblioteca=# GRANT CONNECT ON DATABASE biblioteca TO lectores;
GRANT

biblioteca=# GRANT USAGE ON SCHEMA public TO lectores;
GRANT

biblioteca=# GRANT SELECT ON ALL TABLES IN SCHEMA public TO lectores;
GRANT

biblioteca=# ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO lectores;
ALTER DEFAULT PRIVILEGES
```

c. Asignar el usuario usuario_biblio a este rol.

```text
biblioteca=# GRANT lectores TO usuario_biblio;
GRANT ROLE
```

d. Consultar las tablas del sistema para listar todos los usuarios creados
(pg_roles).

```text
biblioteca=# SELECT rolname FROM pg_roles;
           rolname           
-----------------------------
 postgres
 pg_database_owner
 pg_read_all_data
 pg_write_all_data
 pg_monitor
 pg_read_all_settings
 pg_read_all_stats
 pg_stat_scan_tables
 pg_read_server_files
 pg_write_server_files
 pg_execute_server_program
 pg_signal_backend
 pg_checkpoint
 pg_use_reserved_connections
 pg_create_subscription
 mydb_admin
 admin_biblio
 usuario_biblio
 lectores
(19 rows)
```

e. Cambiar la contraseña del usuario usuario_biblio.

```text
biblioteca=# ALTER USER usuario_biblio WITH PASSWORD '1234';
ALTER ROLE
```

f. Configurar permisos de tal forma que el usuario usuario_biblio no
pueda eliminar registros en ninguna tabla.

```text
biblioteca=# REVOKE DELETE ON ALL TABLES IN SCHEMA public FROM usuario_biblio;
REVOKE
```

## 3. Creación de tablas

a. Crear las siguientes tablas con sus respectivas claves primarias:
- i. autores(id_autor, nombre, nacionalidad)
- ii. libros(id_libro, titulo, año_publicacion, id_autor)
ull.es
- iii. prestamos(id_prestamo, id_libro, fecha_prestamo,
fecha_devolucion, usuario_prestatario)

b. Establecer las claves foráneas correspondientes.

```text
biblioteca=# CREATE TABLE autores (
biblioteca(# id_autor SERIAL PRIMARY KEY,
biblioteca(# nombre VARCHAR(150) NOT NULL,
biblioteca(# nacionalidad VARCHAR(100)
biblioteca(# );
CREATE TABLE

biblioteca=# CREATE TABLE libros (
biblioteca(# id_libro SERIAL PRIMARY KEY,
biblioteca(# titulo VARCHAR(200) NOT NULL,
biblioteca(# año_publicacion INT,
biblioteca(# id_autor INT,
biblioteca(# CONSTRAINT fk_autor FOREIGN KEY (id_autor)
biblioteca(# REFERENCES autores(id_autor) ON DELETE CASCADE
biblioteca(# );
CREATE TABLE

biblioteca=# CREATE TABLE prestamos (
biblioteca(# id_prestamo SERIAL PRIMARY KEY,
biblioteca(# id_libro INT,
biblioteca(# fecha_prestamo DATE NOT NULL DEFAULT CURRENT_DATE,
biblioteca(# fecha_devolucion DATE,
biblioteca(# usuario_prestatario VARCHAR(100),
biblioteca(# CONSTRAINT fk_libro FOREIGN KEY (id_libro)
biblioteca(# REFERENCES libros(id_libro) ON DELETE CASCADE
biblioteca(# );
CREATE TABLE
```

## 4. Inserción de datos

a. Insertar al menos 5 autores, 8 libros y 5 préstamos de ejemplo.

```text
biblioteca=# INSERT INTO autores (nombre, nacionalidad) VALUES
biblioteca-# ('Gabriel García Márquez', 'Colombiana'),
biblioteca-# ('J.K. Rowling', 'Británica'),
biblioteca-# ('Isabel Allende', 'Chilena'),
biblioteca-# ('George Orwell', 'Británica'),
biblioteca-# ('Haruki Murakami', 'Japonesa');
INSERT 0 5

biblioteca=# INSERT INTO libros (titulo, año_publicacion, id_autor) VALUES
biblioteca-# ('Cien años de soledad', 1967, 1),
biblioteca-# ('El coronel no tiene quien le escriba', 1961, 1),
biblioteca-# ('Harry Potter y la piedra filosofal', 1997, 2),
biblioteca-# ('La casa de los espíritus', 1982, 3),
biblioteca-# ('1984', 1949, 4),
biblioteca-# ('Rebelión en la granja', 1945, 4),
biblioteca-# ('Tokio blues', 1987, 5),
biblioteca-# ('Kafka en la orilla', 2002, 5);
INSERT 0 8

biblioteca=# INSERT INTO prestamos (id_libro, fecha_prestamo, fecha_devolucion, usuario_prestatario) VALUES
biblioteca-# (1, '2023-09-01', '2023-09-15', 'Ana Perez'),
biblioteca-# (2, '2023-10-01', NULL, 'Luis Gomez'),
biblioteca-# (5, '2023-10-10', NULL, 'Ana Perez'),
biblioteca-# (7, '2023-10-15', '2023-10-25', 'Carlos Ruiz'),
biblioteca-# (8, '2023-10-20', NULL, 'Marta Lopez');
INSERT 0 5
```

## 5. Consultas básicas

a. Listar todos los libros con su autor correspondiente.

```text
biblioteca=# SELECT l.titulo, a.nombre AS autor
biblioteca-# FROM libros l
biblioteca-# JOIN autores a ON l.id_autor = a.id_autor;
                titulo                |         autor          
--------------------------------------+------------------------
 Cien años de soledad                 | Gabriel García Márquez
 El coronel no tiene quien le escriba | Gabriel García Márquez
 Harry Potter y la piedra filosofal   | J.K. Rowling
 La casa de los espíritus             | Isabel Allende
 1984                                 | George Orwell
 Rebelión en la granja                | George Orwell
 Tokio blues                          | Haruki Murakami
 Kafka en la orilla                   | Haruki Murakami
(8 rows)
```

b. Mostrar los préstamos que aún no tienen fecha de devolución.

```text
biblioteca=# SELECT * FROM prestamos WHERE fecha_devolucion IS NULL;
 id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario 
-------------+----------+----------------+------------------+---------------------
           2 |        2 | 2023-10-01     |                  | Luis Gomez
           3 |        5 | 2023-10-10     |                  | Ana Perez
           5 |        8 | 2023-10-20     |                  | Marta Lopez
(3 rows)
```

c. Obtener los autores que tienen más de un libro registrado.

```text
biblioteca=# SELECT a.nombre, COUNT(l.id_libro) AS total_libros
biblioteca-# FROM autores a
biblioteca-# JOIN libros l ON a.id_autor = l.id_autor
biblioteca-# GROUP BY a.id_autor, a.nombre
biblioteca-# HAVING COUNT(l.id_libro) > 1;
         nombre         | total_libros 
------------------------+--------------
 Haruki Murakami        |            2
 George Orwell          |            2
 Gabriel García Márquez |            2
(3 rows)
```

## 6. Consultas con agregación

a. Calcular el número total de préstamos realizados.

```text
biblioteca=# SELECT COUNT(*) AS total_prestamos FROM prestamos;
 total_prestamos 
-----------------
               5
(1 row)
```

b. Obtener el número de libros prestados por cada usuario.

```text
biblioteca=# SELECT usuario_prestatario, COUNT(*) AS total_libros_prestados
biblioteca-# FROM prestamos
biblioteca-# GROUP BY usuario_prestatario;
 usuario_prestatario | total_libros_prestados 
---------------------+------------------------
 Carlos Ruiz         |                      1
 Marta Lopez         |                      1
 Luis Gomez          |                      1
 Ana Perez           |                      2
(4 rows)
```

## 7. Modificación de datos

a. Actualizar la fecha de devolución de un préstamo pendiente.

```text
biblioteca=# UPDATE prestamos
biblioteca-# SET fecha_devolucion = CURRENT_DATE
biblioteca-# WHERE id_prestamo = 2 AND fecha_devolucion IS NULL;
UPDATE 1
```

b. Eliminar un libro y comprobar el efecto en la tabla de préstamos.

```text
biblioteca=# DELETE FROM libros WHERE id_libro = 5;
DELETE 1

biblioteca=# SELECT * FROM prestamos;
 id_prestamo | id_libro | fecha_prestamo | fecha_devolucion | usuario_prestatario 
-------------+----------+----------------+------------------+---------------------
           1 |        1 | 2023-09-01     | 2023-09-15       | Ana Perez
           4 |        7 | 2023-10-15     | 2023-10-25       | Carlos Ruiz
           5 |        8 | 2023-10-20     |                  | Marta Lopez
           2 |        2 | 2023-10-01     | 2026-09-28       | Luis Gomez
(4 rows)
```

## 8. Creación de vistas

a. Crear una vista que muestre: título del libro, autor y nombre del prestatario.

```text
biblioteca=# CREATE VIEW vista_libros_prestados AS
biblioteca-# SELECT l.titulo, a.nombre AS autor, p.usuario_prestatario
biblioteca-# FROM prestamos p
biblioteca-# JOIN libros l ON p.id_libro = l.id_libro
biblioteca-# JOIN autores a ON l.id_autor = a.id_autor;
CREATE VIEW
```

b. Conceder permisos de consulta sobre esta vista únicamente a usuario_biblio.

```text
biblioteca=# GRANT SELECT ON vista_libros_prestados TO usuario_biblio;
GRANT
```

## 9. Funciones y consultas avanzadas

a. Función que reciba el nombre de un autor y devuelva todos los libros escritos por él.

```text
biblioteca=# CREATE OR REPLACE FUNCTION libros_por_autor(nombre_autor_buscado VARCHAR)
biblioteca-# RETURNS TABLE(titulo_libro VARCHAR) AS $$
biblioteca$# BEGIN
biblioteca$#     RETURN QUERY
biblioteca$#     SELECT l.titulo::VARCHAR
biblioteca$#     FROM libros l
biblioteca$#     JOIN autores a ON l.id_autor = a.id_autor
biblioteca$#     WHERE a.nombre = nombre_autor_buscado;
biblioteca$# END;
biblioteca$# $$ LANGUAGE plpgsql;
CREATE FUNCTION

biblioteca=# SELECT * FROM libros_por_autor('Gabriel García Márquez');
             titulo_libro             
--------------------------------------
 Cien años de soledad
 El coronel no tiene quien le escriba
(2 rows)
```

b. Consulta que devuelva los tres libros más prestados.

```text
biblioteca=# SELECT l.titulo, COUNT(p.id_prestamo) AS veces_prestado
biblioteca-# FROM libros l
biblioteca-# JOIN prestamos p ON l.id_libro = p.id_libro
biblioteca-# GROUP BY l.id_libro, l.titulo
biblioteca-# ORDER BY veces_prestado DESC
biblioteca-# LIMIT 3;
                titulo                | veces_prestado 
--------------------------------------+----------------
 El coronel no tiene quien le escriba |              1
 Tokio blues                          |              1
 Cien años de soledad                 |              1
(3 rows)
```

## 10. Exportación e importación de datos

a. Exportar el contenido de la tabla libros a un archivo CSV.

b. Importar datos adicionales de autores desde un archivo CSV externo.