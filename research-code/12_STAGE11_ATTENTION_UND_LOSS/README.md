# Stage 11 – Routen-Attention und kombinierte Verlustfunktion

Diese Versuchsstufe prüft zwei Verbesserungen des bisherigen Geschwindigkeitsprofilmodells:

1. Die komplette Route wird mit einer bidirektionalen GRU oder einem kleinen Transformer verarbeitet. Für jeden der 48 Profilpunkte gewichtet eine Attention die relevanten Straßenkanten abhängig von ihrer relativen Entfernung.
2. Neben dem robusten Punktfehler werden Soft-DTW für leicht verschobene Profilformen sowie Focal-/Tversky-Loss für seltene Stopp- und Übergangspunkte geprüft.

Die fünf Varianten werden mit denselben routenexklusiven Datenpartitionen und drei festen Zufallsstarts verglichen. Calibration und Test bleiben gesperrt; der Lauf ist ein Entwicklungsvergleich und keine neue finale Modellbestätigung.

## Ausführung

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
cd code
python stage11.py \
  /pfad/student_training_data.jsonl \
  /pfad/neuer_stage11_ergebnisordner
```

Der Ergebnisordner muss neu sein. Der vollständige Lauf erzeugt unter anderem `STAGE11_ERGEBNISBERICHT.md`, `STAGE11_VARIANTENVERGLEICH.csv`, Modellzustände, Vorhersagen, Daten- und Partitionsnachweise sowie `SUCCESS.json`.

## Aussagegrenze

Das Modell kennt weiterhin nur geordnete Edge-IDs und Kantenlängen. Verkehr, Lichtsignale, Tempolimits, Fahrzeugtyp und eine physische Zeitachse fehlen. Ein positiver Befund muss deshalb in einem neuen präregistrierten Voll-Lauf bestätigt werden.
