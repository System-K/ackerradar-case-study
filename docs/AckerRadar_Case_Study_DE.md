# AckerRadar – Case Study

**Eine Multi-Tenant REST API für Präzisionslandwirtschaft mit Geo-Spatial Indexing, Multi-Source Data Aggregation und ALKIS-Integration**

---

## Auf einen Blick

| | |
|---|---|
| **Rolle** | Backend-Entwickler (Umschulung zum Fachinformatiker Anwendungsentwicklung) |
| **Unternehmen** | AckerRadar UG (Berliner AgriTech-Startup) |
| **Umfang** | IHK-Abschlussprojekt, Teil einer mehrmonatigen Mitarbeit |
| **Stack** | Java 17, Spring Boot 3.2, MongoDB (mit Geo-Spatial Indexing), Maven, JWT, OpenAPI 3 |
| **Domäne** | Präzisionslandwirtschaft, Geo-Daten, Sensor-Integration, Marktdaten-Aggregation |
| **Status** | IHK-Abschluss bestanden; Code proprietär (NDA) |

---

## Das Problem

Landwirtschaftliche Betriebe in Deutschland arbeiten mit fragmentierten Datenquellen: Wetterdaten kommen aus einem System, Rohstoffpreise aus einem anderen, Bodensensoren aus einem dritten und amtliche Flurstücksdaten (ALKIS) aus einem vierten. AckerRadar wollte diese Quellen in einem mandantenfähigen System zusammenführen, nahezu in Echtzeit und über standardisierte REST-APIs abrufbar.

Meine Aufgabe: die Backend-API entwerfen und implementieren — mit Fokus auf Plot-Management, Sensor-Daten-Integration und Geo-Spatial Queries.

---

## Architektur

### Systemüberblick

![Systemarchitektur](../diagrams/Systemarchitektur_AckerRadar.png)

### Atomares Datenmodell

Drei wiederverwendbare Basis-Komponenten (`Distinction`, `Timestamp`, `Location`) statt duplizierter Status-/Zeit-/Positionsfelder in jeder Entität.

![ER-Diagramm](../diagrams/ER_Diagramm_Plot_Management_System.png)

### Plot-Management

![UML-Klassendiagramm](../diagrams/UML_Klassendiagramm_Plot_Management.png)

Sensoren werden per GPS-Point-in-Polygon dem Plot zugeordnet — nicht manuell verknüpft.

---

## Technische Entscheidungen

**MongoDB statt PostgreSQL/PostGIS** — Native 2dsphere-Indizes, Schema-Flexibilität (Sensor-Modelle wuchsen von 2 auf 20+ Typen), natürliche Dokumentenstruktur. Trade-off: eingeschränktes Multi-Document-ACID.

**Spring Boot statt Javalin** — Ökosystem (Spring Security, Spring Data MongoDB, springdoc-openapi) sparte Wochen. Trade-off: höherer Speicherverbrauch.

**Hybride Referenzierung** — DBRef für strukturelle Beziehungen, denormalisierte String-IDs für Tenant-Filterung.

**Multi-Tenancy** — Drei Schichten: denormalisierte `tenantId` in jedem Dokument, ThreadLocal `TenantContext` aus JWT, Compound-Indizes mit `tenantId` an führender Position.

---

## Herausforderungen

**GeoJSON-Koordinatenreihenfolge** — `[lon, lat]` vs. `[lat, lon]` ließ Sensoren in der Nordsee erscheinen. Gelöst mit zentralem `CoordinateConverter`.

**Spring Boot 3 Migration** — `javax.*` → `jakarta.*` brach Dependencies mitten im Projekt. Gelöst durch gezieltes Upgrade und Adapter-Layer.

**MockMvc + Tenant-Context** — Transparent in der IHK-Dokumentation beschrieben statt kaschiert. Im Fachgespräch positiv aufgenommen.

---

## Sequenzdiagramm: Plot anlegen

![Sequenzdiagramm](../diagrams/Sequenzdiagramm_Plot_Erstellen.png)

---

## Ergebnis

- Ca. 40 REST-Endpoints, dokumentiert via OpenAPI 3
- Multi-Tenant-Isolation auf DB-, Service- und Controller-Ebene
- Geo-Spatial Queries für Sensor-Plot-Zuordnung und nächste Wetterstation
- Externe Integrationen: OpenWeatherMap, ALKIS (funktionsfähig); MATIF (Stub)
- **IHK-Abschlussprüfung bestanden**

---

## Lessons Learned

- Atomare Datenstrukturen lohnen sich bei Audit-Anforderungen
- MongoDB exzellent für Geo-Daten und flexible Schemata, aber kein Drop-in SQL-Ersatz
- Ehrliche Dokumentation schlägt polierte Dokumentation in technischen Prüfungen
- Multi-Tenancy muss von Tag 1 mitgedacht werden

---

## Über mich

Yves Kühn, Berlin. Quereinsteiger über IHK-Umschulung (bestanden 2026). Schwerpunkte: Java-Backend, MongoDB, REST-API-Design, KI-Integration. Open-Source-Maintainer von **Orbis**.
