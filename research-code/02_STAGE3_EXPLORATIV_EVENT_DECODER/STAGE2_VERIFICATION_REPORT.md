# Technischer Prüfbericht Stage 2

**Stand:** 22. August 2026  
**Status:** Implementierung verifiziert; Full- und Calibration-Ausführung ausstehend

## Ergebnis

Die Stage-2-Kopie implementiert den präregistrierten Ablauf vollständig und
getrennt vom laufenden Full-Quellbaum. Full-Selektor, reiner
Train/Validation-Runner, Drei-Rollen-Freeze, einmaliger Calibration-Ledger,
gemeinsame Calibration-Auswertung und gepaarter Routenbootstrap sind durch
synthetische Tests abgedeckt. Es wurde kein Full-, Calibration- oder Testlauf
gestartet.

Der reale Vorabaufruf gegen den derzeit nur versteckt vorhandenen
Full-Stagingordner endete erwartungsgemäß mit:

```text
FEHLER [NOT_A_PUBLISHED_FULL_DIRECTORY]
Phase: stage2_full_selection
```

Dabei entstanden weder Ergebnisordner noch Calibration-Ledger.

## Abgedeckte technische Verträge

| Vertrag | Nachweis |
|---|---|
| ausschließlich fünf Full-Dateien lesbar | Read-Guard-Systemtest |
| SUCCESS/Run-ID/Hash/Bytes/Config/Daten/Split | Selector-Korruptions- und Provenienztests |
| Trials, Seeds, Parameter, Kennzahlen und Zustände konsistent | Trial-/State-/Roundtriptests |
| Sieger ausschließlich über Validation | deterministische Tie-Break-Tests |
| Training sieht nur Train/Validation | `SelectionData`-Guard und read-only-Matrixtest |
| drei eindeutige Rollen vor Calibration eingefroren | Rehydrations-, Zustandsmutations- und Freeze-Tampertest |
| Full- und Stage-2-Zustände JSON-sicher | Loader- und Hash-Roundtriptests |
| Calibration protokollweit genau einmal und nie Test | stabiler pfadunabhängiger Ledger-, Marker-Binding- und Materialisierungstest |
| gleiche Profile/Reihenfolge für neun Auswertungsrollen | Evaluator- und Mutationsschutztest |
| Ausgabepolicy vor Ensemble | Negativprojektionstest |
| gepaarter Routenbootstrap reproduzierbar | Replikat-, PCG64-, Quantil- und Batchtest |
| Erfolg nur bei CI- und drei Seed-Gates | positiver und negativer Gate-Test |
| Fehler/Abbruch ohne falsches SUCCESS | später Abbruch- und Erhaltungstest |
| bestehende/konkurrierend erscheinende Ziele werden nicht ersetzt | atomarer OS-No-Replace- und Swap-Race-Test |
| Full/App/Ledger und Ausgabe überlappen nicht | Disjunktheits- und Vorabblockadetest |
| Ledger bleibt nach Crash nachweisbar | Parent-/Ledger-Verzeichnis-fsync-Vertragstest |

## Ausgeführte Tests

Gebündeltes Python 3.12, aus `code/`:

```bash
PYTHONDONTWRITEBYTECODE=1 python -B -W error -m unittest discover -v
```

Ergebnis:

```text
Ran 161 tests in 4.120s
OK
```

Klassifikation: 156 Tests unter `code/tests/`, davon 27 neue gezielte
Stage-2-Tests, plus fünf Root-Regressionsprüfungen in
`test_surrogate_pipeline.py`; insgesamt 161/161. Der finale fokussierte
Kernlauf ohne den separaten CLI-Systemtest umfasste 26/26 Stage-2-Tests in
0,213 s. Der finale vollständige Lauf auf dem gehärteten Stand umfasste
161/161 Tests in 4,120 s (4,47 s reale Prozesszeit). Ein unabhängiger
Kontrolllauf desselben Endstands umfasste ebenfalls 161/161 in 4,337 s.

Die 27 Stage-2-Tests teilen sich auf in einen CLI-, zwei Präregistrierungs-,
acht Full-Selector-, sieben Runner-/Publikations- und neun
Calibration-/Bootstraptests.

## Ausstehende wissenschaftliche Nachweise

- Der Full-Lauf ist noch nicht atomar veröffentlicht; aus Teil-Trials wurde
  kein Sieger abgeleitet.
- Die GRU- und Residual-TCN-Kandidaten wurden noch nicht auf dem Vollbestand
  trainiert.
- Calibration wurde nicht geöffnet; es gibt noch keine Gütewerte, kein
  Bootstrapintervall und keine bestätigte Verbesserung.
- Nicht-GRU-Sieger blockieren absichtlich, weil dafür keine gezielte
  Verbesserung präregistriert ist.
- Ein manueller GUI-Test ist für Stage 2 nicht einschlägig, da der neue Ablauf
  bewusst einen separaten CLI-Einstieg besitzt.
- Die Split-Kapselung ist eine interne API-Grenze: Der vollständige Audit-Scan
  hält Record-Objekte aller Splits im Prozess. Training erhält nachweislich nur
  `SelectionData` mit Train/Validation, und nur Calibration wird nach dem
  Ledger-Gate materialisiert; gegen absichtlich veränderten Prozesscode wäre
  eine externe Isolationsumgebung nötig.
- Der Full-Selector prüft die vollständige interne SUCCESS-, Hash-, Schema-,
  Trial- und State-Konsistenz, aber keine externe digitale Signatur oder
  unabhängig verwaltete Herkunftsattestation des lokalen Full-Ordners.
- Der Once-only-Ledger ist lokale Dateisystem-Custody. Andere Ergebnisnamen,
  Prozesse und Full-Pfade innerhalb derselben Paketablage bleiben gesperrt;
  administratives Löschen des Ledgers oder Verschieben der gesamten App in
  einen neuen Elternordner liegt außerhalb dieses lokalen Schutzmodells.
- Der atomare No-Replace-Pfad wurde auf macOS real ausgeführt. Linux
  (`RENAME_NOREPLACE`) und Windows sind statisch implementiert; unbekannte
  Systeme brechen ab, statt auf eine unsichere Umbenennung zurückzufallen.

Diese Grenzen sind wissenschaftliche Ausführungsgrenzen, keine still
überbrückten Implementierungslücken.
