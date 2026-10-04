# Technische Kurzbeschreibung

## Kinematisches Grundmodell

Aus den Trainingsrouten wird für jede bekannte Straßenkante eine mittlere Geschwindigkeit bestimmt. Bei seltenen Kanten wird dieser Wert zur globalen mittleren Geschwindigkeit hin stabilisiert. Die Kantenlängen bilden daraus ein Geschwindigkeitsprofil entlang der Route. Ein vorwärts und rückwärts gerechneter Grenzwert auf Basis von \(v^2\) begrenzt zu starke Beschleunigungs- und Bremsänderungen.

## Neuronales Residualmodell

Eine bidirektionale GRU liest die Straßenkanten in beiden Richtungen. Für jeden der 48 Ausgabepunkte wählt eine distanzbezogene Attention die relevanten Kanten aus. Das Netz prognostiziert nicht die gesamte Geschwindigkeit, sondern nur die Korrektur zum kinematischen Grundprofil. Der Ausgabekopf startet bei null; das Training beginnt daher exakt beim Grundmodell.

## Verlustfunktion und Vergleich

Die Hauptkomponente ist eine robuste Huber-Loss des Geschwindigkeitsprofils. Zwei Versuchsvarianten ergänzen Strafen für überschrittene Beschleunigungs- und Bremsgrenzen sowie für große Residuen. Verglichen werden Gewichte 0, 0,02 und 0,10 mit je drei Startwerten. Eine regularisierte Variante darf höchstens 0,05 m/s MAE verlieren und muss die Grenzverletzungen im Auswahlteil strikt reduzieren.

## Befund

`physics_residual_w000` wird ausgewählt. Das physikgestützte Grundprofil plus gelernte Korrektur verbessert den Holdout-MAE um 0,0422 m/s beziehungsweise 0,92 % gegenüber Stage 11. Die Summe der Proxy-Grenzverletzungen sinkt um 11,75 %. Die zusätzlichen Loss-Strafen liefern keinen robusten Mehrwert und bleiben deaktiviert.

## Einschränkung

Die 48 Profilpunkte besitzen keine reale Zeit- oder Wegkoordinate. Die verwendeten Beschleunigungswerte sind deshalb keine direkt messbaren \(m/s^2\), sondern kinematische Proxy-Werte. Für belastbare physikalische Aussagen sind insbesondere die originalen Profilkoordinaten und die SUMO-Netz-, Fahrzeug- und Verkehrsdaten erforderlich.
