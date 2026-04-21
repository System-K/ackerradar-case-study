# AckerRadar – Plot Management REST API

> Enterprise-grade REST API for precision agriculture: multi-tenant, geo-spatial indexing, external data source aggregation.

---

## Summary

AckerRadar is a multi-tenant REST API for agricultural businesses. The system aggregates data from multiple external sources — weather data (OpenWeatherMap), commodity prices (MATIF/Euronext, regional exchanges), and official cadastral land records (ALKIS) — and exposes them via standardized endpoints.

My contribution: **architecture and implementation of the backend API**, focusing on plot management, sensor data integration, and geo-spatial queries.

> **Note:** The source code is proprietary (NDA). This case study documents architecture decisions and technical learnings.

---

## Tech Stack

| Component | Technology |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot 3.2 |
| Database | MongoDB with 2dsphere Geo-Spatial Indexing |
| Auth | JWT with Tenant Context Propagation |
| Build | Maven |
| Documentation | OpenAPI 3 (Swagger UI) |
| External APIs | OpenWeatherMap, ALKIS, MATIF/Euronext |

---

## Architecture Highlights

### Atomic Data Structure

Instead of duplicating status, timestamps, and location in every entity, all domain entities use three reusable base components:

- **`Distinction`** — status, category, timestamp, optional location
- **`Timestamp`** — isolated time value, separately indexable
- **`Location`** — coordinates + GeoJSON Point

→ Consistent audit trails without a separate audit table. Required for German fertilizer regulation and organic certification.

### Multi-Tenant Isolation

Three-layer approach:
1. `tenantId` as a denormalized mandatory field in every document
2. `TenantContext` as ThreadLocal (extracted from JWT, cleared per request via filter)
3. Compound indexes with `tenantId` in first position

### Geo-Spatial Queries

MongoDB 2dsphere indexes for:
- Sensor-to-plot assignment (point-in-polygon)
- Nearest weather station lookup (`$nearSphere`)
- Polygon-based plot search (`$geoWithin`)

### Hybrid Reference Strategy (DBRef + Denormalization)

- **DBRef** for structural relationships (Plot → PlotProperties, Plot → Polygon)
- **Denormalized string IDs** for tenant references (performance on filtered queries)

---

## Interesting Problems

**GeoJSON Coordinate Order** — GeoJSON uses `[lon, lat]`, humans think `[lat, lon]`, and different external APIs mix both. Solved with a central `CoordinateConverter` using explicit method names (`fromLatLon`, `fromGeoJson`).

**Spring Boot 3 Migration** — `javax.*` → `jakarta.*` namespace change broke several dependencies. Solved via targeted upgrades and a thin adapter layer.

**MockMvc + Tenant Context** — `@WithMockUser` is insufficient for JWT-based tenant isolation. Documented transparently in the IHK submission rather than hidden.

---

## Results

- ~40 REST endpoints, documented via OpenAPI 3
- Multi-tenant with isolation at DB, service, and controller levels
- External integrations: OpenWeatherMap, ALKIS (functional); MATIF (stub)
- **IHK certification exam passed** (Fachinformatiker Anwendungsentwicklung)

---

## Full Case Study

The complete case study with architecture diagrams, detailed trade-off analyses, and lessons learned:

- [Case Study (English)](./docs/AckerRadar_Case_Study_EN.md)
- [Case Study (Deutsch)](./docs/AckerRadar_Case_Study_DE.md)

---

## Contact

**Yves Kühn** — Backend Developer, Berlin\
Focus: Java, Spring Boot, MongoDB, REST API Design, AI Integration
