# SUMO-Profil-Benchmark – isolierte Stage-2-Arbeitskopie

> **Status dieser Kopie (22. August 2026):** Dies ist die getrennte
> Entwicklungsvariante `SUMO_Profile_Benchmark_stage2_work`. Sie baut auf der
> v2-Arbeitskopie auf, enthält aber keine übernommenen Benchmarkläufe oder
> Caches und verändert weder `SUMO_Profile_Benchmark`, dessen laufenden
> Full-Prozess noch `SUMO_Profile_Benchmark_v2_work`. Stage 2 ist technisch
> implementiert, wurde jedoch wissenschaftlich noch nicht ausgeführt. Solange
> der präregistrierte Full-Lauf nicht atomar mit `SUCCESS.json` publiziert ist,
> blockiert der Einstieg kontrolliert vor Datenscan, Training, Calibration und
> Ausgabeerzeugung. Details stehen in
> [STAGE2_PREREGISTRATION.md](STAGE2_PREREGISTRATION.md) und
> [STAGE2_VERIFICATION_REPORT.md](STAGE2_VERIFICATION_REPORT.md).

Schema 1 vergleicht weiterhin unverändert sechs Ansätze zur Prognose eines
normierten Geschwindigkeitsprofils aus **statischen Routeninformationen**:

- globaler Mittelwert und Ridge als Referenzen,
- Gradient Boosting und MLP als tabellarische Modelle,
- GRU und TCN als Modelle der vollständigen geordneten Edge-Sequenz.

Schema 2 ergänzt additiv die eigene Kennung `causal_residual_tcn`: ein echtes
dreiblöckiges kausales Residual-TCN mit Dilatationen 1/2/4, präfixkausalem
numerischem Encoder, trainierbaren Edge-Embeddings sowie separaten Stopp- und
Übergangs-Outputs. Das bisherige Modell `tcn` und alle Schema-1-Presets bleiben
inhaltlich und namentlich unverändert.

Alle Modelle verwenden denselben validierten Datenbestand, denselben
routenexklusiven Split, dasselbe 48-Punkt-Ziel und dieselben
Bewertungsdatensätze. Die Anwendung verändert die Rohdaten nicht.

> **Wissenschaftlicher Geltungsbereich:** Der Benchmark untersucht die
> Abbildung *statische Route → normiertes Geschwindigkeitsprofil*. Im Datensatz
> fehlen insbesondere eine physische Zeitbasis, Verkehrszustände,
> SUMO-Szenarien und -Seeds, Tempolimits, Lichtsignalanlagen,
> Knotensteuerungen sowie eine Netzversion. Die Ergebnisse belegen daher weder
> Verkehrssensitivität noch SUMO-Gleichwertigkeit oder einen Speed-up. Die
> ausgegebene Energiegröße ist ausschließlich ein flachstraßenbasierter
> Sensitivitätsproxy.

## Schnellstart

Vorausgesetzt werden Python mit Tk-Unterstützung für die GUI sowie die in
`code/requirements.txt` festgelegten Laufzeitbibliotheken NumPy und Pillow.

```bash
cd outputs/SUMO_Profile_Benchmark_stage2_work/code
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

Für die exakt geprüfte Referenzumgebung mit Python 3.12 stehen die fest
aufgelösten Bibliotheksversionen in `requirements-lock.txt`. Der allgemein
unterstützte Bereich bleibt in `requirements.txt` beziehungsweise
`code/requirements.txt` dokumentiert.

### Direkter Start unter macOS

Nach der Installation kann `START_BENCHMARK_GUI.command` im Finder per
Doppelklick gestartet werden. Das Skript verwendet bevorzugt
`code/.venv/bin/python` und andernfalls `python3`. `RUN_TESTS.command` startet
auf demselben Weg die strenge Testsuite mit Warnungen als Fehlern.

### Kommandozeile

Der zweite Positionsparameter muss ein **neuer, noch nicht existierender**
Laufordner sein. Bestehende Ergebnisse werden nicht überschrieben.

```bash
python benchmark.py /pfad/student_training_data.jsonl /pfad/neuer_lauf --preset smoke
python benchmark.py /pfad/student_training_data.jsonl /pfad/neuer_lauf --preset extended_smoke
```

Eine vollständig eigene, schema-konforme Konfiguration wird so verwendet:

```bash
python benchmark.py /pfad/student_training_data.jsonl /pfad/neuer_lauf \
  --config ../configs/meine_konfiguration.json
```

`python benchmark.py --help` zeigt die aktuellen Optionen.

### Präregistrierte Stage 2

Stage 2 besitzt einen separaten Einstieg und ändert den bisherigen Benchmark
nicht. Die drei Pfade sind der unveränderte Rohdatensatz, der **publizierte**
Full-Ergebnisordner und ein neuer, noch nicht existierender Stage-2-Ordner:

```bash
python stage2.py \
  /pfad/student_training_data.jsonl \
  /pfad/SUMO_Profile_Benchmark_full_20260822 \
  /pfad/SUMO_Profile_Benchmark_stage2_result
```

Der Full-Selector öffnet aus dem Full-Ordner ausschließlich `SUCCESS.json`,
`validation_selection.json`, `selected_model_artifacts.json`,
`config_resolved.json` und `split_manifest.json`. Ranking, Bericht,
`benchmark_result.json` und `test_predictions.npz` bleiben geschlossen. Erst
nach einem erneuten identischen Datenscan, ausschließlich Train/Validation-
basierter Auswahl und einem gehashten Prä-Calibrations-Freeze wird der externe
Marker im stabilen Ledgerstamm
`../.SUMO_Profile_Benchmark_stage2_calibration_ledgers/calibration-<SHA-256>/CALIBRATION_OPENED`
exklusiv angelegt. Seine Identität hängt von Full-ID, Protokoll-, Daten- und
Split-Hash ab, nicht von Ergebnisname oder Full-Pfad. Ein anderer Ausgabeordner
oder eine byteidentische Full-Kopie kann Calibration daher nicht erneut öffnen.
Ergebnis- und Ledgerordner werden nie überschrieben.

Die aktuell noch laufende/versteckte Full-Stagingkopie ist absichtlich kein
zulässiger Eingang. Für sie endet `stage2.py` mit
`NOT_A_PUBLISHED_FULL_DIRECTORY`, ohne einen Ergebnis- oder Ledgerordner zu
erzeugen.

### Grafische Oberfläche

```bash
python benchmark_gui.py
```

Die GUI bietet:

- Auswahl der JSONL-Datei und eines neuen Laufordners,
- unveränderte Schema-1-Presets `smoke`, `quick`, `standard`, `full` sowie
  additive Schema-2-Presets `extended_smoke`, `extended_quick`,
  `extended_standard`, `extended_full`,
- ein einzelnes Modell oder den gemeinsamen Lauf aller Modelle,
- vollständige editierbare Parameter als strikt validiertes JSON,
- nicht blockierende Ausführung in einem Worker-Thread,
- sicheren Abbruch an expliziten Prüfpunkten,
- Phasen-/Modellfortschritt, Laufprotokoll, Ranking, Parameter,
  automatische Interpretation und drei Ergebnisgrafiken.

Ein Abbruch wirkt kooperativ am nächsten sicheren Prüfpunkt; er beendet keinen
Dateischreibvorgang mitten in einer Veröffentlichung.

## Presets richtig verwenden

Jeder Lauf scannt die gesamte Eingabedatei und erstellt das vollständige
Daten-Audit. Die Splitgrenzen bestimmen, wie viele gültige Datensätze je Split
deterministisch für Training und Bewertung behalten werden.

| Preset | Train / Validation / Calibration / Test | Seeds | HPO-Kandidaten je Modell | Dispersionsanalyse | Zweck |
|---|---:|---:|---:|---|---|
| `smoke` | 128 / 48 / 48 / 64 | 1 | 1, HPO aus | aus | schneller Funktionsnachweis |
| `quick` | 2.000 / 500 / 500 / 500 | 1 | bis 2 | an | Entwicklung und Vorprüfung |
| `standard` | 12.000 / 3.000 / 2.000 / 3.000 | 3 | bis 3 | an | begrenzter Vergleichslauf |
| `full` | alle gültigen Datensätze des jeweiligen Splits | 3 | bis 2 | an | vollständiger Lauf auf dem aktuellen Datensatz |

Die vier `extended_*`-Presets besitzen dieselben Splitgrenzen, Seeds und
Suchbudgets wie ihr jeweiliges Schema-1-Gegenstück, führen aber alle sieben
Registrymodelle einschließlich `causal_residual_tcn`. Ein Wechsel auf Schema 2
ist damit explizit und nicht als stille Änderung eines alten Presets möglich.

Alle mitgelieferten Presets behalten Routenreplikate und verwenden
`route_target_policy = "route_mean"`. Der vollständige Parametersatz und der
begrenzte Suchraum stehen in `configs/*.json`.

Diese Entwicklungskopie enthält bewusst keinen `verified_smoke_run/` und keine
anderen Ergebnisordner. Der automatisierte Systemtest führt einen temporären
Schema-2-Einmodelllauf aus; er ist ein Funktionsnachweis und kein Ranking für
die Masterarbeit.

## Datenvertrag

Die Eingabe ist UTF-8-JSONL: eine JSON-Zeile pro Datensatz. Fünf Felder sind
verpflichtend:

| Feld | Typ und Bedeutung |
|---|---|
| `fahrzeug_id` | nicht leere Zeichenfolge oder ganze Zahl; Identifikation des Datensatzes |
| `route_edges` | nicht leere, geordnete Liste unveränderter Edge-ID-Zeichenfolgen |
| `edge_laengen_m` | nicht leere Liste endlicher, nicht negativer Längen in Metern; positionsgleich zu `route_edges` |
| `distanz_gesamt_m` | endliche, positive Gesamtdistanz in Metern |
| `v_profil_ms` | nicht leere Liste endlicher, nicht negativer Geschwindigkeiten in m/s |

Die Summe der Kantenlängen muss bis auf
`max(0,05 m; 10⁻⁶ × distanz_gesamt_m)` zur Gesamtdistanz passen. Leere,
syntaktisch fehlerhafte, doppelt belegte, nicht endliche oder außerhalb der
technischen Schutzgrenzen liegende Datensätze werden mit Grund gezählt und
ausgeschlossen. Der dokumentierte negative Sentinelwert wird **nicht** auf
eine Geschwindigkeit umgedeutet oder korrigiert. Höchstens drei Beispiele je
Fehlergrund erscheinen im Audit.

Die angenommenen Einheiten folgen den Feldnamen. Innerhalb plausibler Werte
kann die Anwendung eine falsch deklarierte Einheit ohne externe Metadaten
nicht erkennen.

## Ablauf des fairen Vergleichs

1. Die gesamte JSONL-Datei wird gestreamt, validiert und gehasht.
2. Der SHA-256-Hash der kanonischen geordneten Edgefolge ordnet jede Route
   deterministisch genau einem Split zu: 60 % Train, 15 % Validation,
   10 % Calibration und 15 % Test.
3. Ein Splitmanifest prüft paarweise, dass keine Routengruppe zwei Splits
   angehört. Ein Überlappungsbefund bricht den Lauf ab.
4. Bei `route_mean` erhält jede Route in Train und Validation ein gemitteltes
   Zielprofil. Die rohen Replikate bleiben für Audit, Dispersionsanalyse und
   routegewichtete Bewertung erhalten.
5. Skalierung, tabellarische Transformationen und das Edge-Vokabular werden
   ausschließlich aus dem Trainingssplit bestimmt. Fit und HPO erhalten eine
   technisch begrenzte Datenansicht mit Train und Validation; ihre Arrays sind
   schreibgeschützt.
6. Der begrenzte, vorab konfigurierte HPO-Suchraum wird nur auf
   Train/Validation ausgewertet. Der Calibration-Split bleibt reserviert und
   wird in der aktuellen Benchmarkversion nicht zur Modellauswahl verwendet.
7. Calibration- und Testmatrizen werden erst im ausdrücklich autorisierten
   Endauswertungspfad materialisiert. Nachdem alle Kandidaten feststehen, wird
   der Testsatz für alle Modelle einmal ausgewertet.
8. Ergebnisse werden zunächst in einem Staging-Ordner erzeugt und anschließend
   als vollständiger Laufordner veröffentlicht. Der Runner prüft Abbruchstatus
   und Zielpfad unmittelbar vor und erneut nach der Artefakt-Hashbildung;
   Zielpfade, die an diesen Prüfpunkten vorhanden sind, bleiben unangetastet.

Die ausführliche Definition steht in
[METHODIK_UND_GUELTIGKEIT.md](METHODIK_UND_GUELTIGKEIT.md).

## Ergebnisartefakte

Ein erfolgreicher Lauf enthält mindestens:

| Artefakt | Inhalt |
|---|---|
| `SUCCESS.json` | Abschlussmarker sowie SHA-256 und Größe aller übrigen Laufartefakte |
| `benchmark_result.json` | vollständige maschinenlesbare Ergebnisse, Metriken und Aussagegrenzen |
| `config_resolved.json` | tatsächlich verwendete Konfiguration |
| `data_audit.json` | Scan-, Ausschluss-, Wertebereichs- und Splitstatistik |
| `split_manifest.json` | eingefrorene Record-/Routenbelegung und Überlappungskontrolle |
| `validation_selection.json` | jeder HPO-Versuch und die deterministische Auswahlentscheidung |
| `selected_model_artifacts.json` | Spezifikationen und serialisierte Zustände der gewählten Modelle |
| `test_predictions.npz` | Referenz, Seed- und Ensembleprognosen ohne Pickle-Objektarrays |
| `model_ranking.csv` | kompakte Rang- und Metriktabelle |
| `SCIENTIFIC_REPORT.md` | automatisch erzeugter wissenschaftlicher Laufbericht |
| `model_ranking.png` | Rangvisualisierung |
| `learning_curves.png` | Trainings-/Validierungsverläufe, soweit vorhanden |
| `profile_examples.png` | Referenz- und Prognoseprofile ausgewählter Testdatensätze |
| `run_log.jsonl` | strukturiertes, phasenbezogenes Laufprotokoll |
| `trials/<modell>/*.json` | vollständige Konfiguration und Validierung jedes Kandidaten |

Wenn die Dispersionsanalyse aktiv ist, enthält der Lauf zusätzlich
`within_route_dispersion.json`. Bei aktiviertem Cache liegt
`SUMO_Profile_Dispersion_Cache_v1.json` im übergeordneten Ausgabeordner. Ein
fehlgeschlagener oder abgebrochener Staging-Ordner wird mit einem eindeutigen
`FAILED`- beziehungsweise `CANCELLED`-Suffix erhalten, damit Log und Ursache
prüfbar bleiben. Solche Diagnoseordner enthalten keinen `SUCCESS.json`-Marker.
Zwischen letzter Existenzprüfung und portablem `os.rename` bleibt ein sehr
kleines Konkurrenzfenster: Ein exakt dort parallel angelegtes **leeres**
Verzeichnis oder ein Symlink kann auf POSIX-Systemen nicht mit einer
plattformübergreifenden No-replace-Garantie ausgeschlossen werden. Deshalb
soll der laufbezogene Zielname während des Publikationsmoments nicht von einem
zweiten Prozess angelegt werden.

## Kennzahlen und Interpretation

Primärmetrik ist der **routegewichtete Profil-MAE** in m/s: Jede eindeutige
Route erhält dasselbe Gewicht, unabhängig von ihrer Zahl an Replikaten.
Zusätzlich werden Profil-RMSE, zentrierter Form-MAE, Gradienten-MAE auf dem
normierten Gitter, Stillstandsanteil sowie Präzision/Recall/F1 für
Stillstandspunkte und Stopp-/Anfahrübergänge ausgegeben. Der Schwellenwert der
Presets beträgt `v ≤ 0,5 m/s`.

Die beiden disjunkten Generalisierungsstrata lauten:

- neue Route, deren Edges sämtlich im Trainingsvokabular vorkommen;
- neue Route mit mindestens einer im Training unbekannten Edge.

Ein Stratum ohne Datensätze wird als leer ausgewiesen. Vergleiche sind stets
mit Zahl der Profile und Routen zu berichten.

Trainings- und Inferenzzeiten beschreiben ausschließlich die gemessenen
Modelloperationen in der protokollierten Laufumgebung. Es gibt im Benchmark
keine korrespondierende SUMO-Laufzeitmessung; aus den Zeiten folgt daher keine
Speed-up-Aussage.

## Geprüfter Stand

- [VERIFICATION_REPORT.md](VERIFICATION_REPORT.md) trennt automatisierte
  Prüfungen, den synthetischen Durchstich und noch erforderliche Vollergebnisse.
- [V2_DEVELOPMENT_NOTES.md](V2_DEVELOPMENT_NOTES.md) dokumentiert Architektur,
  Präfixkausalität, Schemaabgrenzung und offene wissenschaftliche Nachweise.
- [DISPERSION_RESULT.md](DISPERSION_RESULT.md) dokumentiert die geprüfte
  Vollbestands-Dispersionsauswertung des übernommenen Schema-1-Kontexts; sie
  wurde in dieser lauflosen Entwicklungskopie nicht neu erzeugt.
- [CHANGELOG.md](CHANGELOG.md) hält den aktuellen Funktions- und
  Dokumentationsstand fest.

Die Integrität des ausgelieferten Pakets wird aus dessen Wurzelverzeichnis
geprüft mit:

```bash
python3 verify_package.py
```

Das Skript vergleicht alle in `SHA256SUMS` aufgeführten Dateien mit ihren
SHA-256-Werten und lehnt zusätzliche, nicht manifestierte Dateien oder
Symlinks ab. Optional kann zugleich der unverändert extern verbleibende
Rohdatensatz gegen die dokumentierte Größe und SHA-256 geprüft werden:

```bash
python3 verify_package.py --jsonl /pfad/student_training_data.jsonl
```

## Tests

Im Codeverzeichnis:

```bash
python -B -W error -m unittest discover -v
```

Die Suite umfasst Datenvertrag, Split/Leakage, route-mean-Politik,
Modelltraining und -serialisierung, variable Sequenzen, Metriken,
Dispersionsanalyse, HPO-/Testreihenfolge, atomaren Export, CLI/GUI-Helfer,
Abbruch und Fehlerfälle. Die gezielte Suite unter `code/tests/` umfasst 156
Tests. Die vollständige Discovery ergänzt fünf Root-Regressionsprüfungen aus
`test_surrogate_pipeline.py`; damit sind 161/161 Tests bestanden. Davon sind
27 gezielte Stage-2-Prüfungen für CLI, Präregistrierung, Full-Selector, Runner,
Calibration-Ledger und Bootstrap. Die genaue Klassifikation und aktuelle
Ausführung sind im Stage-2-Prüfbericht dokumentiert.
