# DLX OpenAPI Spezifikation - Neuerungen seit DIN/TS 19455:2025-03
**Referenz:** DIN/TS 19455:2025-03 https://www.dinmedia.de/de/vornorm/din-ts-19455/387178367

Diese Dokumentation beschreibt die neuen Features und Erweiterungen, die seit dem ursprünglichen DIN/TS 19455:2025-03 Standard hinzugefügt wurden.
Die DLX API wurde von einem einfachen Token-basierten Link-Management-System zu einer umfassenden Plattform für den sicheren Austausch medizinischer Bilder mit erweiterten Funktionen und professioneller Authentifizierung weiterentwickelt.

---

## 📋 Einführung

Die DLX (DICOM Link Exchange) API basiert ursprünglich auf der deutschen DIN-Norm und ermöglicht den sicheren Austausch medizinischer Bilder über Token-basierte Links. Seit der Initialisierung wurden umfangreiche Erweiterungen implementiert, um die API für industrielle Einsatzszenarien zu professionalisieren.

**Version:** 2.0  
**Basis:** DIN/TS 19455:2025-03
**Letzte Aktualisierung:** April 2026

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

**Security:** 
- `adminBearerAuth` mit Scope `dlx.tokens` (via `/admin_auth`)
- `bearerAuth` mit Scope `dlx.derive` (via TFA-Authentifizierung)

---

### 2. Admin-Authentifizierung

JWT-basierte Authentifizierung für administrative Zugriffe.

**Neuer Endpoint:**
- `POST /admin_auth` - Authentifizierung mit Admin-Credentials → JWT

**Neue Security-Scheme:**
- `adminBearerAuth` - JWT Bearer für administrative Endpoints

**OAuth 2.0 Scopes:**
- `dlx.tokens` - Erstellen und Verwalten von Token-Links
- `dlx.import` - Importieren externer DLX-Links

---

### 3. Import-Funktionalität

Austausch von Daten zwischen DLX-Systemen.

**Neuer Endpoint:**
- `POST /import` - Importieren externer DLX-Links

**Beschreibung:**
- Importiert Daten von einem externen DLX-Link
- Synchroner Trigger mit asynchronem Hintergrundprozess
- Sofortige Antwort (200) oder Status-Update (202) mit `statusUrl`

**Security:** `adminBearerAuth` mit `dlx.import` Scope

---

### 4. DICOMweb-Integration

Unterstützung für DICOMweb-Dienste (WADO-RS und QIDO-RS) zur direkten Abfrage von DICOM-Daten.

**Neue Capability:** `dicomweb` in `/api_info`

Die `dicomweb` Capability in der `/api_info` Antwort gibt an, ob WADO-RS (Retrieve) und QIDO-RS (Query) für DICOM-Daten verfügbar sind. DLX Bearer-Tokens können für die Authentifizierung an diesen Endpoints verwendet werden, eingeschränkt auf die Study UIDs des jeweiligen Tokens.

**JWT-Authentifizierung:**
- DLX Bearer-Token wird für DICOMweb-Endpoints akzeptiert
- Eingeschränkt auf Study UIDs des Tokens

---

### 5. Context-basierte Tags

Zusätzliche Tags zur Klassifizierung der Authentifizierungsmethoden:

| Tag | Beschreibung |
|-----|--------------|
| `DLX-ConsumerContext` | Endpoints via TFA-Authentifizierung (/token → /tokentfa → JWT). Scope: `dlx.derive` |
| `DLX-ManagementContext` | Endpoints via Admin-Authentifizierung (/admin_auth → JWT). Scopes: `dlx.tokens`, `dlx.import` |

Einige Endpoints (z.B. `/tokens` CRUD) unterstützen beide Contexte mit unterschiedlichen Authentifizierungsmethoden.

---

## 📊 Aktuelle API-Struktur

### Endpoints nach Rolle

#### DLX-Download (Consumer)
| Endpoint | Methode | Auth | Beschreibung |
|------|---------|------|-----------|
| `/token/{value}` | GET | None | TFA-Fragen abrufen |
| `/tokentfa/{value}` | POST | None | TFA-Antworten → JWT |
| `/list` | GET | JWT | Liste aller Datenobjekte |
| `/download/{id}` | GET | JWT | Download einzelnes Objekt |
| `/downloadall` | GET | JWT | Download aller Objekte als ZIP |

#### DLX-Tokens
| Endpoint | Methode | Auth | Beschreibung |
|------|---------|------|-----------|
| `/tokens` | POST | adminBearerAuth / bearerAuth¹ | Token erstellen |
| `/tokens` | GET | adminBearerAuth / bearerAuth¹ | Alle Token auflisten (paginiert) |
| `/tokens/{token}` | GET | adminBearerAuth / bearerAuth¹ | Token-Details |
| `/tokens/{token}` | PUT | adminBearerAuth / bearerAuth¹ | Token aktualisieren |
| `/tokens/{token}` | DELETE | adminBearerAuth / bearerAuth¹ | Token löschen |

¹ `bearerAuth` mit `dlx.derive` Scope für Token Derivation (nur eigene abgeleitete Tokens)

#### DLX-Import
| Endpoint | Methode | Auth | Beschreibung |
|------|---------|------|-----------|
| `/import` | POST | adminBearerAuth | Externen DLX-Link importieren |

#### DLX-Info
| Endpoint | Methode | Auth | Beschreibung |
|------|---------|------|-----------|
| `/api_info` | GET | None | API-Version und Capabilities |
| `/tfa_info` | GET | adminBearerAuth | Unterstützte TFA-Optionen |

---

## 🔐 Authentifizierung

### JWT Bearer Authentication (`bearerAuth`)
Für DLX-Download Endpoints nach TFA-Authentifizierung.
- **Scope:** `dlx.derive` - Eingeschränkte Untertokens erstellen (Token Derivation)

### Admin Bearer Authentication (`adminBearerAuth`)
Für administrative Endpoints nach `/admin_auth` Authentifizierung.
- **Scope:** `dlx.tokens` - Token-Management
- **Scope:** `dlx.import` - Import-Endpoints

---

## 🎯 Capabilities in `/api_info`

| Capability | Typ | Beschreibung |
|------|-----|---|
| `download` | boolean | DLX-Download Endpoints (token, list, download, downloadall) |
| `tokens` | boolean | Token-Management Endpoints (POST/GET/PUT/DELETE /tokens) |
| `import` | boolean | Import-Endpoint (/import) |
| `derivation` | boolean | Token-Derivation (erstellen abgeleiteter Token) |
| `dicomweb` | object | DICOMweb-Capabilities (WADO-RS, QIDO-RS) für direkte DICOM-Abfrage |

---

## 📈 Versionshistorie

| Version | Datum | Änderungen |
|---------|-----|--------|
| **2.0** | April 2026 | Token-Management, Admin-Authentifizierung, Import-Funktionalität, DICOMweb-Integration, Context-basierte Tags |
| **1.0** | März 2025 | DIN/TS 19455:2025-03 |
