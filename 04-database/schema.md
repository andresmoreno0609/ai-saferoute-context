# 🗄️ Esquema de Base de Datos - SafeRoute

---

## 1. Visión General

- **Motor:** PostgreSQL 15+
- **Extensión Espacial:** PostGIS 3.4+
- **Esquema:** `public`
- **PK:** UUID con `uuid_generate_v4()`
- **Soft Delete:** Campo `status` con valores 'ACTIVE', 'INACTIVE', 'DELETED'

---

## 2. Entidades Principales

### 2.1 users

**Descripción:** Tabla principal de usuarios del sistema (cuenta de autenticación)

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| id | UUID | PK, NOT NULL | Identificador único |
| email | VARCHAR(255) | UNIQUE, NOT NULL | Email del usuario |
| password_hash | VARCHAR(255) | NOT NULL | Hash de contraseña |
| name | VARCHAR(255) | NOT NULL | Nombre completo |
| roles | SET(VARCHAR) | NOT NULL | Roles (soporta multi-rol) |
| status | VARCHAR(20) | NOT NULL | Estado: ACTIVE, INACTIVE, DELETED |
| created_at | TIMESTAMP | NOT NULL | Fecha de creación |
| last_login_at | TIMESTAMP | NULL | Último login |

**Índices:**
```sql
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_status ON users(status);
```

---

### 2.2 drivers

**Descripción:** Información extendida del conductor (relación 1:1 con users)

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| id | UUID | PK, NOT NULL | Identificador único |
| user_id | UUID | FK → users(id), UNIQUE | Referencia al usuario |
| name | VARCHAR(255) | NOT NULL | Nombre del conductor |
| phone | VARCHAR(20) | NOT NULL | Teléfono |
| vehicle_id | UUID | FK → vehicles(id) | Vehículo asignado |
| is_verified | BOOLEAN | DEFAULT false | Verificado por ADMIN |
| document_number | VARCHAR(50) | NULL | Número de identificación |
| birth_date | DATE | NULL | Fecha de nacimiento |
| license_number | VARCHAR(50) | NULL | Número de licencia |
| emergency_contact | VARCHAR(255) | NULL | Contacto de emergencia |
| photo_url | VARCHAR(500) | NULL | URL de foto |
| created_at | TIMESTAMP | NOT NULL | Fecha de creación |

---

### 2.3 vehicles

**Descripción:** Vehículos del sistema

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| id | UUID | PK, NOT NULL | Identificador único |
| plate | VARCHAR(20) | UNIQUE, NOT NULL | Placa del vehículo |
| model | VARCHAR(100) | NULL | Modelo |
| brand | VARCHAR(100) | NULL | Marca |
| color | VARCHAR(50) | NULL | Color |
| capacity | INTEGER | NULL | Capacidad de pasajeros |
| created_at | TIMESTAMP | NOT NULL | Fecha de creación |

---

### 2.4 vehicle_documents

**Descripción:** Documentos de los vehículos

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| id | UUID | PK, NOT NULL | Identificador único |
| vehicle_id | UUID | FK → vehicles(id) | Vehículo |
| document_type | VARCHAR(30) | NOT NULL | SOAP, SEGURO, TECNOMECANICA, TARJETA_PROPIEDAD |
| file_url | VARCHAR(500) | NULL | URL del documento |
| start_date | DATE | NULL | Fecha inicio vigencia |
| end_date | DATE | NULL | Fecha fin vigencia |
| is_active | BOOLEAN | DEFAULT true | Documento activo |

**Regla:** Solo un documento ACTIVO por tipo.

---

### 2.5 driver_documents

**Descripción:** Documentos del conductor

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| id | UUID | PK, NOT NULL | Identificador único |
| driver_id | UUID | FK → drivers(id) | Conductor |
| document_type | VARCHAR(30) | NOT NULL | CEDULA, LICENCIA, PASAPORTE |
| document_number | VARCHAR(50) | NULL | Número del documento |
| license_category | VARCHAR(10) | NULL | Categoría licencia |
| file_url | VARCHAR(500) | NULL | URL del documento |
| is_active | BOOLEAN | DEFAULT true | Documento activo |

---

### 2.6 students

**Descripción:** Estudiantes registrados

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| id | UUID | PK, NOT NULL | Identificador único |
| name | VARCHAR(255) | NOT NULL | Nombre del estudiante |
| address | VARCHAR(500) | NOT NULL | Dirección de residencia |
| location | GEOGRAPHY(POINT) | NOT NULL | Ubicación geográfica |
| school_name | VARCHAR(255) | NULL | Nombre del colegio |
| school_location | GEOGRAPHY(POINT) | NULL | Ubicación del colegio |
| birth_date | DATE | NULL | Fecha de nacimiento |
| grade | VARCHAR(50) | NULL | Grado escolar |
| emergency_contact | VARCHAR(255) | NULL | Contacto de emergencia |
| emergency_phone | VARCHAR(20) | NULL | Teléfono de emergencia |
| medical_info | VARCHAR(1000) | NULL | Info médica |
| photo_url | VARCHAR(500) | NULL | URL de foto |
| student_code | VARCHAR(50) | NULL | Código interno |
| created_at | TIMESTAMP | NOT NULL | Fecha de creación |

**Índices:**
```sql
CREATE INDEX idx_students_location ON students USING GIST(location);
CREATE INDEX idx_students_student_code ON students(student_code);
```

---

### 2.7 student_nfc

**Descripción:** Tarjetas NFC asignadas a estudiantes (histórico)

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| id | UUID | PK, NOT NULL | Identificador único |
| student_id | UUID | FK → students(id) | Estudiante |
| nfc_uid | VARCHAR(100) | UNIQUE, NOT NULL | UID de la tarjeta NFC |
| is_active | BOOLEAN | DEFAULT true | ¿NFC activo? |
| assigned_at | TIMESTAMP | NOT NULL | Fecha de asignación |
| deactivated_at | TIMESTAMP | NULL | Fecha de desactivación |
| assigned_by | UUID | NULL | Usuario que asignó |
| notes | VARCHAR(500) | NULL | Notas adicionales |

**Reglas:**
- Solo UN NFC activo por estudiante
- Al asignar nuevo, anterior se inactiva automáticamente

---

### 2.8 guardians

**Descripción:** Acudientes/Padres de estudiantes

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| id | UUID | PK, NOT NULL | Identificador único |
| user_id | UUID | FK → users(id), UNIQUE | Referencia al usuario (opcional) |
| name | VARCHAR(255) | NOT NULL | Nombre del acudiente |
| phone | VARCHAR(20) | NOT NULL | Teléfono móvil |
| email | VARCHAR(255) | NULL | Email |
| fcm_token | TEXT | NULL | Token de Firebase |
| document_number | VARCHAR(50) | NULL | Número de identificación |
| address | VARCHAR(500) | NULL | Dirección |
| emergency_contact | VARCHAR(255) | NULL | Contacto de emergencia |
| occupation | VARCHAR(100) | NULL | Ocupación |
| created_at | TIMESTAMP | NOT NULL | Fecha de creación |

---

### 2.9 student_guardians

**Descripción:** Relación muchos a muchos entre estudiantes y acudientes

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| id | UUID | PK, NOT NULL | Identificador único |
| student_id | UUID | FK → students(id) | Estudiante |
| guardian_id | UUID | FK → guardians(id) | Acudiente |
| relationship | VARCHAR(50) | NOT NULL | father, mother, guardian, other |
| is_emergency_contact | BOOLEAN | DEFAULT false | Contacto de emergencia |
| notify_events | BOOLEAN | DEFAULT true | Recibe notificaciones |

---

### 2.10 routes

**Descripción:** Rutas escolares

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| id | UUID | PK, NOT NULL | Identificador único |
| name | VARCHAR(255) | NOT NULL | Nombre de la ruta |
| driver_id | UUID | FK → drivers(id) | Conductor asignado |
| status | VARCHAR(20) | DEFAULT 'PENDING' | PENDING, IN_PROGRESS, COMPLETED |
| start_time | TIMESTAMP | NULL | Hora de inicio real |
| end_time | TIMESTAMP | NULL | Hora de finalización |
| scheduled_date | DATE | NOT NULL | Fecha programada |
| notes | TEXT | NULL | Notas |
| created_at | TIMESTAMP | NOT NULL | Fecha de creación |

---

### 2.11 stops

**Descripción:** Paradas de una ruta

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| id | UUID | PK, NOT NULL | Identificador único |
| route_id | UUID | FK → routes(id) | Ruta asociada |
| student_id | UUID | FK → students(id) | Estudiante |
| order | INTEGER | NOT NULL | Orden de la parada |
| location | GEOGRAPHY(POINT) | NOT NULL | Ubicación de la parada |
| arrival_time | TIMESTAMP | NULL | Hora de llegada real |
| picked_up | BOOLEAN | DEFAULT false | Ya fue recogido |
| dropped_off | BOOLEAN | DEFAULT false | Ya fue dejado |
| created_at | TIMESTAMP | NOT NULL | Fecha de creación |

---

### 2.12 gps_positions

**Descripción:** Historial de posiciones GPS

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| id | UUID | PK, NOT NULL | Identificador único |
| driver_id | UUID | FK → drivers(id) | Conductor |
| route_id | UUID | FK → routes(id) | Ruta activa |
| location | GEOGRAPHY(POINT) | NOT NULL | Coordenadas |
| timestamp | TIMESTAMP | NOT NULL | Timestamp del GPS |
| speed | DOUBLE | NULL | Velocidad en km/h |
| heading | DOUBLE | NULL | Dirección (0-360) |
| accuracy | DOUBLE | NULL | Precisión en metros |

**Índices:**
```sql
CREATE INDEX idx_gps_positions_route ON gps_positions(route_id);
CREATE INDEX idx_gps_positions_location ON gps_positions USING GIST(location);
```

---

### 2.13 student_events

**Descripción:** Eventos críticos del estudiante (BOARD, ARRIVAL, DROP)

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| id | UUID | PK, NOT NULL | Identificador único |
| student_id | UUID | FK → students(id) | Estudiante |
| driver_id | UUID | FK → drivers(id) | Conductor |
| route_id | UUID | FK → routes(id) | Ruta |
| event_type | VARCHAR(20) | NOT NULL | BOARD, ARRIVAL, DROP |
| location | GEOGRAPHY(POINT) | NOT NULL | Ubicación |
| timestamp | TIMESTAMP | NOT NULL | Hora del evento |
| notes | TEXT | NULL | Notas |
| created_at | TIMESTAMP | NOT NULL | Fecha de creación |

**Nota:** Esta tabla es **inmutable** (INSERT ONLY).

---

### 2.14 observations

**Descripción:** Observaciones/novedades reportadas por el conductor

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| id | UUID | PK, NOT NULL | Identificador único |
| student_id | UUID | FK → students(id) | Estudiante |
| driver_id | UUID | FK → drivers(id) | Conductor |
| description | TEXT | NOT NULL | Descripción |
| photo_url | TEXT | NULL | URL de la foto |
| severity | VARCHAR(20) | DEFAULT 'LOW' | LOW, MEDIUM, HIGH |
| timestamp | TIMESTAMP | NOT NULL | Hora del registro |

---

### 2.15 notifications

**Descripción:** Registro de notificaciones enviadas

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| id | UUID | PK, NOT NULL | Identificador único |
| guardian_id | UUID | FK → guardians(id) | Acudiente receptor |
| title | VARCHAR(255) | NOT NULL | Título |
| body | TEXT | NOT NULL | Cuerpo del mensaje |
| type | VARCHAR(50) | NOT NULL | BOARD, ARRIVAL, DROP, OBSERVATION |
| reference_id | UUID | NULL | ID de referencia |
| sent_at | TIMESTAMP | NOT NULL | Fecha de envío |
| read_at | TIMESTAMP | NULL | Fecha de lectura |

---

## 3. Relaciones

| Relación | Tipo | Descripción |
|----------|------|-------------|
| users → drivers | 1:1 | Cada conductor tiene un usuario |
| drivers → vehicles | 1:1 | Un conductor tiene un vehículo |
| vehicles → vehicle_documents | 1:N | Un vehículo tiene muchos documentos |
| drivers → driver_documents | 1:N | Un conductor tiene muchos documentos |
| drivers → routes | 1:N | Un conductor puede tener varias rutas |
| routes → stops | 1:N | Una ruta tiene varias paradas |
| students → guardians | N:M | Relación a través de student_guardians |
| students → student_nfc | 1:N | Un estudiante puede tener varios NFCs |
| drivers → gps_positions | 1:N | Un conductor tiene muchas posiciones GPS |
| routes → student_events | 1:N | Una ruta genera muchos eventos |

---

## 4. Reglas de Negocio

1. **Un conductor solo puede tener UNA ruta activa a la vez**
2. **Eventos inmutables** — Solo INSERT en `student_events`
3. **Un estudiante puede tener múltiples guardianes**
4. **Solo un documento ACTIVO por tipo** (vehículo y conductor)
5. **Solo un NFC activo por estudiante** (histórico)

---

## 5. Convenciones

| Convención | Regla |
|------------|-------|
| **Tablas** | snake_case plural (users, student_events) |
| **Campos** | snake_case (created_at, vehicle_plate) |
| **PK** | UUID con `uuid_generate_v4()` |
| **Fechas** | `TIMESTAMP NOT NULL DEFAULT NOW()` |
| **Soft delete** | Campo `status` |
| **Enums** | VARCHAR con CHECK constraint |
| **Índices** | Nombrar `idx_tabla_campo` |

---

## 6. Véase También

- [App Overview](../01-general/app-overview.md)
- [PostGIS](../02-backend/postgis.md)
- [Sistema NFC](./nfc.md)

---

*Documento generado: 2026-04-13*
*Versión: 1.0.0*