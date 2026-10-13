# STAC Tooling – Werkzeuge für AIHealthMap

[@k-tocek](https://github.com/k-tocek)

*Geosoftware II, WS 26/27 – Handout zum Thema „STAC Tooling“*

## 1. Worum es geht

Dieses Handout stellt gängige Werkzeuge rund um STAC vor und bewertet, welche davon für AIHealthMap (KI-gestützte Web-Mapping-Anwendung zur Entdeckung von Gesundheitsdaten) sinnvoll sind. Es soll als Einstieg und als Entscheidungshilfe für die Projektarbeit dienen. Behandelt werden STAC Index, STAC Browser, pystac, pystac-client, stac-js und ol-stac samt OpenLayers sowie Alternativen, Stolperfallen und eine Empfehlung.

## 2. STAC in Kürze

STAC (*SpatioTemporal Asset Catalog*) ist ein Standard, um räumlich-zeitliche Daten einheitlich zu beschreiben und durchsuchbar zu machen. STAC beschreibt nur die **Metadaten**, die Daten selbst liegen woanders. Die vier Bausteine sind:

| Baustein | Bedeutung |
|---|---|
| **Item** | Ein einzelnes räumlich-zeitliches Objekt mit Geometrie, Zeitangabe und Eigenschaften (technisch ein GeoJSON-Feature) |
| **Asset** | Verweis auf die eigentliche Datei, z. B. ein GeoTIFF; hängt an einem Item oder einer Collection |
| **Collection** | Beschreibt einen Datensatz mit gemeinsamen Metadaten wie räumlicher und zeitlicher Ausdehnung, Lizenz und Anbieter |
| **Catalog** | Verknüpft Items, Collections und weitere Kataloge über Links |

Zwei Dinge werden oft vermischt:

- **STAC-Spezifikation:** legt nur die Struktur der JSON-Dateien fest. Ein *statischer Katalog* ist lediglich eine Sammlung solcher Dateien und nicht dynamisch durchsuchbar.
- **STAC API:** eine Web-Schnittstelle (aufbauend auf OGC API – Features), die dynamische Suche ermöglicht. Welche Funktionen sie bietet (Item-Suche, Filter mit CQL2, Freitext, Sortierung, Collection-Suche), steht im Feld `conformsTo` der Startseite und ist **je Anbieter unterschiedlich**.

## 3. Anforderungen von AIHealthMap an die Werkzeuge

Aus der Projektbeschreibung ergeben sich folgende Anforderungen:

1. Gesundheitsbezogene Geodaten sollen **providerübergreifend auffindbar** sein.
2. Die Zielgruppe sind **Forschende aus der Medizin ohne Geo-Kenntnisse**, daher braucht es eine einfache, geführte Oberfläche.
3. Ein **KI-Agent** unterstützt bei Suche, Darstellung und Analyse, die Suche muss also maschinell ansteuerbar sein.
4. Die Lösung ist eine **Web-Anwendung auf Basis von STAC** mit einer **Erweiterung der STAC-(API-)Spezifikation** für Gesundheitsdaten.
5. Optional: PDF-Gesundheitsdaten mit textuellen Geomarkern und erweiterte Geoanalysen.

Daraus folgen fünf Aufgabenfelder, an denen die Werkzeuge gemessen werden: **Entdecken, Suchen, Darstellen, Erzeugen/Erweitern, KI-Anbindung.**

## 4. Die Werkzeuge im Einzelnen

### 4.1 STAC Index – Entdecken

Der [STAC Index](https://stacindex.org) ist ein Verzeichnis öffentlicher STAC-Kataloge und -APIs sowie zugehöriger Software und Tools. Der Katalogbereich basiert auf dem STAC Browser.

- **Stärken:** Erster Anlaufpunkt, um zu sehen, was es überhaupt gibt.
- **Grenzen:** Ein Eintrag garantiert weder Erreichbarkeit noch Datenqualität. Ein Gesundheitsbezug lässt sich nur über die Stichwortsuche herausfinden, er ist kein Filterkriterium.
- **Nützlich für uns:** Es gibt Community-Projekte, die die Katalogliste maschinenlesbar machen (z. B. [giswqs/stac-index-catalogs](https://github.com/giswqs/stac-index-catalogs)). Mit [STAC-Atlas](https://github.com/SpatioCore/STAC-Atlas) existiert außerdem ein Projekt, das Collections aus den im STAC Index gelisteten Katalogen per Crawler indiziert und per API bereitstellt. Das ist inhaltlich sehr nah an unserem Projektziel. Reifegrad und Eignung müssten wir noch prüfen.

**Bewertung:** Für die Recherche in der Anfangsphase sehr gut, als Laufzeitkomponente der App eher nicht geeignet, da er keinen Ersatz für eine eigene Suche bietet.

### 4.2 STAC Browser – Entdecken und Vorlage für die Oberfläche

Der [STAC Browser](https://github.com/radiantearth/stac-browser) ist eine Webanwendung (Vue) zum Durchsuchen von statischen STAC-Katalogen **und** STAC APIs. Er lässt sich lokal oder per Docker betreiben und gegen einen beliebigen Katalog konfigurieren. Er unterstützt viele STAC-Versionen und kann inoffiziell auch einige OGC-API-Records/Features-Server lesen.

- **Anpassbarkeit:** Eigene Metadatenfelder lassen sich über die Registry von `stac-fields` registrieren und dann schön darstellen. Das passt direkt zur Aufgabe, STAC auf Gesundheitsdaten zu erweitern. Dazu kommen konfigurierbare Basiskarten, Themes, Mehrsprachigkeit und steckbare Erweiterungen.
- **Stärken:** Sofort nutzbare Katalog-Oberfläche inklusive Suche. Sehr gut, um zu verstehen, wie STAC-Daten aussehen.
- **Grenzen:** Der Browser richtet sich an Leute, die STAC kennen, nicht an medizinische Forschende. Er ist kein KI-gestütztes Portal. Das README weist darauf hin, dass die Finanzierung für die Wartung begrenzt ist, und für die Zukunft ist ein größerer Versionssprung mit neuem Layout geplant. Als Basis einer eigenen App ist er daher ein bewegliches Ziel.

**Bewertung:** Zum Erkunden und als Referenz für gute STAC-Bedienung unverzichtbar. Als Grundlage unserer App nur, wenn das Team bewusst Vue und einen Fork in Kauf nimmt. Sonst ist eine eigene schlanke Oberfläche besser, bei der man Ideen übernimmt.

### 4.3 pystac – Erzeugen und Lesen

[PySTAC](https://github.com/stac-utils/pystac) ist eine Python-Bibliothek, mit der STAC-Objekte (Catalog, Collection, Item, Asset) gelesen, erstellt, verändert, gespeichert und gegen die Schemata validiert werden. Sie unterstützt STAC-Extensions.

- **Einsatz bei uns:** Beschreibung eigener Gesundheitsdatensätze (z. B. Auswertungen pro Landkreis) und Aufbau eines eigenen Katalogs, einschließlich der Felder unserer Health-Extension.
- **Stärken:** Weit verbreitet, solide Dokumentation, Validierung.
- **Grenzen:** Reine Metadaten-Bibliothek, keine Suchoberfläche und keine Geodatenanalyse.

### 4.4 pystac-client – Suchen

[pystac-client](https://github.com/stac-utils/pystac-client) greift auf STAC APIs zu. Zentral sind vier Funktionen:

| Funktion | Zweck |
|---|---|
| Verbindung öffnen | Zugriff auf eine STAC API |
| Item-Suche | Suche nach Collection, Gebiet (bbox oder Geometrie), Zeitraum, Filter (CQL2), Sortierung, Feldauswahl und Maximalzahl |
| Collection-Suche | Suche nach Datensätzen inklusive Freitext (nur, wenn der Server die Collection-Search-Erweiterung anbietet) |
| Queryables | Welche Eigenschaften sind filterbar? |

**Warum ist das für AIHealthMap zentral?** Ein Sprachmodell kann Nutzerfragen in genau diese strukturierten Parameter übersetzen. Die Suche ist eine klar definierte „Funktion“, die sich als Tool für einen KI-Agenten bereitstellen lässt (Tool-Calling).

- **Stärken:** Deckt die gängigen Suchfunktionen ab, inklusive Seitenweiser Abfrage, gute Dokumentation.
- **Grenzen:** Python, also Backend und nicht direkt im Browser. Der Funktionsumfang hängt vom Server ab. Ein gefundenes Item heißt nicht, dass das Asset frei abrufbar ist (z. B. müssen beim Microsoft Planetary Computer die URLs signiert werden).

### 4.5 stac-js – Lesen im Frontend

[stac-js](https://github.com/moregeo-it/stac-js) ist ein schreibgeschützter Wrapper um ein bereits geladenes STAC-JSON-Objekt. Die Bibliothek lädt nichts per HTTP und erzeugt nichts. Ihr Nutzen liegt in Hilfsmethoden (Ausdehnung, Zeitraum, Vorschaubilder, Links) und darin, dass sie Objekte älterer STAC-Versionen auf die neueste Version hebt. Das ist hilfreich, weil APIs unterschiedliche Versionen liefern.

**Bewertung:** Praktisch für Ergebnislisten und Detailansichten, ersetzt aber weder HTTP-Client noch Suche. Wer nur zwei Felder braucht, kommt auch mit rohem JSON aus. Bei vielen Versionen und Objekttypen lohnt sich stac-js. Außerdem bildet es die Grundlage von ol-stac.

### 4.6 OpenLayers und ol-stac – Darstellen

[OpenLayers](https://openlayers.org/) ist eine Bibliothek für interaktive Webkarten. [ol-stac](https://github.com/moregeo-it/ol-stac) ist eine STAC-Ebenengruppe für OpenLayers: Man übergibt eine URL oder STAC-JSON und erhält passende Karten-Layer, etwa für Geometrien, Vorschaubilder und GeoTIFF-Assets.

- **Wichtig – Item ist nicht gleich Asset:** Die Geometrie eines Items lässt sich immer zeichnen. Ob ein Asset sichtbar wird, hängt vom Format ab. COG/GeoTIFF funktioniert direkt, NetCDF oder Parquet nicht ohne Weiteres.
- **Stärken:** Spart viel Eigenentwicklung, gute Unterstützung für Projektionen und Raster, aktiv gepflegt, TypeScript-Typen, viele Beispiele.
- **Grenzen:** Stark auf Raster-Assets und Item-Footprints zugeschnitten. Für Vektor- und Tabellendaten (z. B. Inzidenzen pro Landkreis) brauchen wir vermutlich eigene Layer. 
- **Versionen und Links:** ol-stac und OpenLayers müssen zusammenpassen (siehe README). Das Repository ist umgezogen, ältere Links und Tutorials sind daher teils veraltet.

**Alternativen für die Karte:**

| Bibliothek | STAC-Anbindung | Einschätzung |
|---|---|---|
| **OpenLayers + ol-stac** | ol-stac | Beste STAC-Integration, gut für Projektionen und GeoTIFFs |
| **Leaflet + stac-layer** | stac-layer (ähnliche Schnittstelle, ebenfalls auf stac-js) | Einfacher, aber weniger Funktionen für Raster und Projektionen |
| **MapLibre GL JS** | keine offizielle STAC-Schicht (nicht geprüft) | Gut für Vektorkacheln und Optik, STAC-Anbindung müsste selbst gebaut werden |

**Empfehlung:** OpenLayers + ol-stac, weil es das am besten passende Paket für STAC ist. Setzt das Team stark auf Vektorkacheln und Choroplethenkarten, kann MapLibre für Gesundheitsdaten besser sein. Die Entscheidung hängt an den Datenformaten.

## 5. Vergleich und Entscheidungshilfe

| Tool | Aufgabe | Umgebung | Relevanz für AIHealthMap |
|---|---|---|---|
| STAC Index | Entdecken | Web | mittel (Recherche) |
| STAC Browser | Erkunden, Referenz-Oberfläche | Vue/Web | mittel (Vorlage, evtl. Fork) |
| pystac | Erzeugen, Validieren | Python | **hoch** (eigene Metadaten, Extension) |
| pystac-client | Suchen | Python | **hoch** (KI-Tool-Calling, Suche) |
| stac-js | Lesen, Normalisieren | JavaScript | mittel bis hoch (Frontend) |
| OpenLayers | Karte | JavaScript | **hoch** |
| ol-stac | STAC-Layer in OpenLayers | JavaScript | **hoch** (für Raster und Footprints) |

**Entscheidungsleitfaden – „Ich will …“:**

| Ich will … | Nimm … |
|---|---|
| wissen, welche STAC-Kataloge es gibt | STAC Index |
| sehen, wie ein Katalog aufgebaut ist | STAC Browser |
| per Code suchen (Gebiet, Zeit, Collection, Filter) | pystac-client |
| eigene Datensätze als STAC beschreiben | pystac |
| STAC-JSON im Browser sauber auslesen | stac-js |
| ein Item oder eine Collection auf die Karte bringen | OpenLayers + ol-stac |
| einen eigenen STAC-API-Server betreiben | z. B. [stac-fastapi](https://github.com/stac-utils/stac-fastapi), meist mit pgstac als Datenbank (nicht vertieft) |

## 6. Vorschlag: Architektur mit KI-Agent

Dies ist ein Vorschlag, keine Festlegung:

1. Nutzer:innen stellen eine Frage in Alltagssprache.
2. Ein KI-Agent (LLM) übersetzt sie per Tool-Calling in strukturierte Suchparameter (Datensatz, Gebiet, Zeitraum).
3. Das Backend führt mit pystac-client die Suche in den STAC APIs der Anbieter aus.
4. Das Frontend zeigt die Ergebnisse mit stac-js, OpenLayers und ol-stac auf der Karte. Der Agent liefert Erklärungen und Folgevorschläge.

**Praktische Hinweise für den Agenten:**

- **Collection-IDs nicht raten lassen:** Die echte Liste der Collections als Kontext mitgeben, sonst erfindet das Modell IDs.
- **Parameter validieren:** Vor der Suche gegen die Queryables des Servers prüfen.
- **Interpretation sichtbar machen:** Den Nutzenden zeigen, wie ihre Frage verstanden wurde (Gebiet, Zeitraum, Datensatz). Das ist für Nicht-Geo-Expert:innen und für die Nachvollziehbarkeit wichtig.
- **Suche im Frontend:** Für einfache Fälle reicht eine direkte Anfrage an den Such-Endpunkt der API. Eine Entsprechung zu pystac-client in JavaScript wurde hier nicht untersucht.

## 7. Passt STAC zu Gesundheitsdaten?

**Gut zu finden** sind Umweltfaktoren mit Gesundheitsbezug:

- **Luftqualität:** Sentinel-5P (u. a. NO₂, O₃, CO, SO₂, Aerosol), etwa in den AWS Open Data und im Microsoft Planetary Computer.
- **Hitze:** Sentinel-3 Landoberflächentemperatur (Planetary Computer).
- **Vegetation und Grünflächen:** Sentinel-2 (z. B. NDVI selbst berechnet).
- **Bevölkerungsraster und Landbedeckung** gibt es in mehreren Katalogen. Konkrete Collections wurden in dieser Recherche nicht verifiziert und sollten vor Projektstart im STAC Index geprüft werden.

**Meist nicht vorhanden** sind eigentliche Gesundheitsdaten wie Inzidenzen, Krankenhausauslastung oder Surveillance-Statistiken. Sie sind überwiegend tabellarisch oder Vektordaten mit Verwaltungsgrenzen und oft zugriffsbeschränkt. STAC ist historisch auf Raster- und Satellitendaten ausgelegt. Genau deshalb enthält die Projektaufgabe die Erweiterung der Spezifikation. Konkret bedeutet das:

- Eine eigene **STAC-Extension** (Präfix z. B. `health:`) mit Feldern wie Indikator, Einheit, Erhebungsmethode, Bezugseinheit (Landkreis, Gemeinde), Altersgruppe und Datenschutzstufe. Mit pystac lassen sich solche Felder setzen, der STAC Browser kann sie über `stac-fields` darstellen.
- **Vektor- und Tabellendaten** werden als Assets referenziert und im Frontend selbst dargestellt. Hilfreich sind dafür die Best Practices zu Vektor-Layern in STAC, die Table-Extension und STAC-GeoParquet (nicht vertieft).
- **Fachliche Vergleichbarkeit** garantiert STAC nicht. Begriffe, Einheiten und Methoden müssen über die Extension oder zusätzliche Metadaten abgebildet werden.
- **Datenschutz und Zugriffsrechte** sind unabhängig von den Tools zu klären, da Gesundheitsdaten besonders schützenswert sind. Das beeinflusst auch die Architektur, etwa ob nur Metadaten oder auch Daten bereitgestellt werden.


## 8. Typische Stolperfallen

| Problem | Hinweis |
|---|---|
| **CORS** | Ohne aktiviertes CORS funktioniert kein browserbasiertes STAC-Tool. Prüfen oder einen Proxy nutzen. |
| **bbox-Reihenfolge** | West, Süd, Ost, Nord (Länge vor Breite). OpenLayers rechnet standardmäßig in EPSG:3857, STAC nutzt EPSG:4326, daher ggf. transformieren. |
| **Zeitformat** | RFC 3339; Intervalle mit Schrägstrich, offene Intervalle mit `..`. |
| **APIs sind nicht gleich** | `conformsTo` prüfen: Filter, Freitext, Sortierung, Collection-Suche? |
| **Asset-Zugriff** | Metadaten sind öffentlich, Daten teils nicht (z. B. signierte URLs beim Planetary Computer). |
| **Asset-Format** | COG/GeoTIFF ist direkt kartierbar, NetCDF und Parquet nur mit zusätzlicher Verarbeitung (z. B. Tile-Server wie TiTiler, nicht vertieft). |
| **STAC-Versionen** | Kataloge liefern unterschiedliche Versionen. stac-js migriert, andere Tools nicht unbedingt. |
| **Versionskompatibilität** | ol-stac und OpenLayers müssen zusammenpassen. |
| **Veraltete Links** | Viele Tutorials verweisen noch auf das alte Repository von ol-stac. |
| **Paging und Limits** | Immer eine Maximalzahl an Ergebnissen setzen, sonst werden sehr viele Seiten geladen. |

## 9. Fazit und Empfehlung

1. **Erkunden:** STAC Index und STAC Browser (lokal gegen eine bekannte API) nutzen, um ein Gefühl für STAC-Daten zu bekommen.
2. **Backend und Agent:** pystac-client als Suchschnittstelle, pystac zum Erzeugen eigener Metadaten mit Health-Extension.
3. **Frontend:** OpenLayers + ol-stac (mit stac-js im Hintergrund) für Raster und Footprints, eigene Layer für Vektor- und Tabellendaten.
4. **Früh klären:** Welche konkreten Gesundheitsdatensätze in welchem Format eine Rolle spielen. Davon hängt ab, ob ol-stac reicht oder MapLibre bzw. eigene Layer nötig sind.
5. **Nicht überbauen:** Nicht jedes Projekt braucht alle Tools. Der STAC Browser ist eine Referenz, kein Pflichtbaustein.


## Quellen

- STAC: [stacspec.org](https://stacspec.org), [STAC-Spezifikation](https://github.com/radiantearth/stac-spec), [Best Practices](https://github.com/radiantearth/stac-spec/blob/master/best-practices.md)
- STAC Index: [stacindex.org](https://stacindex.org), [giswqs/stac-index-catalogs](https://github.com/giswqs/stac-index-catalogs), [STAC-Atlas](https://github.com/SpatioCore/STAC-Atlas)
- STAC Browser: [radiantearth/stac-browser](https://github.com/radiantearth/stac-browser)
- PySTAC: [stac-utils/pystac](https://github.com/stac-utils/pystac)
- pystac-client: [Repository](https://github.com/stac-utils/pystac-client), [Dokumentation](https://pystac-client.readthedocs.io/)
- stac-js: [moregeo-it/stac-js](https://github.com/moregeo-it/stac-js)
- ol-stac: [moregeo-it/ol-stac](https://github.com/moregeo-it/ol-stac)
- OpenLayers: [openlayers.org](https://openlayers.org/)
- Leaflet-Alternative: [stac-utils/stac-layer](https://github.com/stac-utils/stac-layer)
- Server: [stac-utils/stac-fastapi](https://github.com/stac-utils/stac-fastapi)
- Daten: [Sentinel-5P auf AWS Open Data](https://registry.opendata.aws/sentinel5p/), [Planetary Computer](https://planetarycomputer.microsoft.com/)