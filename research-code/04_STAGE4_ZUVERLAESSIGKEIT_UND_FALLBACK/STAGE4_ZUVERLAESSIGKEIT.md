# Stage 4 – Zuverlässigkeitsintervalle und SUMO-Fallback

Dieser Softwarestand verändert den eingefrorenen Stage-3-Prädiktor nicht. Er ergänzt eine getrennte Zuverlässigkeitsschicht.

## Funktionsweise

- verwendet ausschließlich die bisherige Stage-3-Holdout-Hälfte;
- teilt deren Routen deterministisch in Conformal-Calibration, Methodenselektion und finalen Holdout;
- bildet ein gemeinsames Intervall für alle 48 Profilpunkte und alle Wiederholungen einer Route;
- vergleicht fünf vorab definierte Unsicherheitsskalen;
- akzeptiert in v2 nur Methoden mit mindestens 90 % Routenabdeckung und tatsächlich variierender Unsicherheit;
- sortiert Routen nach Intervallbreite und weist die übrigen Fälle explizit SUMO zu.

Calibration- und Testsplit des ursprünglichen Benchmarks bleiben unberührt. Auch die frühere Stage-3-Tuninghälfte wird nicht verwendet.

## Ausführung

```bash
python3 -B -W error code/stage4_reliability.py \
  /pfad/stage3_validation_20260826 \
  /pfad/neues_stage4_ergebnis \
  --protocol configs/stage4_reliability_gate_v2.json \
  --json-summary
```

## Versionen

- `Code_v1_Diagnostik`: erster konservativer Lauf; ehrliche Abdeckung, aber konstante und damit nicht diskriminierende Unsicherheit.
- `Code`: verschärfter v2-Stand; konstante Skalen sind als Fallback-Ranking unzulässig, und die selektive Risikowirkung wird separat geprüft.

Stage 4 ist weiterhin explorativ. Ein positives empirisches Holdout-Ergebnis ersetzt keine externe Bestätigung.

