# Stage 3 – Event-Aware Stop Decoder

Dieser Softwarestand ist vom bestätigten Stage-2-Code getrennt. Er verändert weder Trainingsdaten, Splitlogik, Zielvektor noch eingefrorene Modellgewichte.

## Methode

1. Die über drei Seeds gemittelte verbesserte GRU liefert das Geschwindigkeitsprofil.
2. Das eingefrorene Causal-Residual-TCN liefert über drei Seeds gemittelte Stoppwahrscheinlichkeiten.
3. Ein deterministischer Decoder setzt Profilpunkte oberhalb einer gewählten Stoppwahrscheinlichkeit auf 0 m/s.
4. Schwellenwert und Mindestlänge eines Stopplaufs werden nur in einer routengruppierten Hälfte des Validation-Splits gewählt.
5. Die zweite Validation-Hälfte bleibt bis zur Auswahl unangetastet und dient als explorativer Holdout.
6. Calibration und Test sind im Protokoll ausdrücklich verboten.

Die fest eingefrorene Konfiguration liegt in `configs/stage3_event_decoder_v1.json`.

## Ausführung

```bash
python3 -B -W error code/stage3_event_decoder.py \
  /pfad/student_training_data.jsonl \
  /pfad/stage2-ergebnis \
  /pfad/neues-stage3-ergebnis \
  --json-summary
```

Das Ausgabeziel darf noch nicht existieren. Der Runner prüft vor der Inferenz alle Stage-2-Artefakthashes, die Modellzustände, den Rohdatenhash und das vollständige Splitmanifest.

## Aussagegrenze

Der Code ist explorativ. Der Validation-Holdout mindert unmittelbares Schwellentuning-Overfitting, ist aber kein vollständig unabhängiger Bestätigungssplit, weil die Upstream-Modelle zuvor mit Validation ausgewählt wurden.

