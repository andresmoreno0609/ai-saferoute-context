# ADR 0001 — Arquitectura hexagonal en el backend

**Estado:** Aceptada
**Fecha:** ~2026-03 (bootstrap del proyecto)
**Documentado retroactivamente:** 2026-07-21

---

## Contexto

Al arrancar el backend de SafeRoute había que elegir cómo estructurar el código Java + Spring Boot. Las opciones sobre la mesa eran:

1. **Arquitectura en capas clásica** (Controller → Service → Repository).
2. **Arquitectura hexagonal** (Adapter → UseCase → Domain → Repository).
3. **Clean Architecture / Screaming Architecture** (paquetes por feature con capas adentro).

SafeRoute tiene muchas reglas de negocio no triviales:
- Un guardian puede administrar múltiples students.
- Un driver debe estar validado (documentos + `infoValidate`) para tomar rutas.
- Los events de student tienen máquina de estados (BOARD → ARRIVAL → DROP).
- NFC scans tienen validación cruzada de student activo + ruta en curso + driver asignado.

Con capas clásicas, esa lógica termina en Services de 500 líneas mezclando validación, orquestación y persistencia.

---

## Decisión

Se adoptó **arquitectura hexagonal por feature**, con esta estructura por módulo:

```
{feature}/
├── adapter/       ← Orquesta use cases, hace mapping HTTP ↔ dominio
├── controller/    ← REST controllers (endpoints)
├── usecase/       ← Un archivo por caso de uso, extiende UseCaseAdvance
├── service/       ← Servicios de soporte del feature (opcional)
└── dto/           ← DTOs del feature (opcional)
```

Y una capa `common/` con lo compartido:

```
common/
├── entity/        ← Entidades JPA
├── repository/    ← JpaRepository interfaces
├── service/       ← Servicios cross-cutting
├── dto/           ← DTOs compartidos
├── usecase/       ← Template base UseCaseAdvance
├── config/        ← Security, Jackson, config
└── security/      ← JWT filters, auth providers
```

**Regla:** cada endpoint del controller invoca a un adapter, el adapter invoca uno o varios use cases, los use cases hablan con repositorios y servicios. **Los controllers NUNCA llaman directo a services o repositorios.**

---

## Consecuencias

### Positivas

- **Un caso de uso, un archivo.** Fácil de encontrar, fácil de testear, fácil de reemplazar.
- **Los controllers quedan finos** — solo mapping HTTP + delegación al adapter.
- **Reglas de negocio aisladas** — el use case no sabe si se está invocando desde REST, un cron, o un test.
- **Escalable a más módulos** — agregar un nuevo feature es replicar la estructura, no toquetear services gigantes existentes.
- **`UseCaseAdvance` template** obliga a estructura consistente.

### Negativas

- **Verboso.** Un CRUD trivial requiere Controller + Adapter + 4 UseCases + Repository + Entity + DTO. Es mucho archivo para poca lógica.
- **Curva de aprendizaje** para alguien nuevo que espera capas clásicas de Spring.
- **Boilerplate.** Se resiente sobre todo en features simples que serían 20 líneas en capas clásicas.

### Mitigaciones adoptadas

- Se usan **`@Builder` en records** para reducir código de construcción de DTOs (convención documentada en `02-backend/architecture.md`).
- Para features con lógica mínima (vehicle documents, driver documents) se aceptan 1-2 use cases en vez de forzar el patrón completo.
- Los adapters pueden orquestar múltiples use cases en operaciones compuestas (por ejemplo `RegisterAdapter` invoca `CreateUser` + `CreateGuardianProfile` + `GenerateTokens`).

---

## Alternativas descartadas

### Capas clásicas (Controller → Service → Repository)

**Por qué no:** con 16 módulos y features complejos (NFC, tracking, events con máquina de estados), los services habrían crecido a >500 líneas mezclando orquestación con lógica de negocio. Difícil de testear en aislamiento.

### Clean Architecture pura (con entidades de dominio separadas de entidades JPA)

**Por qué no:** duplicaría el modelo (Entity JPA + Entity de dominio) y agregaría mapeo entre las dos capas. Para un proyecto MVP en solitario, el costo/beneficio no cierra. Si el proyecto crece a un equipo o se llega a necesitar swap del ORM, se puede evolucionar hacia esto sin romper todo — la separación adapter/usecase ya está.

### DDD con bounded contexts como módulos separados de Maven

**Por qué no:** overkill absoluto para un MVP. Se dejó como opción futura si el proyecto crece a múltiples equipos.

---

## Revisión

Esta decisión se revisa si:
- El boilerplate empieza a frenar la velocidad de desarrollo (más de un día perdido por semana en estructura).
- Se necesita compartir dominio con otro cliente (backoffice web, batch processor, etc.) y aparece la necesidad de separar dominio puro de JPA.
- El equipo crece a más de 3 personas y la consistencia del patrón se rompe.

Al día de hoy (2026-07-21) la decisión sigue siendo válida y el patrón se está sosteniendo bien.
