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
ALTER ROLE pepe NOLOGIN; (bloquear usuario)

-- Restablecer login
ALTER ROLE alumno1 LOGIN;

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
REVOKE ALL PRIVILEGES ON  FROM lectura_escritura;

```

### 3. Roles como "grupos" de permisos

- La mejor práctica es `crear roles` sin login que agrupen permisos.
- Luego `asignar` esos roles a los usuarios.
  
```sql
-- Crear un rol de grupo
CREATE ROLE marketing;

-- Dar permisos al rol
GRANT SELECT ON customer TO marketing;
GRANT SELECT ON invoice TO marketing;

-- Asignar el rol a un usuario
GRANT marketing TO empleado1;
-- Dar SELECT a todas las tablas del esquema public
GRANT SELECT ON ALL TABLES IN SCHEMA public TO lectura_escritura;
```

Ahora empleado1 hereda todos los permisos de marketing.
Con esto estamos asociadno roles a usuarios que ya tenemos creados previamente y no es necesario ir dando permisos de uno en uno.

--- 

### 4. Dar permisos futuros automáticamente
Para que las nuevas tablas también tengan permisos:

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT ON TABLES TO solo_lectura;
```

- `ALTER DEFAULT PRIVILEGES` → Cambia los permisos que tendrán por defecto los objetos nuevos que se creen.
- `IN SCHEMA public` → Indica que se aplicará a las tablas nuevas creadas dentro del esquema public.
- `GRANT SELECT` → Permiso que tendrán automáticamente las nuevas tablas: lectura.
- `ON TABLES` → Se aplica específicamente a tablas recién creadas.
- `TO lectura` → Rol o usuario que recibirá esos permisos por defecto.
---

### 5. Permisos sobre bases de datos y esquemas

Permitir crear objetos en un esquema
- Da permiso para `crear` objetos (tablas, vistas, funciones…).
```sql   
GRANT CREATE ON SCHEMA public TO alumno1;
-- GRANT CREATE → Da permiso para crear objetos (tablas, vistas, funciones…).
-- ON SCHEMA public → Ese permiso se aplica dentro del esquema public.
-- TO empleado1 → El usuario empleado1 podrá crear objetos allí.
```
Permitir conectarse a una base de datos
```sql
GRANT CONNECT ON DATABASE chinook TO empleado1;

-- GRANT CONNECT → Autoriza a conectarse a esa base de datos.
-- ON DATABASE chinook → Afecta a la base de datos chinook.
-- TO empleado1 → Ese usuario puede entrar a la base de datos, pero no hacer nada más si no tiene otros permisos.
```
---

### 6. Limitar número de conexiones por usuario

```sql
ALTER ROLE empleado1 CONNECTION LIMIT 1;
-- ALTER ROLE empleado1 → Modifica las propiedades del rol empleado1.
-- CONNECTION LIMIT 1 → Ese usuario solo puede tener una conexión activa al mismo tiempo.
```
### 7. Fecha de caducidad de la contraseña
```sql
ALTER ROLE empleado VALID UNTIL '2026-01-01';
-- VALID UNTIL '2026-01-01' → Indica que la contraseña dejará de ser válida en esa fecha.
-- A partir de ese día el usuario no podrá iniciar sesión hasta que un administrador le cambie o renueve la contraseña.
```
Ejercicio: Diseño de Roles, Usuarios y Permisos en PostgreSQL

Tu objetivo es construir la arquitectura de seguridad completa de TechNova Corp. siguiendo estas indicaciones:

1. Crea las tablas del sistema

La empresa maneja diferentes tipos de datos. Imagina que existen las siguientes tablas en el esquema public:

Tablas comerciales:

clientes

productos

ventas

ingresos

pagos

Tablas operativas:

proyectos

tareas

empleados

tickets

incidencias

usuarios_sistema

Tablas internas para desarrolladores:

dev_pruebas_junior

dev_pruebas_senior

Tablas estructurales:

logs_sistema

configuracion

metricas

No necesitas crear las tablas con columnas, solo el listado para poder asignar permisos.

2. Crea los roles base (sin LOGIN)

Debes crear roles que representen permisos específicos, separados por áreas y tipos de acceso. Los roles necesarios son:

lectura_basica

lectura_informes

lectura_escritura

edicion_completa

admin_datos

dev_junior

dev_senior

dev_lider

marketing

sistemas

becarios

proyectos

soporte

auditoria

direccion

Cada rol debe tener una misión clara dentro de TechNova.

3. Define el propósito de cada rol

Asigna, en el propio ejercicio, para qué sirve cada uno. Por ejemplo:

lectura_basica: acceso mínimo, solo lectura a clientes y productos.

lectura_informes: lectura avanzada (ventas, ingresos, pagos…).

lectura_escritura: operaciones CRUD básicas en proyectos, tareas y empleados.

edicion_completa: puede insertar, actualizar y borrar filas, pero no puede borrar tablas.

admin_datos: puede crear, alterar y borrar tablas del esquema public, pero no administrar la base al completo.

dev_junior: solo lectura en proyectos, tareas y empleados.

dev_senior: lectura + escritura en esas tablas y creación de sus propias tablas.

dev_lider: controla permisos sobre los desarrolladores.

marketing: lectura de clientes, productos y ventas.

sistemas: crear/borrar tablas, administrar roles, pero sin acceder a datos comerciales.

becarios: permisos mínimos, solo lectura en clientes.

proyectos: acceso común para consultar y modificar tareas y proyectos.

soporte: lectura + actualización en tickets, incidencias y usuarios.

auditoria: solo lectura absoluta sobre todas las tablas.

direccion: lectura total y gestión de permisos global.

4. Crea los usuarios reales

Crea los usuarios:

ana_junior

carlos_senior

laura_lider

maria_marketing

juan_sistemas

luis_becario

sofia_soporte

roberto_auditor

ceo_empresa

Cada usuario debe heredar el rol principal apropiado. Además:

laura_lider → también pertenece a proyectos.

juan_sistemas → también pertenece a admin_datos.

luis_becario → también pertenece a lectura_basica.

5. Reglas de seguridad adicionales

Debes definir dentro del ejercicio:

Todas las tablas nuevas del esquema public deben otorgar automáticamente:

SELECT a auditoria

SELECT y INSERT/UPDATE a dev_senior

El rol marketing debe poder conectarse a la BD, pero no crear objetos.

El rol sistemas debe poder conectarse y tener un límite máximo de 2 conexiones.

El rol direccion y el usuario ceo_empresa pueden conectarse siempre, sin restricciones de conexión.

6. Preguntas de razonamiento obligatorio

Dentro del mismo ejercicio, debes responder:

Diferencia entre un rol que solo tiene SELECT y uno que tiene SELECT, INSERT y UPDATE.

Por qué es útil separar lectura_basica y lectura_informes.

Cómo funciona la herencia de permisos si dev_junior y dev_senior pertenecen al rol proyectos.

Quién puede borrar tablas y quién no.

Qué ocurre si un usuario pertenece a un rol con SELECT y a otro que tiene REVOKE SELECT.

Cómo cambiar la contraseña de un usuario, bloquearlo o reactivarlo.

Por qué auditoria no debe tener permisos de escritura.

Qué ventaja tiene un rol como proyectos para agrupar permisos entre varios perfiles.


