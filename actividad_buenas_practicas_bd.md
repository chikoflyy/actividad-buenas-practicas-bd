# Propuesta de Buenas Practicas De Base de Datos - TecnoStore

## Parte 1: Organización y nomenclatura.
## Parte 2: Estrategia de Respaldo.
## Parte 3: Registro de Cambios.  
## Parte 4: Situación Final.

------------------------------------------


## Parte 1: Organización y Nomenclatura.

| Nombre Actual | Problema Identificado | Nombre Propuesto | Justificación |
| :--- | :--- | :--- | :--- |
| `tabla1` | Le faltan detalles; si entro a la base de datos no tengo idea de qué información guarda esta tabla. | `ordenes_compra` | Elegí este nombre porque deja 100% claro que aquí se almacenan las transacciones que realizan los clientes de TecnoStore. |
| `datosx` | Es un nombre ambiguo y poco profesional; la palabra "datos" no me dice nada en un sistema relacional. | `catalogo_productos` | Decidí renombrarla así para identificar inmediatamente que se trata de la lista de artículos disponibles en la tienda. |
| `clientes2026` | Amarra el nombre a un año específico. Cuando pase el tiempo, la tabla quedará desactualizada o tendré que crear una por año. | `compradores` | Propongo usar un sustantivo genérico en plural para que la estructura sea escalable y no dependa de fechas. |
| `user1` | Está en inglés, en singular y usa un número arbitrario que no aporta nada. | `cuentas_acceso` | Prefiero este término para reflejar las credenciales de los usuarios del sistema manteniendo una convención clara en español. |
| `prueba` | Es un objeto basura que alguien dejó en producción. Me genera desorden y riesgo de manipular datos erróneos. | *Eliminar objeto* | Considero que lo correcto es borrarlo para mantener el esquema limpio y evitar confusiones en el equipo. |
| `indice1` | Es un índice "a ciegas"; no sé a qué tabla ni a qué columna está optimizando. | `idx_compradores_rut` | Apliqué la convención `prefijo_tabla_columna` para saber al instante qué consulta se está acelerando en el sistema. |

------------------------------------------

## Parte 2: Estrategia de Respaldo

Actualmente, TecnoStore corre un riesgo enorme al guardar copias ocasionales en el mismo servidor. Si el servidor falla, perderíamos tanto la operación como los respaldos. Para solucionar esto, respondo a las siguientes preguntas clave de la estrategia:

### ¿Cada cuánto realizaría un respaldo?
**Respuesta:** 
No podemos depender de respaldos ocasionales. Propongo una estrategia combinada para no saturar el sistema pero garantizar la recuperación de datos:
* **Respaldo Completo (Full):** Lo realizaría una vez por semana (los domingos a las 02:00 AM), aprovechando que son las horas de menor tráfico en la tienda online.
* **Respaldo Incremental:** Lo programaría diariamente al final de la jornada (23:59 PM) para guardar únicamente las ventas y nuevos registros del día.

### ¿Dónde almacenaría los respaldos?
**Respuesta:** 
Prohíbo rotundamente almacenar las copias dentro del mismo servidor de producción. Implementaría un almacenamiento en la nube (como AWS S3 o Google Cloud Storage) o en un servidor secundario seguro. Si el servidor principal sufre un daño físico o un ataque informático, nuestros respaldos estarán completamente a salvo fuera de la red local.

### ¿Cómo identificaría cada archivo?
**Respuesta:** 
Para evitar confusiones en una emergencia y saber exactamente qué archivo restaurar, utilizaría la siguiente convención de nombres:
* **Estructura:** `bkp_[tipo]_[nombre_bd]_[YYYYMMDD_HHMM].sql`
* **Ejemplo práctico:** `bkp_full_tecnostore_20260817_0200.sql`
De esta forma, cualquier integrante del equipo sabe al instante de qué base de datos es el respaldo, si es completo o incremental, y la fecha y hora exacta en que se generó.

### ¿Por qué sería importante probar periódicamente la restauración?
**Respuesta:** 
Considero que "un respaldo que no se ha probado, simplemente no existe". De nada sirve tener un archivo `.sql` de 10 GB si al momento de una caída el archivo está corrupto o da errores de sintaxis. Realizar simulacros periódicos en un ambiente de pruebas nos asegura que la información es 100% recuperable y que el negocio podrá volver a operar sin sorpresas.

------------------------------------------

## Parte 3: Registro de Cambios.

Para evitar que en TecnoStore se sigan haciendo modificaciones a ciegas y sin trazabilidad, implementé un control de bitácora obligatorio. A continuación, presento la resolución y el registro formal del caso:

### ¿Cómo registraría el cambio realizado para el equipo de ventas?
**Respuesta:**
Construí una bitácora en formato de tabla para auditar el evento. Esta estructura nos permite saber en cualquier momento quién hizo la modificación, qué comandos ejecutó, cuándo lo hizo y por qué razón.

| Campo | Información Registrada |
| :--- | :--- |
| **Fecha y hora** | 2026-08-17 14:30 hrs |
| **Responsable** | Administrador de Base de Datos (DBA) |
| **Cambio realizado** | Creación de cuenta de usuario con permisos de acceso restringido a lectura. |
| **Script/comando utilizado** | `CREATE USER 'ventas_user'@'%' ...`  |
| **Motivo del cambio** | Habilitar al equipo de ventas para consultar información de la tienda sin arriesgar la integridad de los datos. |

### Script SQL Ejecutado
Como estándar de trazabilidad, todo registro de bitácora debe ir acompañado del código o comando exacto ejecutado en el servidor:

```sql
-- 1. Crear el usuario para el área de ventas con contraseña segura
CREATE USER 'ventas_consultas'@'%' IDENTIFIED BY 'VentasSeguras2026!';

-- 2. Otorgar únicamente permisos de lectura (SELECT) sobre las tablas de producción
GRANT SELECT ON tecnostore_db.* TO 'ventas_consultas'@'%';

-- 3. Aplicar y refrescar los privilegios en la base de datos
FLUSH PRIVILEGES;
```

------------------------------------------

## Parte 4: Situación Final.

Ante la solicitud del desarrollador de aplicar un cambio directo sobre la base de datos de producción sin respaldo previo ni registro en bitácora, mi postura y procedimiento como DBA son los siguientes:

### ¿Autorizaría el procedimiento?
**Respuesta:**
**No, bajo ninguna circunstancia autorizo este procedimiento.** En la administración de bases de datos no existen los "cambios pequeños e inofensivos". Saltarse los protocolos de seguridad es la causa principal de caídas masivas de sistemas, corrupción de información y pérdidas irrecoverables de datos.

### ¿Qué debería realizar antes del cambio?
**Respuesta:**
Antes de tocar cualquier tabla en el entorno de producción, exigiría cumplir obligatoriamente con tres pasos de control:
1. **Validación en Staging/Desarrollo:** Probar el script SQL en un ambiente de pruebas para confirmar que no genera bloqueos de tablas ni errores de sintaxis.
2. **Respaldo preventivo (Pre-change backup):** Generar un respaldo inmediato de la tabla o esquema afectado para asegurar una vía de retorno limpia si algo sale mal.
3. **Plan de reversión (Rollback script):** Contar con el script inverso documentado para deshacer los cambios rápidamente en caso de emergencia.

### ¿Qué debería registrar después?
**Respuesta:**
Una vez ejecutado el cambio en producción de forma controlada, registraría inmediatamente en la bitácora:
* **Fecha y hora precisa:** Momento exacto de la ejecución.
* **Trazabilidad:** Nombre del desarrollador que solicitó la modificación y del DBA que la autorizó/ejecutó.
* **Comando ejecutado:** El script SQL definitivo que se aplicó sobre la base de datos.
* **Estado final:** Confirmación de que las consultas y la tienda online siguen operando con normalidad.

### Justificación de mi decisión
**Respuesta:**
Mi responsabilidad principal es garantizar la estabilidad, seguridad y continuidad operativa de TecnoStore. Permitir modificaciones indocumentadas y "al vuelo" en producción deja a la empresa totalmente expuesta a pérdidas económicas e interrupciones del servicio sin posibilidad de volver atrás. Un protocolo de cambios no busca entorpecer el trabajo de desarrollo, sino proteger el negocio.
