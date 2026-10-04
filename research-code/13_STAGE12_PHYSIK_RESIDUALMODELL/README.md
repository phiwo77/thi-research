# Stage 12 – Physikgestütztes Residualmodell

Das Modell zerlegt die Prognose in zwei Teile:

1. Ein kinematisches Grundmodell schätzt aus den Trainingsdaten eine typische Geschwindigkeit je Straßenkante. Kantenlängen ordnen diese Werte entlang der Route an. Vorwärts- und Rückwärtsgrenzen begrenzen zu schnelle Geschwindigkeitsänderungen.
2. Eine bidirektionale GRU mit distanzbezogener Attention lernt nur die verbleibende Abweichung zwischen Grundmodell und gemessenem 48-Punkte-Profil.

Getestet werden ein Residualmodell ohne zusätzliche Physikstrafe sowie zwei Stärken für Physik- und Residualregularisierung. Jede Variante wird mit drei festen Zufallsstarts trainiert. Die Modellwahl verwendet ausschließlich den vorab festgelegten Auswahlteil der Validierungsdaten; Calibration und Test bleiben gesperrt.

## Ergebnis des vorliegenden Laufs

Die ausgewählte Variante `physics_residual_w000` verbessert den Holdout-MAE gegenüber Stage 11 von 4,5945 auf 4,5524 m/s. Gleichzeitig sinkt die Summe der Beschleunigungs- und Verzögerungsüberschreitungen von 0,00563 auf 0,00497. Die zusätzlichen Physikstrafen erfüllen auf dem Auswahlteil die vorab festgelegte Bedingung einer strikt geringeren Verletzungsrate nicht und werden daher nicht ausgewählt.

Das Modell bleibt physikgestützt, weil die neuronale Prognose als Korrektur eines kinematisch begrenzten Grundprofils formuliert ist. `w000` bedeutet nur, dass keine zusätzliche Physikstrafe im Loss aktiv ist.

## Ausführung

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
./RUN_STAGE12.command \
  /pfad/student_training_data.jsonl \
  /pfad/stage11_ergebnisordner \
  /pfad/neuer_stage12_ergebnisordner
```

Der Zielordner muss neu sein. Die Integrität eines erfolgreichen Laufs wird anschließend geprüft mit:

```bash
python verify_stage12.py /pfad/stage12_ergebnisordner
```

## Wissenschaftliche Aussagegrenze

Vorhanden sind geordnete Edge-IDs, Kantenlängen, Gesamtstrecke und ein auf 48 normierte Punkte umgerechnetes Geschwindigkeitsprofil. Es fehlt die reale Weg- oder Zeitkoordinate jedes Profilpunkts. Beschleunigung und Verzögerung sind daher nur Proxy-Größen auf einer angenommenen äquidistanten Streckenachse.

Für ein physikalisch identifizierbares Modell werden zusätzlich Tempolimits, Spurzahl, Steigung, Kurvenradius, Lichtsignale und Vorfahrtsregeln, Verkehrsnachfrage, Fahrzeugparameter, Netzversion sowie SUMO-Szenario und Seed benötigt. Der aktuelle Befund ist explorativ und sollte in einem präregistrierten Voll-Lauf bestätigt werden.
