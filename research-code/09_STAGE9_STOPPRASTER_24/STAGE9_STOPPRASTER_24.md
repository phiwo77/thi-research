# Stage 9 – 24 Stoppwahrscheinlichkeiten

Getrennte Sensitivitätsanalyse: nur der Event-Kopf wird von 48 auf 24
Stützstellen geändert. Das Regressionsprofil und alle Bewertungsmetriken
bleiben auf 48 Punkten. Eventwahrscheinlichkeiten werden vor Decoder und
Bewertung linear auf 48/47 Punkte projiziert.

```bash
./RUN_STAGE9_EVENT_GRID_24.command /pfad/student_training_data.jsonl
```

Verbindliche Bewertung: `../Berichte/STAGE9_STOPPRASTER_24_BERICHT.md`.
