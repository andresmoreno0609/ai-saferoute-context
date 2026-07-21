# Project Status — SafeRoute

> Fecha de este snapshot: **2026-07-21**
> Última sesión de trabajo real: **2026-05-04** (backend y app)
> Fase actual: **Construcción de MVP** (pre-lanzamiento)

---

## Resumen ejecutivo

SafeRoute tiene tres repos activos: backend Spring Boot, app React Native/Expo, y este repo de contexto. **Ambos repos de código están limpios y pusheados al remoto — no hay trabajo perdido.** El backend tiene un núcleo sólido de 16 módulos y ~81 use cases. La app tiene 29 pantallas y design system, pero solo la mitad está cableada en el navegador.

Lo que falta para MVP se lista en `roadmap.md` y `next-steps.md`.

---

## Backend — `saferoute`

**Ubicación:** `C:\Users\User\Documents\SafeRoute\Repositories\saferoute`
**Branch actual:** `main` — limpio, sincronizado con `origin/main`.
**Stack:** Spring Boot 4.0.5, Java 21, JPA, Spring Security, JWT, PostgreSQL.
**Arquitectura:** Hexagonal (Controller → Adapter → UseCase → Service/Repository → Entity).

### Módulos implementados (16)

| Módulo | Estado | Notas |
|---|---|---|
| `auth` | Funcional | 5 use cases. JWT access 15min + refresh 7d. Multi-role. |
| `user` | Funcional | 5 use cases. CRUD + role assignment. |
| `guardian` | Funcional | 13 use cases. Recientemente expandido con endpoints de students. |
| `student` | Funcional | 5 use cases. Lat/lng como columnas simples (post-refactor JTS). |
| `studentguardian` | Funcional | 6 use cases. Mapeo N:N. |
| `driver` | Funcional | 6 use cases. Perfiles + disponibilidad + verificación. |
| `driverdocument` | Funcional | 2 use cases. Documentos LICENCIA, CEDULA. |
| `vehicle` | Funcional | 2 use cases. Registro + estado. |
| `vehicledocument` | Funcional | 2 use cases. SOAT, SEGURO, etc. |
| `route` | Funcional | 5 use cases. Estados PENDING → IN_PROGRESS → COMPLETED. |
| `stop` | Funcional | 5 use cases. Paradas ordenadas por ruta. |
| `gps` | Funcional | 3 use cases. Tracking en tiempo real + histórico. |
| `studentevent` | Funcional | 5 use cases. Eventos BOARD/ARRIVAL/DROP con contexto de ruta. |
| `studentnfc` | Funcional | 8 use cases. Asignación de tarjetas + tracking activo. |
| `notification` | **Parcial** | 7 use cases. Guarda en DB. **FCM push NO integrado** (ver next-steps). |
| `observation` | Funcional | 5 use cases. Reporte de incidentes con metadata. |

### Gaps del backend

1. **FCM push no implementado.** `NotificationService.java:39` tiene un TODO. Las notificaciones se persisten pero no se envían push real.
2. **Sin migraciones (Flyway/Liquibase).** Se usa `spring.jpa.hibernate.ddl-auto=update` en dev y `validate` en prod. Riesgoso para evolución de esquema en producción.
3. **README sin "getting started" concreto.** Faltan pasos de setup local (build, DB, env vars).
4. **Sin tests automatizados visibles** en el flujo de CI (verificar en la Fase 1).

### Configuración por perfil

- **dev / default:** DB `localhost:5432/saferoute` (postgres/root), logs DEBUG, JWT 15min/7d.
- **prod:** DB por env vars (`${DB_URL}`, `${DB_USERNAME}`, `${DB_PASSWORD}`), Hibernate `validate`, cookies seguras (HttpOnly + Secure), logs WARN.

---

## App móvil — `saferoute-app`

**Ubicación:** `C:\Users\User\Documents\SafeRoute\Repositories\saferoute-app`
**Branch actual:** `main` — limpio, sincronizado con `origin/main`.
**Stack:** Expo 54.0.33, React Native 0.81.5, React 19.1.0, TypeScript, React Navigation 7, Zustand 5, AsyncStorage, TanStack React Query (instalado, no usado).

### Pantallas implementadas (29)

| Rol | Cantidad | Cableadas en navigation |
|---|---|---|
| Auth (Login, Register) | 2 | Sí (2/2) |
| Guardian | 8 | Parcial (5/8) |
| Driver | 7 | **No (0/7)** |
| Admin | 11 | **No (0/11)** |
| Shared (Profile) | 1 | Sí (1/1) |

**Solo 9 rutas están hardcodeadas en `App.tsx`.** Los flows de driver y admin están construidos como pantallas pero son inalcanzables.

### Design system

- Documentado en `docs/design-system.md` y `docs/design-specs.md`.
- 12 componentes base en `src/components/index.tsx`: Button, Input, Card, Header, BottomNav, Sidebar, Avatar, Badge, EmptyState, Spinner, ListItem, Divider.

### State management

- **Un solo store Zustand:** `src/store/authStore.ts` (user, token, isLoading + setAuth/logout/restoreAuth).
- Persistencia en AsyncStorage.
- No hay stores para driver, guardian, route, notifications. Cada pantalla fetchea directo.

### API client

- **Ubicación:** `src/lib/api.ts` (fetch nativo, no Axios).
- **Base URL:** `http://192.168.1.8:8080/api/v1` **hardcodeada**.
- Refresh automático de token en 401. **Sin límite de retries — riesgo de loop infinito.**
- 11 namespaces: `authApi`, `usersApi`, `driversApi`, `vehiclesApi`, `routesApi`, `studentsApi`, `guardianApi`, `driverApi`, `eventsApi`, `notificationsApi`, `dashboardApi`.

### Red flags de la app

1. **Debug code en `main`.** `ProfileScreen.tsx` tiene 8 `console.log()` y un box amarillo de debug visible en la UI (líneas 74, 83, 88, 97, 118).
2. **IP hardcodeada.** `src/lib/api.ts:19` y `ProfileScreen.tsx:11`. Sin `.env`.
3. **TanStack React Query instalado pero SIN USAR.** Desperdicio de dep + falta de caching de listados.
4. **Sin README de setup.**
5. **Sin tests, sin scripts de test, sin Jest configurado.**
6. **ProfileScreen usa paths hardcodeados** en vez de `guardianApi.getProfile()`. Inconsistencia.
7. **Sin manejo global de errores** (no hay ErrorBoundary, cada pantalla implementa try/catch a mano).

---

## Repo de contexto — `ai-saferoute-context`

**Ubicación:** `C:\Users\User\Documents\SafeRoute\Repositories\ai-saferoute-context`
**Branch actual:** `master` — hay 1 archivo modificado sin commitear.

### Cobertura de documentación

| Área | Estado |
|---|---|
| Overview del producto (`01-general/`) | Completo. |
| Backend architecture (`02-backend/architecture.md`) | Completo (14KB). |
| PostGIS (`02-backend/postgis.md`) | Completo (14KB) — pero ojo, el backend ya migró de PostGIS a columnas lat/lng simples. **Este doc está desactualizado.** |
| Frontend (`03-frontend/`) | **Stub.** Solo README con TODOs. |
| DB schema (`04-database/schema.md`) | Completo (12.8KB). |
| NFC (`04-database/nfc.md`) | Completo. |
| API contracts (`05-shared/`) | Completo — Postman collection actualizada. |
| Roadmap / status / next-steps | **Nuevo — esta carpeta 00-status/**. |

### Cambios pendientes de commit

- `05-shared/SafeRoute-API-Postman.json` — modificado con endpoints de become-driver / become-guardian y refactor `role` → `roles[]`. Falta commitear.

---

## Ecosistema completo

```
SafeRoute/
├── saferoute/               ← Backend Spring Boot (branch main)
├── saferoute-app/           ← App React Native (branch main)
└── ai-saferoute-context/    ← Docs para agentes de IA (branch master) ← estás acá
```

Los tres repos apuntan a GitHub. Los tres están sincronizados. No hay trabajo local no pusheado excepto el Postman modificado.
