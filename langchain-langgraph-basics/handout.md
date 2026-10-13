Author: [@lukasholscher-hue]

# LangChain und LangGraph Grundlagen
Dieses Handout gibt einen Überblick über die Grundlagen von LangChain und LangGraph.

## 1. Einführung: LangChain und LangGraph

### 1.1 Was ist LangChain?
LangChain ist ein Open-Source-Framework, das entwickelt wurde, um die Erstellung von Anwendungen auf Basis von Large Language Models (LLMs) zu vereinfachen und zu strukturieren. Es stellt modulare Komponenten und Abstraktionen bereit, die die Anbindung und Verknüpfung von Sprachmodellen mit externen Datenquellen, Schnittstellen und Werkzeugen erleichtern.

### 1.2 Was ist LangGraph?
LangGraph ist ein Open-Source-Framework zur Entwicklung zustandsbehafteter KI-Anwendungen und Agenten. Es ermöglicht, Abläufe als Graphen aus einzelnen Verarbeitungsschritten und Übergängen zu modellieren. Dadurch lassen sich sowohl lineare als auch verzweigte und zyklische Workflows umsetzen, in denen Verarbeitungsschritte bei Bedarf wiederholt werden können.

### 1.3 Kontrollfluss: Chains und Graphen
Der Kontrollfluss beschreibt die Reihenfolge, in der einzelne Verarbeitungsschritte innerhalb einer KI-Anwendung ausgeführt werden.

**Chains (LangChain)**
- Bei klassischen sequenziellen Chains steht die Reihenfolge der Verarbeitungsschritte im Voraus fest.
- Informationen werden von einem Baustein zum nächsten weitergegeben.
- Klassische sequenzielle Chains lassen sich als gerichtete azyklische Graphen darstellen und enthalten keine Schleifen oder Rücksprünge

**Graphen (LangGraph)**
- Der nächste Verarbeitungsschritt wird abhängig vom aktuellen Zustand und anhand definierter Bedingungen ausgewählt.
- Ein gemeinsamer State speichert die für den Workflow relevanten Zustandsdaten und ermöglicht deren Austausch zwischen den Verarbeitungsschritten.
- Nodes führen Verarbeitungsschritte aus, während Edges die Übergänge zwischen ihnen festlegen.
- Conditional Edges ermöglichen unterschiedliche Ausführungspfade und iterative Abläufe, bei denen Verarbeitungsschritte wiederholt werden können.

### 1.4 Gemeinsamkeiten und Unterschiede
**Gemeinsamkeiten**
- Beide dienen dem Aufbau von KI-Anwendungen und Agenten auf Basis von Large Language Models (LLMs)
- Beide unterstützen die Integration standardisierter Modell-Schnittstellen (OpenAI oder Anthropic) sowie externe Werkzeuge

**Unterschiede**
|Kriterium|LangChain|LangGraph|
|---|---|---|
|**Hauptzweck**|Vereinfacht die Entwicklung von LLM-Anwendungen und Agenten durch vorgefertigte Abstraktionen.|Ermöglicht die flexible Modellierung und Steuerung zustandsbehafteter Workflows und Agenten.|
|**Kontrollfluss**|Sequenzieller und linearer Ablauf ohne Schleifen|Zyklischer Graph mit Schleifen und Verzweigungen|
|**Zustandsverwaltung**|Konzentriert sich auf Modell-Aufrufe, Prompts und einfache Ausführungen|Bietet einen zentralen State, dauerhafte Ausführung und Zustandsspeicherung|


## 2. State

### 2.1 Das Konzept des State
Der State ist die zentrale Datenstruktur, die den aktuellen Zustand der Anwendung speichert und den einzelnen Komponenten als gemeinsame Grundlage für den Datenaustausch dient.

### 2.2 Definition des State-Schemas
Das State-Schema legt die Struktur und die Datentypen des zentralen Zustands eines LangGraph-Graphen fest. Es definiert, welche Felder der State enthält und welche Werte diese aufnehmen können. Die Nodes nutzen diese Felder, um auf Zustandsdaten zuzugreifen und Aktualisierungen zurückzugeben.

### 2.3 Lesen und Aktualisieren des State
Das Lesen und Aktualisieren des State regelt, wie Zustandsdaten zwischen den einzelnen Verarbeitungsschritten eines LangGraph-Graphen verwendet und verändert werden.

**1. Lesen des State**
- Eine Node-Funktion erhält den für sie vorgesehenen State als Eingabe.
- Die Node liest die benötigten Informationen aus den Feldern des definierten State-Schemas (z.B. TypedDict).
- Die Nodes arbeiten mit dem gemeinsamen Zustand des Graphen. Welche Daten sie benötigen, hängt von ihrer jeweiligen Aufgabe ab.

**2. Aktualisieren des State**
- Eine Node muss nicht den gesamten State zurückgeben. Sie kann ein Dictionary mit den Feldern zurückgeben, die aktualisiert werden sollen.
- LangGraph verarbeitet die Rückgabewerte und übernimmt die Änderung in den State. Ohne einen expliziten Reducer wird ein vorhandener Feldwert standardmäßig durch den neuen Wert ersetzt.
- Für bestimmte State-Felder können Reducer festlegen, wie der bisherige Wert und das Update miteinander kombiniert werden.
- Bei einem Reducer bezeichnet *left* den bisherigen Wert und *right* das neue Update. Die Funktion bestimmt, wie daraus der aktualisierte Wert entsteht.

## 3. Nodes und Edges

### 3.1 Nodes
Nodes sind die Verarbeitungseinheiten eines LangGraph-Graphen. Sie führen die Logik des Workflows aus wie z.B. Berechnungen, Datenverarbeitung oder Aufrufe eines Sprachmodells.

### 3.2 Edges
Edges steuern den Kontrollfluss innerhalb eines Graphen in LangGraph. Sie bestimmen auf Basis des aktuellen State, welcher Node als Nächstes aufgerufen wird.
Während Nodes die eigentlichen Verarbeitungsschritte ausführen, legen Edges fest, wie diese miteinander verbunden sind und in welcher Reihenfolge sie durchlaufen werden.

### 3.3 Start- und Endpunkte: START und END
START und END sind spezielle Nodes, die den Einstieg und den Abschluss eines Graphen definieren.
- **START**: Markiert den Einstiegspunkt des Graphen und legt fest, welche Nodes zu Beginn der Ausführung aufgerufen werden.
- **END**: Markiert den regulären Abschluss der Graphausführung 

## 4. Conditional Edges

### 4.1 Bedingte Übergänge und Routing
Conditional Edges ermöglichen eine dynamische Steuerung des Kontrollflusses in LangGraph.
Sie bestimmen anhand des aktuellen State, welcher Node als Nächstes ausgeführt wird oder ob der Workflow beendet werden soll.

Im Gegensatz zu normalen Edges, die einen festen Übergang definieren, ermöglichen Conditional Edges eine Auswahl des Ausführungspfads zur Laufzeit.

Dabei erhält eine Routing-Funktion den aktuellen State als Eingabe und wertet die relevanten Informationen aus.
Daraufhin gibt sie einen Routing-Wert zurück, der direkt oder über ein optionales Mapping-Dictionary einem Zielknoten zugeordnet wird.

Typische Anwendungsfälle wären dynamische Verzweigungen oder iterative Schleifen

### 4.2 Routing-Funktionen und Anwendungsbeispiel
Conditional Edges werden mit der Methode `add_conditional_edges()` in den Graphen eingebunden.
Sie erhält den Ausgangsknoten, die Routing-Funktion und optional ein Mapping-Dictionary, das Rückgabewerte den jeweiligen Zielknoten zuordnet.

**Beispiel: Weiterleitung einer Supportanfrage**
```python
from langgraph.graph import StateGraph, START, END

def route_request(state):
    if state["category"] == "billing":
        return "billing"
    return "general"

builder.add_conditional_edges(
    "classify",
    route_request,
    {
        "billing": "billing_node",
        "general": "general_node"
    }
)
```
*Hier wertet die Routing-Funktion die Kategorie einer Supportanfrage aus. Bei einer Rechnungsanfrage wird der Graph an `billing_node` weitergeleitet, bei allen anderen Anfragen an `general_node`. Das Mapping-Dictionary ordnet die Rückgabewerte den entsprechenden Zielknoten zu.*

## 5. Annotated Fields und (Custom) Reducers

### 5.1 Annotated Fields
Annotated Fields verwenden das Python-Konstrukt `typing.Annotated`, um einem Feld im State-Schema zusätzliche Metadaten zuzuordnen. Damit kann insbesondere festgelegt werden, wie neue Updates mithilfe einer Reducer-Funktion mit dem bestehenden Feldwert zusammengeführt werden.

**Funktionsweise**
- `Annotated[Typ, Reducer_Funktion]` koppelt den Datentyp eines Feldes direkt an eine Update-Logik
- Ohne `Annotated` überschreibt jedes neue Update den bisherigen Wert des Feldes vollständig (Default Reducer).
- Ein Reducer erhält den bisherigen Wert (*left*) und das neue Update (*right*) als Argumente und bestimmt, wie daraus der aktualisierte Wert entsteht

### 5.2 Standardverhalten der State-Aktualisierung
In LangGraph bestimmen Reducer-Funktionen, wie Updates von Nodes auf den State angewendet werden.
Ohne expliziten Reducer wird der bisherige Wert eines Feldes standardmäßig durch den neuen Wert ersetzt.
- **Partielle Updates**: Eine Node muss nur die Felder zurückgeben, die aktualisiert werden sollen.
- **Unveränderte Felder**: Alle nicht aktualisierten Felder behalten ihren bisherigen Wert.

### 5.3 Standard-Reducers und Reducer-Beispiele
Reducer sind Funktionen, die den bisherigen Feldwert (*left*) und ein neues Update (*right*) miteinander kombinieren.
- `operator.add`: Verknüpft bspw. zwei Listen, indem die Elemente der neuen Liste angehängt werden.
- `add_messages`: Verwaltet Chat-Nachrichten, indem neue Nachrichten hinzugefügt und bestehende Nachrichten anhand ihrer ID aktualisiert werden.

### 5.4 Custom Reducers
Custom Reducers sind selbst definierte Funktionen, die festlegen, wie ein bisheriger Wert und ein neues Update kombiniert werden.
```python
def merge_lists(left, right):
    return left + right
```
*Hier werden die bisherigen Listenelemente mit den neuen Elementen zusammengeführt. Die Funktion kann über Annotated einem State-Feld zugeordnet werden*    

```python
from typing import Annotated
from typing_extensions import TypedDict

class State(TypedDict):
    foo: int
    bar: Annotated[list[str], merge_lists]
```
- *foo: Ohne Reducer wird der bisherige Wert durch das neue Update ersetzt.*
- *bar: Der selbst definierte Reducer `merge_lists` hängt die neuen Listenelemente an die bisherigen an.*

## 6. Zusammenfassung
LangChain und LangGraph sind Frameworks zur Entwicklung LLM-basierter Anwendungen, die unterschiedliche Architekturansätze verfolgen. Während LangChain modulare Komponenten für den Aufbau von Sprachmodell-Anwendungen bereitstellt, stellt LangGraph die Infrastruktur zur Steuerung zyklischer und zustandsbehafteter Workflows bereit.

Die wichtigsten Konzepte von LangGraph sind: 
- **State**: Speichert den aktuellen Anwendungszustand über den gesamten Prozessverlauf
- **Nodes und Edges**: Nodes führen Verarbeitungsschritte aus, während Edges den Kontrollfluss zwischen ihnen festlegen.
- **Conditional Edges**: Ermöglicht bedingte Verzweigungen und iterative Abläufe.
- **Annotated Fields und Reducers**: Legen fest, wie neue Updates mit bestehenden State-Werten zusammengeführt werden.

## 7. Quellen
- https://docs.langchain.com/oss/python/langgraph/overview
- https://docs.langchain.com/oss/python/langchain/overview
- https://docs.langchain.com/oss/python/langgraph/graph-api
- https://www.ibm.com/de-de/think/topics/langchain
- https://www.ibm.com/de-de/think/topics/langgraph
- https://docs.python.org/3/library/typing.html#typing.Annotated
- Taulli, Tom, und Gaurav Deshmukh. *Building Generative AI Agents Using LangGraph, AutoGen, and CrewAI.*