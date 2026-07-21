# Next Steps — SafeRoute

> **Este archivo se pisa seguido.** Cuando cerrás un ticket, movelo a `changelog.md` y agregá el siguiente acá.
> Última actualización: **2026-07-21** (post-sesión de prueba en device físico)

---

## Contexto rápido

Sesión del 2026-07-21: durante la prueba en device físico se descubrieron y arreglaron **dos bugs bloqueantes** que estaban en el Ticket 2 (IP hardcodeada) y Ticket 3 (navegación). Ver `changelog.md` para el detalle.

Estado de Fase 0 (higiene) ahora:

| Ticket | Estado |
|---|---|
| Ticket 1 — Debug code en ProfileScreen | Pendiente |
| Ticket 2 — Base URL a `.env` | **Parche aplicado, fix definitivo pendiente** |
| Ticket 3 — Navigation completa | **Completo** (todas las 20 pantallas cableadas) |
| Ticket 4 — Retry limit de token refresh | Pendiente |
| Ticket 5 — Commit del Postman | Pendiente |

Los tickets abajo son los siguientes en línea. Cuando termines los 5, actualizá esto con los tickets de Fase 1.

---

## Ticket 1 — Sacar debug code de ProfileScreen

**Repo:** `saferoute-app`
**Archivos:** `src/screens/shared/ProfileScreen.tsx`
**Prioridad:** Alta
**Estimación:** 30 min

**Qué hacer:**
- Eliminar los 8 `console.log()` (líneas aprox 74, 83, 88, 97, 118).
- Eliminar el debug box amarillo visible en la UI (buscar `debugText`, `debugIcon`, `debugBox` en styles).
- Verificar que `ProfileScreen` use `guardianApi.getProfile()` en vez de path hardcodeado `/guardians/user/{id}`.

**Criterio de "done":** grep por `console.log` y `debug` en `ProfileScreen.tsx` no devuelve nada. La pantalla se ve limpia sin cajas amarillas.

---

## Ticket 2 — Base URL a variables de entorno (FIX DEFINITIVO)

**Repo:** `saferoute-app`
**Archivos:** los 9 identificados el 2026-07-21 + nuevo `.env` + `.env.example`
**Prioridad:** Alta
**Estimación:** 1-2h

**Qué hacer:**
- Instalar y configurar `expo-constants` con `extra` en `app.json`, o `react-native-dotenv`.
- Crear `.env` con `EXPO_PUBLIC_API_URL=http://192.168.40.9:8080/api/v1` (ajustar IP según red).
- Crear `.env.example` con valor dummy.
- Agregar `.env` a `.gitignore` (verificar primero).
- **Refactor MÁS PROFUNDO:** eliminar las 9 constantes `API_URL` duplicadas en los 9 archivos. Todas las llamadas fetch deben usar el cliente centralizado en `src/lib/api.ts`. Las pantallas no deberían tener URLs de API adentro.
- Ver ADR pendiente sobre esto (se puede escribir en esta iteración).

**Criterio de "done":** grep por `192.168` en toda la app devuelve solo `.env`. Grep por `const API_URL` devuelve 0 matches en `src/screens/`.

---

## Ticket 3 — Navigation completa

**COMPLETO** — movido a `changelog.md` el 2026-07-21.

Detalles del refactor y decisión arquitectónica ver `decisions/0002-navegacion-react-navigation-clasico.md`.

---

## Ticket 4 — Limitar retries del refresh de token

**Repo:** `saferoute-app`
**Archivos:** `src/lib/api.ts`
**Prioridad:** Media
**Estimación:** 30 min

**Qué hacer:**
- Agregar un flag `isRefreshing` para evitar refreshes concurrentes.
- Agregar contador de retries — máximo 1 retry por request. Si el segundo intento también da 401, forzar logout.
- Considerar cola de requests que esperan el refresh (si es simple; si no, dejarlo para después).

**Criterio de "done":** un 401 persistente no cuelga la app en loop. Después de 1 retry fallido, se dispara logout y navega a Login.

---

## Ticket 5 — Commit del Postman pendiente

**Repo:** `ai-saferoute-context`
**Archivos:** `05-shared/SafeRoute-API-Postman.json`
**Prioridad:** Baja (limpieza)
**Estimación:** 5 min

**Qué hacer:**
- Verificar el diff del Postman.
- **Agregar además** el nuevo endpoint `GET /api/v1/dashboard/stats` (implementado el 2026-07-21).
- Commitear con mensaje descriptivo: `docs(api): update Postman with roles[], info-validate, and dashboard endpoints`.
- Push.

**Criterio de "done":** `git status` en `ai-saferoute-context` limpio.

---

## Ticket 6 — Prueba end-to-end en device físico

**Repo:** `saferoute-app`
**Prioridad:** Alta
**Estimación:** 1h (manual)

Después del refactor de navegación del 2026-07-21, probar en device físico los 3 flujos completos:
1. Login como admin → navegar por Dashboard, Users, Routes, Vehicles, Verify.
2. Login como driver → Dashboard, MyRoutes, StartRoute, Tracking, ScanNFC, EventForm.
3. Login como guardian → Dashboard, Children, ChildForm, ChildDetail, Notifications, RouteStatus.

**Criterio de "done":** los 3 roles pueden atravesar sus flujos principales sin crashes ni errores en consola. Documentar bugs encontrados como nuevos tickets.

---

## Después de estos 6

Cuando termines la Fase 0, los próximos tickets van a salir de la **Fase 1**:

1. Integrar FCM push notifications (backend + app).
2. Migrar el backend a Flyway con `V1__initial_schema.sql`.
3. Usar TanStack Query en todos los listados de la app.
4. Agregar `ErrorBoundary` global en la app.
5. Completar `03-frontend/` en el repo de contexto (design-system, navigation, state).
