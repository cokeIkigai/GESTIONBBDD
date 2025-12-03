##  🔒 Gestión de Usuarios y Permisos en PostgreSQL

### 1. ¿Qué es un ROL en PostgreSQL?

- En PostgreSQL, un rol es una entidad que puede representar:
- Un usuario real (si tiene LOGIN).
- Un grupo de permisos (si NO tiene LOGIN).
- Normalmente se le asocia a unos ciertos permisos.

CREAR/BORRAR
```sql
 -- Crear rol sin login	
CREATE ROLE Sergio;
 -- Crear rol sin login	
CREATE ROLE lectura;
CREATE ROLE lectura_escritura;

-- Crear usuario real	
CREATE ROLE pepe LOGIN PASSWORD '1234';
-- Borrar rol	
DROP ROLE nombre;
-- Forzar borrado	
DROP OWNED BY nombre; DROP ROLE nombre;
```
MODIFICAR
```sql
-- Renombrar rol	
ALTER ROLE viejo RENAME TO nuevo;
-- Cambiar contraseña	
ALTER ROLE pepe PASSWORD '7890';
-- Bloquear login	
ALTER ROLE pepe NOLOGIN;
-- Ver todos los roles que existen	
SELECT rolname FROM pg_roles;
```

### 2. ¿Qué es un privilegio?

Un privilegio es una `autorización` que permite a un rol realizar  `acciones específicas` sobre objetos de la base de datos.
Define qué puede hacer un usuario: 

`leer`, `insertar`, `modificar` o `borrar` datos.
- Sin privilegios asignados, un rol no puede operar sobre esos objetos.

Recordamos: 

```sql
SELECT → Leer datos
INSERT → Insertar filas
UPDATE → Modificar
DELETE → Borrar
REFERENCES → Crear claves externas
TRIGGER → Crear triggers
```

#### Los roles y los privilegios permiten:

- `Proteger` datos sensibles
- `Organizar` qué puede hacer cada usuario
- Crear `estructuras de seguridad` fáciles de mantener
- `Delegar` permisos según departamentos (marketing, ventas, administración…)
- `Controlar` qué usuarios pueden conectarse o modificar información
---

#### Dar permiso de lectura sobre una tabla
```sql
-- Permiso [SELECT/INSERT/UPDATE/DELETE] ON <TABLA> TO <ROL>
GRANT SELECT ON customer TO lectura;

Ejemplo: dar permisos de lectura y escritura
GRANT SELECT, INSERT, UPDATE ON invoice TO lectura_escritura;

GRANT ALL PRIVILEGES ON track TO admin;
```
#### Quitar permisos (REVOKE)

Sirve para eliminar permisos que antes se concedieron.
```sql
-- quitar UPDATE
REVOKE UPDATE ON customer FROM lectura;
-- quitar todo
REVOKE ALL PRIVILEGES ON track FROM lectura_escritura;
```

#### 3. Roles como "grupos" de permisos

- La mejor práctica es `crear role`s sin login que agrupen permisos.
- Luego asignar esos roles a los usuarios.
  
```sql
-- Crear un rol de grupo
CREATE ROLE marketing;

-- Dar permisos al rol
GRANT SELECT ON customer TO marketing;
GRANT SELECT ON invoice TO marketing;

-- Asignar el rol a un usuario
GRANT marketing TO alumno1;
```

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
