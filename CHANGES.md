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

**Security:** `bearerAuth` mit `dlx.derive` Scope

**Wichtige Hinweise:**
- Der `dlx.derive` Scope wird nur vergeben, wenn der erstellende Token mit `allowDerivation: true` erstellt wurde
- Bei Verwendung von bearerAuth für DLX-Creator Endpunkte müssen die erstellten/aktualisierten Token ein Subset der erstellenden Token-Berechtigungen sein
- `expiresAt` darf nicht über die Gültigkeit des erstellenden Tokens hinausgehen
- Die Datenobjekte (Studien) können eingeschränkt werden (weniger Studien als Eltern-Token)

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
| `/tokens` | POST | Bearer¹ | Token erstellen |
| `/tokens` | GET | Bearer¹ | Alle Token auflisten (paginiert) |
| `/tokens/{token}` | GET | Bearer¹ | Token-Details |
| `/tokens/{token}` | PUT | Bearer¹ | Token aktualisieren |
| `/tokens/{token}` | DELETE | Bearer¹ | Token löschen |

¹ Bearer mit `dlx.derive` Scope für Token Derivation (nur eigene abgeleitete Tokens)

**Wichtige Hinweise:**
- Der `dlx.derive` Scope wird nur vergeben, wenn der erstellende Token mit `allowDerivation: true` erstellt wurde
- Bei Verwendung von bearerAuth für DLX-Creator Endpunkte müssen die erstellten/aktualisierten Token ein Subset der erstellenden Token-Berechtigungen sein
- `expiresAt` darf nicht über die Gültigkeit des erstellenden Tokens hinausgehen
- Die Datenobjekte (Studien) können eingeschränkt werden (weniger Studien als Eltern-Token)

#### Info
| Endpoint | Methode | Auth | Beschreibung |
|------|---------|------|-----------|
| `/api_info` | GET | None | API-Version und Capabilities |
| `/tfa_info` | GET | Bearer | Unterstützte TFA-Optionen (authentifiziert) |

---

## 🔐 Authentifizierung

### JWT Bearer Authentication
- **Scope:** `dlx.consumer` - Download-Endpoints
- **Scope:** `dlx.derive` - Token-Management über `/tokens`

**Wichtige Hinweise:**
- Der `dlx.derive` Scope wird nur vergeben, wenn der erstellende Token mit `allowDerivation: true` erstellt wurde
- Bei Verwendung von bearerAuth für DLX-Creator Endpunkte müssen die erstellten/aktualisierten Token ein Subset der erstellenden Token-Berechtigungen sein

---

## 🎯 Capabilities in `/api_info`

| Capability | Typ | Beschreibung |
|------|-----|---|
| `download` | boolean | DLX-Consumer Endpoints (token, list, download, downloadall) |
| `tokens` | boolean | Token-Management Endpoints (POST/GET/PUT/DELETE /tokens) |
| `derivation` | boolean | Token-Derivation (erstellen abgeleiteter Token) |
| `dicomweb` | object | DICOMweb-Capabilities (WADO-RS, QIDO-RS) für direkte DICOM-Abfrage |

---

## 📈 Versionshistorie

| Version | Datum | Änderungen |
|---------|-----|--------|
| **2.0** | April 2026 | Token-Management, bearerAuth mit dlx.derive für DLX-Creator, DICOMweb-Integration |
| **1.0** | März 2025 | DIN/TS 19455:2025-03 |