# Changelog — SafeRoute

> Historial de cambios importantes agrupados por fecha. **Append-only** — nunca se borra nada.
> Formato: `YYYY-MM-DD` — descripción con contexto de POR QUÉ, no solo QUÉ.

---

## 2026-07-21 — Wiki de retorno + Ticket 2 y 3 de Fase 0 completos

### `ai-saferoute-context`

- Creada la carpeta `00-status/` como punto de entrada al retomar el proyecto. Incluye `project-status.md`, `roadmap.md`, `next-steps.md`, `changelog.md`, y `decisions/`. **Motivación:** después de 2+ meses sin tocar SafeRoute, era imposible saber dónde había quedado el trabajo. La docs técnica estaba (schema, architecture, API contracts) pero faltaba la capa de "estado".
- Escrito **ADR 0002** — decisión de estandarizar la app en React Navigation clásico (no Expo Router). Documentado durante el fix del bug de useRouter.

### `saferoute-app`

**Ticket 2 (parche parcial) — IP hardcodeada actualizada:**

Descubierto durante prueba en device físico: **9 archivos** de la app tenían la URL del backend hardcodeada como `192.168.1.8:8080` (IP vieja de otra red). Actualizada a la IP actual `192.168.40.9:8080` en los 9 archivos. **Este es un parche, no la solución definitiva** — el `.env` sigue pendiente (Ticket 2 completo). Cuando cambie de red, mismo problema.

**Ticket 3 (completo) — Navegación arreglada + 20 pantallas migradas:**

Descubierto durante prueba en device físico como bug latente: **20 pantallas de la app** (11 admin + 7 driver + 2 guardian) fueron escritas asumiendo Expo Router (`useRouter`, `useLocalSearchParams`), pero la app usa React Navigation clásico. Los símptomas: `ReferenceError: Property 'useRouter' doesn't exist` al navegar a esas pantallas. En varios casos el `import` de expo-router ni siquiera existía — `router` era completamente undefined.

Migración de 20 pantallas hecha en una pasada:
- Removidos todos los `import { useRouter, useLocalSearchParams } from 'expo-router';`.
- Cambio de signatura: `function XxScreen()` → `function XxScreen({ navigation, route }: { navigation?: any; route?: any })`.
- `router.push('/path')` → `navigation?.navigate('RouteName', params)`.
- `router.back()` → `navigation?.goBack()`.
- `useLocalSearchParams<{ x: string }>()` → `route?.params`.
- Bonus: `DriverDashboardScreen` (no estaba en la lista) también estaba roto — fixed.
- Bonus: varias pantallas tenían `useNavigation` importado del paquete equivocado (`@react-navigation/native-stack` en vez de `/native`) — removidos.

`App.tsx` actualizado: **29 Stack.Screen** registradas (2 auth + 8 guardian + 11 admin + 8 driver). Antes eran 9 rutas.

Verificación: `useRouter | expo-router | useLocalSearchParams | router.` en `src/` = **0 matches**.

**Motivación de la sesión:** Andres estaba probando la app en su celular físico y no podía loguearse. Un diagnóstico paso a paso (chequeo de IPs, puertos, firewall) reveló primero el problema de IP hardcodeada, y después el bug latente de `useRouter`. Ambos se resolvieron en la sesión.

---

## 2026-05-04 — Última sesión de trabajo real

### Backend
- **Notifications:** agregados tipos `ERROR` y `WARNING` para clasificar alertas.
- **Guardian:** endpoint GET para que un guardian obtenga un student individual (antes solo tenía listado).

### App
- **Guardian dashboard polish:** colores para tipos de notificación (matching backend), refactor de tipografía/spacing/badges.
- **Design system aplicado consistente** en pantallas de guardian.
- **Feature de foto del estudiante** en el formulario de gestión de hijos.
- **UX de flujo de children mejorado.**
- Sacados los últimos "debug in loading" antes de commitear (pero quedó debug en `ProfileScreen` — pendiente en Fase 0).

---

## 2026-05-02 — Sesión intensa de estabilización

### Backend
- **Migración de PostGIS a columnas lat/lng simples.** Después de ida y vuelta con JTS Point, WKTReader y PointAttributeConverter, se decidió que el front envía números simples y el backend guarda columnas `latitude`/`longitude` planas. **Nota:** `02-backend/postgis.md` quedó desactualizado con este cambio.
- **Deserialización de enum `Relationship` con valores en español** (@JsonCreator + Jackson deserializer + converter case-insensitive).
- **Permisos:** GUARDIAN y DRIVER ahora pueden crear/actualizar students (antes solo ADMIN).
- **Refactor:** lógica de guardian-students movida a UseCases.
- Nuevos endpoints de guardian-student con verificación de token.

### App
- **Auto-refresh de token:** access token se renueva automáticamente al expirar.
- **Logout funcional** con redirect y autoload de hijos.
- **Perfil con datos de guardian** (documento, dirección, ocupación, contacto de emergencia) — solo lectura.
- **Pantallas de gestión de hijos** (Flujo 17): ChildrenScreen, ChildDetailScreen, ChildFormScreen.
- Mapeo de valores de `Relationship` frontend → backend.
- **Ciclo intensivo de debug del perfil** — quedó código de debug committeado a `main` (deuda técnica).

---

## 2026-05-01 — Endpoints multi-rol y auto-perfiles

### Backend
- **`GET /me` refactorizado:** devuelve solo info del user, no tokens.
- **Auto-creación de perfil guardian/driver on register:** al registrarse, si el usuario elige un rol, se crea el perfil correspondiente automáticamente.
- **Endpoints become-driver / become-guardian:** un usuario existente puede agregarse un rol adicional.
- **Campo `infoValidate`** en perfiles guardian/driver + endpoints dedicados para setear validación.
- **Fix:** `BadCredentialsException` en login devuelve error friendly en vez de 500.
- Postman collection regenerada completa.

### App
- **`CompleteProfileScreen` (Flujo 02C)** con 7 campos.
- **Endpoint específico de notifications para guardian.**
- **Navigation base + auth store Zustand + guardian dashboard.**

---

## 2026-04-30 — Documentación de flujos y pantallas

### App
- 22 flujos de usuario documentados en `docs/flujos.md`.
- TODO de 25+ pantallas en `docs/TODO_PANTALLAS.md`.

---

## 2026-04-13 — Contexto remoto para agentes de IA

### Backend
- Movida la carpeta `docs/` local a este repo `ai-saferoute-context`.
- Agregado `opencode.json` para cargar contextos remotos vía URL.

**Motivación:** centralizar la documentación de contexto en un solo repo consumible por múltiples agentes (OpenCode, Cursor, Claude).

---

## 2026-04-10 — Multi-rol

### Backend
- `UserEntity.roles` cambió de single `Role` a `Set<Roles>`.
- JWT ahora soporta múltiples roles en el claim.
- `GuardianEntity` tiene `user_id` como FK a `UserEntity`.
- Todos los use cases y services actualizados.

**Motivación:** un mismo usuario puede ser guardian de sus hijos Y driver contratado — el modelo anterior no lo permitía.

---

## 2026-04-07 — README del backend + campos adicionales de guardian

### Backend
- README con arquitectura, features, endpoints.
- Guardian con campos: `documentNumber`, `birthDate`, `address`, `emergencyContact`, `occupation`.

---

## 2026-04-06 — Sistema NFC + campos adicionales de student

### Backend
- **Sistema NFC completo:** endpoints assign, deactivate, history, scan + DTOs + UseCases + Adapter + Controller.
- **Student con campos:** `birthDate`, `grade`, `emergencyContact`, `medicalInfo`, `photoUrl`, `studentCode`.
- **Driver con campos:** `documentNumber`, `birthDate`, `address`, `license`, `emergency`, etc.
- **Convención:** usar `@Builder` en DTO records en vez de builders manuales.

---

## Antes de 2026-04-06

Bootstrap del proyecto. Estructura inicial hexagonal, entidades base (User, Driver, Guardian, Student, Vehicle, Route, Stop), auth con JWT, estructura de módulos, PostgreSQL, Spring Boot 4.0.5.
