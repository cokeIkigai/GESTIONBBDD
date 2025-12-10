# Recuperación de Datos

La `pérdida` de datos puede venir por `fallos` de hardware, `errores` humanos o `corrupción`. El objetivo es volver a un estado anterior fiable, con la mínima pérdida posible y en el menor tiempo.

**¿Qué pasa si alguien borra una tabla accidentalmente?**

**¿Podemos recuperar datos concretos sin restaurar toda la base?**

**¿Cómo debe diseñarse una estrategia profesional de recuperación?**

--- 
### Fundamentos de Recuperación

**1.** `Garantizar` disponibilidad, integridad y continuidad.

**2.** `Reducir` al mínimo la pérdida de datos.

**3.** `Restaurar` el sistema a un punto coherente anterior.

#### Elegir la estrategia según el fallo:

- **Recuperación física:** ficheros completos, datafiles, clusters, backups del sistema.

- **Recuperación lógica:** tablas, registros o estados previos usando herramientas de logs/undo.

#### Escenarios típicos de fallo

- `Fallos` de hardware.

- `Fallos` de software o corrupción.

- `Errores` humanos (DROP TABLE, UPDATE masivo mal hecho).

*--Ejemplo de riesgo: DROP TABLE empleados(Sin backup o logs, es irrecuperable)--*

2.3 Herramientas en PostgreSQL

En PostgreSQL no existe RMAN, pero sí alternativas equivalentes:

Herramienta	Tipo	Uso típico
pg_dump / pg_restore	Lógico	Copias de esquemas, tablas o BD completas.
pg_basebackup	Física	Réplica o copia completa del cluster.
PITR (Point-In-Time Recovery)	Física temporal	Restaurar la BD a un instante anterior usando WAL.
WAL (Write-Ahead Logging)	Registro	Permite recuperación a puntos previos.
3. Copias de Seguridad en PostgreSQL
3.1 Copias lógicas (pg_dump)

Adecuadas para recuperar objetos concretos.

Ejemplos:

pg_dump empresa > empresa.sql
pg_dump -t empleados empresa > empleados.sql
pg_restore -d empresa restauracion.dump


Ventajas:

Granularidad (tablas, esquemas, BD).

Portables entre versiones.

Limitaciones:

No sirven para PITR ni para sistemas muy grandes en caliente.

3.2 Copias físicas (pg_basebackup)

Copia byte a byte del cluster completo. Necesarias para PITR.

pg_basebackup -D /backups/full -Ft -z -P -U replicacion


Incluye:

Archivos de datos.

Configuración.

WAL (si se configura).

3.3 WAL y Point-In-Time Recovery (PITR)

Permite volver a un instante exacto, ideal ante errores humanos.

Pasos generales:

Tener activado archive_mode y archive_command.

Restaurar un backup físico.

Incluir los WAL archivados.

Especificar el punto de recuperación:

recovery_target_time = '2025-02-10 14:35:00'

4. Recuperación Lógica
4.1 Restauración de objetos específicos

Usando copias lógicas:

pg_restore -d empresa -t empleados empleados.dump


Ideal cuando:

Se borra una tabla concreta.

Se necesita restaurar un esquema aislado.

4.2 Restauración por error humano

Si activaste PITR:

Restaura el cluster a un punto anterior al fallo.

Exporta solo lo necesario.

Importa en el cluster actual sin perder cambios posteriores.

5. Estrategias de Restauración
5.1 Restauración completa

Restaurar todo el sistema desde backup físico + WAL.

Útil cuando:

El servidor está corrupto.

Se han perdido varios ficheros de datos.

5.2 Restauración parcial

Tablas concretas (pg_dump/pg_restore).

Esquemas completos.

Porciones de datos mediante scripts preparados.

5.3 Ejemplo de recuperación PITR

Detener PostgreSQL.

Restaurar backup físico.

Copiar WAL archivados.

Crear recovery.signal y configurar destino.

Arrancar.

6. Buenas Prácticas

Establecer frecuencia clara de backups (diaria física + lógicas selectivas).

Probar restauraciones periódicamente.

Documentar el proceso de recuperación.

Monitorizar el espacio de WAL.

Validar backups:

pg_verifybackup /backups/full


Separar almacenamiento de datos y copias.

Automatizar tareas con cron o systemd timers.

7. Esquema Resumen para tus Alumnos

Backup lógico → tablas/esquemas → pg_dump

Backup físico → cluster completo → pg_basebackup

Recuperación por error humano → PITR + export/import parcial

Recuperación rápida → objetos concretos vía pg_restore

Estrategia profesional → combinación de ambas + validación periódica
