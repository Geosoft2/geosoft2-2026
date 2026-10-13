Author: [@torsten-geoloek](https://github.com/torsten-geoloek)

# STAC-Spezifikation
STAC macht Geodaten durch eine einheitliche Struktur leichter auffindbar, durchsuchbar und nutzbar.

## 1 Einführung 

### 1.1 Was ist STAC?
- SpatioTemporal Asset Catalog
- Ziel: eine einheitliche Standardsprache für die Beschreibung von Geodaten
    - Dadurch können geografische Daten einfacher gefunden, durchsucht und abgefragt werden.
- Basiert auf miteinander verknüpften JSON-Dateien.
- Ist flexibel und erweiterbar und kann an unterschiedliche Anwendungsbereiche angepasst werden.



## 2 Grundkonzept
- STAC besteht aus drei zentralen Komponenten: Item, Catalog und Collection.
- Die Komponenten werden über Links miteinander verknüpft.
- Ein Item beschreibt ein oder mehrere räumlich-zeitliche Assets und enthält zugehörige Metadaten.
- Ein Catalog dient zur Organisation und Verknüpfung von Items, Collections und weiteren Catalogs.
- Eine Collection fasst zusammengehörige Items zusammen und enthält zusätzliche Informationen über die Datensammlung.
- Die eigentlichen Daten werden als Assets referenziert; die STAC-Dateien dienen dazu, diese Daten zu beschreiben und auffindbar zu machen.


## 3 Wichtige Begriffe

### 3.1 Item
- Grundbaustein von STAC.
- Beschreibt ein oder mehrere Assets mithilfe von GeoJSON und zusätzlichen Metadaten.
- Enthält beispielsweise Informationen zur geografischen Lage und zum Zeitpunkt der Datenerfassung.

#### Item fields 
(von hier: https://github.com/radiantearth/stac-spec/blob/master/item-spec/item-spec.md#item-fields)


| Field Name | Type | Description |
|---|---|---|
| `type` | string | **REQUIRED.** Type of the GeoJSON Object. MUST be set to `Feature`. |
| `stac_version` | string | **REQUIRED.** The STAC version the Item implements. |
| `stac_extensions` | `[string]` | A list of extensions the Item implements. |
| `id` | string | **REQUIRED.** Provider identifier. The ID should be unique within the [Collection](../collection-spec/collection-spec.md) that contains the Item. |
| `geometry` | GeoJSON Geometry Object \| null | **REQUIRED.** Defines the full footprint of the asset represented by this item, formatted according to RFC 7946, [section 3.1](https://tools.ietf.org/html/rfc7946#section-3.1) if a geometry is provided or [section 3.2](https://tools.ietf.org/html/rfc7946#section-3.2) if *no* geometry is provided. |
| `bbox` | `[number]` | **REQUIRED if `geometry` is not `null`, prohibited if `geometry` is `null`.** Bounding Box of the asset represented by this Item, formatted according to [RFC 7946, section 5](https://tools.ietf.org/html/rfc7946#section-5). |
| `properties` | [Properties Object](#properties-object) | **REQUIRED.** A dictionary of additional metadata for the Item. |
| `links` | [Link Object](#links) | **REQUIRED.** List of link objects to resources and related URLs. See the [best practices](../best-practices.md#use-of-links) for details on when the use of `self` links is strongly recommended. |
| `assets` | `Map<string, Asset Object>` | **REQUIRED.** Dictionary of asset objects that can be downloaded, each with a unique key. |
| `collection` | string | The `id` of the STAC Collection this Item references to. This field is **required** if a link with a `collection` relation type is present and is **not allowed** otherwise. |

### 3.2 Collection
- Fasst zusammengehörige STAC Items zu einer Sammlung zusammen.
- Enthält zusätzliche Metadaten, beispielsweise zur räumlichen und zeitlichen Ausdehnung, zur Lizenz und zu den Datenanbietern.
![alt text](image-2.png)

### 3.3 Catalog
- Organisiert und verknüpft STAC Items, Collections und weitere Kataloge.
- Dient als Einstiegspunkt zur Navigation durch die Daten.
![alt text](image-1.png)

### 3.4 Asset
- Eine Datei, die räumlich-zeitliche Informationen über die Erde enthält, z. B. ein Satellitenbild oder eine LiDAR-Punktwolke.
- Das Asset enthält die eigentlichen Daten; ein STAC Item verweist darauf und beschreibt sie mit Metadaten.

### 3.5 Link
- Verknüpft STAC-Komponenten miteinander.
- Ermöglicht die Navigation zwischen Items, Catalogs und Collections sowie den Verweis auf zugehörige Daten.

### 3.6 GeoJSON
- Format zur Beschreibung geografischer Objekte und ihrer räumlichen Eigenschaften.
- Ein STAC Item basiert auf einem GeoJSON-Feature.

### 3.7 Metadaten
- Informationen, die einen Datensatz näher beschreiben.
- Beispiele sind Aufnahmezeitpunkt, räumliche Ausdehnung, Lizenz und Datenanbieter.


## 4 Erweiterungen
- STAC ist flexibel und kann durch sogenannte "Extensions" erweitert werden.
- Extensions ergänzen die STAC-Spezifikation um zusätzliche Felder und Funktionen für bestimmte Datentypen oder Anwendungsbereiche.
- Sie ermöglichen es beispielsweise, spezielle Informationen zu bestimmten Geodaten zu beschreiben.
- Die STAC-Community entwickelt gemeinsam Erweiterungen und zusätzliche Metadatenstandards.

## 5 Quellen
- https://stacspec.org/en/tutorials/intro-to-stac/
- https://github.com/radiantearth/stac-spec/