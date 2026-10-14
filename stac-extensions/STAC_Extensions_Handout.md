# STAC Extensions

Geosoftware II · WiSe 2026/27 · Timo, Paul

## Worum geht es bei STAC?

STAC steht für *SpatioTemporal Asset Catalog*. Die Spezifikation legt fest, wie raum- und zeitbezogene Daten beschrieben werden. Dadurch lassen sich beispielsweise Satellitenaufnahmen leichter finden und in verschiedenen Anwendungen nutzen.

Die Daten sind über **Catalogs**, **Collections** und **Items** organisiert: Ein Catalog verlinkt andere STAC-Objekte, eine Collection fasst zusammengehörige Daten zusammen und ein Item beschreibt eine einzelne Aufnahme. Die eigentlichen Dateien, etwa die einzelnen Bänder eines Sentinel-2-Bildes, werden als **Assets** mit dem Item verknüpft.

Der STAC Core enthält vor allem grundlegende Angaben: Was ist das für ein Datensatz, wo und wann wurde er aufgenommen und wo liegen die Dateien?

## Warum gibt es Extensions?

Für viele Anwendungen reichen diese Angaben nicht aus. Bei Satellitendaten interessiert zum Beispiel auch, wie stark eine Aufnahme bewölkt ist oder welches Koordinatenreferenzsystem verwendet wurde. Solche zusätzlichen Informationen werden über **STAC Extensions** einheitlich beschrieben. Der Core muss dafür nicht um jedes Spezialfeld erweitert werden.

Ein Beispiel ist das Feld `eo:cloud_cover` aus der Electro-Optical Extension:

```json
"properties": {
  "datetime": "2026-10-01T10:15:00Z",
  "eo:cloud_cover": 12.5
}
```

Die Aufnahme hat hier eine Wolkenbedeckung von **12,5 %**. Weil das Feld standardisiert ist, können andere Programme diese Angabe direkt zuordnen.

## Welche Extensions gibt es?

Einige häufige Beispiele sind:

- **Electro-Optical (`eo:`):** Wolkenbedeckung und Angaben zu optischen Daten.
- **Projection (`proj:`):** Koordinatenreferenzsystem und räumliche Transformation.
- **SAR (`sar:`):** Informationen zu Radaraufnahmen, z. B. Polarisation.
- **Raster (`raster:`):** Datentypen, NoData-Werte und Statistiken der Rasterbänder.
- **Scientific Citation (`sci:`):** Angaben zur wissenschaftlichen Zitierung, etwa eine DOI.

Weitere Extensions gibt es beispielsweise für Satellitenbahnen (`sat:`), Verarbeitungsschritte (`processing:`) und ML-Modelle (`mlm:`). Eine Übersicht steht unter https://stac-extensions.github.io/.

## Wie wird eine Extension eingebunden?

Ein STAC-Objekt gibt im Feld `stac_extensions` an, welche Erweiterung es verwendet. Dort steht die **URL des JSON Schemas**, nicht einfach nur der Name. Danach können die Felder der Extension an der vorgesehenen Stelle genutzt werden.

*Das Beispiel zeigt nur einen Ausschnitt eines Items.* Ein Validator kann mithilfe des Core-Schemas und des angegebenen Extension-Schemas überprüfen, ob die Daten korrekt aufgebaut sind.

```json
"stac_extensions": [
  "https://stac-extensions.github.io/eo/v2.0.0/schema.json"
],
"properties": {
  "datetime": "2026-10-01T10:15:00Z",
  "eo:cloud_cover": 12.5
}
```

## Wie definiert man eine eigene Extension?

Zu einer Extension gehört vor allem ein **JSON Schema**. Darin wird festgelegt, welche zusätzlichen Felder es gibt, welche Datentypen sie haben und an welcher Stelle sie verwendet werden dürfen. Eine README erklärt dieselben Felder für Menschen und zeigt die Verwendung anhand von Beispielen. Außerdem gehören eine eindeutige Schema-URL, ein Feldpräfix (z. B. `health:`), eine Version und Angaben zum Reifegrad dazu.

Vor der Entwicklung sollte man nachsehen, ob eine passende Extension bereits existiert. Eigene Feldnamen für bereits standardisierte Inhalte würden den Datenaustausch eher erschweren. Für neue Extensions gibt es eine Vorlage unter https://github.com/stac-extensions/template.

### Beispiel mit Gesundheitsdaten

Für unser Semesterprojekt könnte man sich eine eigene **Health Extension** vorstellen. Ein Item würde dann zum Beispiel folgende Werte enthalten:

```json
"properties": {
  "datetime": "2026-10-01T00:00:00Z",
  "health:disease": "influenza",
  "health:cases": 1284,
  "health:population": 320000
}
```

Das ist **nur ein Entwurf**, keine bereits bestehende offizielle STAC Extension. Bei einer tatsächlichen Umsetzung müsste unter anderem genau beschrieben werden, auf welche Region und welchen Zeitraum sich die Fallzahlen beziehen. Erst danach ließen sich passende Schema-Regeln und überprüfbare Beispiele erstellen.

## Von der Idee zur stabilen Extension

Nach der Definition der Felder folgen JSON Schema, Dokumentation, Beispiel-Dateien und Tests. Anschließend kann Feedback aus der Community eingeholt und die Extension veröffentlicht werden. Die Reifegrade heißen **Proposal**, **Pilot**, **Candidate** und **Stable**; **Deprecated** kennzeichnet Erweiterungen, die nicht mehr verwendet werden sollen.

Entscheidend ist nicht nur, dass zusätzliche Felder technisch funktionieren. Sie müssen auch so beschrieben sein, dass andere Anwendungen sie ohne eigene Sonderlösung verstehen können.

## Quellen

- STAC-Spezifikation: https://github.com/radiantearth/stac-spec
- Übersicht der Extensions: https://stac-extensions.github.io/
- Template für eigene Extensions: https://github.com/stac-extensions/template
