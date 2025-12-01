# 🔍 UT 5: Acceso a la Información. Vistas

# Introducción

- Una **vista** es una consulta almacenada que se presenta como una tabla virtual. 
- No contiene datos propios, sino que permite acceder a los datos reales de otras tablas mediante una consulta predefinida. 
- Las vistas son una herramienta clave para **simplificar el acceso a la información** y para **aplicar políticas de seguridad**.

### ✨ 1. Introducción a las Vistas SQL

####⚡ ¿Qué es una vista en SQL?

Una **vista** es una **representación virtual de una tabla**, generada a partir del resultado de una consulta SQL. 
No almacena los datos, sino que actúa como una forma estructurada de acceder a ellos.

🔔 **Importante:**

- Permite encapsular lógica de consulta.
- Se comporta como una tabla desde el punto de vista del usuario.
- Cada vez que se accede a la vista, se ejecuta la consulta que la define.

 ####⚡ Características principales de las vistas

- **No almacenan datos**, solo definen cómo mostrarlos.
- **Simplifican consultas** complejas.
- **Ocultan columnas sensibles**, limitando el acceso. (columna del ID y otra privacidad)
- **Aportan independencia lógica**, facilitando cambios internos en las tablas.

Las vistas pueden ser tan simples como una selección de columnas, o tan complejas como agregaciones y uniones entre varias tablas.

####⚡  Ventajas del uso de vistas en bases de datos

- **Seguridad:** permiten mostrar solo la información necesaria.
- **Reutilización:** evitan repetir consultas complejas.
- **Simplificación:** facilitan el trabajo a usuarios que no conocen la estructura completa.
- **Independencia lógica**: si cambian las tablas base, la vista puede mantenerse con el mismo nombre.

####⚡  Sintaxis básica para crear vistas (CREATE VIEW)

La sintaxis para crear una vista básica en SQL es:

```sql
CREATE VIEW nombre_vista AS
SELECT columnas
FROM tabla
WHERE condiciones;
```

**Ejemplo**:

```sql
CREATE VIEW empleados_ventas AS
SELECT nombre, apellido, departamento
FROM empleados
WHERE departamento = 'Ventas';
```

Esta vista mostrará solo los empleados que trabajan en el departamento de ventas, ocultando el resto de información.

####⚡Ejemplo de vista simple para filtrar columnas

Si se desea mostrar únicamente el nombre y la ciudad de los clientes:

```sql
CREATE VIEW vista_clientes_ciudad AS
SELECT nombre, ciudad
FROM clientes;
```

Esta vista se podrá consultar como si fuera una tabla:

```sql
SELECT * FROM vista_clientes_ciudad;
```

Con esto, los usuarios podrán acceder a los datos relevantes sin conocer la estructura completa de la tabla `clientes`.

# ✨ 2. Tipos de Vistas y Cláusulas Especiales

####⚡ Vistas simples: definición y operaciones permitidas

Una **vista simple** se basa en una sola tabla y no contiene funciones de agregación ni operaciones complejas como `JOIN`, `DISTINCT` o `GROUP BY`.

**Ventajas:**

- Permite realizar operaciones DML (`INSERT`, `UPDATE`, `DELETE`) si no se violan reglas de integridad.

**Ejemplo:**

```sql
CREATE VIEW empleados_activos AS
SELECT id_empleado, nombre, puesto
FROM empleados
WHERE estado = 'ACTIVO';
```
--- 
####⚡ Vistas complejas: restricciones y consideraciones

Una **vista compleja** se define utilizando más de una tabla o incluye funciones de agregación (`SUM`, `AVG`, etc.), cláusulas `GROUP BY`, `HAVING` o `JOIN`.

**Limitaciones:**

- En general, **no permite operaciones DML** directamente.

**Ejemplo:**

```sql
CREATE VIEW ventas_por_departamento AS
SELECT departamento, SUM(total_venta) AS total
FROM ventas
GROUP BY departamento;
```

####⚡ Inserciones, actualizaciones y eliminaciones a través de vistas

| Tipo de Vista | `INSERT` | `UPDATE` | `DELETE` |
| --- | --- | --- | --- |
| Vista simple | ✅ | ✅ | ✅ |
| Vista con `JOIN` | ⚠️ Parcial | ⚠️ Parcial | ⚠️ Parcial |
| Vista con agregados | ❌ | ❌ | ❌ |
- Las vistas simples permiten DML si existe correspondencia directa con la tabla base.
- Las vistas complejas pueden restringir o impedir estas operaciones.

####⚡ Cláusula `WITH CHECK OPTION`

Evita que las operaciones DML provoquen que los datos queden fuera de las condiciones definidas por la vista.

**Ejemplo:**

```sql
CREATE VIEW empleados_rrhh AS
SELECT * FROM empleados
WHERE departamento = 'RRHH'
WITH CHECK OPTION;
```

Esta opción impide cambiar el departamento de un empleado desde esta vista, si el nuevo valor no cumple la condición.

####⚡Cláusula `WITH READ ONLY`

Restringe completamente las operaciones de modificación a través de la vista. Es útil en vistas con agregaciones o uniones complejas.

**Ejemplo:**

```sql
CREATE VIEW resumen_ventas AS
SELECT departamento, SUM(total)
FROM ventas
GROUP BY departamento
WITH READ ONLY;
```

####⚡Modificación de vistas con `CREATE OR REPLACE VIEW`

Permite redefinir una vista sin necesidad de eliminarla previamente. Los permisos otorgados se conservan.

**Ejemplo:**

```sql
CREATE OR REPLACE VIEW vista_empleados AS
SELECT id, nombre, salario
FROM empleados;

-- Eliminación de vistas con `DROP VIEW`
DROP VIEW nombre_vista;
```

**Precaución:** si existen otras vistas o permisos que dependen de esta vista, pueden generarse errores. Es recomendable consultar `DBA_DEPENDENCIES` antes de su eliminación.

# ✨ 3. Administración de Vistas desde el Diccionario de Datos

####⚡Consultas sobre vistas con `USER_VIEWS`, `ALL_VIEWS` y `DBA_VIEWS`

Oracle proporciona varias vistas del diccionario de datos que permiten consultar la información sobre las vistas creadas en el sistema:

- **`USER_VIEWS`**: Muestra las vistas creadas por el usuario actual.
- **`ALL_VIEWS`**: Muestra las vistas accesibles para el usuario actual, incluyendo las propias y las de otros usuarios con permiso.
- **`DBA_VIEWS`**: Muestra todas las vistas del sistema. Solo accesible para usuarios con privilegios de administrador.

**Ejemplo:**

```sql
SELECT view_name, text
FROM user_views;
```

Este comando permite consultar el nombre de las vistas y el SQL que las define.

####⚡Visualización del SQL original de una vista

El código SQL que define una vista se puede recuperar desde la columna `TEXT` de las vistas del diccionario mencionadas.

**Ejemplo:**

```sql
SELECT view_name, text
FROM user_views
WHERE view_name = 'EMPLEADOS_ACTIVOS';
```

Esto permite revisar y verificar la definición exacta de cualquier vista creada.

####⚡Recomendaciones para la gestión de vistas complejas

**Buenas prácticas:**

- Utilizar nombres descriptivos y coherentes para facilitar la comprensión.
- Documentar el propósito de cada vista.
- Evitar dependencias en cascada (vistas basadas en otras vistas).
- Usar cláusulas como `WITH CHECK OPTION` o `WITH READ ONLY` cuando sea necesario para garantizar la coherencia de los datos y limitar acciones no deseadas.
- Revisar regularmente las vistas no utilizadas para mantener la base de datos organizada y eficiente.

####⚡Ejemplo completo: consulta consolidada mediante vista

**Escenario:** Se requiere mostrar el total de ventas por departamento.

**Vista:**

```sql
CREATE VIEW resumen_ventas_departamento AS
SELECT d.nombre_departamento, SUM(v.total_venta) AS total_ventas
FROM departamentos d
JOIN ventas v ON d.id_departamento = v.id_departamento
GROUP BY d.nombre_departamento;
```

**Consulta sobre la vista:**

```sql
SELECT * FROM resumen_ventas_departamento;
```

Esta vista permite a los usuarios consultar los totales consolidados sin necesidad de conocer la estructura interna de las tablas implicadas.

####⚡Ventajas estratégicas en la administración con vistas

- **Seguridad**: Controlan el acceso a datos sensibles mediante la ocultación de columnas o registros.
- **Simplificación**: Permiten ofrecer al usuario una vista lógica más sencilla de los datos.
- **Modularidad**: Facilitan el mantenimiento dividiendo el acceso a datos en capas lógicas.
- **Compatibilidad**: Permiten cambios internos en la estructura de las tablas sin afectar a las consultas de los usuarios.

Las vistas son herramientas clave para la gestión eficiente, segura y escalable de los datos en entornos empresariales.

# 4. Vistas Materializadas: Rendimiento y Persistencia

####⚡¿Qué es una vista materializada y en qué se diferencia de una vista normal?

Una **vista materializada** es una estructura de base de datos que **almacena físicamente los resultados** de una consulta. A diferencia de una vista normal, que ejecuta la consulta cada vez que se accede a ella, la vista materializada conserva los datos en disco.

Esto permite acceder rápidamente a resultados complejos o pesados sin necesidad de recalcular la consulta cada vez.

**Diferencias principales:**

| Característica | Vista normal | Vista materializada |
| --- | --- | --- |
| Almacena datos físicamente | ❌ No | ✅ Sí |
| Tiempo de actualización | En tiempo real | Bajo demanda o automático |
| Rendimiento en consultas | Medio | Alto |

####⚡Sintaxis básica: `CREATE MATERIALIZED VIEW`

La sintaxis general para crear una vista materializada en Oracle es:

```sql
CREATE MATERIALIZED VIEW nombre_vista
BUILD IMMEDIATE
REFRESH [FAST | COMPLETE | FORCE]
ON [COMMIT | DEMAND]
AS
SELECT ...;
```

**Ejemplo:**

```sql
CREATE MATERIALIZED VIEW resumen_ventas_mat
BUILD IMMEDIATE
REFRESH COMPLETE
ON DEMAND
AS
SELECT departamento, SUM(total)
FROM ventas
GROUP BY departamento;
```

####⚡Mecanismos de actualización (`REFRESH`)

Las vistas materializadas deben sincronizarse periódicamente con los datos reales. Oracle permite varios mecanismos:

- **`FAST`**: Aplica solo los cambios desde el último refresco. Requiere configuración adicional.
- **`COMPLETE`**: Reejecuta toda la consulta y reemplaza los datos.
- **`FORCE`**: Intenta un `FAST`; si no es posible, realiza un `COMPLETE`.

####⚡Tipos de refresco: `ON COMMIT` vs `ON DEMAND`

- **`ON COMMIT`**: La vista se actualiza automáticamente cada vez que se hace `COMMIT` en la tabla base. Es útil si se requiere una sincronización constante.
- **`ON DEMAND`**: La vista solo se actualiza cuando se ejecuta manualmente el comando:

```sql
EXEC DBMS_MVIEW.REFRESH('nombre_vista');
```

Este método es más eficiente para cargas masivas o informes que no necesitan información en tiempo real.

####⚡Escenarios ideales para vistas materializadas

- Informes sobre **grandes volúmenes de datos** con agregaciones frecuentes.
- Consultas **pesadas** que impactan en el rendimiento si se ejecutan constantemente.
- Necesidad de separar la carga de trabajo entre **sistemas de producción y de análisis**.
- **Dashboards** de dirección o áreas de BI donde la información no cambia constantemente.

####⚡Comparativa final entre vistas normales y materializadas

| Característica | Vista normal | Vista materializada |
| --- | --- | --- |
| Almacenamiento físico | ❌ No | ✅ Sí |
| Tiempo de actualización | Tiempo real | Bajo demanda o al commit |
| Rendimiento en consultas | Medio (según caso) | Alto (preprocesado) |
| Soporte de DML directo | ✅ A veces | ❌ No (solo lectura) |
| Ideal para... | Acceso dinámico | Consultas pesadas y BI |

Con esta comparativa se cierra la explicación sobre vistas, destacando su rol en el acceso eficiente y seguro a la información en bases de datos empresariales.
