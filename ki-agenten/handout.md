@pjb161

# KI-Agenten für Health Data Discovery: Architektur & Konzepte

Dieses Handout dient als technische Grundlage und Entscheidungshilfe für das Geosoftware II Projekt (Entwicklung einer KI-gestützten Web-Mapping-Anwendung für Gesundheitsdaten). Es behandelt die essenziellen Konzepte von KI-Agenten, deren Planungsprozesse, den Einsatz von Tools sowie die Integration in moderne Benutzeroberflächen.

---

## 1. KI-Agent vs. herkömmlicher LLM-Chatbot

Der Kernunterschied zwischen einem Standard-Chatbot (wie der Weboberfläche von ChatGPT) und einem KI-Agenten liegt in der **Handlungsfähigkeit (Agency)** und der **Autonomie**.

| Eigenschaft | Herkömmlicher LLM-Chatbot | KI-Agent |
| :--- | :--- | :--- |
| **Primäre Funktion** | Textgenerierung, Beantwortung von Fragen aus dem internen (teils veralteten) Wissen. | Zielgerichtetes Lösen von Problemen durch Interaktion mit der Außenwelt. |
| **Ablauf** | Single-Turn (Frage → Antwort). | Multi-Turn iterativ (Planung → Aktion → Beobachtung → Anpassung). |
| **Systemzugriff** | Isoliert (meist keine externen APIs). | Hat Zugriff auf **Tools** (APIs, Datenbanken, Web-Suche). |
| **Beispiel im Projekt** | "Erkläre mir, was Asthma ist." | "Finde Gesundheitsdaten zu Asthma in Münster, berechne die räumliche Verteilung, lade sie auf die Karte und markiere Hotspots." |

> **Fazit für unser Projekt:** Wir bauen keinen simplen Chatbot, der nur medizinisches Wissen wiedergibt, sondern ein handelndes System, das Geodaten-APIs (wie WFS/WMS) oder Datenbanken (z. B. PostgreSQL/PostGIS) aktiv abfragen und die Web-Karte steuern kann.

---

## 2. Planung und Aufgabenzerlegung (Task Decomposition)

Komplexe Anfragen (z.B. *"Zeige mir den Zusammenhang zwischen Luftqualität und Atemwegserkrankungen in NRW"*) überfordern ein LLM, wenn es sie in einem einzigen Schritt lösen soll. Der Agent muss die Aufgabe **zerlegen**:

1. **Sub-Goal Decomposition:** Das LLM bricht das Hauptziel in kleine Teilaufgaben herunter.
   * *Schritt 1:* API für Luftqualitätsdaten (z.B. Umweltbundesamt) abfragen.
   * *Schritt 2:* API für Gesundheitsdaten abfragen.
   * *Schritt 3:* Daten räumlich verschneiden (Spatial Join).
   * *Schritt 4:* GeoJSON an das Web-Frontend senden.
2. **Methoden:**
   * **Chain of Thought (CoT):** Das Modell wird angewiesen, "laut zu denken" (z.B. durch den Prompt-Zusatz: *"Let's think step by step"*). Dies verbessert die Planungsqualität enorm.
   * **Tree of Thoughts (ToT):** Der Agent generiert mehrere Lösungswege und bewertet diese, bevor er sich für den vielversprechendsten Pfad entscheidet.

---

## 3. AI Agent Tools: Was ist das und wofür benötigt man sie?

Ein LLM ist wie ein Gehirn ohne Hände. **Tools** sind die Hände. 
Technisch gesehen ist ein Tool eine Funktion, deren Parameter dem LLM als JSON-Schema übergeben werden (*Function Calling*). Das LLM generiert dann ein JSON, um die Funktion mit spezifischen Argumenten aufzurufen.

**Beispiel-Schema für ein Tool, das wir nutzen könnten:**
```json
{
  "name": "fetch_health_data",
  "description": "Holt Krankheitsfälle für eine bestimmte Bounding Box.",
  "parameters": {
    "disease": {"type": "string", "description": "z.B. 'Asthma'"},
    "bbox": {"type": "array", "description": "[minLon, minLat, maxLon, maxLat]"}
  }
}
```

**Wofür brauchen wir sie im Geosoftware-Projekt?**
* **Datenbeschaffung:** Abfragen von OGC-APIs, Overpass API (OSM) oder FHIR-APIs für Gesundheitsdaten.
* **Analytik:** Ausführen von Python-Skripten (z.B. mit `geopandas`) zur räumlichen Analyse.
* **UI-Interaktion:** Ein Tool wie `add_layer_to_map(geojson)`, welches dem Frontend signalisiert, dass neue Geodaten auf der Karte gerendert werden sollen.

---

## 4. Der ReAct Loop (Reasoning + Acting)

Der ReAct-Ansatz kombiniert schlussfolgerndes Denken (*Reasoning*) mit Handlungen (*Acting*). Es ist die Standard-Architektur für Agenten:

1. **Thought (Gedanke):** Der Agent überlegt, was als Nächstes zu tun ist.
2. **Action (Handlung):** Der Agent ruft ein Tool mit bestimmten Parametern auf.
3. **Observation (Beobachtung):** Das System führt das Tool aus (z.B. API Request) und gibt das Ergebnis an das LLM zurück.
4. **Wiederholung:** Der Loop läuft so lange weiter, bis der Agent feststellt, dass er das Ziel erreicht hat und die **Final Answer** generieren kann.

---

## 5. Einzelagenten- vs. Multiagenten-Architekturen

| Architektur | Beschreibung | Bewertung für unser Projekt |
| :--- | :--- | :--- |
| **Einzelagent** | Ein einzelnes, mächtiges LLM, das Zugriff auf alle Tools hat und als zentraler Router agiert. | **+** Einfach zu implementieren (gut für den Start).<br>**-** Das LLM wird bei zu vielen Tools (>7-10) schnell verwirrt und vergisst Instruktionen. |
| **Multiagent** | Ein Team von Agenten mit spezifischen Rollen (z. B. `DataFetcher`, `SpatialAnalyst`, `MapUICoordinator`). Frameworks: *CrewAI, LangGraph* | **+** Hoch skalierbar, saubere Trennung von Aufgaben ("Separation of Concerns").<br>**-** Hoher API-Kostenaufwand (Tokens), sehr komplexes Debugging. |

---

## 6. Human-in-the-Loop (HITL) - Konzept und Zweck

Beim **Human-in-the-Loop**-Konzept handelt der Agent nicht komplett autonom, sondern pausiert an kritischen Stellen und fordert menschliches Feedback oder eine Freigabe an.

* **Zweck im Health-Data Kontext:** Gesundheitsdaten sind extrem sensibel (Datenschutz/Ethik). Zudem können fehlerhafte Geodaten-Berechnungen Server überlasten.
* **Anwendungsfall:** Bevor der Agent eine riesige Datenbankabfrage auslöst oder sensible demografische Gesundheitsdaten auf eine öffentliche Web-Karte lädt, unterbricht das Framework den ReAct-Loop. Der Nutzer muss in der UI bestätigen: *"Die Abfrage erfordert 500 MB an Daten. Möchtest du diese räumliche Abfrage wirklich ausführen? (Ja/Nein)"*.

---

## 7. Erstellung von Benutzeroberflächen (UIs) für KI-Agenten

Um einen Agenten für "Health Data Discovery" nutzbar zu machen, benötigen wir ein Frontend, das Chat und Web-Mapping vereint.

1. **Streamlit / Gradio**
   * *Vorteile:* Reines Python. Extrem schnelles Prototyping. Geodaten können via `streamlit-folium` eingebunden werden.
   * *Nachteile:* Sehr starr. Bei komplexen Map-Interaktionen (z. B. Nutzer klickt auf ein Polygon in der Karte und der Chatbot reagiert darauf) stoßen diese Frameworks sofort an ihre Grenzen.
2. **Benutzerdefinierte UI (HTML, CSS, React/Vue + Leaflet/OpenLayers)**
   * *Vorteile:* Volle Kontrolle. Nahtlose, bidirektionale Kommunikation (Agent malt auf Karte ↔ Nutzer interagiert mit Karte, Agent analysiert).
   * *Nachteile:* Höherer Entwicklungsaufwand (Trennung von Backend via FastAPI/Node.js und Frontend via WebSockets/REST).

---

## 8. Guidelines & Empfehlungen für unser Projekt

Basierend auf den obigen Konzepten, hier meine persönlichen Empfehlungen für die Umsetzung in unseren Projektgruppen:

1. **Architektur klein halten:** Start mit einer **Einzelagenten-Architektur** (z.B. mit *LangChain*). Erst dann auf Multiagenten-Systeme umbauen, wenn der Agent aufgrund zu vieler Tools an Genauigkeit verliert.
2. **UI-Entscheidung treffen:** 
   * *Streamlit* nur nutzen, wenn man in Woche 1 einen schnellen Proof-of-Concept des Agenten testen will.
   * **Meine Meinung:** Für die finale Version der *AIHealthMap* sollten wir zwingend eine **benutzerdefinierte UI (React/Vanilla JS + Leaflet)** bauen. Nur so lässt sich eine professionelle, nahtlose User Experience zwischen Karte und Chat realisieren.
3. **Datenschutz durch HITL:** Ein **Human-in-the-Loop**-Breakpoint sollte zwingen implementiert werden. Wenn der Agent externe, unbekannte Gesundheitsdaten-APIs anbindet, sollte der Nutzer immer die Kontrolle darüber haben, was letztendlich auf der Karte gerendert wird.
4. **Tool-Design:** Die Tools sollten "atomar" (eine Funktion = ein klarer Zweck) gehalten werden. Präzise Docstrings für Python-Funktionen schreiben, da das LLM diese Beschreibungen liest, um zu verstehen, wie es das Tool nutzen muss.