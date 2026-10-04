# Stage 7 – kontrollierte Kernelvariation

## Zweck

Stage 7 prüft, ob ein kürzeres oder längeres kausales Sichtfeld entlang der
Edge-Sequenz die Stopp-/Anfahrerkennung des Causal-Residual-TCN verbessert. Der bestehende Kernel-5-
Referenzstand wird unverändert aus Stage 6 geladen; neu trainiert werden nur
Kernel 3, 7 und 9.

## Unveränderter Vertrag

- Modell: `causal_residual_tcn`
- Kanäle: 24; Embedding: 8
- Epochen: 12; Batch: 64; Lernrate: 0,0015; Patience: 5
- identische Verlustgewichte, Unknown-Dropout, Seeds und Stage-3-Decoder
- nur `kernel_size` wird verändert
- keine Verwendung von Validation, Calibration oder Test für Training,
  Auswahl oder Bewertung
- keine Nachoptimierung anhand des Reserve-Holdouts

Die Kernel 3/5/7/9 besitzen nominelle Sichtfelder von 29/57/85/113
Edge-Sequenzpositionen. Diese Größe ist nicht mit den 48 Stützstellen des
ausgegebenen Geschwindigkeitsprofils gleichzusetzen. Wie viel reales Netz ein
Sichtfeld umfasst, hängt von Zahl und Länge der Edges einer Route ab; die
kausale Randauffüllung bleibt Teil der unveränderten Architektur.

## Frische Bewertungsreserve

Der vollständige 1,9-GB-Quelldatensatz wird geprüft und gehasht. Die ersten
6.000 deterministisch gezogenen Train-Datensätze rekonstruieren exakt die
Stage-6-Basis. Aus den nächsten 3.000 Train-Datensätzen werden sämtliche
Routen entfernt, die bereits in der Basis vorkommen. Die verbleibende Reserve
wird deterministisch und routenexklusiv in Auswahl und einmaliges Holdout
geteilt. Damit werden die in Stage 6 bereits offengelegten Validierungswerte
nicht erneut zur Auswahl benutzt.

## Ausführung

```text
./RUN_STAGE7_KERNEL_VARIANTS.command /pfad/student_training_data.jsonl
```

Ein bestehender Ergebnisordner wird niemals überschrieben. Jeder Lauf enthält
`SUCCESS.json`, Artefakthashes, Protokoll, Daten- und Partitionsaudit,
Vorhersagen sowie drei Modellzustände pro neuer Variante.

## Aussagegrenze

Stage 7 ist eine kontrollierte reduzierte Sensitivitätsanalyse. Sie ersetzt
weder einen präregistrierten Full-Scale-Lauf noch die versiegelte finale
Testauswertung. Ein positiver Befund wäre nur ein Kandidat für eine separate
Bestätigung, keine automatische Modellfreigabe.
