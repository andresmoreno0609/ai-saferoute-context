# 🏷️ Sistema NFC - SafeRoute

---

## 1. Visión General

El sistema NFC de SafeRoute permite **identificar estudiantes** mediante tarjetas o tags NFC durante el transporte escolar. El sistema registra automáticamente eventos (BOARD, DROP) cuando se detecta un tag NFC.

---

## 2. Componentes del Sistema

### 2.1 Entidad: `student_nfc`

| Campo | Tipo | Descripción |
|-------|------|-------------|
| id | UUID | PK |
| student_id | UUID | FK → students(id) |
| nfc_uid | VARCHAR(100) | UID único de la tarjeta NFC |
| is_active | BOOLEAN | ¿NFC activo actualmente? |
| assigned_at | TIMESTAMP | Fecha de asignación |
| deactivated_at | TIMESTAMP | Fecha de desactivación (null si activo) |
| assigned_by | UUID | Usuario que realizó la asignación |
| notes | VARCHAR(500) | Notas adicionales |

---

## 3. Reglas de Negocio

### 3.1 Reglas Principales

1. **Un solo NFC activo por estudiante** — Al asignar un nuevo NFC, el anterior se desactiva automáticamente
2. **Histórico preserved** — Todos los NFCs asignados se mantienen en la tabla (soft delete no aplica a NFCs)
3. **UID único** — No puede haber dos estudiantes con el mismo NFC UID activo
4. **Operaciones permitidas:**
   - Asignar NFC a estudiante
   - Desactivar NFC manualmente
   - Consultar NFC activo de un estudiante
   - Ver historial de NFCs de un estudiante

---

## 4. Flujo de Uso

### 4.1 Asignar NFC a Estudiante

```
1. ADMIN o DRIVER registra un NFC
   └─> POST /api/v1/students/{id}/nfc
   
2. Sistema verifica que el UID no esté activo en otro estudiante
   
3. Si hay NFC anterior activo → se desactiva automáticamente
   
4. Se crea nuevo registro en student_nfc
   └─> is_active = true
   └─> assigned_at = NOW()
   
5. Retorna el NFC asignado
```

### 4.2 Escanear NFC (Detectar Estudiante)

```
1. App móvil lee un tag NFC
   └─> POST /api/v1/nfc/scan
   
2. Sistema busca NFC por UID
   └─> WHERE nfc_uid = :uid AND is_active = true
   
3. Si existe:
   └─> Retorna datos del estudiante
   
4. Si no existe:
   └─> Retorna error: "NFC no registrado o inactivo"
```

### 4.3 Consultar Historial de NFCs

```
1. Solicitar historial de un estudiante
   └─> GET /api/v1/students/{id}/nfc/history
   
2. Sistema retorna todos los registros (activos e inactivos)
   └─> Ordenados por assigned_at DESC
   
3. Incluye información de quién asignó y cuándo
```

---

## 5. Endpoints API

### 5.1 Asignar NFC a Estudiante

```
POST /api/v1/students/{studentId}/nfc

Request:
{
  "nfcUid": "04A1B2C3D4E5F6",
  "notes": "Tarjeta asignada al estudiante"
}

Response (201):
{
  "id": "uuid",
  "studentId": "uuid",
  "nfcUid": "04A1B2C3D4E5F6",
  "isActive": true,
  "assignedAt": "2026-04-13T10:00:00Z",
  "notes": "..."
}
```

### 5.2 Obtener NFC Activo

```
GET /api/v1/students/{studentId}/nfc

Response (200):
{
  "id": "uuid",
  "nfcUid": "04A1B2C3D4E5F6",
  "isActive": true,
  "assignedAt": "2026-04-13T10:00:00Z"
}
```

### 5.3 Ver Historial NFC

```
GET /api/v1/students/{studentId}/nfc/history

Response (200):
[
  {
    "nfcUid": "04A1B2C3D4E5F6",
    "isActive": true,
    "assignedAt": "2026-04-13T10:00:00Z"
  },
  {
    "nfcUid": "04Z9Y8X7W6V5U4",
    "isActive": false,
    "assignedAt": "2026-01-15T08:00:00Z",
    "deactivatedAt": "2026-04-13T10:00:00Z"
  }
]
```

### 5.4 Escanear NFC

```
POST /api/v1/nfc/scan

Request:
{
  "nfcUid": "04A1B2C3D4E5F6"
}

Response (200):
{
  "studentId": "uuid",
  "studentName": "Juan Pérez",
  "schoolGrade": "5to Primaria"
}

Response (404):
{
  "error": "NFC no registrado o inactivo"
}
```

### 5.5 Desactivar NFC

```
DELETE /api/v1/students/{studentId}/nfc/{nfcId}

Response (204): No content
```

---

## 6. Integración con Eventos de Ruta

### 6.1 Flujo Propuesto (v2)

```
1. Conductor inicia la ruta
2. App del conductor tiene lector NFC conectado
3. Estudiante aborda → conductor escanea NFC
   └─> POST /api/v1/nfc/scan
   └─> Sistema detecta estudiante
   └─> Crea evento BOARD automáticamente
   └─> Notifica al guardian
4. Estudiante baja → proceso similar → Evento DROP
```

### 6.2 Pendiente de Implementación

- [ ] Integrar escaneo NFC con creación de eventos
- [ ] Validar que NFC corresponda a estudiante de la ruta actual
- [ ] Notificaciones automáticas al guardian al detectar NFC

---

## 7. Consideraciones de Seguridad

1. **UID puede ser clonado** — El NFC UID no es seguro por sí solo; considerar autenticación adicional
2. **Validar estudiante en ruta** — Solo permitir eventos si el estudiante pertenece a la ruta activa
3. **Logs de escaneo** — Registrar todos los intentos de escaneo (exitosos y fallidos)
4. **Timeouts** — Definir tiempo mínimo entre escaneos del mismo estudiante (evitar duplicados)

---

## 8. Véase También

- [App Overview](../01-general/app-overview.md)
- [Esquema de BD](./schema.md)
- [Arquitectura Backend](../02-backend/architecture.md)

---

*Documento generado: 2026-04-13*
*Versión: 1.0.0*