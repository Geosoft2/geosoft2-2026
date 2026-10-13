@hxnnx22
# STAC API – Collection Search / Filter / Sort / Free Text Extensions (ohne CQL2)

# 1. STAC Collection Search: 
Collection Search dient dazu, passende STAC Collections anhand bestimmter Kriterien zu finden.
Collections enthalten Metadaten zu zusammengehörigen Geodaten, bspw. Satellitenaufnahmen
eines bestimmten Gebiets oder Datensatzes.

Ziel ist es, die Suche nach relevanten Datensammlungen einzugrenzen und geeignete Daten
schneller zu finden.

# 2. Grundlegende Suchparameter: 
bbox: -> räumliche Suche 
-> Suche nach Collections innerhalb eines bestimmten Gebiets
datetime: -> zeitliche Suche 
-> Suche nach Collections innerhalb eines bestimmten Zeitraums
limit: -> Begrenzung der Ergebnisse
-> Legt fest, wie viele Ergebnisse pro Seite zurückgegeben werden

### Beispiel: Räumliche Suche (bbox)
```http
GET /collections?bbox=7.5,51.8,7.7,52.0
```
Die Bounding Box wird durch 4 Werte beschrieben:
min. Längengrad, min. Breitengrad, max. Längengrad, max. Breitengrad
Damit wird die Suche auf einen geographischen Bereich begrenzt, sofern der verwendete Endpunkt
diesen Parameter unterstützt. 

### Beispiel: Zeitliche Suche (datetime)
```http
GET /collections?datetime=2021-01-01T00:00:00Z/2021-12-31T23:59:59Z
```
Die Anfrage sucht nach Collections, deren zeitliche Ausdehnung das angegebene Intervall schneidet. Die zeitliche Filterung bezieht sich dabei auf die Ausdehnung der Collections
und nicht automatisch auf einzelne Items. 

### Beispiel: Begrenzung der Ergebnisse (limit)
```http
GET /collections?limit=10
```
Mit limit=10 werden höchstens zehn Ergebnisse pro Seite angefordert. Bei weiteren Ergebnissen kann die API zusätzliche Seiten bereitstellen.

# 3. Filtermöglichkeiten ohne CQL2
Filter dienen dazu, Suchergebnisse anhand bestimmter Eigenschaften einzugrenzen. Neben komplexen 
Filtersprachen können STAC APIs auch einfachere Suchparameter anbieten. 

Die verfügbaren Möglichkeiten hängen davon ab, welcher Endpunkt und welche Erweiterung von den
jeweiligen API unterstützt werden. 

# 3.1 Freitextsuche (q):
Der Parameter q kann bei entsprechender Unterstützung eine opzionale Suche nach Begriffen ermöglichen, 
bspw. nach Namen, Beschreibung oder Schlagwörtern von Collections.
### Beispiel: 
```http
GET /collections?q=sentinel
```
Die Anfrage veranschaulicht  eine Suche nach dem Begriff "sentinel". Welche Metadaten durchsucht
werden, hängt von der jeweiligen Implementierung ab.

# 4. Sortierung (sortby):
Die optionale Sortierung ordnet Suchergebnisse nach bestimmten Eigenschaften. So können Ergebnisse übersichtlicher 
dargestellt und leichter verglichen werden. 
Mögliche Suchkriterien sind bspw.:
      - ID
      - Titel
      - zeitliche Ausdehnung
      - Erstellungszeitpunkt
### Beispiel: 
```http
GET /collections?sortby=id
```
Die Anfrage veranschaulicht eine Sortierung nach der Collection-ID. Die konkrete Syntax und Verfügbarkeit
richten sich nach der unterstützten Sortierfunktion der API. 

Eine Sortierung ist besonders hilfreich, wenn viele Collections gefunden werden und die Ergebnisse
in einer nachvollziehbaren Reihenfolge angezeigt werden sollen.

# 5. Feldauswahl(fields):
Die optionale Fields-Erweiterung ermöglicht es, bestimmte Felder in der Antwort einzuschließen oder auszuschließen. Dadurch können unnötige Metadaten reduziert und Antworten kompakter gestaltet werden.

### Beispiel:
```http
GET /collections?fields=id,title
```
Die konkrete Syntax und die verfügbaren Felder richten sich nach der implementierten Erweiterung. Die Fields-Erweiterung muss von der API für den Collection-Search-Endpunkt unterstützt werden. 

# 6. Zusammenfassung
Collection Search ermöglicht es, relevante Datensammlungen gezielt zu finden und die Suche anhand räumlicher und zeitlicher Kriterien einzugrenzen. 
Zusätzliche Erweiterungen können weitere Funktionen bereitstellen, bspw. Freitextsuche, Sortierung und Feldauswahl.
Die verfügbaren Funktionen hängen von der jeweiligen STAC-API-Implementierung ab. Vor der Verwendung eines Parameters sollte deshalb geprüft werden, ob der entsprechende Endpunkt die benötigte Funktion unterstützt. 

# 7. Quellen
https://stacspec.org/en
https://github.com/stac-api-extensions/collection-search
https://stac-api-extensions.github.io/
