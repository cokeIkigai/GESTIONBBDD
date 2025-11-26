UT 4: Gestión de Usuarios y Permisos en PostgreSQL
Objetivos

Comprender los principios fundamentales de seguridad en bases de datos.

Crear, modificar y eliminar roles y usuarios en PostgreSQL.

Conceder y revocar privilegios de sistema y de objetos.

Gestionar permisos mediante roles heredables.

Aplicar políticas de seguridad: límites, conexiones y configuración del rol.

Activar y consultar auditoría básica mediante log_statement.

Aplicar buenas prácticas de diseño de permisos en entornos multiusuario.

1. Seguridad en Bases de Datos y Roles en PostgreSQL
1.1 Seguridad en bases de datos

Conjunto de políticas y permisos que controlan quién accede a los datos y qué puede hacer.

Importancia:

Protección de información sensible.

Cumplimiento legal.

Prevención de accesos indebidos.

1.2 Seguridad del sistema vs seguridad de datos

Seguridad del sistema: Conectarse (login), crear base de datos, crear otros roles.

Seguridad de los datos: SELECT, INSERT, UPDATE, DELETE, REFERENCES.

1.3 Conceptos fundamentales en PostgreSQL
Roles (sustituyen a usuarios)

En PostgreSQL NO existe un comando CREATE USER real. Internamente todo es un rol, con o sin capacidad de login.

CREATE ROLE pepe LOGIN PASSWORD '1234';


Los roles pueden:

Tener contraseña.

Heredar permisos de otros roles.

Ser usados como grupos de permisos.

Privilegios

Sobre objetos:

SELECT, INSERT, UPDATE, DELETE

REFERENCES

TRIGGER

Sobre el sistema:

CREATEDB

CREATEROLE

SUPERUSER

LOGIN

Ejemplo
CREATE ROLE solo_lectura;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO solo_lectura;
GRANT solo_lectura TO alumno1;

2. Gestión de Roles, Permisos y Seguridad
2.1 Crear usuarios (roles con login)
CREATE ROLE analista LOGIN PASSWORD 'Analista2025';

2.2 Modificar un rol
ALTER ROLE analista PASSWORD 'NuevaPass2025';
ALTER ROLE analista NOLOGIN;

2.3 Eliminar rol
DROP ROLE analista;

2.4 Conceder privilegios sobre tablas
Conceder
GRANT SELECT ON customer TO analista;
GRANT SELECT, INSERT ON invoice TO analista;

Revocar
REVOKE INSERT ON invoice FROM analista;

Conceder permisos en todas las tablas del esquema
GRANT SELECT ON ALL TABLES IN SCHEMA public TO analista;

2.5 Roles como grupos
CREATE ROLE marketing;
GRANT SELECT ON customer TO marketing;
GRANT marketing TO analista01;

2.6 Permisos de creación
GRANT CREATE ON DATABASE chinook TO alumno1;
GRANT USAGE, CREATE ON SCHEMA public TO alumno1;

3. Políticas de Seguridad en PostgreSQL (equivalente a perfiles Oracle)

PostgreSQL no tiene perfiles como Oracle, pero puedes configurar restricciones:

Limitar número de conexiones
ALTER ROLE etl_user CONNECTION LIMIT 1;

Expulsar por inactividad (a nivel de servidor)

Se recomienda vía postgresql.conf:

idle_in_transaction_session_timeout = 30000   # 30s

Política de contraseñas

Puedes usar:

PASSWORD

VALID UNTIL

Ejemplo:

ALTER ROLE etl_user VALID UNTIL '2025-12-31';

Bloqueo manual del usuario
ALTER ROLE etl_user NOLOGIN;

4. Auditoría y monitorización

PostgreSQL no incluye auditoría fina por defecto (como Oracle FGA).

Tiene:

Log de sentencias

En postgresql.conf:

log_statement = 'all'


Consultar logs.

Ver roles y permisos
SELECT * FROM pg_roles;
SELECT * FROM information_schema.role_table_grants;
SELECT * FROM information_schema.table_privileges;

Mini proyecto guiado

Crear un rol de solo lectura para Marketing en PostgreSQL

Crear rol:

CREATE ROLE marketing;


Conceder lectura:

GRANT SELECT ON customer TO marketing;
GRANT SELECT ON invoice TO marketing;


Crear usuario:

CREATE ROLE analista01 LOGIN PASSWORD 'Marketing2025!';


Asignarle el rol:

GRANT marketing TO analista01;


Probar permisos:

SELECT * FROM customer LIMIT 5;     -- OK
DELETE FROM customer;               -- ERROR
