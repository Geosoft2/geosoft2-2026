Author: [@FinjaAus](https://github.com/FinjaAus)
# OGC Common Query Language 2 (CQL2)

## Motivation und Anwendungsbereich
Die Common Query Language 2 (CQL2) ist ein OGC (Open Geospatial Consortium) Standard. Sie ist kein eigener API Standard, sondern eine Filtersprache, die in APIs wie STAC oder OGC API-Features eingebaut werden kann, um Geodaten räumlich und zeitlich, aber auch nach normalen Attributen, filtern zu können.
Der Filter wird zusammen mit einer Abfrage an eine Sammlung von Ressourcen (z. B. Features oder STAC-Items) geschickt, und die Antwort enthält nur die Ressourcen, die dem Filter entsprechen. (https://www.ogc.org/standards/cql2/)

Ohne CQL2 versucht man, Daten über einfache Query-Strings wie ?city=Münster zu filtern. Komplexere Bedingungen wie ODER-Verknüpfungen oder zeitliche und räumliche Abfragen müssen je nach API unterschiedlich umgesetzt werden oder sind gar nicht möglich. CQL2 löst dieses Problem: Es reicht ein einziger Parameter "filter", dessen Inhalt, der gesamte Filterausdruck, durch den Standard einheitlich festgelegt ist. Voraussetzung ist, dass die jeweilige API CQL2 unterstützt.
Welche Felder sich filtern lassen, legt jede API bzw. Collection selbst fest. CQL2 vereinheitlicht die Filtersprache (Syntax und Operatoren). Welche Parameter verfügbar sind, kann man bei der API über die sogenannten Queryables abfragen. 

Ein typischer Anwendungsfall ist die Suche in einem STAC-Katalog mit Satellitenszenen: Einfache Parameter wie bbox und datetime grenzen nur Region und Zeitraum ein. Mit CQL2 lassen sich zusätzlich Bedingungen auf beliebigen Metadaten-Feldern und deren Verknüpfung ausdrücken, z. B. “weniger als 20 % Wolkenbedeckung und Plattform Sentinel-2A oder Sentinel-2B”.  
`eo:cloud_cover < 20 AND (platform = 'sentinel-2a' OR platform = 'sentinel-2b')`



## Syntax & Operatoren
### Operatoren
CQL2 ist modular aufgebaut und kann aus mehreren Bausteinen bestehen ("Requirement Classes"). Nicht jede Software, die die Filter auswertet, unterstützt alle Funktionen, bevor man die Filter schreibt, sollte man also prüfen was die API alles unterstützt. **Basic CQL2** muss jede Implementierung können, das beinhaltet Vergleiche und Verknüpfungen.
Mehrere Bedingungen werden mit `AND`, `OR` und `NOT` verknüpft. Es gibt noch weitere Gruppen, die unterstützt werden können, die räumlichen, zeitlichen und Array-Operatoren erkennt man dabei am Präfix `S_` (spatial), `T_` (temporal) und `A_` (array).


| Gruppe | Zweck | Operatoren |
|---|---|---|
| Basic CQL2 (Pflicht) | Werte vergleichen, Bedingungen verknüpfen | `=` `<>` `<` `<=` `>` `>=` `IS NULL`, `AND` `OR` `NOT` |
| Advanced Comparison | Textmuster, Bereiche, Listen | `LIKE` `BETWEEN` `IN` |
| Spatial | Räumliche Beziehungen | `S_INTERSECTS` `S_WITHIN` `S_CONTAINS` u. a. |
| Temporal | Zeitliche Beziehungen | `T_BEFORE` `T_AFTER` `T_DURING` `T_INTERSECTS` u. a. |
| Case-/Accent-insensitive | Groß-/Kleinschreibung bzw. Akzente ignorieren | `CASEI` `ACCENTI` |
| Array | Listen vergleichen, z. B. Schlagwörter | `A_OVERLAPS` `A_CONTAINS` u. a. |
| Weitere | Rechnen, eigene Funktionen, zwei Properties vergleichen | `+` `-` `*` `/`, Funktionen |

 
Diese Übersicht ist zusammenfassend und zeigt nicht vollständig alle einzelnen Gruppen und Operatoren, eine komplette Übersicht ist hier zu finden: [OGC CQL2 Standard, Kapitel 2 (6 und 7)](https://docs.ogc.org/is/21-065r2/21-065r2.html)

### Syntax: CQL2 Text und CQL2 JSON

zwei Schreibweisen (encodings) sind möglich, beide Drücken den gleichen Filter aus:
- **cql2-text**: Menschenlesbare Textkodierung (an SQL angelehnt), eignet sich bei HTTP-Query-Parameter
- **cql2-json**: json Kodierung für Kommunikation zwischen Maschinen, eignet sich für POST-Request Body

#### Aufbau
Ein CQL2-Filter besteht aus einer oder mehreren Bedingungen.

CQL2 Text:  
 **Eigenschaft, Operator, Wert** (z. B. eo:cloud_cover < 10), mehrere Bedingungen werden mit AND, OR und NOT verknüpft und können mit Klammern verschachtelt werden. Räumliche und zeitliche Operatoren werden wie Funktionen geschrieben: `S_INTERSECTS(Eigenschaft, Wert)`
 
 JSON:  
 jede Bedingung hat die gleiche Form und besteht aus 2 Teilen.  
 als erstes gibt `op` den Operator an, dann listet `args` auf was verglichen wird, erst Eigenschaft, dann Wert: `{"op": "Operator xy", "args":[Eigenschaft, Wert]}`  
 Diese können auch ineinander verschachtelt sein: `{"op": "Operator xy", "args":[{"op": "Operator yx", "args": [Eigenschaft, Wert]}]}`  

Gleiche Bedingung in beiden Schreibweisen:

```
eo:cloud_cover < 10
```

```json
{ "op": "<", "args": [ { "property": "eo:cloud_cover" }, 10 ] }
```

### Hinweise
- **Klammern setzen:** In CQL2-Text gilt `NOT` vor `AND` vor `OR`.
- **Koordinatenreihenfolge:** Geometrien werden immer als Länge, Breite (lon/lat) angegeben,
  sowohl in WKT als auch in GeoJSON (Standard-CRS: CRS84).

## Filter-Ausdrücke (Beispiele)
### 1. einfache Verknüpfung
Items mit mehr als 50 % Datenabdeckung und weniger als 10 % Wolken.

```
sentinel:data_coverage > 50 AND eo:cloud_cover < 10
```

```json
{
  "op": "and",
  "args": [
    { "op": ">", "args": [ { "property": "sentinel:data_coverage" }, 50 ] },
    { "op": "<", "args": [ { "property": "eo:cloud_cover" }, 10 ] }
  ]
}
```

Der Filter besteht aus zwei Vergleichen, die mit `AND` verbunden sind. In JSON bildet das `and` die äußere Ebene, und die beiden `args` sind die zwei Vergleiche. Bei einem POST-Request steht dieses Objekt im Feld `filter` des Request-Body (`{ "filter-lang": "cql2-json", "filter": { ... } }`).

### 2. Temporal Intersection
prüft, ob sich Datum oder Zeitangaben überschneiden

cql2-text
Operator funktioniert wie eine Funktion, hier steht in der Klammer Eigenschaft und Wert
```
T_INTERSECTS(datetime, INTERVAL('2020-11-11T00:00:00Z', '2020-11-12T00:00:00Z'))
``` 

 cql2-json
```json
{
    "op": "t_intersects",
    "args": [
      { "property": "datetime" },
      { "interval" : [ "2020-11-11T00:00:00Z", "2020-11-12T00:00:00Z"] }
    ]
  }
```
funktioniert für **s_intersects** genauso, statt `datetime` wird mit `geometry` verglichen: [weitere Beispiele](https://github.com/stac-api-extensions/filter#example-8-s_intersects-cql2-json-post)  
Der einfache STAC-Parameter `intersects` erlaubt nur eine Geometrie. Mit CQL2 lassen sich
mehrere räumliche Bedingungen per `OR` kombinieren:

**S_INTERSECTS**(geometry,POLYGON((-77.0824 38.7886,-77.0189 38.7886, ...))) **OR** **S_INTERSECTS**(geometry,POLYGON((-79.0935 38.7886,-79.0290 38.7886, ...)))

json: [code Beispiel](https://github.com/stac-api-extensions/filter#example-9-s_intersects-cql2-json-post)

### 3. Mehrere Collections mit unterschiedlichen Feldnamen filtern

Bei einer Suche über mehrere Collections können Felder unterschiedlich benannt sein, obwohl sie
dasselbe angeben, z. B. die Datenabdeckung bei Sentinel-2 (`sentinel:data_coverage`) und
Landsat (`landsat:coverage_percent`). Bei Landsat-Items ist `sentinel:data_coverage` NULL,
der Vergleich ergibt also „unbekannt“. Durch die `OR`-Verknüpfung reicht es, wenn eine Bedingung erfüllt wird, dadurch lassen sich mehrere collections auf einmal durchsuchen. Der dritte Teil nimmt zusätzlich Items auf, die keines
der beiden Felder haben und sonst herausfallen würden.
(angepasstes Beispiel nach [README Example 10](https://github.com/stac-api-extensions/filter#example-10-using-is-null))
```
sentinel:data_coverage > 50 OR landsat:coverage_percent > 50 OR (sentinel:data_coverage IS NULL AND landsat:coverage_percent IS NULL)
```
json: [code Beispiel](https://github.com/stac-api-extensions/filter#example-10-cql2-json-post)  
(dort mit `landsat:coverage_percent < 10`, damit es Sinn ergibt denkt euch aber > 50)


**Weitere Beispiele:** [STAC Filter Extension](https://github.com/stac-api-extensions/filter/tree/v1.0.0-rc.4#examples),
[CQL2-Spezifikation](https://docs.ogc.org/is/21-065r2/21-065r2.html)

## Einsatz im STAC/OGC-API-Kontext

CQL2 kann nur bei APIs genutzt werden, die es unterstützen, z. B. OGC API – Features oder STAC

Welche Operatoren ein Server unterstützt (vgl. Tabelle oben), steht auf seiner Startseite
im Feld `conformsTo`. Welche Felder filterbar sind, zeigen die Queryables (`/queryables`).
Der Filter wird im Parameter `filter` geschickt, `filter-lang` gibt die Schreibweise an:

```
GET /search?filter=eo:cloud_cover < 10
```
```json
POST /search
{ "filter-lang": "cql2-json",
  "filter": { "op": "<", "args": [ { "property": "eo:cloud_cover" }, 10 ] } }
```