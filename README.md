# 📚 ai-saferoute-context

> Contextos técnicos documentados para el desarrollo de SafeRoute - Sistema de gestión de transporte escolar.

---

## 🎯 Propósito

Este repositorio centraliza la documentación técnica, arquitectura, patrones de código y decisiones de diseño para el desarrollo de SafeRoute. Está diseñado para ser consumido por agentes de IA (como OpenCode, Cursor, etc.) para mantener contexto consistente del proyecto.

---

## 📁 Estructura

```
ai-saferoute-context/
├── 00-status/               ← ⭐ EMPEZAR ACÁ al retomar el proyecto
│   ├── README.md            ← Guía de uso de esta carpeta
│   ├── project-status.md    ← Estado real de backend, app y docs
│   ├── roadmap.md           ← Fases hasta MVP y después
│   ├── next-steps.md        ← Los 3-5 próximos tickets
│   ├── changelog.md         ← Historial de cambios importantes
│   └── decisions/           ← ADRs — decisiones arquitectónicas
│
├── 01-general/              ← Contexto general (todas las apps)
│   ├── app-overview.md      ← ¿Qué es SafeRoute?
│   └── stack.md             ← Stack tecnológico
│
├── 02-backend/              ← Contextos backend
│   ├── architecture.md      ← Arquitectura y patrones de código
│   └── postgis.md           ← PostGIS y datos geoespaciales (⚠️ desactualizado)
│
├── 03-frontend/             ← Contextos frontend (pendiente completar)
│
├── 04-database/             ← Contextos de base de datos
│   ├── schema.md            ← Esquema y entidades
│   └── nfc.md               ← Sistema NFC
│
└── 05-shared/               ← Compartido entre front y back
    ├── api-contracts.md     ← Contratos de API
    └── SafeRoute-API-Postman.json  ← Colección Postman para testing
```

---

## 🧭 ¿Volvés al proyecto después de tiempo?

Abrí primero **`00-status/README.md`** y seguí el orden que indica ahí. Está diseñado para que en 5-10 minutos sepas dónde estás y qué toca hacer.

---

## 🚀 Uso con OpenCode

En cada proyecto, configurar `.opencode/opencode.json` para cargar contextos automáticamente:

```json
{
  "contextPaths": [
    "https://raw.githubusercontent.com/andresmoreno0609/ai-saferoute-context/main/01-general/app-overview.md",
    "https://raw.githubusercontent.com/andresmoreno0609/ai-saferoute-context/main/02-backend/architecture.md",
    "docs/contextLocal.md"
  ]
}
```

---

## 📝 Contribución

1. Crear/actualizar archivos de contexto en la carpeta correspondiente
2. Mantener consistencia con el proyecto SafeRoute
3. Actualizar este README si se agrega nueva documentación
4. **IMPORTANTE:** Al agregar nuevos endpoints en el código, actualizar `05-shared/SafeRoute-API-Postman.json`

---

## 📅 Última Actualización

- **Fecha:** 2026-07-21
- **Versión:** 1.2.0
- **Proyecto:** SafeRoute - Sistema de gestión de transporte escolar
- **Cambio de esta versión:** agregada carpeta `00-status/` con wiki de estado del proyecto (project-status, roadmap, next-steps, changelog, ADRs).

---

*Documentos generados desde SafeRoute - Repositorio de contextos para IA*