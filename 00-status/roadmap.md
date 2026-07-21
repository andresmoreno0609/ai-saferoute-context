# Roadmap — SafeRoute hacia MVP

> Objetivo: **lanzar el MVP.**
> Este documento cambia una vez al mes o cuando cambia el objetivo del proyecto. No se toca todos los días.

---

## Definición de MVP

**Un MVP de SafeRoute que se pueda lanzar** significa poder entregarle la app a un colegio piloto con:

1. Padres (guardians) pueden registrarse, agregar hijos, y ver la ubicación en tiempo real de la ruta.
2. Conductores (drivers) pueden iniciar ruta, escanear NFC en abordaje/descenso, reportar eventos, y terminar ruta.
3. Notificaciones push funcionan (subida al bus, llegada, eventos importantes).
4. Admin puede ver rutas activas, aprobar conductores, gestionar vehículos.
5. Deployado en un ambiente estable con URL pública, no `192.168.1.8`.
6. Backup de datos y proceso de migración documentado (Flyway).

Todo lo que no esté en esa lista **no es MVP** — va a `Fase 3: Post-MVP`.

---

## Fase 0 — Higiene del código (1-2 días)

Objetivo: dejar los repos en estado presentable antes de agregar features. **Bloqueante** para Fase 1 porque sin esto no se puede iterar limpio.

### Backend
- [ ] Commitear cambio pendiente del Postman en `ai-saferoute-context`.

### App
- [ ] Sacar `console.log` y debug UI box de `ProfileScreen.tsx`.
- [ ] Mover base URL de la API a variable de entorno (`.env` + `expo-constants` o `react-native-dotenv`).
- [ ] Cablear en `App.tsx` las rutas faltantes de driver (7 pantallas) y admin (11 pantallas).
- [ ] Corregir `ProfileScreen` para usar `guardianApi.getProfile()` en vez de path hardcodeado.
- [ ] Agregar límite de retries al refresh automático de token en `src/lib/api.ts` (evitar loop infinito).

### Docs
- [ ] Actualizar `02-backend/postgis.md` — hoy dice PostGIS pero el backend usa lat/lng simples. **O borrarlo si ya no aplica.**

**Criterio de "done":** repos limpios, IP no hardcodeada, todas las pantallas alcanzables desde login, sin logs de debug en `main`.

---

## Fase 1 — Cerrar features MVP (2-3 semanas)

Objetivo: completar la funcionalidad que un colegio piloto necesita para USAR el sistema. Todo lo de acá está en la definición de MVP.

### Backend
- [ ] **Integrar FCM push notifications.** Reemplazar el TODO en `NotificationService.java:39`. Configurar Firebase project, agregar server key en env vars, implementar envío en `NotificationService`.
- [ ] **Migrar a Flyway.** Generar migración inicial desde el esquema actual (`V1__initial_schema.sql`). Cambiar `ddl-auto` a `validate` en dev también. Crear proceso para nuevas migraciones.
- [ ] Agregar tests mínimos de flujos críticos: login, registro guardian, crear student, iniciar ruta, escanear NFC.
- [ ] Agregar validación de datos de entrada consistente (Bean Validation) en todos los controllers.

### App
- [ ] **Integrar `@react-native-firebase/messaging`** para recibir push notifications. Registrar token en backend en login.
- [ ] **Usar TanStack Query** (ya está instalado) para todos los listados: children, routes, notifications, dashboard. Cache + refetch + estados de loading/error consistentes.
- [ ] Agregar `ErrorBoundary` global para no explotar la app en errores no capturados.
- [ ] Completar los flows de driver end-to-end: dashboard → start route → tracking → scan NFC → event form → end route.
- [ ] Completar el flow de admin para verificar drivers.
- [ ] Agregar spinner/skeleton loaders en listados (usando TanStack Query states).

### Docs
- [ ] Completar `03-frontend/` con: design-system.md, navigation.md, state-management.md, screens.md.
- [ ] Escribir README de setup local para backend y app (paso a paso, DB, env vars).

**Criterio de "done":** un usuario nuevo puede clonar los dos repos, seguir el README, correr backend + app localmente, registrar guardian, agregar hijo, ver ruta activa con GPS. Sin tocar código.

---

## Fase 2 — Deploy MVP (1-2 semanas)

Objetivo: **sacar SafeRoute de tu máquina.** Poner el sistema en un lugar accesible desde afuera para un piloto real.

### Infra backend
- [ ] Elegir hosting: AWS (RDS + EC2/App Runner/ECS) vs alternativas más baratas (Railway, Render, Fly.io). Escribir ADR con la decisión.
- [ ] Provisionar DB PostgreSQL en el hosting elegido, con backup automático diario.
- [ ] Deploy del backend con dominio + HTTPS.
- [ ] Configurar env vars de prod (DB, JWT secret, FCM key).
- [ ] Definir estrategia de logs (CloudWatch, Loki, Papertrail, etc.).

### Distribución app
- [ ] Cambiar base URL a URL de prod via `.env.production`.
- [ ] Configurar Expo EAS Build para builds de Android + iOS.
- [ ] Registrar app en Google Play Console y Apple Developer.
- [ ] Publicar en tracks internos (Play Internal Testing + TestFlight) para el colegio piloto.

### CI/CD
- [ ] GitHub Actions básico: en cada push a `main` del backend, build + tests + deploy automático.
- [ ] En cada push a `main` de la app, disparar build EAS.

**Criterio de "done":** el colegio piloto puede descargar la app desde Play/TestFlight, registrarse, y usar el sistema contra el backend en producción.

---

## Fase 3 — Post-MVP (a definir después del lanzamiento)

Todo lo que NO es crítico para el piloto va acá. Se prioriza según feedback del piloto.

Posibles items (sin orden ni compromiso):
- Panel web de admin (hoy solo hay mobile).
- Reportes exportables (PDF/Excel) de asistencia por estudiante/ruta.
- Notificaciones por email además de push.
- Geofencing (alertas si el bus sale de la zona esperada).
- Historial de rutas por padre (últimos 30 días).
- Chat conductor-padre.
- Modo offline en la app del conductor (sync después).
- Integración con billing / mensualidades.
- App para el rol de "escolta" (adulto que acompaña la ruta).
- Multi-tenant (varios colegios en la misma instancia).

---

## Estado actual del roadmap

- **Fase 0:** No iniciada.
- **Fase 1:** No iniciada.
- **Fase 2:** No iniciada.
- **Fase 3:** N/A.

Ver `next-steps.md` para los tickets concretos del momento.
