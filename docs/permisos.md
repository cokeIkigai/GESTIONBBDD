##  🔒 Gestión de Usuarios y Permisos en PostgreSQL


#### 1. ¿Qué es un ROL en PostgreSQL?

- En PostgreSQL, un rol es una entidad que puede representar:

- Un usuario real (si tiene LOGIN).

- Un grupo de permisos (si NO tiene LOGIN).
  
## Rol simple (sin login)
```sql
 -- Crear rol sin login	
CREATE ROLE nombre;
-- Crear usuario real	
CREATE ROLE pepe LOGIN PASSWORD '1234';
-- Borrar rol	
DROP ROLE nombre;
-- Forzar borrado	
DROP OWNED BY nombre; DROP ROLE nombre;
-- Renombrar rol	
ALTER ROLE viejo RENAME TO nuevo;
-- Cambiar contraseña	
ALTER ROLE pepe PASSWORD '7890';
-- Bloquear login	
ALTER ROLE pepe NOLOGIN;
-- Ver roles	
SELECT rolname FROM pg_roles;
``` 
CREATE ROLE pepe LOGIN PASSWORD '1234';
Crear un rol que actúa como usuario (con login)

CREATE ROLE alumno1 LOGIN PASSWORD '1234';

Crear un rol que es un grupo de permisos
CREATE ROLE ventas;

1. ¿Qué son los PRIVILEGIOS?

Los privilegios determinan qué puede hacer un rol dentro de la base de datos.
Los más comunes sobre tablas son:

SELECT → Leer datos

INSERT → Insertar filas

UPDATE → Modificar

DELETE → Borrar

REFERENCES → Crear claves externas

TRIGGER → Crear triggers

Ejemplo: dar permiso de lectura sobre una tabla
GRANT SELECT ON customer TO solo_lectura;

Ejemplo: dar permisos de lectura y escritura
GRANT SELECT, INSERT, UPDATE ON invoice TO alumno1;

Ejemplo: dar todos los permisos sobre una tabla
GRANT ALL PRIVILEGES ON track TO alumno1;

3. Quitar permisos (REVOKE)

Sirve para eliminar permisos que antes se concedieron.

Ejemplo: quitar UPDATE
REVOKE UPDATE ON customer FROM alumno1;

Ejemplo: quitar todo
REVOKE ALL PRIVILEGES ON track FROM alumno1;

4. Roles como "grupos" de permisos

La mejor práctica es crear roles sin login que agrupen permisos.
Luego asignar esos roles a los usuarios.

4.1 Crear un rol de grupo
CREATE ROLE marketing;

4.2 Dar permisos al rol
GRANT SELECT ON customer TO marketing;
GRANT SELECT ON invoice TO marketing;

4.3 Asignar el rol a un usuario
GRANT marketing TO alumno1;


Ahora alumno1 hereda todos los permisos de marketing.

5. Permisos en todas las tablas del esquema

PostgreSQL permite aplicar permisos en lote.

Dar SELECT a todas las tablas del esquema public
GRANT SELECT ON ALL TABLES IN SCHEMA public TO solo_lectura;

Dar permisos futuros automáticamente

Para que las nuevas tablas también tengan permisos:

ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT ON TABLES TO solo_lectura;

6. Permisos sobre bases de datos y esquemas
Permitir crear objetos en un esquema
GRANT CREATE ON SCHEMA public TO alumno1;

Permitir conectarse a una base de datos
GRANT CONNECT ON DATABASE chinook TO alumno1;

7. Modificar roles
Cambiar contraseña
ALTER ROLE alumno1 PASSWORD 'Nueva1234';

Quitar capacidad de login (bloquear usuario)
ALTER ROLE alumno1 NOLOGIN;

Restablecer login
ALTER ROLE alumno1 LOGIN;

8. Limitar número de conexiones

Similar a “sesiones por usuario”.

ALTER ROLE etl_user CONNECTION LIMIT 1;

9. Establecer fecha de caducidad para contraseña
ALTER ROLE etl_user VALID UNTIL '2026-01-01';

10. Eliminar roles o usuarios
Borrar usuario sin dependencias
DROP ROLE alumno1;

Borrar usuario con objetos creados (requiere transferir o borrar antes)
DROP OWNED BY alumno1;
DROP ROLE alumno1;
