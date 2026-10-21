Author: [@ThConze]

# Grundlagen von Large Language Models (LLMs)
Dieses Handout gibt einen Überblick über die Grundlagen von LLMs. Es erläutert, wie LLMs Texte erzeugen und wie sich Antworten durch geeignetes Prompting verbessern lassen. Außerdem werden Begriffe wie Tokens oder Fine-Tuning erklärt und die wichtigsten Bibliotheken und Tools vorgestellt. 

## 1. Was sind LLMs?
Large Language Models (große Sprachmodelle) sind generative KI-Sprachmodelle die mit sehr großen Textmengen trainiert werden. Sie sind in der Lage komplexe sprachliche Aufgaben wie z.B. Zusammenfassen, Übersetzen, Beantwortung von Fragen oder Textgenerierung zu bearbeiten.

## 2. Wie funktionieren LLMs? 
Das LLM funktioniert ähnlich wie ein neuronales Netzwerk. Es analysiert den Eingabetext und berechnet die Wahrscheinlichkeit für das Wort, welches als nächstes drankommt. Dafür wird der Eingabetext in kleine Bausteine (Tokens) zerlegt und ermittelt dann die Wahrscheinlichkeit für das nächste Wort. 

Ein Token kann ein Wort, ein Wortteil, ein Satzzeichen oder ein anderes Textfragment sein. 

### 2.1 Ablauf der Textgenerierng:

**Text → Tokens → Embedding-Vektoren → Transformer → Wahrscheinlichkeiten → Nächstes Token**

Hinweise:
- Die berechneten Wahschreinlichkeiten beruhen auf Mustern, die das Modell während des Trainings gelernt hat und auf dem aktuellen Kontext. 
- Je nach Modell werden unterschiedliche Tokens ausgewählt.
- Ein LLM ist keine Datenbank und kann deshalb auch falsche oder erfundene Aussagen erzeugen. 

### 2.2 Beispiel Anfrage an ein LLM
Die Eingabe lautet: "Die Hauptstadt von Frankreich ist "
1. Text: Die Hauptstadt von Frankreich ist
2. Tokens: [„Die“, „Haupt“, „stadt“, „ von“, „ Frankreich“, „ ist“]
3. Embedding-Vektoren: Jeder Token wird zu Zahlen umgewandelt.
4. Transformer: Die Tokens verrechnen sich gegenseitig → "ist" wird in Zusammenhang mit den vorherigen Tokens gesetzt → es wird klar, dass eine Haupstadt gesucht wird
5. Wahrscheinlichkeiten: Berrechnung der Wahrscheinlichkeit der nächsten Tokens. z.B. Paris 0.92, London 0.03, Berlin 0.02, ...
6. Nächstes Token: Paris wird ausgewählt
7. Schleife beginnt von vorne

### 2.3 Kontextfenster

Ein Kontextfenster/Kontextlänge ist die maximale Anzahl der Tokens die ein Modell gleichzeitig verarbeiten kann. Dazu können sowohl die Eingabe als auch bereits erzeugte Ausgaben gehören. Je nach Anbierter und Modellversion unterscheiden sich die Kontextlängen. 
Wird die maximale Anzahl der Tokens überschritten, können die Eingabe in kleinere Abschnitte unterteilt werden oder durch z.B. Retrieval-Augmented Generation (RAG) eingebunden werden.


## 3. Promting Techniken
Es gibt verschiedene Promting Techniken, also die Art wie man einen Promt (Eingabetext) schreibt, die zu unterschiedlich genauen Ergebnissen führen. Die wichtigsten sind: 

### 3.1 Zero-Shot Promting
- Aufgabe ohne zusätzlichen Kontext oder Beispiele
- Bsp. : *Was ist Vektordaten?*
- Vorteil: schnell und einfach
- Nachteil: oft unpräzise und allgemeine Antworten

### 3.2 One / Few-Shot Prompting
- Enthält ein oder wenige Beispiele um dem Modell die Aufgabe zu veranschaulichen
- Bsp. : *Rasterdaten sind flächenhafte Daten die aus Pixeln bestehen wie z.B. Luftbilder. Was sind Vektordaten?*
- Vorteil: gezieltere Antworten
- Nachteil: mehr Vorarbeit im Promting

### 3.3 Chain-of-Thought (CoT) Prompting
- fordert Modell auf Lösung in kleinen Schritten zu erklären
- Bsp.: *Erkläre Schritt für Schritt, was Vektordaten sind und wie sie entstehen?*
- Vorteil: genauer Ablauf für komplexe Aufgaben 
- Nachteil: kann zu sehr langen Antworten fürhen

### 3.4 Step-Back Prompting
- Zunächst wird eine allgemeine Frage zum Thema erfragt, danach die konkrete Aufgabe
- Bsp. : *Welche grundlegenden Eigenschaften unterscheiden Vektordaten von Rasterdaten? Erkläre anschließend, warum Vektordaten besonders geeignet sind, um Verwaltungsgrenzen darzustellen.*
- Vorteil: tieferes Wissen, genauere Antworten
- Nachteil: benötigt mehr Schrtitte, kann bei einfachen Aufgaben unnötig komplex sein

### 3.5 Role-Based Prompting
- KI soll sich in bestimmte Rolle hereinversetzen, umd die Antwort kontextbezogen zu beantworten
- Bsp. : *Du bist GIS-Spezialist auf einer Messe. Erkläre was Vektordaten sind.*
- Vorteil: kontextbezogene Antworten
- Nachteil: kann zu subjektiven Antworten führen

### 3.6 **Retrieval-Augmented Generation (RAG) Prompting**
- eine der wichtigsten Techniken
- LLM kombiniert interenes Wissen mit externen Datenquellen, um genauere Antworten zu liefern
Bsp: *Nutze die folgenden GeoJSON‑Dateien mit Punktdaten zu Krankheitsfällen und die Polygon‑Grenzen der Stadtteile. Fasse die räumlichen Muster zusammen und identifiziere mögliche Hotspots.*
Vorteil: Geeignet für dynamische und datenintensive Aufgaben mit exteren Datenquellen
Nachteil: Qualität hängt von den verwendeten Quellen ab

### 3.7 Weitere Promting-Techniken:
- Meta Promting
- Self-Consistency Promting
- Tree of Thoughts (ToT) Promting
- ReAct Promting
- Automatic Promt Engineering (APE)


## 4. Wissensgrenzen von LLMs und dessen Überwindung
LLMs können nur das wissen, was in den Trainingsdaten enthalten war. Der sogenannte **Knowledge Cutoff** bezeichnet den Zeitpunkt, wenn die Grenze des Wissens erreicht ist. Von dort an treten bei Sprachmodellen häufig diese Probleme auf:
- **halluzinieren** (neue, falsche Inhalte erfinden)
- **veraltete Infomationen liefern**
- **Fakten falsch rekonstruieren**

### 4.1 Überwindung von Wissensgrenzen
- Retrival Aufgemnted Generation (externe Daten nutzen)
- Few Shot (Beispiele)
- Chain of Thought (Schrittweises Denken reduziert logische Fehler)

## 5. Fine-Tuning
Fine-Tuning ist das Weitertrainieren eines bereits vortrainierten Modells. Dadurch lassen sich die Verhalten und Vorhersagen an einen bestimmten Anwendungsfall anpassen. Wichtige Ansätze sind u.a.:

- Supervised Fine-Tuning (SFT): Modell wird mit ausgewählten Beispielen weitertrainiert
- Full Fine Tuning: Alle Modellparameter werden weitertrainiert
- Parameter-Efficient Fine-Tuning (PEFT): Es werden nur wenige Parameter angepasst, anstatt das gesamte Modell zu trainieren


## 6. Lokale Ausführung von LLMs vs. API-basierter Zugriff
Große Sprachmodelle können lokal auf eigener Hardware oder über die API eines externen Anbieters genutzt werden.

### 6.1 Lokale Zugriff
Das Modell wird direkt auf dem eigenen Rechner oder einem selbst betriebenen Server ausgeführt.

**Vorteil:** 
- Offline Nutzung
- mehr Kontrolle über Modellversionen, Datenschutz, Kosten

**Nachteil:** 
- große Modelle benötigen viel Arbeitsspeicher
- begrenzte Modellgröße durch Leistung der vorhandenen Hardware
- Hoher Setup Aufwand (Installation, Updates, Konfiguration)

### 6.2 API-basierter Zugriff
Das Modell läuft auf Servern eines Anbieters (z.B. OpenAI, Anthropic). Das Modell sendet die Anfrage über eine API.

**Vorteil:** 
- keine Installation notwendig
- schnelle Integration
- Zugriff auf sehr leistungsstarke Modelle

**Nachteil:** 
- abhängig vom Anbieter (Datenschutz, Kosten, etc.)
- Internet Zugriff erforderlich

## 7. Überblick über die wichtigsten Tools und Bibliotheken
Die Arbeit mit LLMs kann mithilfe von verschiedenen Tools und Bibliotheken vereinfacht werden. Die bekanntesten Tools und Bibliotheken werden im folgenden Abschnitt genannt:

### 7.1 Ollama 
- ermöglicht die lokale Ausführung auf dem eigenen Rechener
- bietet eine CLI für einfachen Zugriff

Offizielle Dokumentation: https://docs.ollama.com/

### 7.2 vLLM
- Libary von Open Source Code für schnelle und produktive Umgebung bei gropßen und paralleen Anfragen
- stellt Inferenz-Engine und Inferenz-Server bereit

Offizielle Dokumentation: https://docs.vllm.ai/en/latest/ 

### 7.3 Hugging Face
- zentrale Plattform für Modelle, Datensätze und Bibliotheken im Bereich KI
- stellt verschiedenen offene Modelle bereit

Offizielle Dokumentation: https://huggingface.co/docs

### 7.4 OpenRouter
- stellt einheitliche API/Schnittstelle für Modelle unterschiedlicher Anbieter bereit
- ermöglicht einfachen Vergleich zwischen verschiedenen Modellen

Offizielle Dokumentation: https://openrouter.ai/docs/quickstart

### 7.5 Sentence Transformers
- Modelle zur Erzeugunng von Embeddings (Embaddings=Vektoreinbettung)

Offizielle Dokumentation: https://sbert.net/

### Quellen
- https://www.iese.fraunhofer.de/blog/wie-funktionieren-llms/
- https://services.informatik.hs-mannheim.de/~mechler/see2/SEE2-LLM-GPT-Basics.pdf
- https://digitaleneuordnung.de/blog/prompting-techniken#h-step-back-prompting
- https://www.ibm.com/de-de/think/topics/context-window#