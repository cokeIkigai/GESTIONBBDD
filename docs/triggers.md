# Triggers en PostgreSQL

## 📌 1. ¿Qué es un Trigger? 

Un trigger es un bloque de código que se ejecuta automáticamente cuando ocurre un evento en la base de datos.

No lo llama el programador: lo llama el propio motor de BD.

Actúa como un disparador que se activa cuando pasa algo concreto.

## ⏱️ 2. ¿Cuándo se dispara un trigger?

- INSERT → cuando se inserta una fila
- UPDATE → cuando se actualiza una fila
- DELETE → cuando se elimina una fila
- TRUNCATE → en PostgreSQL solo a nivel de sentencia
- INSTEAD OF INSERT/UPDATE/DELETE → en vistas

Cada vez que ocurre cualquiera de estos eventos, el trigger “salta”.

## 🔵 3. ¿Qué ventajas tiene?

1. Auditoría
  - Quién hizo la operación.
  - Cuándo la hizo.
  - Qué valores cambió (OLD vs NEW).
  - Guardar histórico.

2. Validación y seguridad
  - Proteger la BD incluso si el backend falla.
  - Impedir datos inválidos.
  - Bloquear inserciones o modificaciones peligrosas.
  - Evitar borrados críticos.

## 🔸 4.  Diferencia BEFORE (filtro) vs AFTER (post-proceso).

  - **BEFORE**: Se ejecuta antes de que la operación se realice en la tabla (filtro). Decide si permitir o no la operación.
    
    - Validación de datos (filtro).
    - Modificar valores antes de guardarlos.
    - Bloquear un INSERT/UPDATE con RETURN NULL.
      
  - **AFTER**: Se ejecuta después de que la operación ya se ha realizado. Aquí no puedes impedir la operación, ya ocurrió.
    
    - Actualizaciones en otras tablas.
    - Estadísticas.
    - Replicación interna.
   
*-Se suelen combinar ambas-*

## Ejemplo

#### Trigger BEFORE → validar

```sql
CREATE OR REPLACE FUNCTION validar_precio()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.unitprice < 0 THEN
        RETURN NULL; -- BLOQUEA el insert
    END IF;

    RETURN NEW; -- Lo permite
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_validar_precio
BEFORE INSERT ON track
FOR EACH ROW
EXECUTE FUNCTION validar_precio();
```

#### Trigger AFTER → auditar

```sql
CREATE OR REPLACE FUNCTION auditar_insert_track()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO trackaudit (trackid, name, fecha)
    VALUES (NEW.trackid, NEW.name, CURRENT_TIMESTAMP);
    RETURN NULL; 
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_auditar_insert
AFTER INSERT ON track
FOR EACH ROW
EXECUTE FUNCTION auditar_insert_track();
```

### Se ejecuta con:

```sql
INSERT INTO track (...) VALUES (...);
```
---
## 🧠 Estructura de un trigger en PostgreSQL
- Para crear un trigger en **PostgreSQL** es necesario:
  - Crear una **función** que devuelva tipo TRIGGER
  - Crear el **trigger** que llama a esa función
    
#### 1.  Crear función:

``` sql
--Crea o remplaza la función existente <Nombre de la función>
CREATE OR REPLACE FUNCTION auditar_insert_track()
-- la función es para un trigger. empieza el código del trigger.
RETURNS TRIGGER AS $$
-- Empieza el bloque de la lógica
BEGIN
     IF NEW.unitprice < 0 THEN
     -- Anulas el insert/update (muy usado en BEFORE).
        RETURN NULL; 
    END IF;
    -- Permite que la operación continúe.
    RETURN NEW;
--Termina el bloque de la lógica
END;
-- Termina el delimitador. Es el lenguaje utilizado.
$$ LANGUAGE plpgsql;
```

#### 2. Crear trigger:

``` sql
-- Crea el trigger y se nombra
CREATE TRIGGER nombre_trigger
-- se le dice cuando AFTER/BEFORE y que ejecución INSERT/UPDATE/DELETE
AFTER INSERT ON tabla
-- Se ajecuta a nivel de línea (Acceso a NEW y OLD)
FOR EACH ROW
-- Ejecuta la función creada para ese trigger
EXECUTE FUNCTION ejemplo();
```
🎯 **FOR EACH ROW VS FOR EACH STATEMENT** 
- Si se hace un update que afecte a varias filas la respuesta es distinta para:
  -  **For each row**: Lanza el trigger por cada fila afectada.
  -  **For each statement**: Lo lanza un sola vez aunque haya muchas filas afectadas.
---


### 🧱 ¿Qué son NEW y OLD?

**1.** Son registros especiales (pseudo-tablas) que PostgreSQL te da dentro del trigger. 

**2.** Funcionan como “fotografías” de la fila antes y después del cambio. Y puedes acceder si se pone en la creación del trigger FOR EACH ROW.

**3.** NEW y OLD pertenecen SIEMPRE a la tabla donde ocurre el trigger.

Se va a crear un trigger donde se lance cuando se haga un update de un cambio en un campo pero se mantiene en el resto. Pero

TABLA Track

```sql
CREATE TABLE Track (
    TrackId       SERIAL PRIMARY KEY,
    Name          VARCHAR(200) NOT NULL,
    AlbumId       INT,
    MediaTypeId   INT,
    GenreId       INT,
    Composer      VARCHAR(220),
    Milliseconds  INT,
    Bytes         INT,
    UnitPrice     NUMERIC(10,2),
);
```
TABLA TrackAudit (registro)

```sql
CREATE TABLE TrackAudit (
    AuditId         SERIAL PRIMARY KEY,
    TrackId         INT,
    Name            VARCHAR(200),
    Accion          VARCHAR(50),     -- INSERT / UPDATE / DELETE
    OldUnitPrice    NUMERIC(10,2),
    NewUnitPrice    NUMERIC(10,2),
    Fecha           TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

FUNCIÓN DEL TRIGGER
```sql
CREATE OR REPLACE FUNCTION fn_audit_track_update()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO TrackAudit (TrackId, Name, Accion, OldUnitPrice, NewUnitPrice)
    VALUES (
        NEW.TrackId,
        NEW.Name,
        'UPDATE',
        OLD.UnitPrice,
        NEW.UnitPrice
    );

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

```
TRIGGER
```sql
CREATE TRIGGER trg_audit_track_update
AFTER UPDATE ON Track
FOR EACH ROW
EXECUTE FUNCTION fn_audit_track_update();
```
UPDATE
```sql
UPDATE Track
SET Name = 'Canción A Remaster'
WHERE TrackId = 1;
```

--- 

### 1. Función del trigger (DELETE)

```sql
CREATE OR REPLACE FUNCTION fn_audit_track_delete()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO TrackAudit (TrackId, Name, Accion, OldUnitPrice, NewUnitPrice)
    VALUES (
        OLD.TrackId,         -- ID antes de borrar
        OLD.Name,            -- Nombre antes de borrar
        'DELETE',            -- Acción
        OLD.UnitPrice,       -- Precio antiguo
        NULL                 -- No hay NEW en DELETE
    );

    RETURN OLD;              -- Para DELETE se devuelve OLD
END;
$$ LANGUAGE plpgsql;
```
### 2. Trigger DELETE

```sql
CREATE TRIGGER trg_audit_track_delete
AFTER DELETE ON Track
FOR EACH ROW
EXECUTE FUNCTION fn_audit_track_delete();
```
### 3. Ejecución

```sql
DELETE FROM Track
WHERE TrackId = 1;
```

--- 
### 🔥 CÓDIGO COMPLETO DEL TRIGGER COMBINADO (TG_OP)

TG_OP: operación para la que se activó el disparador: INSERT, UPDATE, DELETE, o TRUNCATE.

```sql
CREATE OR REPLACE FUNCTION fn_audit_track_all()
RETURNS TRIGGER AS $$
BEGIN
    -- INSERT
    IF TG_OP = 'INSERT' THEN
        INSERT INTO TrackAudit (TrackId, Name, Accion, OldUnitPrice, NewUnitPrice)
        VALUES (
            NEW.TrackId,
            NEW.Name,
            'INSERT',
            NULL,
            NEW.UnitPrice
        );
        RETURN NEW;
    END IF;

    -- UPDATE
    IF TG_OP = 'UPDATE' THEN
        INSERT INTO TrackAudit (TrackId, Name, Accion, OldUnitPrice, NewUnitPrice)
        VALUES (
            NEW.TrackId,
            NEW.Name,
            'UPDATE',
            OLD.UnitPrice,
            NEW.UnitPrice
        );
        RETURN NEW;
    END IF;

    -- DELETE
    IF TG_OP = 'DELETE' THEN
        INSERT INTO TrackAudit (TrackId, Name, Accion, OldUnitPrice, NewUnitPrice)
        VALUES (
            OLD.TrackId,
            OLD.Name,
            'DELETE',
            OLD.UnitPrice,
            NULL
        );
        RETURN OLD;
    END IF;

END;
$$ LANGUAGE plpgsql;
```
TRIGGER ÚNICO
```sql
CREATE TRIGGER trg_audit_track_all
AFTER INSERT OR UPDATE OR DELETE ON Track
FOR EACH ROW
-- Condición solo para DELETE,  si OLD.UnitPrice > 1, 
-- para INSERT/UPDATE se hace siempre
WHEN (TG_OP <> 'DELETE' OR OLD.UnitPrice > 1)
EXECUTE FUNCTION fn_audit_track_all();
```



