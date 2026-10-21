[@epetrianina](https://github.com/epetrianina)

# STAC API Spezifikation

## 1. Einleitung

Die **STAC API (SpatioTemporal Asset Catalog API)** ist eine HTTP-basierte Schnittstelle für den Zugriff auf STAC-Daten. Sie baut auf der STAC-Spezifikation auf und ermöglicht es Clients, STAC-Objekte wie **Catalogs, Collections und Items** über definierte API-Endpunkte abzurufen und zu durchsuchen.

Während ein statischer STAC Catalog hauptsächlich aus miteinander verlinkten JSON-Dateien besteht, ermöglicht eine STAC API dynamische Anfragen an einen Server. Dadurch können Anwendungen gezielt nach passenden räumlichen und zeitlichen Daten suchen, ohne den gesamten Katalog selbst durchsuchen zu müssen.

Für das Projekt **AIHealthMap** ist dies besonders relevant, da ein Web-Frontend oder ein KI-Agent über eine standardisierte Schnittstelle passende räumliche Gesundheitsdaten auffinden und abrufen kann.

## 2. STAC-Spezifikation vs. STAC API

Die **STAC-Spezifikation** beschreibt, wie räumlich-zeitliche Daten und ihre Metadaten strukturiert werden. Sie definiert unter anderem die zentralen STAC-Objekte **Catalog, Collection, Item und Asset** sowie deren Beziehungen untereinander.

Die **STAC API** baut auf dieser Datenstruktur auf und definiert eine standardisierte Web-Schnittstelle, über die STAC-Objekte über HTTP abgerufen und durchsucht werden können. Die STAC API beschreibt also vor allem, wie Clients auf STAC-Daten zugreifen können.

| STAC-Spezifikation | STAC API |
|---|---|
| definiert die Struktur von STAC-Daten | definiert den Zugriff auf STAC-Daten über eine Web-API |
| beschreibt z. B. Catalogs, Collections, Items und Assets | stellt HTTP-Endpunkte bereit |
| kann als statischer Katalog aus verlinkten JSON-Dateien veröffentlicht werden | ermöglicht dynamische Requests an einen Server |
| Navigation erfolgt hauptsächlich über Links zwischen STAC-Objekten | Daten können gezielt über Endpunkte abgefragt werden |
| benötigt nicht zwingend ein Backend | benötigt einen API-Server bzw. ein Backend, das Requests verarbeitet |

Die STAC API ersetzt die STAC-Spezifikation nicht, sondern verwendet deren Datenmodelle. Ein über die API zurückgegebenes **Item** bleibt also ein STAC Item; die API legt nur fest, wie auf dieses Item zugegriffen wird.

## 3. Architektur der STAC API

Bei einer STAC API kommunizieren Client und Server miteinander. Ein Client, zum Beispiel ein Web-Frontend, ein Skript oder ein KI-Agent, sendet einen HTTP-Request an einen Endpoint. Der Server verarbeitet die Anfrage und sendet eine Response zurück, meistens im JSON-Format.

```text
Client (Web-Frontend, Skript, KI-Agent)
            |
            | HTTP Request
            | z. B. GET /collections
            v
        STAC API
      (API-Server)
            |
            | Zugriff auf interne Datenquelle
            v
   Datenbank / STAC-Daten /
   andere Datenquellen
            |
            v
       JSON Response
            |
            v
          Client
```

## 4. Foundation Specifications und Conformance

Die STAC API ist modular aufgebaut. Das bedeutet, dass nicht jeder Server alle Funktionen unterstützen muss.

Es gibt drei wichtige Foundation Specifications:

- **Core**: enthält die grundlegende API-Struktur und die Landing Page `/`
- **Item Search**: ermöglicht die Suche nach Items über `/search`
- **Features**: stellt Collections und Items über Endpunkte wie `/collections` oder `/collections/{collectionId}/items` bereit

Über Conformance zeigt ein Server, welche Funktionen er unterstützt.

Das Feld conformsTo auf der Landing Page enthält diese Informationen.

Ein Client kann dadurch prüfen, welche Endpunkte und Möglichkeiten verfügbar sind, bevor er sie benutzt.

## 5. Wichtige Endpunkte

Welche Endpunkte verfügbar sind, hängt davon ab, welche Teile der STAC API ein Server unterstützt. Je nachdem, welche Teile der STAC API unterstützt werden, können zum Beispiel folgende Endpunkte verfügbar sein:

| Endpoint | Methode | Funktion |
|---|---|---|
| `/` | `GET` | Landing Page und Einstiegspunkt der API |
| `/collections` | `GET` | Liste der verfügbaren Collections |
| `/collections/{collectionId}` | `GET` | Informationen zu einer bestimmten Collection |
| `/collections/{collectionId}/items` | `GET` | Items innerhalb einer bestimmten Collection |
| `/collections/{collectionId}/items/{itemId}` | `GET` | ein bestimmtes Item |
| `/search` | `GET` / `POST` | Suche nach Items, auch über mehrere Collections hinweg |
| `/conformance` | `GET` | Informationen über unterstützte Standards |
| `/api` | `GET` | Beschreibung der API und ihrer Endpunkte |

## 6. Statischer STAC Catalog vs. STAC API

Ein statischer STAC Catalog besteht hauptsächlich aus miteinander verlinkten JSON-Dateien. Die einzelnen Catalogs, Collections und Items können über ihre Links gefunden und geöffnet werden.

Mit einer STAC API kann ein Client gezielt Daten über HTTP anfragen und suchen.

| Statischer STAC Catalog | STAC API |
|---|---|
| besteht aus verlinkten JSON-Dateien | stellt HTTP-Endpunkte bereit |
| sehr einfach zu hosten | benötigt einen API-Server |
| gut für kleinere oder selten veränderte Datenbestände | gut für größere oder häufiger veränderte Datenbestände |
| Navigation über Links | gezielte Requests und Suche möglich |
| wenig serverseitige Logik notwendig | Server verarbeitet Anfragen |

Ein statischer Catalog ist daher sinnvoll, wenn die Datenmenge überschaubar ist und sich nur selten ändert.

Eine STAC API eignet sich besonders bei großen Datenmengen, häufigen Suchanfragen oder wenn andere Anwendungen automatisch auf die Daten zugreifen sollen.

Für **AIHealthMap** ist eine STAC API wahrscheinlich besser geeignet, da das Web-Frontend und der KI-Agent gezielt nach passenden Daten suchen und nur relevante Ergebnisse abrufen können.

## 7. Typische Anwendungsfälle
### Beispiel 1: Suche nach Satellitenbildern

Satellitenarchive können sehr viele einzelne Aufnahmen enthalten. Ein Nutzer benötigt aber meistens nur Bilder für einen bestimmten Ort und Zeitraum. Mit einer STAC API können passende Items gezielt gesucht werden, anstatt einen kompletten statischen Katalog durchsuchen zu müssen.

### Beispiel 2: Suche nach Gesundheits- und Umweltdaten

In einer Anwendung wie AIHealthMap könnten regelmäßig neue Datensätze hinzukommen, zum Beispiel zu Luftqualität, Temperatur oder anderen gesundheitsrelevanten Faktoren. Ein Web-Frontend oder KI-Agent kann über die STAC API gezielt passende Collections und Items abrufen und anschließend im Web-Portal darstellen.

## 8. Bedeutung für AIHealthMap

Für AIHealthMap kann die STAC API als standardisierte Schnittstelle zwischen dem Web-Portal, dem KI-Agenten und den verfügbaren Geodaten dienen.

Ein möglicher Ablauf wäre:

```text
Nutzer
   ↓
KI-Agent interpretiert die Anfrage
   ↓
STAC-API-Request
   ↓
passende Collections und Items
   ↓
Darstellung im Web-Portal
```

## 9. Quellen

- STAC API Specification, Version 1.0.0:  
  https://github.com/radiantearth/stac-api-spec/tree/release/v1.0.0

- STAC API Overview:  
  https://github.com/radiantearth/stac-api-spec/blob/release/v1.0.0/overview.md

- STAC Specification Overview:  
  https://github.com/radiantearth/stac-api-spec/blob/release/v1.0.0/stac-spec/overview.md

- STAC API - Core OpenAPI Specification:  
  https://github.com/radiantearth/stac-api-spec/blob/release/v1.0.0/core/openapi.yaml

- Official STAC Website:  
  https://stacspec.org/