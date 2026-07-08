# Especificaciones Funcionales — Sistema Web de Toma de Inventarios

Bodega / Distribuidora de Repuestos Industriales y Automotrices
Versión 1.0 · Junio 2026

## 1. Objetivo del sistema

Desarrollar un sistema web de gestión y toma de inventarios para una bodega
distribuidora de repuestos industriales y automotrices. Debe permitir controlar el
stock en tiempo real, registrar entradas y salidas, gestionar múltiples ubicaciones,
generar reportes automáticos y mantener un historial completo y auditable de todos
los movimientos.

## 2. Alcance del proyecto

- Gestión completa del catálogo de repuestos (industriales y automotrices)
- Control de inventario en una o más bodegas / ubicaciones físicas
- Registro de movimientos de stock: entradas, salidas y ajustes
- Toma de inventario físico periódico con conciliación automática
- Sistema de alertas por stock mínimo configurable por producto
- Historial y trazabilidad de movimientos por usuario y fecha
- Generación de reportes exportables (Excel / PDF)
- Panel de control con indicadores clave del inventario

## 3. Usuarios y roles

| Rol | Permisos y accesos |
| --- | --- |
| Jefe / Supervisor de Bodega | Acceso total: gestión de productos, bodegas, usuarios, reportes y configuración |
| Gestor de Inventarios Externo | Acceso limitado: toma de inventario, registro de movimientos y consulta de stock. Sin configuración ni reportes financieros. |

Cada usuario accede con credenciales únicas. El sistema registra al usuario
responsable de cada acción.

## 4. Prioridades de desarrollo

| Prioridad | Módulo / función | Descripción |
| --- | --- | --- |
| Alta | Registro de movimientos de stock | Entradas y salidas en tiempo real |
| Alta | Reportes y alertas de stock bajo | Notificación automática bajo el mínimo |
| Media | Control multi-bodega | Inventario en varias ubicaciones |
| Media | Historial y trazabilidad | Registro de quién hizo qué y cuándo |

## 5. Módulos funcionales

| Código | Módulo | Descripción |
| --- | --- | --- |
| M-01 | Autenticación y gestión de usuarios | Login seguro, roles y permisos |
| M-02 | Catálogo de productos | Repuestos con código, descripción, categoría y proveedor |
| M-03 | Gestión de bodegas / ubicaciones | Administración de múltiples bodegas |
| M-04 | Registro de entradas de stock | Compras, recepciones, devoluciones de clientes |
| M-05 | Registro de salidas de stock | Ventas, despachos, devoluciones a proveedor |
| M-06 | Toma de inventario físico | Conteo periódico vs stock del sistema |
| M-07 | Alertas de stock mínimo | Notificación por umbral configurable |
| M-08 | Historial y trazabilidad | Registro auditable de todos los movimientos |
| M-09 | Reportes y exportación | Reportes en pantalla + export Excel/PDF |
| M-10 | Panel de control (dashboard) | Resumen visual del inventario |

### 5.1 Autenticación (M-01)
- Login con usuario y contraseña
- Sesión con expiración configurable
- Recuperación de contraseña por correo
- Gestión de usuarios: crear, editar, activar/desactivar
- Asignación de roles: Jefe de Bodega / Gestor Externo

### 5.2 Catálogo de productos (M-02)
- Campos: código interno y/o de proveedor, nombre/descripción, categoría
  (industrial / automotriz / subcategoría), unidad de medida, stock mínimo
  configurable por producto, proveedor(es) asociados, precio de costo referencial
  (opcional)
- Búsqueda rápida por código, nombre o categoría
- Importación de productos desde archivo Excel

### 5.3 Gestión de bodegas (M-03)
- Creación de múltiples bodegas / ubicaciones
- Cada producto puede tener stock independiente por bodega
- Transferencias de stock entre bodegas con registro del movimiento

### 5.4 Registro de entradas (M-04)
- Tipos: compra a proveedor, devolución de cliente, ajuste positivo
- Campos: producto, cantidad, bodega destino, fecha, proveedor, referencia (ej. nº factura)
- Confirmación antes de aplicar el movimiento
- Actualiza el stock en tiempo real al confirmar

### 5.5 Registro de salidas (M-05)
- Tipos: despacho a cliente, devolución a proveedor, ajuste negativo, merma
- Campos: producto, cantidad, bodega origen, fecha, destino/motivo, referencia
- Validación: no permitir salidas sin stock suficiente
- Opción de salida parcial con saldo pendiente

### 5.6 Toma de inventario físico (M-06)
- El sistema genera una planilla de conteo con el stock teórico actual
- El usuario ingresa las cantidades físicamente contadas
- El sistema muestra automáticamente las diferencias (sobrantes / faltantes)
- El Jefe de Bodega aprueba y aplica el ajuste
- Queda registrado el historial de cada toma realizada

### 5.7 Alertas de stock mínimo (M-07)
- Nivel de stock mínimo configurable por producto
- Alerta visible en el panel cuando el stock baja del mínimo
- Notificación por correo al Jefe de Bodega
- Listado de productos en estado crítico desde el dashboard

### 5.8 Historial y trazabilidad (M-08)
- Registro inmutable de todos los movimientos: entrada, salida, ajuste, transferencia
- Cada registro: fecha/hora, tipo, producto, cantidad, bodega, usuario y referencia
- Filtros por rango de fechas, tipo, producto y usuario
- Exportación a Excel o PDF

### 5.9 Reportes (M-09)
- Stock actual por bodega (filtro por categoría o producto)
- Movimientos del período (rango de fechas)
- Productos con stock bajo el mínimo
- Tomas de inventario realizadas y sus diferencias
- Ranking de productos con mayor rotación
- Todos exportables en Excel (.xlsx) y PDF

### 5.10 Panel de control / Dashboard (M-10)
- Total de productos en inventario
- Nº de productos con stock crítico
- Últimos movimientos registrados
- Acceso rápido a los módulos más usados
- Gráfico de entradas vs. salidas del mes actual

## 6. Requerimientos técnicos

### 6.1 Plataforma
- Web, accesible desde navegador (Chrome, Firefox, Edge)
- Responsivo: computadora y tablet
- Sin instalación de software adicional

### 6.2 Seguridad
- HTTPS
- Contraseñas con hash seguro (bcrypt o equivalente)
- Cierre de sesión automático por inactividad
- Registro de auditoría de accesos

### 6.3 Base de datos
- Soporte para múltiples bodegas y gran volumen de productos
- Base relacional recomendada (PostgreSQL o MySQL)
- Respaldo automático periódico

### 6.4 Rendimiento
- Respuesta ≤ 3 s en operaciones estándar
- ≥ 10 usuarios concurrentes sin degradación

## 7. Entregables esperados

- Sistema web funcional y desplegado
- Manual de usuario por rol (Jefe de Bodega y Gestor Externo)
- Manual técnico / documentación de la base de datos
- Código fuente
- Sesión de capacitación para usuarios finales

## 8. Consideraciones finales

Consultar cualquier duda sobre los requisitos antes de desarrollar cada módulo.
Desarrollo por fases: primero los módulos de alta prioridad (M-01 a M-08), luego
reportes avanzados y dashboard. Cualquier cambio al alcance debe acordarse y
documentarse por escrito antes de implementarse.
