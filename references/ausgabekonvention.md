# Ausgabekonvention DOCX

Diese Konvention gilt für alle drei Extraktoren. Ziel ist ein DOCX, das der nachfolgende Pseudonymisierungslauf ohne Sonderbehandlung verarbeiten kann.

## Kopfzeile

Der erste Absatz jedes DOCX enthält Metadaten in einer Zeile:

```
Quelle: <Originaldateiname> | Format: <PDF|PPTX|XLSX> | <Seiten/Folien/Blätter>: <n> | Extraktion: <ISO-Zeitstempel>
```

Beispiel: `Quelle: 260828_Schulung_Mailverlauf_Nordmark.pdf | Format: PDF | Seiten: 6 | Extraktion: 2026-09-05T19:48:08+00:00`

## Überschriftshierarchie

- `Heading 1` — obere Gliederungsebene je Format:
  - PDF: `Seite <n>`
  - PPTX: `Folie <n>: <Titel>`
  - XLSX: `Blatt: <Blattname>`
- `Heading 2` — nachrangige Struktur, wenn im Original erkennbar (aktuell nicht automatisch vergeben, für Handnachbesserung reserviert)
- `Heading 3` — weitere Ebene, ebenfalls reserviert

Der Anwender darf im DOCX Überschriften nachträglich vertiefen (z. B. „Kapitel 3.2" in `Heading 2` umwandeln). Der Pseudonymisierungsskill respektiert die Formatvorlagen und rührt sie nicht an.

## Absätze

- Formatvorlage `Normal`
- Jeder Absatz des Originals wird zu einem eigenen DOCX-Absatz
- Bei PDF wird an Doppel-Newlines (`\n\n`) getrennt; einfache Zeilenumbrüche innerhalb eines Absatzes bleiben als Leerzeichen erhalten (PyPDF-Standardverhalten)
- Bei PPTX wird an jedem Zeilenumbruch getrennt (jeder Aufzählungspunkt ist ein Absatz)

## Tabellen

- Formatvorlage `Table Grid`
- Kopfzeile nicht gesondert formatiert (die erste Zeile ist die Kopfzeile des Originals, bleibt inhaltlich unverändert)
- Zellinhalte als Text; Zahlen werden als String übernommen, damit die spätere Ersetzung stabil greift
- Excel-Formeln: der berechnete Wert wird übernommen (`data_only=True` in openpyxl). Formeltext geht verloren; wenn er benötigt wird, das Excel vor der Extraktion in einem Formeltextexport speichern.

## Bilder

Bilder werden nicht eingebettet. Für jedes übersprungene Bild fügt der Extraktor eine sichtbare Zeile ein:

- PDF: derzeit keine Bildmarkierung (PyPDF kann die Bildpositionen nicht zuverlässig zurückgeben)
- PPTX: `[Bild auf Folie <n>: <Shape-Name>]`
- XLSX: keine Bildmarkierung (Bilder in Excel sind selten inhaltsrelevant)

## Notizen (PPTX)

Nur aktiv, wenn `--mit-notizen` gesetzt ist. Standardmäßig deaktiviert, weil Notizen meist Vortragsanweisungen enthalten, die im Extrakt keinen Nutzen haben. Wenn aktiv:

```
[Notizen Folie <n>]
<Notiztext>
```

## Zeichenkodierung

UTF-8 durchgängig. Umlaute und ß werden buchstabengetreu übernommen. Wichtig für den nachfolgenden Pseudonymisierungsskill: die Klarnamen im DOCX sind das, was in der Änderungskarte stehen muss.

## Was NICHT im DOCX steht

- Kopf- und Fußzeilen des Originals (bewusst weggelassen; enthalten meist Seitenzahlen, kein extraktrelevanter Inhalt)
- Wasserzeichen
- Kommentare und Änderungsverfolgung
- Verknüpfte Objekte, SmartArt, Diagramme (nur Titel/Beschriftung, wenn als Textrahmen zugreifbar)
- Verborgene Folien (PPTX) und ausgeblendete Zeilen/Spalten (XLSX)

Diese Auslassungen sind bewusst. Wenn sie im konkreten Fall relevant sind, muss die Extraktion durch einen Fachanwender manuell ergänzt werden.

## Prüfbericht (JSON auf stdout)

Jedes Skript gibt am Ende einen Bericht auf stdout aus:

```json
{
  "eingabe": "…",
  "ausgabe": "…",
  "seiten|folien|blaetter": ...,
  "absaetze": ...,
  "tabellen": ...,
  "bilder_uebersprungen": ...,
  "warnungen": ["…"],
  "status": "OK" | "FEHLER"
}
```

Der Bot referenziert diesen Bericht in seiner Antwort an den Anwender. Bei `status: FEHLER` liefert der Bot kein DOCX aus, sondern gibt den Fehlertext und einen Vorschlag zum weiteren Vorgehen.
