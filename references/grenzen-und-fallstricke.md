# Grenzen und Fallstricke

Ehrliche Auflistung dessen, was die drei Extraktoren nicht leisten. Der Bot muss diese Grenzen im Prüfbericht ansprechen, wenn sie im konkreten Fall auftreten.

## PDF

### Scan-PDF ohne Textebene

Der häufigste harte Fall. `pypdf.extract_text()` liefert für Scan-PDF leere Strings, weil die Seiten nur Bilddaten enthalten.

- Der Extraktor erkennt das (alle Seiten leer → Abbruch mit klarer Meldung).
- Auflösung: das PDF vorher OCR-en, z. B. mit `ocrmypdf` (unter Linux/macOS: `ocrmypdf input.pdf output.pdf -l deu`). Danach den Extraktor auf dem OCR-Ergebnis ausführen.
- Kritisch: OCR-Ergebnisse sind fehlerhaft, insbesondere bei Fachbegriffen, Zahlen und Sonderzeichen. Der Anwender muss das DOCX nach der Extraktion gründlicher als bei Text-PDF prüfen.

### Mehrspaltige Layouts

`pypdf` liest Textblöcke in der Reihenfolge, in der sie im PDF-Dokumentbaum stehen, nicht nach visueller Anordnung. Bei zweispaltigen Fachaufsätzen kann die Reihenfolge zerrissen werden (Spalte 1 → Spalte 2 durcheinander).

- Der Extraktor gibt keinen Warnhinweis (weil er die Layoutstruktur nicht analysiert).
- Auflösung: bei bekannten mehrspaltigen Vorlagen manuell auf einspaltigem Export bestehen, oder Extraktion mit einem visuellen Layout-Parser (z. B. `pdfplumber`) — nicht Teil dieses Skills.

### Tabellen in PDF

`pypdf` kennt keine Tabellenstruktur. Tabellenzellen erscheinen als Text, entweder in einer Zeile mit Trennzeichen oder in mehreren Zeilen. Der DOCX enthält deshalb keine echte Tabelle, sondern Fließtext.

- Auflösung, wenn Tabellenstruktur benötigt wird: `pdfplumber` oder `camelot` einsetzen — nicht Teil dieses Skills.

### Verschlüsselte PDF

`pypdf` scheitert an passwortgeschützten PDFs.

- Der Extraktor gibt den Fehler unverändert weiter.
- Auflösung: PDF vor der Übergabe entschlüsseln (Passwort entfernen).

### Beschädigte PDF

Manche PDF-Dateien lösen bei `pypdf` einzelne Warnungen aus (defekte Cross-Reference-Tabelle etc.). Der Extraktor fängt Exceptions je Seite ab und markiert die Seite mit einer Warnung, verarbeitet aber die restlichen Seiten weiter.

- Auflösung: das PDF vorher mit `qpdf --linearize` oder ähnlichem reparieren, wenn viele Seiten betroffen sind.

## PPTX

### Gruppierte Shapes

PowerPoint erlaubt beliebige Verschachtelung: Gruppen von Gruppen von Shapes. Der Extraktor durchläuft nur die oberste Ebene und ignoriert den Inhalt gruppierter Shapes.

- Der Extraktor gibt eine Warnung je Folie mit gruppierter Shape aus.
- Auflösung: die Gruppierung in PowerPoint aufheben und die Extraktion wiederholen.

### SmartArt-Grafiken

SmartArt ist keine Text-Shape, sondern eine gerenderte Zeichnung. `python-pptx` kann die enthaltenen Texte nicht zuverlässig zurückgeben.

- Der Extraktor überspringt SmartArt kommentarlos.
- Auflösung: SmartArt in PowerPoint in normale Textrahmen umwandeln (Rechtsklick → In Text konvertieren) und Extraktion wiederholen.

### Master- und Layout-Elemente

Text, der auf dem Folienmaster oder in einem Folienlayout steht (z. B. Fußzeilen-Vorlagen), erscheint nicht in der einzelnen Folie und wird deshalb nicht extrahiert.

- Auflösung: bei relevanten Master-Inhalten diese in einer separaten Handnotiz mitgeben.

### Verborgene Folien

Werden extrahiert wie sichtbare Folien. Wenn das nicht gewünscht ist, muss der Anwender sie vorher aus dem PPTX löschen.

### Notizen

Werden nur mit `--mit-notizen` extrahiert. Standardmäßig aus, weil sie meistens Vortragshilfen enthalten.

## XLSX

### Formeln

Der Extraktor liest mit `data_only=True`: er sieht den berechneten Wert, den Excel beim letzten Speichern in die Datei geschrieben hat, nicht den Formeltext.

- Konsequenz: eine Datei, die nach dem Speichern nie neu berechnet wurde (z. B. ein Vorlagenblatt), kann leere Zellen zeigen, wo Excel eigentlich einen berechneten Wert anzeigt.
- Auflösung: die Excel-Datei einmal öffnen, `Strg+Alt+F9` (alles neu berechnen), speichern, dann extrahieren.

### Verbundene Zellen

`openpyxl` gibt den Wert der oberen linken Zelle einer Verbundzelle zurück, die anderen Zellen im Verbund sind leer. Im DOCX entsteht dadurch eine Tabelle mit einigen leeren Zellen dort, wo im Original der Verbund war.

- Auflösung: bei kritischen Blättern Verbund vorher auflösen (Excel: „Zellen zusammenführen" abschalten) oder das DOCX nachbearbeiten.

### Große Blätter

Der Extraktor beschneidet auf 50.000 Zellen je Blatt (siehe `MAX_ZELLEN_PRO_BLATT` in `extract_xlsx.py`). Alles darüber wird abgeschnitten mit Warnung.

- Auflösung: das Blatt vorher auf die relevanten Zellen begrenzen (Löschen, Kopieren in neues Blatt).

### Ausgeblendete Zeilen/Spalten

Werden mitextrahiert. `openpyxl` unterscheidet nicht standardmäßig zwischen sichtbaren und ausgeblendeten Zellen.

- Konsequenz für Datenschutz: wenn eine Spalte in Excel ausgeblendet war und deshalb übersehen wurde, taucht sie im DOCX wieder auf. Der Anwender muss das wissen.

### Diagramme, Bilder, Kommentare

Nicht extrahiert. Diagramme und Bilder sind nicht Teil der Zelldaten. Kommentare (`ws.comment_at`) werden aus Zeitgründen aktuell nicht gelesen.

## Was gemeinsam für alle drei gilt

- Der Extraktor überträgt Text, nicht Bedeutung. Ob der extrahierte Text vollständig genug ist, entscheidet der Anwender.
- Für einen Freigabeprozess sollte der Anwender das erzeugte DOCX vor der Pseudonymisierung stichprobenartig mit dem Original vergleichen (z. B. drei zufällige Seiten oder Folien öffnen).
- Kein Automatismus ersetzt die Verantwortung des Bearbeiters, Mandatsdaten korrekt zu behandeln.

## Kein Ersatz für Rechts-, Steuer- oder Datenschutzberatung
