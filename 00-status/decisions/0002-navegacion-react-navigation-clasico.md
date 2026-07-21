# ADR 0002 — Navegación con React Navigation clásico (no Expo Router)

**Estado:** Aceptada
**Fecha:** 2026-07-21

---

## Contexto

Durante una sesión de debugging (2026-07-21) se descubrió que **20 pantallas de la app** (11 admin + 7 driver + 2 guardian) estaban escritas asumiendo **Expo Router** (usaban `useRouter` de `expo-router` y `useLocalSearchParams`), pero la app está configurada con **React Navigation clásico** en `App.tsx` (Stack Navigator con rutas registradas manualmente).

El síntoma: al intentar navegar a cualquiera de esas 20 pantallas, la app crasheaba con `ReferenceError: Property 'useRouter' doesn't exist`. En varios casos el import de `expo-router` ni siquiera existía — el código usaba `router.xxx` con `router` completamente undefined.

Este bug estaba latente porque solo 9 rutas estaban cableadas en `App.tsx` (todas Guardian). Cuando en la Fase 0 se empezaron a cablear las pantallas de admin y driver, el bug se materializó.

**El problema de fondo:** las pantallas fueron generadas o copiadas de templates asumiendo un stack (Expo Router) sin verificar que el proyecto usara ese stack. La decisión de "qué framework de navegación usar" **nunca se tomó explícitamente**. Cuando algo se pincha por un default no revisado, este es el resultado.

---

## Decisión

Se estandariza toda la app en **React Navigation 7 clásico** (`@react-navigation/native` + `@react-navigation/native-stack`), con:

- **`App.tsx` como único punto de registro de rutas.**
- **Todas las pantallas reciben `navigation` y `route` como props** (no hooks tipo `useRouter`).
- **Params via `route.params`**, no `useLocalSearchParams`.
- **Nombres de ruta en PascalCase** siguiendo el patrón `RoleContext + Screen` (ej: `AdminUsers`, `DriverStartRoute`, `GuardianChildren`).

Pattern canónico:

```typescript
export default function XxScreen({ navigation, route }: { navigation?: any; route?: any }) {
  const { id } = route?.params || {};

  const handleBack = () => navigation?.goBack();
  const handleGoNext = () => navigation?.navigate('OtherScreen', { id });

  // ...
}
```

Se descartó Expo Router — pero se deja el camino abierto para migrar en Fase 3 (post-MVP) si aparece una razón concreta (deep linking universal, web version, etc.).

---

## Consecuencias

### Positivas

- **Consistencia inmediata.** Todo el equipo (o cualquier agente asistiendo) usa un solo modelo.
- **Zero magia.** Las rutas están en un solo archivo (`App.tsx`), fácil de auditar.
- **Deep linking manual pero explícito** (via `linking` prop de NavigationContainer si se necesita).
- **Migración fue mecánica** — patrón claro, refactor de 20 archivos en una sesión.

### Negativas

- **`App.tsx` va a crecer.** Con 29 pantallas registradas y creciendo, mantenerlo será feo. Ya se debería considerar dividirlo en `AuthNavigator`, `AdminNavigator`, `DriverNavigator`, `GuardianNavigator` anidados.
- **Sin file-based routing.** Agregar una pantalla requiere: crear el archivo + agregar import + agregar `Stack.Screen` en `App.tsx`. Tres pasos, no uno.
- **Deep linking a rutas anidadas es verboso.** Si se lanza el MVP y aparece la necesidad de deep links (`saferoute://route/123`), habrá que agregar `linking` config a mano.

### Mitigaciones adoptadas

- **Prohibición explícita** de importar `expo-router` en cualquier pantalla nueva. Si aparece en un review, se rechaza.
- Se va a evaluar la partición de `App.tsx` en navegadores por rol como próxima iteración (post-MVP o si `App.tsx` supera las 50 rutas).

---

## Alternativas descartadas

### Migrar toda la app a Expo Router

**Por qué no ahora:** el estándar de Expo 54 es file-based routing, y hay valor a largo plazo. Pero:
- Requeriría reescribir `App.tsx`, migrar `src/screens/` a estructura de carpeta `app/`, y actualizar las 5 pantallas Guardian que YA funcionan.
- El MVP está a semanas de estar listo. Migrar la navegación entera meses antes del lanzamiento no aporta valor de negocio.
- La migración es reversible en cualquier momento. Se puede revisar en Fase 3.

### Mezclar Expo Router para admin/driver y clásico para guardian

**Por qué no:** híbridos de navegación son notoriamente frágiles. Deep linking cruzado no funcionaría, y cualquier programador que llegara nuevo no entendería qué patrón aplicar. Consistencia > flexibilidad.

### Usar React Navigation Bottom Tabs desde el vamos

**Por qué no todavía:** los flujos actuales están pensados como stack lineal (login → dashboard → detail). Bottom tabs se puede agregar por rol cuando sean necesarios (probablemente ya en Fase 1 para el driver: Dashboard | Mis Rutas | Perfil).

---

## Revisión

Esta decisión se revisa si:
- `App.tsx` supera las **50 pantallas registradas**.
- Aparece un requerimiento fuerte de **deep linking universal** o **versión web** de la app.
- El equipo crece y la falta de file-based routing empieza a doler en las code reviews.

Hasta entonces: **React Navigation clásico es el estándar de SafeRoute app.**

---

## Referencias

- Refactor completo del bug: memoria engram `saferoute/navigation-refactor` (2026-07-21).
- Pattern de referencia: `src/screens/guardian/ChildFormScreen.tsx`.
- Registro central de rutas: `App.tsx` en `saferoute-app`.
