# Sicherheitsmeldungen

Dieser Skill wird als Vorstufe fuer die Verarbeitung von Mandatsdokumenten eingesetzt. Sicherheitsprobleme werden deshalb nicht als oeffentliches GitHub-Issue gemeldet, sondern intern.

## Meldeweg

**Interner Ansprechpartner (Uebergangsstand):**
Daniel Wernicke, RWT-Gruppe / Die TaxMaxen
E-Mail: daniel.wernicke@rwt-gruppe.de

Der Ansprechpartner leitet die Meldung an RWT-IT und, wenn Mandatsbezug moeglich ist, an RWT-Datenschutz weiter.

## Was gemeldet werden sollte

- Fehlverhalten der Extraktoren, das zu Datenverlust fuehrt (Text im Original, aber nicht im DOCX, ohne Warnung)
- Fehlverhalten, das zu Datenpreisgabe fuehrt (z. B. verborgene Excel-Spalten oder Notizen, die entgegen den Voreinstellungen im Extrakt landen)
- Auffaelligkeiten in Abhaengigkeiten (`pypdf`, `python-pptx`, `openpyxl`, `python-docx`), die fuer den Anwendungsfall relevant sind
- Fehlerhaftes Verhalten bei verschluesselten oder beschaedigten Eingabedateien, das keinen sauberen Fehler wirft

## Was NICHT hier gemeldet wird

- Bugs ohne Sicherheitsbezug: normales GitHub-Issue im jeweiligen Repository
- Fragen zur Bedienung: siehe `references/ausgabekonvention.md`, `references/grenzen-und-fallstricke.md` und die zentrale IT-Uebergabe-Notiz

## Vertrauliche Kommunikation

Wenn eine Meldung mandatsbezogene Beispiele enthaelt, ausschliesslich verschluesselt uebermitteln.

## Reaktionszeiten (Richtwerte)

- Eingangsbestaetigung: 3 Arbeitstage
- Erste inhaltliche Rueckmeldung: 10 Arbeitstage
- Verbindliche Regelung nach Vollintegration in die RWT-GitHub-Organisation
