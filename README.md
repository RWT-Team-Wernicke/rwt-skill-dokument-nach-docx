# rwt-skill-dokument-nach-docx

Agent-Skill zur Extraktion von PDF-, PPTX- und XLSX-Dokumenten in ein strukturiertes DOCX. Vorstufe für den Skill `rwt-skill-pseudonymisierung`, der ausschließlich DOCX verarbeitet.

Interner Skill der RWT-Gruppe / Die TaxMaxen. Zielumgebung: Claract (OMM Solutions, Stuttgart) oder eine andere Umgebung mit Unterstützung für [Open Agent Skills](https://github.com/agentskills/agentskills).

---

## Übersicht

- Drei Extraktoren: PDF (`pypdf`), PPTX (`python-pptx`), XLSX (`openpyxl`)
- Ein Ausgabeformat: strukturiertes DOCX mit Kopfzeile, Überschriften, Absätzen und Tabellen
- Deterministisch, kein Sprachmodell in der Skriptausführung
- Ehrliche Grenzen: Scan-PDF, SmartArt, verbundene Excel-Zellen und andere Fallstricke sind dokumentiert und werden im Prüfbericht gemeldet

## Repositoriumsaufbau

```
rwt-skill-dokument-nach-docx/
├── SKILL.md                              Systemprompt und Regeln (vom Agenten gelesen)
├── README.md                             diese Datei (für Menschen)
├── scripts/
│   ├── requirements.txt
│   ├── extract_pdf.py                    PDF → DOCX (pypdf)
│   ├── extract_pptx.py                   PPTX → DOCX (python-pptx)
│   └── extract_xlsx.py                   XLSX → DOCX (openpyxl)
├── references/
│   ├── ausgabekonvention.md              Aufbau des DOCX
│   └── grenzen-und-fallstricke.md        Bekannte Grenzen der drei Extraktoren
└── assets/
    ├── 260828_Schulung_Mailverlauf_Nordmark.pdf     Test-PDF
    └── beispiel_beteiligungen.xlsx                  Test-XLSX (synthetisch)
```

## Einrichtung im GitHub-Repo (einmalig)

1. Repository in der RWT-GitHub-Organisation anlegen (private): `rwt-skill-dokument-nach-docx`
2. Diese Inhalte in den Hauptzweig commiten (`main`)
3. In Claract unter „Skills → Import" die Kennung `<org>/rwt-skill-dokument-nach-docx` eingeben (z. B. `rwt-gruppe/rwt-skill-dokument-nach-docx`)

## Test in einer beliebigen Python-Umgebung

```bash
cd rwt-skill-dokument-nach-docx
python -m pip install -r scripts/requirements.txt

# PDF-Test
python scripts/extract_pdf.py \
  --input assets/260828_Schulung_Mailverlauf_Nordmark.pdf \
  --outdir out/

# XLSX-Test
python scripts/extract_xlsx.py \
  --input assets/beispiel_beteiligungen.xlsx \
  --outdir out/

# PPTX-Test (eigene Datei mitbringen)
python scripts/extract_pptx.py \
  --input <ihre_praesentation>.pptx \
  --outdir out/
```

## Typischer Einsatz mit Pseudonymisierung

```
Anwender:                                                  Skill / Bot:
─────────────────────────────────────────────────────────────────────
lädt Original-PDF hoch                                     "EXTRAHIEREN"
                                                           → extract_pdf.py
                                                           → Original.docx + Prüfbericht

lädt Original.docx zurück                                  "PSEUDONYMISIEREN, Alias MANDAT-XY"
                                                           → Vorschlagsliste in Phase 1
sagt "FREIGABE"                                            → pseudonymize.py
                                                           → Original_PSEUDO.docx
                                                             + MANDAT-XY_LEGENDE_v1.md/.json

lädt Original_PSEUDO.docx in
offene KI-Umgebung, arbeitet dort
und lädt Ergebnis zurück                                   (Legende bleibt geschlossen)

Ergebnis_PSEUDO.docx + Legende                             "RUECKUMWANDELN, Alias MANDAT-XY"
                                                           → depseudonymize.py
                                                           → Ergebnis_KLAR.docx
```

## Bekannte Grenzen (Kurzform)

- Scan-PDF ohne Textebene: der Extraktor bricht mit klarer Meldung ab und empfiehlt OCR (`ocrmypdf`)
- Mehrspaltige PDF-Layouts: die Lesereihenfolge kann durcheinander geraten
- PPTX-SmartArt: wird nicht extrahiert
- PPTX-Gruppen: nur die oberste Ebene wird gelesen
- Excel-Formeln: nur berechnete Werte, kein Formeltext
- Verbundene Excel-Zellen: nur die obere linke Zelle enthält den Wert

Vollständige Auflistung mit Auswegen in `references/grenzen-und-fallstricke.md`.

## Kein Ersatz für Rechts-, Steuer- oder Datenschutzberatung

Der Skill überträgt Inhalte technisch. Er trifft keine Aussage über die Zulässigkeit der Weiterverarbeitung. Für die datenschutz- und berufsrechtliche Bewertung gelten die RWT-Richtlinien.

## Version

Version 1.0 vom 05.09.2026 (getestet in isolierter Sandbox, nicht in Claract-Produktion).
