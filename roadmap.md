# Roadmap por fases

Trabaja de arriba hacia abajo, un módulo a la vez. Marca cada casilla al terminar.
Referencias de módulo (M-01…M-10) en `docs/spec.md`.

## Fase 0 — Setup

- [ ] `create-next-app` (TypeScript, App Router, ESLint), `tsconfig` en modo estricto
- [ ] Prettier + ESLint configurados
- [ ] Prisma + PostgreSQL en Railway; `schema.prisma` inicial (ver `docs/data-model.md`)
- [ ] Primera migración + `seed.ts` (1 usuario `JEFE` y 1 `GESTOR` de prueba, 1 bodega)
- [ ] Auth.js (credentials) + bcrypt; middleware de protección de rutas por rol
- [ ] Layout base y navegación responsive (desktop + tablet)
- [ ] Setup de Zod y patrón de errores de dominio (`InsufficientStockError`, etc.)
- [ ] Servicio de movimientos (`MovementsService`) con la transacción base y lock de fila

## Fase 1 — Núcleo (alta prioridad, M-01 a M-08)

### M-01 · Autenticación y usuarios
- [ ] Login, logout, expiración de sesión por inactividad
- [ ] CRUD de usuarios (crear, editar, activar/desactivar) — solo `JEFE`
- [ ] Recuperación de contraseña por correo (Resend)
- [ ] Registro en `access_logs`

### M-02 · Catálogo de productos
- [ ] CRUD de productos (código interno/proveedor, nombre, unidad, `min_stock`)
- [ ] CRUD de categorías (con subcategorías) y proveedores
- [ ] Búsqueda por código, nombre o categoría
- [ ] Importación desde Excel (exceljs / SheetJS) con validación y reporte de errores
- [ ] `cost_price` oculto al `GESTOR` en API y UI

### M-03 · Bodegas
- [ ] CRUD de bodegas
- [ ] Stock por bodega funcionando (`stock_by_warehouse`)
- [ ] Transferencias entre bodegas (movimiento `TRANSFERENCIA`, transacción atómica)

### M-04 · Entradas
- [ ] Registrar entrada (compra / devolución de cliente / ajuste positivo)
- [ ] Confirmación previa; aplica al stock en la transacción del movimiento

### M-05 · Salidas
- [ ] Registrar salida (despacho / devolución a proveedor / ajuste negativo / merma)
- [ ] Validación de stock suficiente (no permite negativo)
- [ ] Salida parcial con saldo pendiente

### M-07 · Alertas de stock mínimo
- [ ] Cálculo de productos bajo el mínimo
- [ ] Badge / indicador en el panel + listado de críticos
- [ ] Notificación por correo al `JEFE` (Resend)

### M-08 · Historial y trazabilidad
- [ ] Vista de movimientos con filtros (rango de fechas, tipo, producto, usuario)
- [ ] Exportación del historial a Excel y PDF

### M-06 · Conteo físico (al final de la fase 1)
- [ ] Sesión de conteo + snapshot del stock teórico (`count_lines`)
- [ ] Ingreso de cantidades contadas y cálculo de diferencias
- [ ] Aprobación del `JEFE` → genera movimientos de ajuste (`CONTEO_FISICO`)
- [ ] Marcado de líneas cuyo stock teórico cambió desde el snapshot
- [ ] **Capa offline**: PWA con Serwist + IndexedDB (Dexie) + cola de sync (outbox)

## Fase 2 — Reportes y dashboard (prioridad media/baja)

### M-09 · Reportes
- [ ] Stock actual por bodega (filtro por categoría/producto)
- [ ] Movimientos del período
- [ ] Productos bajo el mínimo
- [ ] Tomas de inventario y sus diferencias
- [ ] Ranking de productos por rotación
- [ ] Export Excel (.xlsx) + PDF en todos

### M-10 · Dashboard
- [ ] Total de productos e ítems críticos
- [ ] Últimos movimientos
- [ ] Gráfico entradas vs. salidas del mes
- [ ] Accesos rápidos a módulos frecuentes

## Fase 3 — Cierre y entregables

- [ ] Escaneo de códigos como mejora (BarcodeDetector nativo + fallback `@zxing/browser`)
- [ ] Backups automáticos verificados en Railway
- [ ] Endurecer seguridad: HTTPS, expiración por inactividad, auditoría de accesos;
      revisar que ningún dato financiero se filtre al `GESTOR`
- [ ] Manual de usuario por rol (Jefe / Gestor)
- [ ] Manual técnico / documentación de la base de datos
- [ ] Sesión de capacitación

## Transversal (aplica en todas las fases)

- [ ] Permisos verificados en la API, no solo en la UI
- [ ] Validación Zod en cada endpoint
- [ ] Tests de la lógica de dominio: validación de stock, diferencias de conteo,
      permisos por rol
- [ ] Toda mutación de stock pasa por `MovementsService`
