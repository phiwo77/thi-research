# Änderungsprotokoll

Dieses Protokoll beschreibt den im Workspace prüfbaren Stand. Es ersetzt keine
Versionsverwaltung und behauptet keine nicht belegte historische Reihenfolge
früherer Zwischenstände.

## 2026-08-27 – isolierte Stage-7-Kernelanalyse

- kontrollierte Variation ausschließlich von `kernel_size` (3/7/9) gegen die
  unverändert geladene Stage-6-Referenz (5);
- frische interne Train-Reserve mit vollständigem Ausschluss sämtlicher
  Stage-6-Basisrouten und routenexklusiver Auswahl-/Holdout-Teilung;
- Validation, Calibration und Test für Training, Auswahl und Metriken gesperrt;
- neun neue Modellzustände, Seed-Streuung, Ereignis-, Übergangs-, Profil- und
  Wahrscheinlichkeitsmetriken sowie atomarer SUCCESS-Export;
- keine Modellpromotion, da keine Variante alle Auswahlgates erfüllt;
- 187/187 Regressionstests bestanden.

## 2026-08-22 – isolierte Stage-2-Arbeitskopie

### Hinzugefügt

- strikt eingefrorene Stage-2-Präregistrierung mit Full-Run-, Daten-, Split-,
  Seed-, Kandidaten-, Rollen- und Bootstrapvertrag;
- Full-Selector mit einer Lese-Whitelist von fünf Artefakten, Hash-/Byte-,
  Trial-, Zustands- und Roundtripprüfung sowie deterministischer
  Validation-Siegerregel;
- reiner Train/Validation-Runner für zwei präregistrierte GRU-Verbesserungen
  und zwei `causal_residual_tcn`-Kandidaten über je drei Seeds;
- gehashtes Drei-Rollen-Modellmanifest, Rehydration ausschließlich aus dem
  Freeze sowie Post-Calibration-Hash- und Artefaktkontrollen;
- protokollweit stabiler, pfadunabhängiger und crashfester
  `CALIBRATION_OPENED`-Ledger mit gebundenem Markerinhalt, gemeinsame Auswertung
  der drei Rollen und aller sechs eingefrorenen Full-Modelle sowie ein
  gebatchter gepaarter 10.000er-Routenbootstrap;
- Vorabblockade überlappender Ergebnis-, Full-, App- und Ledgerpfade;
- atomare No-Replace-Publikation über das Betriebssystem (`RENAME_EXCL` auf
  macOS, `RENAME_NOREPLACE` auf Linux; sonst fail-closed) einschließlich
  deterministischem Swap-Race-Test;
- eigener CLI-Einstieg `code/stage2.py` und gezielte Gate-, Fehler-, Abbruch-,
  Kausalitäts-, Hash-, Reihenfolge-, Bootstrap- und Publikationstests.

### Verifiziert

- 156/156 Tests unter `code/tests/` plus fünf Root-Regressionsprüfungen,
  insgesamt 161/161 mit Python 3.12, `-B` und Warnungen als Fehler bestanden;
- der noch unveröffentlichte Full-Stagingordner wird vor Datenscan und jeder
  Ausgabe mit `NOT_A_PUBLISHED_FULL_DIRECTORY` abgelehnt;
- kein Full-, Calibration- oder Testlauf wurde aus dieser Kopie gestartet;
- keine Ergebnis-, Run- oder Cacheordner wurden in die Kopie übernommen.

## 2026-08-22 – isolierte v2-Entwicklungskopie

### Hinzugefügt

- additive Modellkennung `causal_residual_tcn` mit eigenem präfixkausalem
  numerischem Encoder, trainierbaren Edge-Embeddings, drei Residualblöcken,
  Dilatationen 1/2/4, zwei kausalen Faltungen je Block und expliziter
  Residualprojektion bei Kanalwechsel;
- 49-dimensionaler Regressionskopf sowie trainierbare Stopp- und
  Übergangsköpfe mit gemeinsamer Validation-selektierter Multitask-Loss;
- strikt validierter JSON-Zustand v2 ohne Pickle- oder Legacy-Fallback;
- additive Schema-2-Presets `extended_smoke`, `extended_quick`,
  `extended_standard`, `extended_full` und getrennte CLI-/GUI-Auswahl;
- gezielte Kausalitäts-, Receptive-Field-, Residual-, Determinismus-,
  Event-Head-, Zustands-, Korruptions-, Abbruch- und Integrationstests;
- selbstständige Legacy-Inferenzfixtures unter `code/tests/fixtures`, damit die
  Tests ohne kopierte Laufordner reproduzierbar bleiben.

### Verifiziert

- 129/129 Tests unter `code/tests/` sowie fünf Root-Regressionsprüfungen,
  insgesamt 134/134 Tests mit Python 3.12, `-B` und Warnungen als Fehler
  bestanden;
- temporärer synthetischer Schema-2-Einmodelllauf einschließlich Runner,
  Serialisierungsroundtrip, Spezifikation, Auswertung und Export bestanden;
- keine Benchmarkläufe oder Caches in die Entwicklungskopie übernommen;
- kein `extended_full`-Lauf gestartet und kein neues wissenschaftliches
  Ranking behauptet.

## 2026-08-21 – dokumentierter Benchmarkstand

### Hinzugefügt

- gemeinsamer Benchmark-Runner für globalen Mittelwert, Ridge, Gradient
  Boosting, MLP, GRU und TCN;
- versionierte Presets `smoke`, `quick`, `standard` und `full` mit expliziten
  Seeds, Splitgrenzen und begrenzten Kandidatenlisten;
- systematischer JSONL-Datenvertrag mit Sentinel-, Typ-, Werte-, Längen- und
  Distanzbilanzprüfung;
- eingefrorenes routenexklusives Splitmanifest auf Basis der geordneten
  Edgefolge;
- technisch gekapselte Train-/Validation-Auswahlansicht, schreibgeschützte
  Matrizen und verzögerte Materialisierung von Calibration/Test;
- `route_mean`-Zielpolitik mit Erhalt roher Routenreplikate für Bewertung und
  Dispersion;
- train-only Standardisierung sowie trainierbare PAD-/UNK-Edge-Embeddings für
  GRU und TCN;
- explizite Profil-, Form-, Gradienten-, Stillstands- und
  Stopp-/Anfahrübergangsmetriken mit profil- und routegewichteter Aggregation;
- getrennte Generalisierungsstrata für neue Kombinationen bekannter Edges und
  Routen mit mindestens einer unbekannten Edge;
- klar begrenzter flachstraßenbasierter Energiesensitivitätsproxy;
- deterministische automatische Interpretation mit verpflichtenden
  Aussagegrenzen zu Verkehr, SUMO-Ersatz/Speed-up, Zeitbasis, Tempolimits,
  Signalen, Knoten, Szenarien, der fehlenden strikten Noise-Floor-Aussage und
  Energie;
- streamingbasierte Within-Route-Dispersionsanalyse mit Welford-Momenten,
  Multiplikitätsklassen und atomarem Cache;
- atomarer Laufexport mit Konfiguration, Audit, HPO-Versuchen,
  Modellzuständen, Prognosen, Ranking, Bericht, Grafiken, Log und
  Artefakthashes;
- erneute Abbruch- und Zielpfadprüfung vor und nach der Artefakt-Hashbildung;
  in den getesteten Fehlerpfaden bleibt der fremde Zielinhalt erhalten und
  Diagnoseordner tragen keinen falschen `SUCCESS.json`-Marker;
- nicht blockierende Tk-GUI mit Presets, editierbarem JSON,
  Ein-/Alle-Modell-Auswahl, Fortschritt, sicherem Abbruch und Ergebnisansicht;
- automatisierte Tests für den modularen Benchmark und seine wiederverwendeten
  Komponenten, einschließlich echter Benchmark-CLI-Unterprozesse;
- Paketprüfer und `SHA256SUMS` zur unabhängigen Integritätskontrolle der
  ausgelieferten Dateien.

### Verifiziert

- 122/122 Tests des gesamten Codepakets bestanden am 21. August 2026;
- vollständiger Smoke-Pfad aller sechs Modelle bis `SUCCESS.json` ausgeführt;
- aktueller verifizierter Smoke-Lauf `smoke-136266439df0` mit Datensatz-,
  Konfigurations-, Splitmanifest-, Artefakt- und Quellcodehashes veröffentlicht;
- im Smoke-Audit 184.456 Zeilen, 165.616 gültige und 18.840 wegen des
  negativen Sentinelwerts ausgeschlossene Datensätze erfasst;
- im Smoke-Split alle sechs paarweisen Routenüberlappungen mit 0 bestätigt;
- vorhandenes Vollbestands-Dispersionsartefakt rechnerisch geprüft:
  110.615 Routengruppen, davon 35.344 Mehrfachgruppen mit 90.345 Profilen;
- Route-Makro-Within-RMSE des 48-Punkt-Profils
  `4.713243270315268 m/s` und profilgewichteter Wert
  `4.732783587002383 m/s` als empirischer Conditional-Dispersion-/
  Noise-Floor-Proxy bestätigt.

### Dokumentiert

- neue Einstiegshilfe in `README.md`;
- vollständige Definitionen und Gültigkeitsgrenzen in
  `METHODIK_UND_GUELTIGKEIT.md`;
- Nachweise, offene Freigabepunkte und korrekte Einordnung des Smoke-Laufs in
  `VERIFICATION_REPORT.md`;
- exakte Vollbestands-Dispersionsprüfung in `DISPERSION_RESULT.md`;
- `code/README.md` auf einen kurzen, aktuellen Einstieg in den modularen
  Benchmark reduziert.

### Wissenschaftliche Klarstellungen

- Die Rangfolge in `verified_smoke_run/` ist ausschließlich ein
  Funktionsnachweis und kein belastbares Modellranking.
- `verified_full_dispersion.json` beschreibt einen empirischen
  Conditional-Dispersion-/Noise-Floor-Proxy. Es belegt weder irreduzibles
  SUMO-Rauschen noch eine MAE-Untergrenze oder reale Verkehrsstreuung.
- Der 48-Punkt-Index ist normiert und besitzt keine dokumentierte physische
  Zeit- oder Distanzbasis.
- Modelllaufzeiten werden ohne korrespondierenden SUMO-Benchmark gemessen und
  tragen keine SUMO-Speed-up-Aussage.
- Der Energieoutput bleibt eine relative Flachstraßen-Sensitivitätsgröße ohne
  Fahrzeug-, Batterie-, Kraftstoff- oder Emissionsvalidierung.

### Noch nicht als Ergebnis vorhanden

- ausgeführtes Modellranking mit dem Preset `full`;
- manuell dokumentierter GUI-Lauf auf dem späteren Zielrechner;
- externer Szenario-/Netz-/Verkehrsvalidierungssatz;
- physische Zeitbasis und darauf aufbauende Fahrzeit-, Beschleunigungs- oder
  Energievalidierung.
