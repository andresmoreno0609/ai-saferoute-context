# 📚 ai-saferoute-context

> Contextos técnicos documentados para el desarrollo de SafeRoute - Sistema de gestión de transporte escolar.

---

## 🎯 Propósito

Este repositorio centraliza la documentación técnica, arquitectura, patrones de código y decisiones de diseño para el desarrollo de SafeRoute. Está diseñado para ser consumido por agentes de IA (como OpenCode, Cursor, etc.) para mantener contexto consistente del proyecto.

---

## 📁 Estructura

```
ai-saferoute-context/
├── 01-general/              ← Contexto general (todas las apps)
│   ├── app-overview.md     ← ¿Qué es SafeRoute?
│   └── stack.md            ← Stack tecnológico
│
├── 02-backend/              ← Contextos backend
│   ├── architecture.md     ← Arquitectura y patrones de código
│   └── postgis.md          ← PostGIS y datos geoespaciales
│
├── 03-frontend/            ← Contextos frontend (pendiente)
│
├── 04-database/            ← Contextos de base de datos
│   ├── schema.md           ← Esquema y entidades
│   └── nfc.md              ← Sistema NFC
│
└── 05-shared/              ← Compartido entre front y back
    ├── api-contracts.md   ← Contratos de API
    └── SafeRoute-API-Postman.json  ← Colección Postman para testing
```

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

---

## 📅 Última Actualización

- **Fecha:** 2026-04-13
- **Versión:** 1.1.0
- **Proyecto:** SafeRoute - Sistema de gestión de transporte escolar

---

*Documentos generados desde SafeRoute - Repositorio de contextos para IA*