---
name: dokument-nach-docx
description: |
  Extraktion von Text und Struktur aus PDF-, PPTX- und XLSX-Dokumenten in ein
  strukturiertes DOCX für die anschließende Weiterverarbeitung, insbesondere für
  den Skill pseudonymisierung-mandatsdokumente. Für Text-PDF, PowerPoint und
  Excel deterministisch. Für gescannte PDF ohne Textebene meldet der Skill den
  Bedarf einer OCR-Vorstufe und arbeitet dann mit dem OCR-Ergebnis weiter.
  Trigger: PDF zu Word, PPTX zu Word, Excel zu Word, Dokument in DOCX
  überführen, extrahieren, konvertieren, Vorbereitung für Pseudonymisierung.
  Ausgang ist immer DOCX. Erhält Überschriftshierarchie, Absätze, Aufzählungen
  und Tabellen; Bilder werden benannt, aber nicht mitgeführt. Formatierung ist
  bewusst minimal, damit der nachfolgende Pseudonymisierungslauf nicht durch
  Formatvielfalt gestört wird.
---

# Extraktion nach DOCX

Dieser Skill erzeugt aus einem Eingangsdokument (PDF, PPTX oder XLSX) eine strukturierte DOCX-Datei. Ziel ist die kontrollierte Weiterverarbeitung, insbesondere durch den Skill `pseudonymisierung-mandatsdokumente`, der ausschließlich DOCX verarbeitet.

## Grundprinzip

- Ein Eingangsformat je Lauf
- Ein DOCX als Ausgang
- Struktur (Überschriften, Absätze, Listen, Tabellen) wird erhalten, wo verfügbar
- Formatierung wird bewusst reduziert (einheitliche Schrift, klare Absätze), damit die spätere Pseudonymisierung stabil arbeitet
- Bilder werden nicht übernommen; ihre Position wird durch einen Platzhalter mit Kurzbeschreibung markiert
- Der Skill entscheidet nicht über Inhalt. Er benennt aufgetretene Grenzen im Prüfbericht, damit der Anwender einschätzt, ob die Ausgabe genügt.

## Aufgabenarten

| Anweisung | Was passiert | Skript |
|---|---|---|
| `EXTRAHIEREN` (PDF) | Textextraktion aus Text-PDF, Erhalt der Überschriftshierarchie soweit erkennbar | `scripts/extract_pdf.py` |
| `EXTRAHIEREN` (PPTX) | Folienweise Textextraktion, jede Folie als Abschnitt mit Titel | `scripts/extract_pptx.py` |
| `EXTRAHIEREN` (XLSX) | Blattweise Tabellen, jede Zelle als Tabellenzelle | `scripts/extract_xlsx.py` |
| `PRUEFEN` | Analyse der Eingabedatei ohne Extraktion (Text-PDF vs. Scan, Bildanteil, Umfang) | interne Prüfung |

## Ablauf

1. Anwender lädt Datei hoch und sagt `EXTRAHIEREN`.
2. Der Skill erkennt am Dateiformat, welches Skript zu verwenden ist.
3. Bei PDF prüft er zuerst, ob eine Textebene vorliegt. Ohne Textebene: Hinweis auf OCR-Bedarf und Abbruch.
4. Der Skill ruft das passende Skript auf. Es erzeugt `<Name>.docx` im Ausgabeverzeichnis.
5. Der Skill liefert das DOCX plus einen Prüfbericht (Anzahl Seiten oder Folien oder Blätter, Anzahl Absätze, Anzahl Tabellen, Anzahl übersprungener Bilder, aufgetretene Warnungen).
6. Der Anwender öffnet das DOCX in Word und prüft, ob die Struktur stimmt. Bei Bedarf: Nachbesserung von Überschriftsebenen oder Absatzumbrüchen von Hand.

## Verbindliche Regeln

1. Keine Inhaltsinterpretation. Der Skill überträgt Text; er fasst nicht zusammen, er kommentiert nicht.
2. Kein Verlust: jede Textstelle des Originals muss im DOCX auftauchen. Wenn eine Stelle nicht extrahierbar ist (z. B. Bildtext ohne OCR), wird ein sichtbarer Platzhalter eingefügt: `[nicht extrahierbar: Bildinhalt Seite 3, ca. 12 Zeilen]`.
3. Reihenfolge wahren: bei PDF Seite für Seite, bei PPTX Folie für Folie, bei XLSX Blatt für Blatt.
4. Metadaten mitliefern: Quelldatei, Format, Seiten- oder Folien- oder Blattzahl, Zeitstempel oben in einer Kopfzeile.
5. Warnungen sichtbar machen: der Prüfbericht listet alle aufgetretenen Warnungen. Der Anwender entscheidet.
6. Keine Bildinhalte im DOCX. Ausschließlich Text und Tabellen. Bilder werden benannt (`[Bild: Diagramm auf Folie 4]`), nicht eingebettet.
7. Wenn die Datei mehr als 100 Seiten oder 200 Folien oder eine Zelle mit über 30.000 Zeichen enthält: Abbruch mit klarer Fehlermeldung. Große Dokumente vorher aufteilen.
8. Wenn das Skript einen Fehler meldet: Fehlertext unverändert an den Anwender geben, nicht paraphrasieren.

## Ausgabekonvention

- Dateiname: `<Originalname_ohne_Endung>.docx`
- Ablageort: gleiches Ausgabeverzeichnis wie das Skript vorgibt (`--outdir`)
- Kopfzeile im DOCX: Originaldateiname, Format, Datum der Extraktion
- Überschriften: Word-Formatvorlagen `Heading 1` (Kapitel bei PDF, Folie bei PPTX, Blatt bei XLSX), `Heading 2` (Unterabschnitt), `Heading 3` (weitere Ebene)
- Absätze: `Normal`
- Tabellen: einfache Rasterformatierung ohne Farben

## Was der Skill nicht kann

- Gescannte PDFs ohne Textebene: braucht OCR (extern; z. B. `ocrmypdf` oder ein OCR-fähiger PDF-Reader). Der Skill erkennt das und bittet den Anwender, das PDF vorher zu OCR-en.
- PowerPoint-Notizen und -Kommentare: werden aktuell nicht extrahiert (kann in einer Folgeversion ergänzt werden).
- Excel-Formeln: werden als berechneter Wert übernommen, nicht als Formeltext.
- Verknüpfte Objekte, eingebettete Videos, SmartArt-Grafiken: erscheinen nur als Platzhalter.
- Verschlüsselte Dokumente: müssen vor der Übergabe entschlüsselt sein.
- Handschrift, Formulare, Screenshots von Tabellen: braucht OCR.
- Sehr lange Dokumente: siehe Regel 7.

Diese Grenzen sind ausführlich in `references/grenzen-und-fallstricke.md` beschrieben.

## Empfohlener Nachfolgeschritt

Nach der Extraktion in einer offenen Umgebung ist das DOCX **noch nicht** pseudonymisiert. Es enthält alle Klarnamen des Originals. Bevor es in eine offene KI-Umgebung geht, ist der Skill `pseudonymisierung-mandatsdokumente` anzuwenden.

Der typische Ablauf für Mandatsdokumente in Nicht-DOCX-Formaten ist deshalb:

1. `dokument-nach-docx` erzeugt DOCX
2. `pseudonymisierung-mandatsdokumente` erzeugt PSEUDO-DOCX und Legende
3. PSEUDO-DOCX geht in die offene Umgebung, Legende bleibt geschlossen

## Skripte

- `scripts/extract_pdf.py` — Textextraktion aus Text-PDF (pypdf)
- `scripts/extract_pptx.py` — Extraktion aus PowerPoint (python-pptx)
- `scripts/extract_xlsx.py` — Extraktion aus Excel (openpyxl)
- `scripts/requirements.txt` — Abhängigkeiten (pypdf, python-pptx, openpyxl, python-docx)

Skriptaufruf, jeweils gleiches Muster:

```
python scripts/extract_pdf.py --input <eingang.pdf> --outdir <verzeichnis>
python scripts/extract_pptx.py --input <eingang.pptx> --outdir <verzeichnis>
python scripts/extract_xlsx.py --input <eingang.xlsx> --outdir <verzeichnis>
```

Jedes Skript gibt einen JSON-Prüfbericht auf stdout aus.

## Selbstkontrollen

Vor Auslieferung des DOCX prüft der Skill:

- Datei ist lesbar (`python-docx` öffnet sie ohne Fehler)
- Kopfzeile ist gefüllt
- Anzahl Absätze > 0
- Prüfbericht enthält alle Warnungen

Bei Verstoß gegen eine der Regeln liefert der Skill kein DOCX aus, sondern beschreibt das Problem und schlägt einen Weg vor.

## Rechtlicher Hinweis

Dieser Skill überträgt Inhalte technisch. Er trifft keine Aussage über die Zulässigkeit der Weiterverarbeitung des Inhalts. Für die datenschutz- und berufsrechtliche Bewertung gelten die RWT-Richtlinien. Keine Rechts-, Steuer- oder Datenschutzberatung.

## Referenzen

- `references/ausgabekonvention.md` — Detaillierte Beschreibung des DOCX-Aufbaus
- `references/grenzen-und-fallstricke.md` — Bekannte Grenzen der drei Extraktoren mit Beispielen
