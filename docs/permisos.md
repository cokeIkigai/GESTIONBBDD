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

Ejercicio:

Crea en PostgreSQL un conjunto completo de roles que representen la estructura de una empresa tecnológica. Debes crear roles sin LOGIN para utilizarlos como grupos de permisos, diferenciando claramente entre los roles que solo leen datos, los que leen y escriben, los que pueden modificar estructuras, y los roles administrativos. Los roles que debes crear son los siguientes: lectura_basica, lectura_informes, lectura_escritura, edicion_completa, admin_datos, dev_junior, dev_senior, dev_lider, marketing, sistemas, becarios, proyectos, soporte, auditoria y direccion. Cada uno debe tener un propósito específico.

El rol lectura_basica debe poder leer únicamente las tablas más simples del esquema public, como clientes y productos. El rol lectura_informes también tiene permisos de lectura, pero incluye además tablas como ventas, ingresos, pagos y cualquier tabla relacionada con informes. El rol lectura_escritura debe poder leer y escribir en tablas operativas, como proyectos, tareas, empleados. El rol edicion_completa debe incluir lectura, escritura y actualización pero no debe tener permisos para borrar tablas.

El rol admin_datos debe ser el más potente dentro de los roles no administrativos del sistema: debe poder crear, modificar y borrar tablas dentro del esquema public, pero sin tener privilegios sobre toda la base de datos. El rol dev_junior debe tener permisos mínimos: solo lectura en proyectos, tareas y empleados. El rol dev_senior debe tener permisos de lectura y escritura en esas mismas tablas, y además poder crear y modificar sus propias tablas de pruebas. El rol dev_lider debe poder hacer todo lo que hacen los anteriores y además gestionar los permisos de otros desarrolladores del área.

El rol marketing debe tener permiso exclusivo de lectura sobre clientes, productos y ventas. El rol sistemas debe poder administrar objetos del esquema, crear tablas, borrar tablas y gestionar usuarios, pero sin tener acceso a leer información comercial como ventas o clientes. El rol becarios debe tener permisos muy reducidos: solo lectura de clientes y ninguna acción sobre otras tablas. El rol proyectos debe agrupar permisos de varios departamentos para que todos los roles que pertenezcan a él puedan consultar y actualizar tareas del área de gestión de proyectos.

El rol soporte debe poder leer y actualizar datos en tickets, incidencias y usuarios. El rol auditoria debe tener lectura total sobre todas las tablas de la base de datos, pero ningún permiso de escritura. El rol direccion debe tener capacidad para leer cualquier tabla, gestionar permisos de otros roles y conectarse con prioridad a la base de datos.

Crea además varios usuarios reales: ana_junior, carlos_senior, laura_lider, maria_marketing, juan_sistemas, luis_becario, sofia_soporte, roberto_auditor, ceo_empresa. Asigna a cada usuario el rol que le corresponda con una relación 1:1, pero también añádeles roles secundarios: por ejemplo, laura_lider también debe pertenecer al rol proyectos, juan_sistemas debe pertenecer también al rol admin_datos, y luis_becario debe pertenecer también a lectura_basica.

Define permisos para cada rol según las características descritas anteriormente. Además crea varias reglas adicionales: todas las tablas nuevas del esquema public deben dar permisos automáticos de lectura a auditoria y permisos de lectura y escritura a dev_senior. También debes permitir que el rol marketing pueda conectarse a la base de datos, pero no crear objetos ni modificar estructuras. El rol sistemas debe poder conectarse y además debe tener un límite de conexión máxima de 2 sesiones simultáneas. El rol ceo_empresa debe poder conectarse en cualquier momento aunque los demás usuarios estén limitados.

Después realiza las siguientes preguntas dentro del propio ejercicio:
– Explica la diferencia entre un rol que solo tiene SELECT y un rol que tiene SELECT, INSERT y UPDATE.
– Indica por qué es útil tener roles como lectura_basica y lectura_informes separados.
– Indica cómo funcionaría la herencia de permisos si dev_junior y dev_senior pertenecen al rol proyectos.
– Explica quién podría borrar tablas y quién no, según los roles creados.
– Indica qué pasaría si un usuario pertenece tanto a un rol que tiene SELECT y a otro que tiene REVOKE SELECT.
– Explica cómo cambiarías la contraseña a uno de los usuarios y cómo la bloquearías o reactivarías.
– ¿Por qué auditoria no debe tener permisos de escritura?
– ¿Qué ventaja tiene tener un rol como proyectos que agrupa permisos de varios perfiles?

Si lo quieres, te genero también la solución completa, un PDF, o una versión más corta.
