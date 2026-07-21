# 00-status — Estado del proyecto SafeRoute

Esta carpeta es el punto de entrada cuando volvés al proyecto después de tiempo sin tocarlo. Si abrís el repo y no sabés por dónde arrancar, **empezá acá**.

---

## Cómo usar esta carpeta

Leé en este orden cuando vuelvas después de un descanso:

1. **`project-status.md`** → ¿En qué estado está cada parte del sistema hoy? Backend, app, docs.
2. **`next-steps.md`** → ¿Qué toca hacer YA? Los 3-5 tickets siguientes.
3. **`roadmap.md`** → ¿Hacia dónde vamos? Las fases hasta MVP y después.
4. **`changelog.md`** → ¿Qué se hizo antes? Historia de cambios importantes.
5. **`decisions/`** → ¿Por qué se decidió X? ADRs (Architecture Decision Records).

---

## Reglas de mantenimiento

Para que esta carpeta NO se convierta en documentación muerta, seguí estas reglas:

- **`next-steps.md` se pisa cada vez que se cierra un ticket.** Cuando terminás algo, movelo a `changelog.md` y agregá el siguiente en `next-steps.md`.
- **`project-status.md` se actualiza al final de cada sesión de trabajo real.** Solo estado concreto, no aspiraciones.
- **`roadmap.md` se revisa una vez al mes o cuando cambia el objetivo.** No se toca todos los días.
- **`changelog.md` es append-only.** Nunca se borra nada, solo se agrega.
- **`decisions/` — un ADR por decisión no obvia.** Si la decisión es "usé la librería estándar de X", no hace falta ADR. Si es "elegí Zustand y no Redux porque…", ahí sí.

---

## Última actualización

- **Fecha:** 2026-07-21
- **Última sesión de trabajo real (backend):** 2026-05-04
- **Última sesión de trabajo real (app):** 2026-05-04
- **Estado general:** MVP en construcción — ver `project-status.md`.
