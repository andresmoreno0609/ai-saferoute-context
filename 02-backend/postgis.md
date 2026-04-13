# 🗺️ PostGIS - Datos Geoespaciales en SafeRoute

---

## 1. Visión General

### 1.1 ¿Qué es PostGIS?

PostGIS es una extensión de PostgreSQL que permite almacenar, indexar y consultar datos geoespaciales (geométricos y geográficos). Es el estándar de facto para bases de datos espaciales en aplicaciones empresariales.

### 1.2 PostGIS en SafeRoute

SafeRoute usa PostGIS para gestionar la información geoespacial relacionada con el transporte escolar:

| Entidad | Tipo Geoespacial | Uso |
|---------|------------------|-----|
| `Student` | `Point` | Ubicación de casa y colegio |
| `Stop` | `Point` | Coordenadas de cada parada |
| `GpsPosition` | `Point` | Posición del vehículo en tiempo real |
| `StudentEvent` | `Point` | Ubicación donde ocurrió el evento |

---

## 2. Responsabilidades de PostGIS

### ¿Qué hace PostGIS?

```
┌─────────────────────────────────────────────────────────────────┐
│                    RESPONSABILIDADES DE POSTGIS                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. 📍 ALMACENAMIENTO                                           │
│     └─> Guardar coordenadas (latitud, longitud) como tipos     │
│         nativos de base de datos                                │
│                                                                  │
│  2. 🔍 INDEXACIÓN ESPACIAL                                      │
│     └─> GIST index para búsquedas rápidas por ubicación         │
│                                                                  │
│  3. 🧮 CÁLCULOS GEOMÉTRICOS                                    │
│     └─> Distancias, áreas, intersecciones, proximidad          │
│                                                                  │
│  4. 📐 TRANSFORMACIONES                                        │
│     └─> Convertir entre sistemas de coordenadas (SRID)          │
│                                                                  │
│  5. 🔎 CONSULTAS ESPACIALES                                     │
│     └─> "Encontrar todas las paradas dentro de 500m"            │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### ¿Qué NO hace PostGIS?

| ❌ No es responsable de | ✅ En su lugar |
|------------------------|----------------|
| Tracking en tiempo real | La aplicación genera los GPS positions |
| Notificaciones a usuarios | Servicio de notificaciones de la app |
| Lógica de negocio | Services/UseCases de Spring |
| Frontend/Mapa | Frontend (React/Mobile) |

---

## 3. Implementación Técnica

### 3.1 Configuración de Hibernate Spatial

```properties
# application.properties
spring.jpa.properties.hibernate.dialect=org.hibernate.spatial.dialect.postgis.PostgisDialect
```

### 3.2 Dependencias Requeridas

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-spatial</artifactId>
</dependency>

<dependency>
    <groupId>org.locationtech.jts</groupId>
    <artifactId>jts-core</artifactId>
    <version>1.18.2</version>
</dependency>
```

### 3.3 Tipos Geométricos en Java

| Tipo JTS | Tipo PostGIS | Uso |
|----------|--------------|-----|
| `Point` | `GEOGRAPHY(POINT)` | Coordenadas individuales |
| `LineString` | `GEOGRAPHY(LINESTRING)` | Ruta recorrida |
| `Polygon` | `GEOGRAPHY(POLYGON)` | Zonas seguras |

### 3.4 Configuración de Entidades

```java
// Ejemplo: StudentEntity.java
@Entity
@Table(name = "students")
public class StudentEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.UUID)
    private UUID id;

    @Column(nullable = false)
    private String name;

    // Tipo geográfico - POINT con SRID 4326 (WGS84)
    @Column(name = "location", columnDefinition = "GEOGRAPHY(POINT, 4326)")
    private Point location;

    @Column(name = "school_location", columnDefinition = "GEOGRAPHY(POINT, 4326)")
    private Point schoolLocation;
}
```

---

## 4. Operaciones CRUD con Datos Espaciales

### 4.1 Crear una Entidad con Punto Geográfico

```java
@Service
@RequiredArgsConstructor
public class StudentService {

    private final StudentRepository repository;

    public StudentResponse createStudent(StudentRequest request) {
        // Crear Point desde latitud y longitud usando JTS
        Point location = createPoint(request.latitude(), request.longitude());

        StudentEntity entity = StudentEntity.builder()
                .name(request.name())
                .address(request.address())
                .location(location)
                .schoolName(request.schoolName())
                .schoolLocation(createPoint(
                    request.schoolLatitude(),
                    request.schoolLongitude()
                ))
                .build();

        StudentEntity saved = repository.save(entity);
        return toResponse(saved);
    }

    // Factory method para crear Point con SRID 4326 (WGS84)
    private Point createPoint(Double longitude, Double latitude) {
        GeometryFactory geometryFactory = new GeometryFactory(
            new PrecisionModel(), 4326
        );
        return geometryFactory.createPoint(new Coordinate(longitude, latitude));
    }
}
```

---

## 5. Consultas Espaciales

### 5.1 Repository con Consultas Espaciales

```java
@Repository
public interface StopRepository extends JpaRepository<StopEntity, UUID>,
        JpaSpecificationExecutor<StopEntity> {

    // Encontrar paradas dentro de un radio (en metros)
    @Query(value = """
        SELECT * FROM stops s
        WHERE ST_DWithin(s.location, 
            ST_SetSRID(ST_MakePoint(:longitude, :latitude), 4326)::geography,
            :radiusInMeters)
        """, nativeQuery = true)
    List<StopEntity> findStopsWithinRadius(
        @Param("longitude") Double longitude,
        @Param("latitude") Double latitude,
        @Param("radiusInMeters") Double radiusInMeters
    );

    // Encontrar parada más cercana a una ubicación
    @Query(value = """
        SELECT * FROM stops s
        ORDER BY s.location <-> 
            ST_SetSRID(ST_MakePoint(:longitude, :latitude), 4326)::geography
        LIMIT 1
        """, nativeQuery = true)
    Optional<StopEntity> findNearestStop(
        @Param("longitude") Double longitude,
        @Param("latitude") Double latitude
    );
}
```

---

## 6. Funcionalidades de SafeRoute con PostGIS

### 6.1 Mapa en Tiempo Real
- GPS del conductor envía posición cada 30 segundos
- Backend guarda en tabla `gps_positions` (con Point)
- Frontend consulta posición actual y dibuja en mapa

### 6.2 Geofencing - Zonas Seguras
- Radio de 200m alrededor de cada parada
- Al entrar/salir del radio, se dispara evento
- Función: `ST_DWithin`

### 6.3 Notificaciones por Ubicación
- Bus llega a parada (150m) → Notificación
- Estudiante sube/baja del bus → Notificación
- Bus cerca de casa (200m) → Notificación

---

## 7. Chuleta de Consultas PostGIS

| Operación | Función PostGIS | Ejemplo |
|-----------|-----------------|---------|
| Crear punto | `ST_MakePoint(lng, lat)` | `ST_MakePoint(-74.0721, 4.7110)` |
| Distancia (m) | `ST_Distance(p1, p2)` | Distancia entre bus y parada |
| Dentro de radio | `ST_DWithin(p1, p2, radio)` | ¿Está a menos de 200m? |
| Punto más cercano | `<->` (operador) | `ORDER BY location <-> bus LIMIT 1` |
| Convertir a GeoJSON | `ST_AsGeoJSON(point)` | `{ "type": "Point", ... }` |
| Longitud línea | `ST_Length(line)` | Distancia total recorrida |
| Centroide | `ST_Centroid(polygon)` | Centro de una zona |

---

## 8. Consideraciones y Mejores Prácticas

### 8.1 SRID (Spatial Reference System Identifier)

| SRID | Sistema | Uso |
|------|---------|-----|
| `4326` | WGS84 (GPS) | ✅ Estándar para coordenadas GPS |
| `32618` | UTM Zone 18N | Colombia (opcional) |
| `0` | Sin referencia | ❌ No usar |

**Importante:** Siempre usar `4326` para coordenadas GPS.

### 8.2 Geography vs Geometry

```
┌─────────────────────────────────────────────────────────────────┐
│                    GEOGRAPHY vs GEOMETRY                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  GEOMETRY:                                                       │
│  - Planar (plano 2D)                                            │
│  - Cálculos más rápidos                                          │
│  - Para áreas pequeñas (< 400km²)                               │
│                                                                  │
│  GEOGRAPHY:                                                      │
│  - Esférico (considera curvatura terrestre)                    │
│  - Cálculos más precisos para largas distancias                │
│  - Usa metros independientemente del SRID                       │
│  - ✅ RECOMENDADO para coordenadas GPS                         │
│                                                                  │
│  SafeRoute usa: GEOGRAPHY(POINT, 4326)                         │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### 8.3 Índices Espaciales

```sql
-- Crear índice GIST para columnas geográficas
CREATE INDEX idx_students_location ON students USING GIST(location);
CREATE INDEX idx_stops_location ON stops USING GIST(location);
CREATE INDEX idx_gps_positions_location ON gps_positions USING GIST(location);
```

**Importante:** Sin índice GIST, las consultas espaciales son lentas.

### 8.4 Performance

| Recomendación | Reason |
|--------------|--------|
| Siempre usar índices GIST | Búsquedas O(log n) en vez de O(n) |
| Limitar resultados con `LIMIT` | Evitar scans completos |
| Usar `GEOGRAPHY` no `GEOMETRY` | Cálculos en metros exactos |
| Batch inserts para GPS | Insertar muchas posiciones a la vez |

---

## 9. Herramientas de Desarrollo

### 9.1 Verificar que PostGIS está instalado

```sql
-- En psql o cualquier cliente PostgreSQL
SELECT postgis_full_version();
```

### 9.2 Visualizar datos espaciales

```sql
-- Ver coordinates como texto
SELECT id, name, ST_AsText(location) as location FROM students;

-- Ver como GeoJSON
SELECT id, name, ST_AsGeoJSON(location) as location FROM students;
```

---

## 10. Resumen para el Equipo

```
┌─────────────────────────────────────────────────────────────────┐
│                    RESUMEN POSTGIS PARA SAFEROUTE              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ✅ USAMOS:                                                     │
│     - PostgreSQL + PostGIS 3.4+                                 │
│     - Hibernate Spatial                                        │
│     - JTS (Java Topology Suite)                                │
│     - SRID 4326 (WGS84 - coordenadas GPS)                      │
│                                                                  │
│  📍 DATOS ESPACIALES:                                           │
│     - Student: location (casa), schoolLocation (colegio)      │
│     - Stop: location (coordenadas de parada)                   │
│     - GpsPosition: location (posición del bus)                 │
│     - StudentEvent: location (donde ocurrió evento)            │
│                                                                  │
│  🔍 CONSULTAS CLAVE:                                            │
│     - ST_DWithin: "¿Está cerca?" (radio)                       │
│     - ST_Distance: "¿A qué distancia?"                          │
│     - <->: "Encontrar más cercano"                             │
│                                                                  │
│  ⚡ PERFORMANCE:                                                │
│     - SIEMPRE crear índice GIST                                 │
│     - Usar GEOGRAPHY no GEOMETRY                                │
│     - Limitar resultados con LIMIT                              │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 11. Véase También

- [App Overview](../01-general/app-overview.md)
- [Arquitectura Backend](./architecture.md)
- [Stack Tecnológico](../01-general/stack.md)

---

*Documento generado: 2026-04-13*
*Versión: 1.0.0*