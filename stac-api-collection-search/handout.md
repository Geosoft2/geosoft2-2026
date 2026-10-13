@hxnnx22
STAC API – Collection Search / Filter / Sort / Free Text Extensions (ohne CQL2)

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

# Beispiel: Räumliche Suche
GET /collections?bbox=7.5,51.8,7.7,52.0
Die Bounding Box wird durch 4 Werte beschrieben:
min. Längengrad, min. Breitengrad, max. Längengrad, max. Breitengrad
Damit wird die Suche auf einen geographischen Bereich begrenzt, sofern der verwendete Endpunkt
diesen Parameter unterstützt. 
 	
# 2. Filtermöglichkeiten ohne CQL2
Filter dienen dazu, Suchergebnisse anhand bestimmter Eigenschaften einzugrenzen. Neben komplexen 
Filtersprachen können STAC APIs auch einfachere Suchparameter anbieten. 

Die verfügbaren Möglichkeiten hängen davon ab, welcher Endpunkt und welche Erweiterung von den
jeweiligen API unterstützt werden. 

# Räumliche Filterung (bbox)
Mit bbox wird ein geografischer Bereich festgelegt. Dadurch können Suchanfragen räumlich
eingeschränkt werden. 

# Freitextsuche (q):
Der Parameter q kann bei entsprechender Unterstützung eine Suche nach Begriffen ermöglichen, 
bspw. nach Namen, Beschreibung oder Schlagwörtern von Collections.
#Beispiel: 
GET/collections?q=sentinel
Die Anfrage veranschaulicht  eine Suche nach dem Begriff "sentinel". Welche Metadaten durchsucht
werden, hängt von der jeweiligen Implementierung ab.

# 3.Sortierung (sort):
Die Sortierung ordnet Suchergebnisse nach bestimmten Eigenschaften. So können Ergebnisse übersichtlicher 
dargestellt und leichter verglichen werden. 
Mögliche Suchkriterien sind bspw.:
      - ID
      - Titel
      - zeitliche Ausdehnung
      - Erstellungszeitpunkt
# Beispiel: 
GET/collections?sortby=id
Die Anfrage veranschaulicht eine Sortierung nach der Collection-ID. Die konkrete Syntax und Verfügbarkeit
richten sich nach der unterstützten Sortierfunktion der API. 

Eine Sortierung ist besonders hilfreich, wenn viele Collections gefunden werden und die Ergebnisse
in einer nachvollziehbaren Reihenfolge angezeigt werden sollen.

# 4.Feldauswahl(fields):
Mit der 
