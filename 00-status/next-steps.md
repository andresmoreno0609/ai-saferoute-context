# Next Steps — SafeRoute

> **Este archivo se pisa seguido.** Cuando cerrás un ticket, movelo a `changelog.md` y agregá el siguiente acá.
> Última actualización: **2026-07-21**

---

## Contexto rápido

Estás en **Fase 0 (Higiene)** del roadmap. El objetivo de esta fase es dejar los repos limpios antes de agregar features. Los tickets abajo salen de esa fase.

Cuando termines los 5 tickets de acá, actualizá este archivo con los 5 tickets siguientes de la Fase 1.

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

## Ticket 2 — Mover base URL de la API a variables de entorno

**Repo:** `saferoute-app`
**Archivos:** `src/lib/api.ts:19`, `src/screens/shared/ProfileScreen.tsx:11`, nuevo `.env` y `.env.example`
**Prioridad:** Alta
**Estimación:** 1h

**Qué hacer:**
- Instalar `expo-constants` (ya viene con Expo) o `react-native-dotenv`.
- Crear `.env` con `EXPO_PUBLIC_API_URL=http://192.168.1.8:8080/api/v1`.
- Crear `.env.example` con la misma clave pero valor dummy (para el repo).
- Agregar `.env` a `.gitignore` (verificar primero).
- Reemplazar los usos de la URL hardcodeada por `process.env.EXPO_PUBLIC_API_URL`.
- Documentar el paso en un README de setup (esto se completa en un ticket posterior).

**Criterio de "done":** grep por `192.168.1.8` en toda la app devuelve solo `.env`.

---

## Ticket 3 — Cablear navigation completa (driver + admin)

**Repo:** `saferoute-app`
**Archivos:** `App.tsx`
**Prioridad:** Alta
**Estimación:** 2-3h

**Qué hacer:**
- Diseñar la estructura de navegación completa. Sugerencia:
  - Stack raíz con `AuthNavigator` (Login, Register) y `AppNavigator` (post-login).
  - `AppNavigator` con tabs bottom o drawer según rol del usuario (Guardian/Driver/Admin).
  - Cada rol tiene su propio stack anidado con sus pantallas.
- Registrar en `App.tsx` las 20+ pantallas hoy inalcanzables:
  - Driver: Dashboard, MyRoutes, StartRoute, EndRoute, Tracking, ScanNFC, EventForm, CompleteProfile.
  - Admin: Dashboard, Users, UserDetail, UserForm, Routes, RouteDetail, RouteForm, Vehicles, VehicleDetail, VehicleForm, VerifyDriver.
- Verificar redirección post-login según rol.

**Criterio de "done":** un usuario que hace login como driver o admin llega a su dashboard y puede navegar a todas sus pantallas.

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
- Commitear con mensaje descriptivo: `docs(api): update Postman with roles[] and info-validate endpoints`.
- Push.

**Criterio de "done":** `git status` en `ai-saferoute-context` limpio.

---

## Después de estos 5

Cuando termines la Fase 0, los próximos 5 tickets van a salir de la **Fase 1**:

1. Integrar FCM push notifications (backend + app).
2. Migrar el backend a Flyway con `V1__initial_schema.sql`.
3. Usar TanStack Query en todos los listados de la app.
4. Agregar `ErrorBoundary` global en la app.
5. Completar `03-frontend/` en el repo de contexto (design-system, navigation, state).

Pero eso lo revisamos cuando lleguemos.
