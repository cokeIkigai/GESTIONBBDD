# Recuperación de Datos (PostgreSQL)

### La recuperación de datos tiene como finalidad:

- Garantizar la disponibilidad e integridad de la información.

- Reducir el tiempo de inactividad (downtime).

- Minimizar la pérdida de datos (RPO).

- Restaurar el sistema a un estado consistente.

### En PostgreSQL distinguimos dos enfoques:

**Recuperación lógica**: Recupera objetos o datos concretos (tablas, esquemas, filas).

**Recuperación física**: Recupera el clúster completo de PostgreSQL (datos + WAL).

### Escenarios comunes de pérdida de datos

Situaciones habituales en entornos reales:

**Errores humanos**
```sql
DELETE FROM clientes;
DROP TABLE pedidos;
```

**Fallos de hardware**

- Discos dañados

- Pérdida del directorio data

- Fallos de software

- Corrupción de datos

- Actualizaciones fallidas

- Ciberataques

- Ransomware

- Accesos no autorizados

*Sin backups, la pérdida suele ser irreversible.*

---

## Herramientas de recuperación en PostgreSQL

|Herramienta	|Tipo	|Uso principal|
|---|---|---|
|pg_dump	|Lógico	|Backups de bases de datos u objetos|
|pg_restore	|Lógico	|Restauración selectiva|
|pg_basebackup	|Físico	|Copia completa del clúster|
|WAL	|Físico	|Recuperación a punto en el tiempo (PITR)|

## Copias de Seguridad en PostgreSQL

### Backup lógico con `pg_dump`

- Es el método más usado en entornos pequeños y medianos.

- Copia estructura y datos

- Portable entre versiones

- Permite restauración selectiva

*Ejemplo*

**Restaurar una Base de Datos:**
```console
 pg_dump -U postgres -F c -f backup_empresa.dump empresa
```
|Comando|Descripción|
|---|---|
|pg_dump| *Crea el backup* |
|-U postgres|*Usuario de PostgreSQL*|
|-F c|*custom → permite restaurar tablas*|
|-f backup_empresa.dump|*Donde se guarda el backup*| 
|empresa|*Base de datos*|

**Restaurar una Tabla**

```console
pg_dump -U postgres -F c -t clientes -f backup_clientes.dump empresa
```

**-t clientes:** Referencia a la tabla

### Restauración con pg_restore

Permite restaurar todo o partes concretas.

**Restaurar Base de Datos:**

```console
pg_restore -U postgres -d empresa backup_empresa.dump
```

Restaurar Tabla:

```console
pg_restore -U postgres -d empresa -t clientes backup_empresa.dump
```


### Backup físico con pg_basebackup

- Copia todo el clúster PostgreSQL.

- Incluye datos, configuraciones y estado interno

- Necesario para recuperación ante desastre

- Base para PITR (Point-In-Time Recovery)

Ejemplo
pg_basebackup -U postgres -D /backups/base -Fp -Xs -P

### WAL y recuperación a un punto en el tiempo (PITR)

PostgreSQL registra todos los cambios en WAL (Write-Ahead Logs).

Esto permite:

- Volver a un instante concreto

- Recuperarse tras errores humanos o ataques

Requiere:

archive_mode = on

archive_command configurado

## Estrategias de Recuperación y Buenas Prácticas
### Tipos de restauración

1. **Restauración lógica:** 

- Tablas o bases concretas

- Menor impacto

2. **Restauración física**

- Todo el sistema

- Tras fallos graves

### Buenas prácticas profesionales

- Realizar backups regulares

- Guardarlos fuera del servidor

- Probar restauraciones periódicamente

- Automatizar con scripts y cron

- Documentar procedimientos

- Un backup que nunca se ha restaurado no es un backup fiable.
