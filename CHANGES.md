# DLX OpenAPI Spezifikation - Neuerungen seit DIN/TS 19455:2025-03
**Referenz:** DIN/TS 19455:2025-03 https://www.dinmedia.de/de/vornorm/din-ts-19455/387178367

Diese Dokumentation beschreibt die neuen Features und Erweiterungen, die seit dem ursprünglichen DIN/TS 19455:2025-03 Standard hinzugefügt wurden.
Die DLX API wurde von einem einfachen Token-basierten Link-Management-System zu einer umfassenden Plattform für den sicheren Austausch medizinischer Bilder mit erweiterten Funktionen und professioneller Authentifizierung weiterentwickelt.

---

## 📋 Einführung

Die DLX (DICOM Link Exchange) API basiert ursprünglich auf der deutschen DIN-Norm und ermöglicht den sicheren Austausch medizinischer Bilder über Token-basierte Links. Seit der Initialisierung wurden umfangreiche Erweiterungen implementiert, um die API für industrielle Einsatzszenarien zu professionalisieren.

**Version:** 2.0  
**Basis:** DIN/TS 19455:2025-03
**Letzte Aktualisierung:** März 2026

---

## 🆕 Neue Features

### 1. Token-Management

Vollständiges CRUD für DLX-Token-Links mit Paginierung, Filterung und Token-Derivation.

**Neue Endpoints:**
- `POST /tokens` - Token erstellen
- `GET /tokens` - Alle Token auflisten (mit Paginierung und Filterung)
- `GET /tokens/{token}` - Token-Details abrufen
- `PUT /tokens/{token}` - Token aktualisieren
- `DELETE /tokens/{token}` - Token löschen

**Neue Funktionen:**
- **Paginierung** - Seite-basierte Auflistung mit `page`, `pageSize`, `sort`
- **Filterung** - Filtern nach `patientId`, `studyInstanceUid`, `accessionNumber`
- **Token-Derivation** - Erstellen abgeleiteter Token mit eingeschränktem Scope

**Security:** `adminOidcAuth` mit `dlx.creator`/`dlx.admin` oder `bearerAuth` mit `dlx.derive`

---

### 2. OIDC-Authentifizierung

Professionelle Authentifizierung für administrative Endpoints.

**Neuer Endpoint:**
- `GET /.well-known/openid-configuration` - OIDC Discovery Document (RFC 8414)

**Neue Security-Schemes:**
- `adminOidcAuth` - OpenID Connect für administrative Endpoints

**OAuth 2.0 Scopes:**
- `dlx.creator` - Erstellen und Verwalten von Token-Links
- `dlx.importer` - Importieren externer DLX-Links
- `dlx.admin` - Vollständiger Admin-Zugriff (überschreibt andere Scopes)

---

### 3. Import-Funktionalität

Austausch von Daten zwischen DLX-Systemen.

**Neuer Endpoint:**
- `POST /import` - Importieren externer DLX-Links

**Beschreibung:**
- Importiert Daten von einem externen DLX-Link
- Synchroner Trigger mit asynchronem Hintergrundprozess
- Sofortige Antwort (200) oder Status-Update (202) mit `statusUrl`

**Security:** `adminOidcAuth` mit `dlx.importer` oder `dlx.admin` Scope

---

### 4. DICOMweb-Integration

Unterstützung für DICOMweb-Dienste (WADO-RS und QIDO-RS) zur direkten Abfrage von DICOM-Daten.

**Neue Capability:** `dicomweb` in `/api_info`

Die `dicomweb` Capability in der `/api_info` Antwort gibt an, ob WADO-RS (Retrieve) und QIDO-RS (Query) für DICOM-Daten verfügbar sind. DLX Bearer-Tokens können für die Authentifizierung an diesen Endpoints verwendet werden, eingeschränkt auf die Study UIDs des jeweiligen Tokens.

**JWT-Authentifizierung:**
- DLX Bearer-Token wird für DICOMweb-Endpoints akzeptiert
- Eingeschränkt auf Study UIDs des Tokens

---

## 📊 Aktuelle API-Struktur

### Endpoints nach Rolle

#### DLX-Consumer (Download)
| Endpoint | Methode | Auth | Beschreibung |
|------|---------|------|-----------|
| `/token/{value}` | GET | None | TFA-Fragen abrufen |
| `/tokentfa/{value}` | POST | None | TFA-Antworten → JWT |
| `/list` | GET | JWT | Liste aller Datenobjekte |
| `/download/{id}` | GET | JWT | Download einzelnes Objekt |
| `/downloadall` | GET | JWT | Download aller Objekte als ZIP |

#### DLX-Creator (Token-Management)
| Endpoint | Methode | Auth | Beschreibung |
|------|---------|------|-----------|
| `/tokens` | POST | OIDC/Bearer | Token erstellen |
| `/tokens` | GET | OIDC/Bearer | Alle Token auflisten (paginiert) |
| `/tokens/{token}` | GET | OIDC/Bearer | Token-Details |
| `/tokens/{token}` | PUT | OIDC/Bearer | Token aktualisieren |
| `/tokens/{token}` | DELETE | OIDC/Bearer | Token löschen |

#### DLX-Importer (Data Import)
| Endpoint | Methode | Auth | Beschreibung |
|------|---------|------|-----------|
| `/import` | POST | OIDC | Externen DLX-Link importieren |

#### Info
| Endpoint | Methode | Auth | Beschreibung |
|------|---------|------|-----------|
| `/api_info` | GET | None | API-Version und Capabilities |
| `/.well-known/openid-configuration` | GET | None | OIDC Discovery Document |

---

## 🔐 Authentifizierung

### JWT Bearer Authentication
- **Scope:** `dlx.consumer` - Download-Endpoints
- **Scope:** `dlx.derive` - Token-Derivation über `/tokens`

### OpenID Connect
- **Scope:** `dlx.creator` - Token-Management
- **Scope:** `dlx.importer` - Import-Endpoints
- **Scope:** `dlx.admin` - Vollständiger Admin-Zugriff

---

## 🎯 Capabilities in `/api_info`

| Capability | Typ | Beschreibung |
|------|-----|---|
| `download` | boolean | DLX-Consumer Endpoints (token, list, download, downloadall) |
| `tokens` | boolean | Token-Management Endpoints (POST/GET/PUT/DELETE /tokens) |
| `import` | boolean | Import-Endpoint (/import) |
| `oidc` | boolean | OIDC Discovery Endpoint (/.well-known/openid-configuration) |
| `derivation` | boolean | Token-Derivation (erstellen abgeleiteter Token) |
| `dicomweb` | object | DICOMweb-Capabilities (WADO-RS, QIDO-RS) für direkte DICOM-Abfrage |

---

## 📈 Versionshistorie

| Version | Datum | Änderungen |
|---------|-----|--------|
| **2.0** | März 2026 | Token-Management, OIDC-Authentifizierung, Import-Funktionalität, DICOMweb-Integration |
| **1.0** | März 2025 | DIN/TS 19455:2025-03 |