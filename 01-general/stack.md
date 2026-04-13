# 🛠️ Stack Tecnológico

---

## 1. Backend

| Componente | Tecnología | Versión |
|------------|------------|---------|
| **Lenguaje** | Java | 17+ (21 recomendado) |
| **Framework** | Spring Boot | 4.0.x |
| **API** | RESTful | OpenAPI 3.0 (Swagger) |
| **Seguridad** | JWT (jjwt) | 0.12.5 |
| **Base de Datos** | PostgreSQL | 15+ |
| **Extensión Espacial** | PostGIS | 3.4+ |
| **ORM** | Hibernate | 6.4.x |
| **Documentación** | SpringDoc OpenAPI | 2.5.0 |

### Dependencias Principales

```xml
<!-- Spring Boot Starters -->
spring-boot-starter-webmvc
spring-boot-starter-data-jpa
spring-boot-starter-security
spring-boot-starter-validation

<!-- Base de Datos -->
postgresql (runtime)
hibernate-spatial
jts-core (1.18.2)

<!-- Seguridad -->
jjwt-api (0.12.5)
jjwt-impl (runtime)
jjwt-jackson (runtime)

<!-- Documentación -->
springdoc-openapi-starter-webmvc-ui (2.5.0)

<!-- Utilidades -->
lombok (optional)
spring-boot-devtools (optional)
```

---

## 2. Base de Datos

| Componente | Tecnología | Notas |
|------------|------------|-------|
| **Motor** | PostgreSQL | 15+ |
| **Extensión** | PostGIS | 3.4+ |
| **ORM** | Hibernate Spatial | Integración PostGIS + JPA |
| **Tipado** | Geography (no Geometry) | Cálculos en metros exactos |
| **SRID** | 4326 (WGS84) | Coordenadas GPS estándar |

### Tipos de Datos Espaciales

| Tipo PostGIS | Uso en SafeRoute |
|--------------|------------------|
| `GEOGRAPHY(POINT)` | Students (casa/colegio), Stops, GPS Positions, Events |
| `GEOGRAPHY(LINESTRING)` | Ruta recorrida (futuro) |
| `GEOGRAPHY(POLYGON)` | Zonas seguras (futuro) |

---

## 3. Seguridad

| Componente | Tecnología | Detalles |
|------------|------------|-----------|
| **Auth** | JWT | Access + Refresh tokens |
| **Hash** | BCrypt | Contraseñas de usuarios |
| **Roles** | Multi-rol | Set<Roles> (ADMIN, DRIVER, GUARDIAN) |
| **Estado** | Soft Delete | status: ACTIVE, INACTIVE, DELETED |

---

## 4. Frontend (Pendiente)

| Componente | Tecnología |
|------------|-------------|
| **Framework** | Por definir |
| **Estado** | Por definir |
| **Maps** | Por definir |
| **Push** | Firebase Cloud Messaging (FCM) |

---

## 5. Infraestructura (Planeada)

| Componente | Propuesta |
|------------|-----------|
| **Hosting** | Cloud (AWS, GCP, o Azure) |
| **DB Hosting** | RDS PostgreSQL o Cloud SQL |
| **Archivos** | S3 o similar |
| **Push Notifications** | Firebase Cloud Messaging |

---

## 6. Herramientas de Desarrollo

| Herramienta | Uso |
|-------------|-----|
| **IDE** | IntelliJ IDEA / VS Code |
| **Build** | Maven |
| **Testing** | JUnit 5, Mockito |
| **API Testing** | Postman / Swagger UI |
| **DB Client** | DBeaver / pgAdmin |

---

## 7. 📚 Véase También

- [App Overview](./app-overview.md)
- [Arquitectura Backend](../02-backend/architecture.md)
- [PostGIS](../02-backend/postgis.md)

---

*Documento generado: 2026-04-13*