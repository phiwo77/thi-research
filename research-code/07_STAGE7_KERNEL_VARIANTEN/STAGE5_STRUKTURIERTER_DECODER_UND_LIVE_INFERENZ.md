# Stage 5: strukturierter Ereignisdecoder und abgesicherte Live-Inferenz

## Ergebnis in einem Satz

Der vorab definierte strukturierte Decoder wird **nicht übernommen**, weil
keine der 36 Varianten alle Tuning-Grenzen erfüllt. Die neue Live-Komponente
verkettet deshalb ausschließlich die bereits akzeptierten Stage-2/3/4-Stände
und erzwingt bei unbekannten oder empirisch zu unsicheren Routen einen
SUMO-Fallback.

## Wissenschaftlicher Vertrag

- Eingefrorene Daten-, Split-, Ziel- und Bewertungsverträge wurden nicht
  verändert.
- Verwendet wurde ausschließlich das bestehende Stage-3-Validation-Archiv.
- Calibration, Test und Stage-4-Holdout-Ziele blieben geschlossen.
- Suchraum, Auswahlgrenzen und Tie-Break wurden vor dem einmaligen Lauf in
  `configs/stage5_structured_decoder_v1.json` festgelegt.
- Ein negativer Lauf wird nicht durch nachträgliche Schwellenänderungen
  „gerettet“.

## Untersuchte Anpassung

`structured_event_decoder.py` implementiert einen deterministischen
Zwei-Zustands-Viterbi-Decoder. Er kombiniert eingefrorene Stopp- und
Übergangswahrscheinlichkeiten und kann eine Mindestdauer von Stopps erzwingen.
Der feste Suchraum umfasst 36 Kombinationen aus Stopp-Bias, Übergangs-Bias,
Übergangsgewicht und Mindeststoppdauer.

Die Auswahl verlangte gleichzeitig:

1. positiven kombinierten Ereignis-F1-Delta,
2. positiven Stoppübergangs-F1-Delta,
3. positiven Anfahrübergangs-F1-Delta,
4. höchstens 0,005 Verlust beim Stopp-F1,
5. höchstens 0,05 m/s Anstieg des Profil-MAE.

## Ergebnis

Keine Variante erfüllte alle fünf Grenzen. Die beste Ereignisvariante
(`stop_logit_bias=1.2`, `transition_logit_bias=1.0`,
`transition_weight=0.25`, `minimum_stop_run_points=1`) erreichte auf dem
Tuning-Teil:

| Kennzahl | Bisherige Stage 3 | Strukturierter Decoder | Delta |
|---|---:|---:|---:|
| Ereignis-F1-Makro | 0,173269 | 0,196397 | +0,023128 |
| Stopp-F1 | 0,297661 | 0,362515 | +0,064854 |
| Stoppübergangs-F1 | 0,113967 | 0,115793 | +0,001826 |
| Anfahrübergangs-F1 | 0,108178 | 0,110883 | +0,002705 |
| Profil-MAE | 4,221627 m/s | 4,373558 m/s | **+0,151931 m/s** |

Damit wurden vier von fünf Grenzen erfüllt; der MAE-Anstieg war rund dreimal
größer als erlaubt. Der Stage-5-Holdout wurde für keinen Kandidaten geöffnet.
Der maschinenlesbare Beschluss lautet `retain_stage3_decoder`.

## Ursache und Aussagegrenze

Die bestehende Stage-3-Ausgabe erkennt auf dem Holdout Stopps in nur 40,43 %
der Profile, während die Referenz in 92,05 % der Profile mindestens einen
Stopp enthält. Sie erzeugt 6.299 statt 47.347 Stoppsegmente und sehr lange
zusammenhängende Stopps (17,81 statt 2,95 Rasterpunkte im Mittel). Das Problem
ist daher nicht nur eine lokal falsche Zustandsfolge. Die Event-Heads liefern
zu wenige getrennte Ereignisse; ein nachgelagerter Decoder kann fehlende
Information nicht zuverlässig rekonstruieren.

## Live-Komponente

`code/hybrid_inference.py` stellt exakt diese Kette her:

1. drei bestätigte GRU-Seeds für das Regressionsensemble,
2. drei Causal-Residual-TCN-Seeds für Stopp- und Übergangswahrscheinlichkeiten,
3. den akzeptierten Stage-3-Hard-Decoder,
4. das Stage-4-Konformalband und den empirischen 50-%-Routen-Gate,
5. expliziten `FALLBACK_SUMO` bei unbekannten Kanten, zu langen Routen,
   ungültigen Ausgaben oder zu breiten Intervallen.

Vor der ersten Anfrage werden alle SUCCESS-Artefakte, Bytegrößen, SHA-256-
Hashes, Modellzustände, Seeds und Upstream-Verweise geprüft. Der abgelehnte
Stage-5-Decoder wird durch einen zusätzlichen Negativbefund-Check technisch
vom Live-Pfad ausgeschlossen.

Beispiel:

```bash
python code/hybrid_inference.py \
  --package-root /pfad/Weiterentwicklung_StopStart_2026-08-26 \
  --input examples/stage5_hybrid_smoke_routes.jsonl \
  --output /pfad/ergebnis.jsonl
```

Die 50-%-Rate ist eine **selektive** empirische Betriebsentscheidung: Im
Stage-4-Holdout betrug der zugehörige Grenzwert 28,878692 m/s maximale
Intervallhalbbreite und der Profil-MAE der akzeptierten Teilmenge
3,951414 m/s. Dies ist keine Garantie für neue Verteilungen.

## Verbindliche Schlussfolgerung

Die Live-Verkettung ist technisch zuverlässiger als eine ungesicherte
Einzelprognose. Sie ist dennoch **kein allgemeiner Simulationsersatz**. Nur
bekannte, innerhalb des empirischen Gates liegende Routen dürfen selektiv als
Surrogat ausgewiesen werden; für alle anderen Fälle bleibt SUMO verbindlich.

