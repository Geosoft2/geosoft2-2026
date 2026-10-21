# Grundlagen von Datenbanksystemen

**Geosoftware II – WiSe 2026/27**

**Autoren:** [@RiekeKwieni](https://github.com/RiekeKwieni), [@Sugrzabka](https://github.com/Sugrzabka)

Dieses Handout stellt vier Datenbankkategorien vor: relationale Datenbanken, NoSQL-Datenbanken, Vektordatenbanken und räumliche Datenbanken. Dabei werden deren grundlegender Aufbau, Funktionsweise, typische Anwendungsgebiete sowie Vor- und Nachteile erläutert.

## 1. Was ist eine Datenbank?

Eine Datenbank ist eine organisierte Sammlung von Daten, die dauerhaft gespeichert, verwaltet und abgefragt werden können.

Ein **Datenbankmanagementsystem (DBMS)** ist die Software, die den Zugriff auf die Daten sowie deren Verwaltung und Sicherung ermöglicht.

Datenbanken unterscheiden sich unter anderem in ihrem Datenmodell, also der Art, wie Daten organisiert werden, und in den unterstützten Abfragen.

Je nach Anwendungsgebiet eignen sich unterschiedliche Datenbanktypen:

- **Relational:** Strukturierte Daten speichern und miteinander verknüpfen.
- **NoSQL:** Unterschiedlich aufgebaute Daten flexibel speichern.
- **Vektor:** Inhaltlich ähnliche Daten erkennen und finden.
- **Räumlich:** Geografische Daten speichern und räumlich abfragen.

## 2. Relationale Datenbanken

Relationale Datenbanken organisieren Informationen in **Tabellen**, die aus Zeilen (Datensätzen) und Spalten (Attributen) bestehen. Mehrere Tabellen können über eindeutige IDs miteinander verknüpft werden.

### 2.1 Aufbau und Schlüssel

Das Beispiel aus der Präsentation zeigt zwei Tabellen, in denen Datensätze und ihre zugehörigen Quellen getrennt gespeichert werden.

**Tabelle `Datensätze`:**

| id | titel | quellen_id |
|---|---|---|
| 17 | Asthma nach Stadtteil | 3 |
| 18 | NO₂-Messwerte | 4 |

**Tabelle `Quellen`:**

| id | name |
|---|---|
| 3 | Gesundheitsamt |
| 4 | Umweltbundesamt |

Die Spalte `quellen_id` verweist auf die entsprechende `id` in der Tabelle `Quellen`. Dadurch kann jedem Datensatz seine Quelle zugeordnet werden, ohne den Quellennamen mehrfach speichern zu müssen.

Wichtige Begriffe sind:

- **Primärschlüssel (Primary Key):** Identifiziert einen Datensatz eindeutig, beispielsweise `id`.
- **Fremdschlüssel (Foreign Key):** Verweist auf einen Datensatz in einer anderen Tabelle, beispielsweise `quellen_id`.
- **SQL (Structured Query Language):** Sprache zum Abfragen und Verwalten relationaler Daten.


### 2.2 Vor- und Nachteile

**Vorteile:**
- Klare Struktur und zuverlässige Beziehungen zwischen Daten.
- Informationen aus mehreren Tabellen können gemeinsam abgefragt werden.
- Gut geeignet für strukturierte und miteinander verknüpfte Daten.

**Nachteile:**
- Änderungen an der Tabellenstruktur müssen geplant werden.
- Weniger flexibel bei häufig wechselnden Datenstrukturen.

**Typische Anwendungen:** Onlineshops mit Kunden, Bestellungen und Produkten, Banken und Buchungssysteme.

## 3. NoSQL-Datenbanken

NoSQL steht für „Not Only SQL“ und umfasst Datenbanksysteme, die nicht ausschließlich das klassische relationale Tabellenmodell verwenden.

Ein Beispiel ist die dokumentenorientierte Datenbank **MongoDB**. Dabei werden Informationen in Dokumenten gespeichert, häufig in einem JSON-ähnlichen Format.

### 3.1 Flexible Datenstrukturen

Anders als bei relationalen Tabellen müssen nicht alle Dokumente dieselben Felder besitzen.

In einem Onlineshop können beispielsweise ein T-Shirt und ein Laptop unterschiedliche Eigenschaften haben:

**T-Shirt:**

```json
{
  "name": "T-Shirt",
  "groesse": "M",
  "farbe": "Blau"
}
```

**Laptop:**

```json
{
  "name": "Laptop",
  "ram": "16 GB",
  "speicher": "512 GB"
}
```

Beide Produkte gehören zum selben Produktkatalog, benötigen aber unterschiedliche Angaben. NoSQL ermöglicht es, solche Daten flexibel zu speichern.

### 3.2 Vor- und Nachteile

**Vorteile:**
- Flexible Datenstrukturen.
- Unterschiedliche Eigenschaften können einfach gespeichert werden.
- Gut geeignet für Anwendungen mit variierenden Datenfeldern.

**Nachteile:**
- Uneinheitliche Daten können die Verarbeitung erschweren.
- Die Datenstruktur kann zwischen einzelnen Dokumenten stark variieren.

**Typische Anwendungen:** Produktkataloge, Content-Management-Systeme und Webanwendungen.

## 4. Vektordatenbanken

Vektordatenbanken ermöglichen es, Daten anhand ihrer **inhaltlichen Ähnlichkeit** zu durchsuchen.

Dafür werden beispielsweise Texte durch ein Modell in Zahlenfolgen umgewandelt. Diese Zahlenfolgen werden als Vektoren bezeichnet.

So kann die Datenbank ähnliche Inhalte finden, auch wenn unterschiedliche Wörter verwendet werden.

### 4.1 Wie funktioniert eine Ähnlichkeitssuche?

Der grundlegende Ablauf lautet:

**Suchanfrage → Umwandlung in einen Vektor → Vergleich mit gespeicherten Vektoren → ähnliche Ergebnisse**

Ein Modell wandelt sowohl gespeicherte Inhalte als auch die Suchanfrage in Vektoren um. Anschließend wird verglichen, welche Vektoren besonders ähnlich sind.

### 4.2 Beispiel: Semantische Suche

Die Suchanfrage lautet:

**„Unterkunft am Meer“**

Eine Vektordatenbank kann dazu beispielsweise folgende Ergebnisse finden:

- Strandhotel
- Ferienhaus an der Küste

Andere Inhalte wie Rezepte oder Fahrradverleih sind in diesem Beispiel weniger ähnlich.

Entscheidend ist nicht die exakte Übereinstimmung der Wörter, sondern deren inhaltliche Bedeutung.

### 4.3 Vor- und Nachteile

**Vorteile:**
- Findet ähnliche Inhalte auch bei unterschiedlichen Formulierungen.
- Ermöglicht die Suche nach Bedeutung statt ausschließlich nach Schlüsselwörtern.
- Geeignet für die Suche nach Texten und Bildern.

**Nachteile:**
- Suchergebnisse sind nicht immer passend oder sachlich richtig.
- Die Qualität hängt vom verwendeten Modell ab.

**Typische Anwendungen:** Semantische Suchmaschinen, Bildsuche und Empfehlungssysteme.

## 5. Räumliche Datenbanken

Räumliche Datenbanken speichern und verwalten Informationen mit einem geografischen Bezug.

Neben normalen Attributen können sie auch geometrische Objekte speichern und räumliche Beziehungen zwischen diesen untersuchen.

Ein Beispiel ist die relationale Datenbank **PostgreSQL**, die durch die Erweiterung **PostGIS** räumliche Funktionen erhält.

### 5.1 Räumliche Datentypen

Geografische Objekte können durch unterschiedliche Geometrien dargestellt werden:

- **Point (Punkt):** Standort einer Haltestelle oder eines Spielplatzes.
- **LineString (Linie):** Verlauf einer Straße oder eines Flusses.
- **Polygon (Fläche):** Umriss eines Parks oder eines Stadtgebiets.

### 5.2 Beispiel: Räumliche Abfrage

Beispiel: eine Karte mit zwei Parks, einer Haltestelle, einem Spielplatz und einem eingezeichneten Suchgebiet.

Die Fragestellung lautet:

**„Welche Parks liegen innerhalb eines bestimmten Stadtgebiets?“**

Mithilfe räumlicher Funktionen kann überprüft werden, welche Parkflächen das Suchgebiet berühren oder überschneiden.

Dabei werden nicht nur die Namen der Parks betrachtet, sondern die tatsächlichen geometrischen Positionen und Ausdehnungen.

Solche Abfragen sind beispielsweise für Kartenausschnitte, Gebietsfilter und räumliche Verknüpfungen nützlich.

### 5.3 Vor- und Nachteile

**Vorteile:**
- Geografische Daten können direkt gespeichert und analysiert werden.
- Räumliche Beziehungen lassen sich automatisch untersuchen.
- Gut geeignet für Karten und standortbezogene Analysen.

**Nachteile:**
- Koordinatensysteme und Geometrien erfordern zusätzliches Fachwissen.
- Komplexe räumliche Berechnungen können rechenintensiv sein.

**Typische Anwendungen:** Navigation, Stadtplanung, Umweltmonitoring und GIS.

## 6. Fazit

Die Wahl einer Datenbank hängt davon ab, welche Daten gespeichert werden und welche Abfragen benötigt werden.

| Datenbanktyp | Besonders geeignet für |
|---|---|
| **Relational** | Strukturierte Daten und Beziehungen |
| **NoSQL** | Flexible Datenstrukturen |
| **Vektor** | Suche nach inhaltlich ähnlichen Daten |
| **Räumlich** | Geografische Daten und räumliche Abfragen |

Die verschiedenen Ansätze müssen nicht getrennt voneinander eingesetzt werden.

Beispielsweise ist PostgreSQL eine relationale Datenbank, die durch **PostGIS** um räumliche Funktionen und durch **pgvector** um Vektorsuche erweitert werden kann.

Somit können unterschiedliche Datenbankfunktionen innerhalb eines Systems kombiniert werden.
