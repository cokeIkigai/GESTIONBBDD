# SQL sintaxis  

[sql_documentación](./SQL_Temario.docx)

CREATE TABLE

```sql
CREATE TABLE Empleados (
  Id        INT           PRIMARY KEY IDENTITY,  -- Clave Primaria (Es única)
  Nombre    VARCHAR(100)  NOT NULL,
  Cargo     VARCHAR(50),
  Salario   DECIMAL(10,2) DEFAULT 0,
  FechaAlta DATE          DEFAULT GETDATE()
);

```

Tipos más comunes: INT · VARCHAR(n) · DECIMAL(p,s) · DATE · DATETIME · BIT · FLOAT

¿Qué tipo de dato usar en cada caso? ↗

restricciones útiles

Email  VARCHAR(200) UNIQUE NOT NULL,
Edad   INT          CHECK (Edad >= 18),
DepId  INT          FOREIGN KEY REFERENCES Departamentos(Id)

SELECT

consulta
consulta simple

```sql
SELECT Nombre, Cargo, Salario
FROM   Empleados
WHERE  Salario > 30000
ORDER BY Nombre ASC;
```

con filtros y funciones

```sql
SELECT TOP 10
  Nombre,
  UPPER(Cargo)        AS Cargo,
  FORMAT(Salario,'N2') AS SalarioFormato
FROM  Empleados
WHERE FechaAlta >= '2024-01-01'
  AND Cargo LIKE '%Manager%'
ORDER BY Salario DESC;
```

Ver más operadores WHERE ↗
con JOIN entre tablas

```sql
SELECT e.Nombre, d.NombreDept
FROM  Empleados    e
INNER JOIN Departamentos d ON e.DepId = d.Id;
```

Ver tipos de JOIN ↗

INSERT
inserción
fila única

```sql
INSERT INTO Empleados (Nombre, Cargo, Salario)
VALUES ('Ana García', 'Desarrolladora', 45000);
```

múltiples filas a la vez

```sql
INSERT INTO Empleados (Nombre, Cargo, Salario)
VALUES
  ('Luis Martínez', 'DBA',        52000),
  ('Sara López',   'QA Engineer', 38000),
  ('Pedro Ruiz',   'DevOps',      48000);
```

insertar desde otra tabla

```sql
INSERT INTO EmpleadosArchivo (Nombre, Cargo)
SELECT Nombre, Cargo
FROM  Empleados
WHERE FechaAlta < '2020-01-01';
```
