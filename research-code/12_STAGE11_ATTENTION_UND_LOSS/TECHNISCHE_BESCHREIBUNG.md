# Technische Kurzbeschreibung der Varianten

## 1. GRU-Referenz

Eine unidirektionale GRU liest die geordnete Kantenfolge und verdichtet sie in einen letzten internen Zustand. Daraus werden alle 48 Geschwindigkeitswerte gemeinsam berechnet. Diese Variante dient als fairer Referenzwert innerhalb derselben PyTorch-Umgebung.

## 2. BiGRU mit distanzbezogener Attention

Eine bidirektionale GRU verarbeitet die vollständig bekannte Route in beide Richtungen. Für jeden der 48 Profilpunkte fragt eine Attention gezielt die relevanten Kanten ab. Neben der inhaltlichen Ähnlichkeit berücksichtigt sie die relative Entfernung zwischen Kantenmittelpunkt und Profilpunkt. Die robuste Huber-Loss begrenzt den Einfluss einzelner großer Abweichungen.

## 3. Leichter Route-Transformer

Zwei Transformer-Schichten erfassen Abhängigkeiten zwischen allen Kanten. Kontinuierliche Distanzkodierungen erhalten die Reihenfolge und relative Lage. Die Ausgabe verwendet dieselbe distanzbezogene Attention wie Variante 2, damit der Architekturvergleich kontrolliert bleibt.

## 4. BiGRU-Attention mit Soft-DTW

Zur Huber-Loss kommt eine begrenzte Soft-DTW-Loss. Sie bewertet ähnlich geformte Profile weniger streng, wenn markante Stellen geringfügig verschoben sind. Der Versuch prüft, ob diese Formtoleranz die Profilprognose verbessert.

## 5. Kombinierte Multi-Task-Loss

Zusätzlich zu Huber und Soft-DTW werden Stopp- und Übergangspunkte als Nebenaufgaben gelernt. Focal-Loss reduziert die Dominanz der häufigen Nicht-Ereignisse; Tversky-Loss gewichtet verpasste seltene Ereignisse stärker. Die unveränderte Hauptbewertung erfolgt weiterhin direkt auf dem vorhergesagten Geschwindigkeitsprofil.
