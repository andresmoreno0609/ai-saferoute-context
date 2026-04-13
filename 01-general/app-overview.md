# 🚍 SafeRoute - Visión General de la Aplicación

---

## 1. 📌 ¿Qué es SafeRoute?

**SafeRoute** es un sistema integral de gestión y monitoreo de rutas escolares en tiempo real. Su objetivo principal es **mejorar la seguridad** de los estudiantes durante el transporte escolar, proporcionando:

- **Trazabilidad completa** del recorrido del vehículo
- **Comunicación efectiva** entre conductores y padres
- **Identificación automatizada** de estudiantes mediante NFC
- **Notificaciones en tiempo real** sobre el estado de los hijos

---

## 2. 🎯 Objetivo del Sistema

El sistema está enfocado en:

1. **Gestionar rutas escolares** de manera eficiente
2. **Monitorear la ubicación** de los vehículos en tiempo real (GPS)
3. **Registrar eventos** de estudiantes (BOARD, ARRIVAL, DROP)
4. **Identificar estudiantes** mediante tarjetas NFC
5. **Notificar a los padres** en tiempo real sobre el estado de sus hijos
6. **Gestionar vehículos y conductores** con validación de documentación

---

## 3. 👥 Actores del Sistema

| Rol | Descripción | Funciones Principales |
|-----|-------------|----------------------|
| **ADMIN** | Administrador | Gestiona usuarios, rutas, verifica conductores y documentos |
| **DRIVER** | Conductor | Ejecuta rutas, envía GPS, registra eventos, reporta observaciones |
| **GUARDIAN** | Acudiente/Padre | Consulta estado del estudiante, recibe notificaciones |

**Nota:** Un usuario puede tener **múltiples roles** (ej: DRIVER + GUARDIAN para un padre que también trabaja como conductor).

---

## 4. 🧩 Módulos del Sistema

### Módulos Implementados

| Módulo | Descripción |
|--------|-------------|
| **auth** | Autenticación JWT, login, registro, refresh token |
| **users** | Gestión de usuarios del sistema |
| **students** | Registro de estudiantes con ubicación geográfica |
| **guardians** | Acudientes y relación con estudiantes |
| **drivers** | Información de conductores con verificación |
| **vehicles** | Gestión de vehículos y documentos obligatorios |
| **routes** | Creación y gestión de rutas escolares |
| **tracking** | GPS en tiempo real |
| **events** | Eventos BOARD, ARRIVAL, DROP |
| **nfc** | Sistema de identificación por NFC |

### Módulos Pendientes (v2)

| Módulo | Descripción |
|--------|-------------|
| **observations** | Registro de novedades con fotos |
| **notifications** | Push notifications avanzadas |

---

## 5. 🔄 Flujo Principal del Sistema

```
1. ADMIN crea una ruta y asigna conductor + vehículo
2. Conductor inicia la ruta
3. App móvil envía ubicación GPS periódicamente (cada 10-30 seg)
4. Estudiante aborda el vehículo → Evento BOARD → Notificación al padre
5. Vehículo llega al colegio → Evento ARRIVAL → Notificación al padre
6. Vehículo retourne a casa → Estudiante baja → Evento DROP → Notificación
7. Conductor termina la ruta
```

---

## 6. 🔑 Características Principales

### 6.1 Gestión de Rutas
- Creación de rutas con paradas ordenadas
- Asignación de conductor y vehículo
- Estados: PENDING → IN_PROGRESS → COMPLETED

### 6.2 Tracking GPS
- Envío de posición cada 10-30 segundos
- Historial de posiciones por ruta
- Visualización en mapa en tiempo real

### 6.3 Eventos de Estudiantes
- **BOARD**: Estudiante sube al vehículo
- **ARRIVAL**: Llega al colegio
- **DROP**: Regresa a casa

### 6.4 Sistema NFC
- Asignación de tarjetas NFC a estudiantes
- Un solo NFC activo por estudiante (histórico)
- Escaneo para detectar estudiante

### 6.5 Validación de Documentos
- **Vehículo**: SOAP, SEGURO, TECNOMECANICA, TARJETA_PROPIEDAD
- **Conductor**: LICENCIA, CEDULA
- Verificación manual por ADMIN

---

## 7. ⚙️ Reglas de Negocio Clave

1. Un estudiante puede tener múltiples acudientes
2. Un conductor debe estar verificado por ADMIN para trabajar
3. Un conductor debe tener vehículo con todos los documentos vigentes
4. Solo un documento activo por tipo (vehículo y conductor)
5. Un estudiante puede tener múltiples NFCs (histórico), solo uno activo
6. Los eventos son inmutables (solo INSERT, no UPDATE/DELETE)
7. Un conductor solo puede tener UNA ruta activa a la vez

---

## 8. 🔐 Seguridad

- **Autenticación:** JWT (JSON Web Tokens)
- **Autorización:** Roles múltiples (ADMIN, DRIVER, GUARDIAN)
- **Contraseñas:** Hash con BCrypt
- **Soft Delete:** Los registros no se eliminan, se marcan como DELETED

---

## 9. 📍 Geolocalización

- Uso de **PostGIS** para consultas espaciales
- Tipo de dato: `GEOGRAPHY(POINT, 4326)` - WGS84
- SRID 4326 (coordenadas GPS estándar)
- Uso de índices GIST para búsquedas rápidas

---

## 10. 📈 Métricas y Datos

- Posiciones GPS por día: ~28,800 (estimado)
- Frecuencia GPS: cada 10-30 segundos
- Precisión típica: 5-10 metros

---

## 11. 🚀 Escalabilidad Futura

- Migración a microservicios
- Uso de mensajería (Kafka o RabbitMQ)
- Optimización automática de rutas
- Integración de pagos
- Integración NFC con eventos de ruta (BOARD/DROP automático)

---

## 12. 📚 Véase También

- [Stack Tecnológico](./stack.md)
- [Arquitectura Backend](../02-backend/architecture.md)
- [PostGIS](../02-backend/postgis.md)
- [Esquema de Base de Datos](../04-database/schema.md)
- [Sistema NFC](../04-database/nfc.md)

---

*Documento generado: 2026-04-13*
*Versión: 1.0.0*