# Propuesta de Buenas Prácticas de Base de Datos - TecnoStore

## Parte 1: Organización y Nomenclatura

| Nombre Actual | Problema Identificado | Nombre Propuesto | Justificación |
| :--- | :--- | :--- | :--- |
| `tabla1` | Nombre genérico que no describe el contenido. | `ventas` / `pedidos` | Identifica claramente los registros almacenados. |
| `datosx` | Ambigüedades y falta de significado profesional. | `inventario` | Define exactamente qué datos de productos guarda. |
| `clientes2026` | Incluye un año/fecha fija dentro del nombre de la tabla. | `clientes` | Mantiene el estándar temporalmente agnóstico y escalable. |
| `user1` | Específico de una prueba o usuario individual. | `usuarios` | Utiliza un sustantivo en plural para la entidad de usuarios. |
| `prueba` | Objeto temporal en entorno de producción. | *Eliminar* / `ambientes_test` | Evita basura y confusión en el esquema activo. |
| `indice1` | Indeterminado; no indica sobre qué tabla/columna aplica. | `idx_clientes_email` | Indica que es un índice (`idx`), la tabla y la columna. |

## Parte 2: Estrategia de Respaldo

- **Frecuencia:** 
  - **Respaldos completos (Full):** Semanalmente (domingos a las 02:00 hrs).
  - **Respaldos incrementales/diferenciales:** Diariamente a la medianoche.
- **Ubicación:** 
  - Almacenamiento seguro en la nube (ej. AWS S3, Google Cloud Storage) o servidor secundario fuera del entorno principal.
- **Identificación de Archivos (Nomenclatura):** 
  - Patrón: `bkp_[tipo]_[nombre_bd]_[YYYYMMDD_HHMM].sql`
  - Ejemplo: `bkp_full_tecnostore_20260817_0200.sql`
- **Importancia de Pruebas de Restauración:** 
  - Un respaldo que nunca se ha probado no garantiza la integridad de los datos. Probar la restauración periódicamente en un entorno de pruebas asegura la continuidad del negocio ante desastres reales.

## Parte 3: Registro de Cambios (Bitácora)

### Tabla de Bitácora

| Fecha y Hora | Responsable | Cambio Realizado | Script/Comando Utilizado | Motivo del Cambio |
| :--- | :--- | :--- | :--- | :--- |
| 2026-08-17 14:00 | Administrador DBA | Creación de usuario con permisos de lectura | *Ver script adjunto abajo* | Permitir consultas de lectura al equipo de ventas |

### Script Utilizado

```sql
-- Creación de usuario para el equipo de ventas
CREATE USER 'ventas_user'@'%' IDENTIFIED BY 'PasswordSeguro2026!';

-- Asignación de permisos de lectura sobre la base de datos de producción
GRANT SELECT ON tecnostore_db.* TO 'ventas_user'@'%';

-- Aplicación de privilegios
FLUSH PRIVILEGES;
