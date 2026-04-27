# Medical Device Risk Ontology
### .*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*

**Ontología del catálogo europeo de dispositivos médicos de alto riesgo** bajo el marco regulatorio MDR (EU 2017/745) e IVDR (EU 2017/746).

Fichero principal: `ontology.ttl`  
Namespace: `https://lvlupcodex.github.io/medical-device-risk-ontology/ontology#`  
Autor: lvlupcodex (github) lilarraza@uoc.edu (UOC) Laura del Carmen Ilarraza Prendes · UOC PEC2 · Primavera 2026

---
# Un README que puede ayudar a otros estudiantes -- Relaciones entre lo aprendido en otras asignaturas y la asignatura de Representación del Conocimiento.


### Este documento sirve como una guia de ayuda para comprender los conceptos relacionados con la asignatura desde el punto de vista de una persona que tiene experiencia práctica en el ámbito de la programación.

> "La mente humana está formada solo por comparaciones hechas para examinar analogías..." — Giacomo Casanova


## Para quien viene de programación orientada a objetos

Si, como tantos otros, vienes de un background de desarrollo de software, la ontología es esencialmente un **modelo de datos semántico**. La analogía más directa que podría decir, es esta:

| Concepto en OWL | Equivalente en POO / SQL |
|---|---|
| `owl:Class` | `class` / tabla en BD |
| `rdfs:subClassOf` | `extends` / herencia |
| `owl:ObjectProperty` | Foreign Key (con nombre y semántica) |
| `owl:DatatypeProperty` | Campo primitivo (`String`, `int`, `boolean`, `Date`) |
| `owl:NamedIndividual` | Instancia / fila de la tabla |
| `owl:Restriction` (cardinality) | `NOT NULL`, `UNIQUE`, `CHECK` |
| `SWRL Rule` | Trigger / stored procedure automático |
| Razonador (Pellet/HermiT) | Motor de inferencia — deduce conocimiento nuevo sin que lo programes explícitamente |

La diferencia clave con una base de datos relacional: las "foreign keys" (object properties) son **ciudadanos de primera clase** con nombre, dominio, rango, y el razonador puede **inferir relaciones transitivas** automáticamente a partir de las reglas declaradas.

---
### .*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*

## Estructura del modelo — T-Box y A-Box

### T-Box (el esquema — equivale al DDL de SQL o a la definición de clases)

El T-Box define las clases, propiedades y restricciones. Es todo lo que **no** es una instancia concreta.

```
MedicalDeviceRiskEntity          ← clase raíz (como Object en Java, o en cualquier otro lenguaje que soporte objtos)
├── MedicalDevice
│   ├── MDRDevice
│   ├── IVDDevice
│   ├── ActiveDevice
│   ├── ImplantableDevice
│   │   └── (herencia múltiple OWL)
│   ├── ActiveImplant         ← subclase de ActiveDevice Y ImplantableDevice a la ¡¡vez!! doble herencia, increible ¿verdad?
│   ├── ImplantableActiveDevice
│   ├── SingleUseDevice
│   ├── ReusableDevice
│   ├── PacemakerDevice          ← subclase de ActiveImplant
│   ├── InVitroDiagnosticDevice
│   └── SelfTestDevice
├── Manufacturer
├── Risk
│   ├── MalfunctionRisk
│   ├── InjuryRisk
│   ├── InfectionRisk
│   ├── DiagnosticRisk
│   ├── SoftwareRisk
│   ├── CybersecurityRisk
│   ├── MaterialRisk
│   ├── RadiationRisk
│   ├── InterferenceRisk
│   └── ContaminationRisk
├── Incident
│   ├── SeriousIncident
│   ├── NearMissIncident
│   └── CorrectedIncident        ← inferida por regla SWRL, no se declara manualmente
├── FieldSafetyCorrectiveAction
│   ├── Recall
│   ├── SafetyAlert
│   ├── UserNotice
│   └── InSituCorrection
├── CompetentAuthority
└── ControlledVocabularyConcept  ← vocabularios controlados (en vez de strings "sueltos")
    ├── RiskClass                 (ClassI, ClassIIa, ClassIIb, ClassIII, ClassA..D)
    ├── RegulatoryFramework       (MDR, IVDR)
    ├── SeverityLevel             (Critical, Serious, Moderate)
    ├── ProbabilityLevel          (Low, Medium, High)
    ├── ActionStatus              (Open, Closed)
    └── Country                   (Germany, Ireland, France, Netherlands...)
```

> **Por qué vocabularios controlados en vez de strings:** si guardamos  `"Critical"` como un string literal, no puedes razonar sobre él, no podemos  enlazarlo a DBpedia, no podemos añadirle traducciones ni definiciones. Como `skos:Concept` con `skos:prefLabel`, es un recurso reutilizable con semántica propia.

---
### .*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*

### A-Box (las instancias — equivale a los datos / filas de la tabla)

Las instancias proceden directamente de los ficheros CSV de la práctica. Cada fila de un CSV = un `owl:NamedIndividual`.

```turtle
# Esto en OWL:
:DEV-001
    rdf:type owl:NamedIndividual , :ImplantableActiveDevice ;
    :deviceName "HeartMate 3 LVAD"^^xsd:string ;
    :hasManufacturer :MAN-001 ;
    :hasRiskClass :ClassIII ;
    :regulatedUnder :MDR .

# Es equivalente a esto en SQL:
-- INSERT INTO Device VALUES ('DEV-001', 'HeartMate 3 LVAD', 'MAN-001', 'ClassIII', 'MDR', ...);
-- Pero además, la clase ImplantableActiveDevice hereda de ActiveDevice Y ImplantableDevice a la vez.
```

---
### .*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*
---
## Las "Foreign Keys" — Object Properties

En SQL una FK es solo un número entero que apunta a otra tabla. Aquí cada relación tiene **nombre propio** y puede tener su inversa declarada:

```turtle
# La "FK" desde MedicalDevice hacia Manufacturer:
:hasManufacturer
    rdf:type owl:ObjectProperty , owl:FunctionalProperty ;  ← FunctionalProperty = exactamente 1 valor (NOT NULL + UNIQUE en ese sentido)
    rdfs:domain :MedicalDevice ;                            ← la "FK está en la tabla" MedicalDevice
    rdfs:range :Manufacturer .                              ← apunta a Manufacturer

# Su inversa (gratis, el razonador la deduce):
:manufacturesDevice
    owl:inverseOf :hasManufacturer ;
    rdfs:domain :Manufacturer ;
    rdfs:range :MedicalDevice .
# Si declares :DEV-001 :hasManufacturer :MAN-001, el razonador infiere automáticamente :MAN-001 :manufacturesDevice :DEV-001
```

El grafo de relaciones del modelo queda así, entonces:

```
MedicalDevice ──hasManufacturer──────▶ Manufacturer
MedicalDevice ──hasRiskClass──────────▶ RiskClass
MedicalDevice ──regulatedUnder────────▶ RegulatoryFramework
Incident      ──concernsDevice────────▶ MedicalDevice
Incident      ──involvesRisk──────────▶ Risk
Incident      ──reportedBy────────────▶ Manufacturer
Incident      ──reportedInCountry─────▶ Country
FSCA          ──addressesIncident─────▶ Incident
Manufacturer  ──hasManufacturerCountry▶ Country
Manufacturer  ──hasCompetentAuthority─▶ CompetentAuthority
Risk          ──hasSeverity────────────▶ SeverityLevel
Risk          ──hasProbability──────────▶ ProbabilityLevel
FSCA          ──hasActionStatus────────▶ ActionStatus
```

---
### .*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*.*
---
## Restricciones de cardinalidad — los constraints

```turtle
# "Todo MedicalDevice DEBE tener exactamente 1 Manufacturer"
:MedicalDevice rdfs:subClassOf [
    rdf:type owl:Restriction ;
    owl:onProperty :hasManufacturer ;
    owl:qualifiedCardinality "1"^^xsd:nonNegativeInteger ;  ← EXACTLY 1
    owl:onClass :Manufacturer
] .

# Equivalente SQL:
-- manufacturer_id INT NOT NULL UNIQUE REFERENCES Manufacturer(id)
```

Si creas una instancia de `MedicalDevice` sin `hasManufacturer`, el razonador detecta inconsistencia (como un constraint de BD que viola la FK).

---

## Reglas SWRL — los "triggers" automáticos

Las reglas SWRL son inferencias que el razonador ejecuta automáticamente. Están declaradas en la sección `SWRL RULES` del fichero TTL. ¡Como triggers!

### Regla 1 — Si `isIVD = true` → clasificar como `IVDDevice`
```
MedicalDevice(?d) ∧ isIVD(?d, true)  →  IVDDevice(?d)
```
Equivale a:
```sql
-- TRIGGER: IF NEW.is_ivd = TRUE THEN clasifica el dispositivo como IVDDevice
```

### Regla 2 — Si `isIVD = false` → clasificar como `MDRDevice`
```
MedicalDevice(?d) ∧ isIVD(?d, false)  →  MDRDevice(?d)
```

### Regla 3 — Si un `Incident` tiene una `FieldSafetyCorrectiveAction` → inferir `CorrectedIncident`
```
Incident(?i) ∧ hasCorrectiveAction(?i, ?f)  →  CorrectedIncident(?i)
```
Esto significa que **no necesitas declarar manualmente** que `INC-001` es un `CorrectedIncident`. El razonador lo deduce porque tiene una FSCA asociada.

---

## Vocabularios externos reutilizados

Siguiendo las mejores prácticas de la web semántica (no reinventar la rueda):

| Prefijo | URI | Para qué se usa |
|---------|-----|-----------------|
| `schema:` | http://schema.org/ | `MedicalDevice` alinea con `schema:MedicalDevice` |
| `foaf:` | http://xmlns.com/foaf/0.1/ | `Manufacturer` extiende `foaf:Organization`; se usan `foaf:name` y `foaf:mbox` |
| `skos:` | https://www.w3.org/2004/02/skos/core# | Todos los vocabularios controlados son `skos:Concept` |
| `dcterms:` | http://purl.org/dc/terms/ | Metadatos de la ontología (título, descripción, autor, fecha) |
| `prov:` | https://www.w3.org/ns/prov# | `CompetentAuthority` extiende `prov:Agent` para trazabilidad regulatoria |
| `rdfs:seeAlso` + DBpedia | https://dbpedia.org/ | REQ-9: enlace externo en `MAN-001`, `DEV-002` (PacemakerDevice), `ImplantableDevice`, países |

---

## Instancias incluidas (A-Box mínima para REQ-7)

| ID | Clase | Descripción |
|----|-------|-------------|
| `DEV-001` | `ImplantableActiveDevice` | HeartMate 3 LVAD |
| `DEV-002` | `ActiveImplant`, `PacemakerDevice` | Micra AV Transcatheter Pacing System |
| `DEV-003` | `ImplantableDevice` | Portico Transcatheter Heart Valve |
| `DEV-016` | `InVitroDiagnosticDevice` | cobas e 801 Immunoassay Analyzer |
| `DEV-017` | `InVitroDiagnosticDevice` | cobas SARS-CoV-2 RT-PCR Test |
| `MAN-001` | `Manufacturer` | Medtronic plc (Ireland) |
| `MAN-002` | `Manufacturer` | Abbott Medical Devices (Germany) |
| `MAN-003` | `Manufacturer` | Siemens Healthineers AG (Germany) |
| `MAN-005` | `Manufacturer` | Roche Diagnostics GmbH (Germany) |
| `RSK-003` | `InjuryRisk` | Thrombosis - device-related |
| `RSK-005` | `MalfunctionRisk` | Device migration - implant |
| `RSK-009` | `MalfunctionRisk` | Battery depletion - premature |
| `RSK-014` | `DiagnosticRisk` | False positive result |
| `RSK-015` | `DiagnosticRisk` | False negative result |
| `INC-001` | `SeriousIncident` | HeartMate 3 thrombosis (→ `CorrectedIncident` inferido) |
| `INC-002` | `SeriousIncident` | Micra AV battery depletion |
| `INC-003` | `SeriousIncident` | Portico valve migration |
| `INC-011` | `SeriousIncident` | cobas e 801 false positive TSH |
| `INC-012` | `SeriousIncident` | cobas SARS-CoV-2 false negative |
| `FSCA-001` | `SafetyAlert` | HeartMate 3 anticoagulation alert |
| `FSCA-002` | `Recall` | Micra AV lot recall |
| `FSCA-003` | `InSituCorrection` | Portico valve technique update |
| `FSCA-009` | `UserNotice` | cobas e 801 biotin notice |
| `FSCA-010` | `Recall` | cobas SARS-CoV-2 reagent recall |

---








## cargar y validar en Protégé

### Abrir la ontología
`File → Open → ontology_fixed.ttl`

### Dónde encontrar cada cosa (para las capturas de los REQ)

| REQ | Navegación en Protégé |
|-----|-----------------------|
| REQ-1 · REQ-2 (clases en inglés, ≥20 clases) | Pestaña **Entities** → subpestaña **Classes** → árbol izquierdo |
| REQ-3 (rdfs:label y rdfs:comment) | Selecciona una clase → panel derecho **Annotations** |
| REQ-4 (Object Properties con dominio/rango) | Pestaña **Entities** → **Object properties** → selecciona `hasManufacturer` |
| REQ-5 (Data Properties con dominio/rango) | Pestaña **Entities** → **Data properties** → selecciona `deviceName` |
| REQ-6 (restricciones) | Pestaña **Entities** → **Classes** → selecciona `MedicalDevice` → panel **Description** → sección *SubClass Of* |
| REQ-7 (≥5 instancias) | Menú **Window → Views → Ontology views → Individuals by class** |
| REQ-8 (reglas SWRL) | Menú **Window → Views → Ontology views → Rules** |
| REQ-9 (enlaces externos) | Selecciona individuo `MAN-001` → panel **Annotations** → `rdfs:seeAlso` |

### Ejecutar el razonador (para REQ consistencia)
1. Menú **Reasoner → Pellet** (o HermiT si Pellet no está instalado)
2. **Reasoner → Start reasoner**
3. La ontología es consistente si ninguna clase aparece en rojo y `owl:Nothing` no tiene instancias
4. Verifica la inferencia: selecciona `INC-001` → el panel **Description** debería mostrar `CorrectedIncident` en cursiva (= inferido, no declarado)

---

## Validación externa

| Herramienta | URL | Qué valida |
|-------------|-----|------------|
| **OOPS!** | https://oops.linkeddata.es/ | Malas prácticas de diseño (pitfalls P01–P41) |
| **FOOPS!** | https://foops.linkeddata.es/FAIR_validator.html | Cumplimiento de principios FAIR |
| **WebVOWL** | https://service.tib.eu/webvowl/ | Visualización del grafo |
| **WIDOCO** | https://github.com/dgarijo/Widoco | Generación de documentación HTML |

---

## Fuentes de datos (PEC_2_Data/)

| Fichero | Clase OWL correspondiente | Instancias de ejemplo |
|---------|--------------------------|----------------------|
| `dispositivos.csv` | `MedicalDevice` y subclases | DEV-001 … DEV-017 |
| `fabricantes.csv` | `Manufacturer` | MAN-001 … MAN-014 |
| `riesgos.csv` | `Risk` y subclases | RSK-001 … RSK-015 |
| `incidentes.csv` | `Incident` y subclases | INC-001 … INC-014 |
| `acciones_correctivas.csv` | `FieldSafetyCorrectiveAction` y subclases | FSCA-001 … FSCA-010 |
