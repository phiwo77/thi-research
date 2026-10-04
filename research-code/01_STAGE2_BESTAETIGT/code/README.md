# Ausführbarer Code

Die vollständige deutschsprachige Nutzungshilfe steht in
[../README.md](../README.md). Methodik, Metriken und wissenschaftliche
Aussagegrenzen sind in
[../METHODIK_UND_GUELTIGKEIT.md](../METHODIK_UND_GUELTIGKEIT.md) beschrieben.

## Installation

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

## Start

Kommandozeile:

```bash
python benchmark.py /pfad/student_training_data.jsonl /pfad/neuer_lauf --preset quick
```

Grafische Oberfläche:

```bash
python benchmark_gui.py
```

Unter macOS kann alternativ im übergeordneten Ordner
`START_BENCHMARK_GUI.command` per Doppelklick verwendet werden.

Als unveränderte Schema-1-Presets stehen `smoke`, `quick`, `standard` und
`full` zur Verfügung. Schema 2 ergänzt `extended_smoke`, `extended_quick`,
`extended_standard` und `extended_full` mit dem neuen Modell
`causal_residual_tcn`. Der
Ausgabeordner muss neu sein; vorhandene Ergebnisse werden nicht überschrieben.
Der Smoke-Lauf prüft ausschließlich die Funktion und liefert kein
wissenschaftlich belastbares Modellranking.

## Zentrale Module

| Datei | Aufgabe |
|---|---|
| `benchmark.py` | CLI-Einstieg |
| `benchmark_gui.py` | nicht blockierende Tk-GUI |
| `benchmark_config.py` | striktes Konfigurationsschema und Presets |
| `benchmark_data.py` | route-mean, Splitmanifest und gemeinsame Datenansicht |
| `benchmark_models.py` | einheitliche Registry und Modellzustände |
| `causal_tcn_model.py` | präfixkausales Residual-TCN mit v2-JSON-Zustand und Event-Heads |
| `benchmark_runner.py` | HPO, versiegelte Testauswertung und atomarer Laufexport |
| `benchmark_metrics.py` | Profil-, Stopp-, Übergangs-, OOV- und Proxy-Metriken |
| `benchmark_interpretation.py` | deterministische Interpretation mit Pflichtwarnungen |
| `benchmark_dispersion.py` | streamingbasierter Conditional-Dispersion-Proxy |
| `benchmark_reporting.py` | Tabellen, Grafiken und wissenschaftlicher Laufbericht |
| `benchmark_errors.py` | Fehlervertrag mit Code, Phase, Ursache und Lösung |
| `benchmark_io.py` | atomare Dateien, kanonische Hashes und strukturiertes Log |

`surrogate_pipeline.py`, `road_load.py`, `sequence_model.py`, `gru_model.py`
und `input_validation.py` stellen wiederverwendete, getestete Komponenten bereit.
Für den fairen Schema-1- oder Schema-2-Vergleich sind `benchmark.py`
beziehungsweise `benchmark_gui.py` die Einstiegspunkte.

Stage 2 ist additiv und verwendet ausschließlich folgende neue Module:

| Datei | Aufgabe |
|---|---|
| `stage2.py` | separater CLI-Einstieg; startet niemals den Full-Benchmark |
| `stage2_config.py` | unveränderlicher, strikt validierter Präregistrierungsvertrag |
| `stage2_full_selector.py` | Allowlist-Lader für genau fünf publizierte Full-Artefakte |
| `stage2_runner.py` | Train/Validation-Auswahl, rehydrierter Rollen-Freeze, stabiler Ledger- und Publikationsvertrag |
| `stage2_calibration.py` | gebundener exklusiver Calibration-Marker, gemeinsame Auswertung und Bootstrap |

Der Stage-2-Runner stoppt, solange der erwartete Lauf
`full-1808fe9ef9bb` nicht mit einem gültigen `SUCCESS.json` veröffentlicht
ist. Er materialisiert oder bewertet nie die Testmatrix und verwendet für
zusätzliches Training nur `SelectionData` mit Train und Validation. Der
Ledger-Schlüssel ist unabhängig
vom gewählten Ergebnis- und Full-Pfad; ein zweiter Name öffnet dieselbe
Präregistrierung nicht erneut.

## Tests

```bash
PYTHONDONTWRITEBYTECODE=1 python -B -W error -m unittest discover -v
```

Der geprüfte Stand und die Einordnung der vorhandenen Ergebnisartefakte stehen
in [../STAGE2_VERIFICATION_REPORT.md](../STAGE2_VERIFICATION_REPORT.md).
