# Stage 8 – Training ohne Routenmittelung

Diese getrennte Variante vergleicht den eingefrorenen Stage-6-Stand
(`route_mean`) mit `raw_profiles`. Bei `raw_profiles` bleibt jedes einzelne
Trainingsprofil erhalten. Architektur, drei Seeds, Optimierung, Decoder,
Auswertungsraster und Auswahlgrenzen sind identisch.

Der Lauf ist explorativ und auf eine bisher ungenutzte Train-Reserve begrenzt.
Validation, Calibration und Test wurden nur auf Integrität geprüft und nicht
für Training, Auswahl oder Metriken verwendet. Es wurde kein finales Modell
ersetzt.

Start:

```bash
./RUN_STAGE8_NO_ROUTE_MEAN.command /pfad/student_training_data.jsonl
```

Verbindliche Bewertung: `../Berichte/STAGE8_OHNE_ROUTENMITTELUNG_BERICHT.md`.
