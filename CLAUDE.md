# Sistema de Inventarios — Contexto del proyecto

Sistema web de gestión y toma de inventarios para una bodega distribuidora de
repuestos industriales y automotrices. ~3 usuarios, 2 roles. Objetivo: control de
stock en tiempo real, movimientos auditables, conteo físico con conciliación,
alertas de stock mínimo y reportes.

- Requisitos completos: `docs/spec.md` (módulos M-01…M-10)
- Modelo de datos: `docs/data-model.md`
- Plan de trabajo por fases: `docs/roadmap.md`

## Stack (decidido — no re-litigar)

- **Next.js (App Router)**, monolito full-stack. UI + API en el mismo proyecto
  (route handlers / server actions).
- **TypeScript** estricto.
- **PostgreSQL + Prisma** (migraciones + tipos).
- **Auth.js (credentials) + bcrypt**. Roles como enum.
- **Zod** para validar toda entrada de la API.
- Reportes: **exceljs** (.xlsx), **@react-pdf/renderer** (PDF).
- Correo: **Resend** (alertas de stock mínimo + reset de contraseña).
- PWA con **Serwist** (`@serwist/next`) — SOLO para el conteo físico offline.
- Almacenamiento offline: **IndexedDB vía Dexie**.
- Deploy: **Railway** (app + Postgres + backups automáticos).
- Gestor de paquetes: **npm**.

## Reglas de dominio NO negociables

Son el corazón del sistema. No las cambies sin confirmarlo explícitamente.

1. **El stock es un LEDGER de movimientos inmutables.**
   - Tabla `movements` append-only. Tipos: `ENTRADA`, `SALIDA`, `TRANSFERENCIA`.
     Los ajustes (incluidos los del conteo físico) son entradas o salidas con
     motivo `AJUSTE_POSITIVO` / `AJUSTE_NEGATIVO` / `CONTEO_FISICO`.
   - NUNCA se edita ni borra un movimiento. Una corrección es un movimiento nuevo.
   - El stock por bodega (`stock_by_warehouse`) se mantiene materializado y se
     actualiza en la **misma transacción** que inserta el movimiento. Nunca por
     separado.
   - Toda mutación de stock pasa por el servicio de movimientos (`MovementsService`).
     Ningún endpoint escribe el stock directamente.

2. **El stock nunca queda negativo.**
   - Una `SALIDA` valida stock suficiente en la bodega origen ANTES de aplicarse,
     dentro de la transacción y con lock de fila (`SELECT … FOR UPDATE`) para
     evitar carreras.
   - Si no hay stock, la operación falla con error claro. Nada de aplicaciones
     parciales, salvo la opción explícita de "salida parcial con saldo pendiente".

3. **Conteo físico (M-06) → ajuste aprobado.**
   - Una sesión de conteo toma un snapshot del stock teórico.
   - El sistema calcula diferencias (sobrantes / faltantes).
   - SOLO el Jefe de Bodega aprueba. Al aprobar se generan los movimientos de
     ajuste (uno por diferencia) dentro de una transacción.
   - Si el stock teórico cambió entre el snapshot y la aprobación, márcalo para
     revisión antes de aplicar.

4. **Permisos por rol — se aplican en la API, no solo en la UI.**
   - `JEFE`: acceso total (productos, bodegas, usuarios, movimientos, conteos,
     reportes, configuración).
   - `GESTOR`: toma de inventario, registro de movimientos y consulta de stock.
     SIN configuración, SIN reportes financieros.
   - El precio de costo y cualquier dato financiero NO se exponen al `GESTOR` — ni
     en la respuesta de la API ni en el payload. Ocultarlo solo en el front NO basta.

5. **Trazabilidad.**
   - Cada movimiento registra: fecha/hora, tipo, motivo, producto, cantidad,
     bodega(s), usuario responsable y referencia.
   - Toda acción sensible queda atribuida al usuario que la ejecutó.

6. **Alcance offline.**
   - SOLO el conteo físico (M-06) funciona offline: snapshot en IndexedDB + outbox
     que sincroniza al reconectar. El servidor es la autoridad al aplicar el ajuste.
   - El resto de la app es online normal. NO conviertas toda la app en offline-first.

## Multi-bodega

Modela el stock por bodega desde el día 1 (`stock_by_warehouse` con FK a bodega),
aunque la UI multi-bodega sea de fase 2. No asumas una sola bodega en el esquema.

## Convenciones de código

- La lógica de negocio vive en el servidor (services / server actions), nunca en
  componentes cliente.
- Valida TODA entrada de API con Zod; deriva los tipos de los esquemas Zod.
- Errores de dominio explícitos (ej. `InsufficientStockError`) traducidos a
  respuestas HTTP claras.
- Código en inglés (tablas, campos, funciones); textos de UI en español.
- Migraciones de Prisma versionadas; nada de `db push` contra algo productivo.
- Dinero/precios: nunca `float`. Usa `Decimal` (Prisma) o enteros en centavos.
- Escribe tests para la lógica crítica: validación de stock, cálculo de diferencias
  de conteo y permisos por rol.

## Comandos (referencia — ajústalos al scaffoldear)

- `npm run dev` — desarrollo
- `npm run build` — build de producción
- `npm run lint` — ESLint
- `npm run typecheck` — `tsc --noEmit`
- `npm test` — tests
- `npx prisma migrate dev` — crear/aplicar migración en dev
- `npx prisma studio` — inspeccionar la DB
- `npm run db:seed` — datos semilla (1 jefe + 1 gestor de prueba)

## Estructura (una vez scaffoldeado — esbozo)

- `app/` — rutas App Router (UI + route handlers)
- `server/` (o `lib/`) — services de dominio: movimientos, conteo, reportes, auth
- `prisma/` — `schema.prisma`, migraciones, `seed.ts`
- `components/` — UI
- `offline/` — conteo offline (Dexie, cola de sync, service worker)
- `docs/` — spec, modelo de datos, roadmap

## Cómo trabajar en este repo

- Antes de tocar stock, relee las reglas de dominio de arriba.
- Sigue `docs/roadmap.md`: trabaja por fases, un módulo a la vez.
- Ante cualquier ambigüedad de alcance, **pregunta antes de implementar** (la propia
  especificación lo exige).
