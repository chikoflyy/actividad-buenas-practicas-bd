# Propuesta de Buenas Practicas De Base de Datos - TecnoStore

## Parte 1: Organización y nomenclatura.
## Parte 2: Estrategia de Respaldo.
## Parte 3: Registro de Cambios.  
## Parte 4: Situación Final.


## Parte 1: Organización y Nomenclatura

| Nombre Actual | Problema Identificado | Nombre Propuesto | Justificación |
| :--- | :--- | :--- | :--- |
| `tabla1` | Le faltan detalles; si entro a la base de datos no tengo idea de qué información guarda esta tabla. | `ordenes_compra` | Elegí este nombre porque deja 100% claro que aquí se almacenan las transacciones que realizan los clientes de TecnoStore. |
| `datosx` | Es un nombre ambiguo y poco profesional; la palabra "datos" no me dice nada en un sistema relacional. | `catalogo_productos` | Decidí renombrarla así para identificar inmediatamente que se trata de la lista de artículos disponibles en la tienda. |
| `clientes2026` | Amarra el nombre a un año específico. Cuando pase el tiempo, la tabla quedará desactualizada o tendré que crear una por año. | `compradores` | Propongo usar un sustantivo genérico en plural para que la estructura sea escalable y no dependa de fechas. |
| `user1` | Está en inglés, en singular y usa un número arbitrario que no aporta nada. | `cuentas_acceso` | Prefiero este término para reflejar las credenciales de los usuarios del sistema manteniendo una convención clara en español. |
| `prueba` | Es un objeto basura que alguien dejó en producción. Me genera desorden y riesgo de manipular datos erróneos. | *Eliminar objeto* | Considero que lo correcto es borrarlo para mantener el esquema limpio y evitar confusiones en el equipo. |
| `indice1` | Es un índice "a ciegas"; no sé a qué tabla ni a qué columna está optimizando. | `idx_compradores_rut` | Apliqué la convención `prefijo_tabla_columna` para saber al instante qué consulta se está acelerando en el sistema. |
