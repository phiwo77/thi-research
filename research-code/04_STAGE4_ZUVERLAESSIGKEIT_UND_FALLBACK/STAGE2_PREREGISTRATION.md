# Präregistrierung der Stage-2-Validierungs- und Calibration-Studie

**Schema:** 1  
**Stand:** 22. August 2026  
**Ausführungsstatus:** technisch implementiert, noch nicht wissenschaftlich ausgeführt

Die maschinenlesbare und maßgebliche Fassung ist
`configs/stage2_preregistered.json`. Der Loader akzeptiert nur deren exakten
kanonischen Inhalt; Laufzeitüberschreibungen des Suchraums, der Seeds oder der
Erfolgsregel sind nicht vorgesehen.

## Eingefrorener Full-Ursprung

- Run-ID: `full-1808fe9ef9bb`
- Konfigurations-SHA-256:
  `bbab8894aee731f53ade317ab0febe1c26aca587e3d344b73e1a519ac8112310`
- Datensatz-SHA-256:
  `52ba5cde12ec102dfbd3b02145aef15fe718fe0f904de2b2753bb8015f87c1b5`
- Splitmanifest-SHA-256:
  `c213b29a1028d7a767c7495ec3243a91ea8f1075f7b3c22d8fc26ef3e345b3fc`
- Seeds: `20260729`, `20260730`, `20260731`
- Full-Modelle: globaler Mittelwert, Ridge, Gradient Boosting, MLP, GRU und
  bestehendes nicht-residuales TCN
- erwartete Profile/Routen: Train 99.055/66.270, Validation 24.680/16.475,
  Calibration 17.071/11.323, Test 24.810/16.547

Der Full-Selector liest ausschließlich:

1. `SUCCESS.json`
2. `validation_selection.json`
3. `selected_model_artifacts.json`
4. `config_resolved.json`
5. `split_manifest.json`

Er öffnet insbesondere weder `benchmark_result.json`, Rankings,
`test_predictions.npz`, Profilgrafiken noch den wissenschaftlichen Bericht.
Fehlt das veröffentlichte `SUCCESS.json`, endet Stage 2 vor jeder weiteren
Aktion.

## Auswahl vor Calibration

Der Full-Validation-Sieger ist das Minimum aus Validation-Mittelwert,
Seed-Standardabweichung und Modellname. Innerhalb eines Suchraums gelten
Validation-Mittelwert, Seed-Standardabweichung und kanonischer Parameterhash.
Calibration und Test sind an dieser Stelle geschlossen.

Die gezielte Verbesserung ist ausschließlich für einen GRU-Sieger
präregistriert. Sie übernimmt `stop_weight` und `transition_weight` des
Full-Siegers und prüft über alle drei Seeds:

| Kandidat | Embedding | Hidden | Epochen | Batch | Lernrate | Patience |
|---|---:|---:|---:|---:|---:|---:|
| 1 | 12 | 48 | 32 | 64 | 0,0012 | 8 |
| 2 | 16 | 64 | 36 | 64 | 0,0008 | 9 |

Zusätzlich werden die beiden unveränderten `causal_residual_tcn`-Kandidaten
aus `configs/extended_full.json` über dieselben drei Seeds ausschließlich auf
Train/Validation ausgewählt. Gewinnt im Full-Lauf kein GRU-Modell, blockiert
Stage 2 fail-closed; es wird kein nachträglicher Suchraum erfunden.

Vor Calibration werden drei Rollen mit allen ausführbaren JSON-Zuständen und
Hashes eingefroren:

- `validation_winner_baseline`: exakte Full-Siegerzustände;
- `validation_winner_improved`: Validation-selektierte GRU-Verbesserung;
- `causal_residual_tcn`: Validation-selektierter Residual-TCN-Kandidat.

Alle sechs Full-Modellfamilien werden zusätzlich unverändert für einen rein
deskriptiven Calibration-Vergleich eingefroren.

## Einmalige Calibration-Auswertung

Nach dem gehashten Prä-Calibrations-Freeze wird der persistente Marker unter
`../.SUMO_Profile_Benchmark_stage2_calibration_ledgers/calibration-<SHA-256>/CALIBRATION_OPENED`
mit `O_CREAT|O_EXCL` angelegt. Der Verzeichnisschlüssel ist ausschließlich an
Full-ID, Protokoll-, Daten- und Split-Hash gebunden; Ausgabe- und Full-Pfad
gehören ausdrücklich nicht zur Identität. Markerdatei, Ledgerverzeichnis und
dessen Verzeichniseinträge werden vor dem Datenzugriff dauerhaft synchronisiert.
Der Markerinhalt bindet zusätzlich den konkreten Stage-2-Lauf und den
Prä-Calibrations-Freeze. Erst danach wird exakt einmal
`materialize_sealed_evaluation_matrix(data, "calibration")` aufgerufen. Ein
vorhandener Ledger oder Ergebnisordner wird nicht überschrieben. Ein Fehler
nach Markeranlage erlaubt keine stille Wiederholung; ein anderer Ergebnisname,
eine umbenannte Full-Kopie oder ein neuer Prozess ändern dieses Gate nicht.

Jeder Seed durchläuft zuerst die explizite Nichtnegativprojektion der
Geschwindigkeiten; erst danach wird das Ensemble gemittelt. Alle drei Rollen
und alle sechs Full-Modelle verwenden dieselben Calibration-Profile in
derselben Reihenfolge. Nur Baseline gegen gezielte Verbesserung ist
inferenzstatistisch. Alle übrigen Vergleiche sind deskriptiv und dürfen keine
nachträgliche Modellwahl auslösen.

## Gepaarter Routenbootstrap

- Zielgröße je Profil: MAE über exakt 48 Geschwindigkeitswerte;
- Cluster: Routenschlüssel, Replikate innerhalb der Route gemeinsam;
- Routenbeitrag: `MAE_Baseline − MAE_Verbesserung`;
- Routen lexikografisch sortiert, Berechnung in `float64`;
- 10.000 Ziehungen mit `numpy.random.Generator(PCG64(20260822))`;
- speicherschonende feste Batches, identische RNG-Ziehfolge;
- 95-%-Intervall mit linearen Quantilen 2,5 % und 97,5 %;
- Konsistenzcheck zur Differenz der gemeinsamen routegewichteten MAE auf zwölf
  Dezimalstellen;
- Erfolg genau dann, wenn die rohe untere Intervallgrenze strikt größer null
  ist und alle drei korrespondierenden Seed-Differenzen strikt positiv sind.

Der Testsatz bleibt in Stage 2 geschlossen. Die Studie belegt weder
SUMO-Gleichwertigkeit noch einen End-to-End-Speed-up oder Verkehrssensitivität.

Die Split-Kapselung ist eine geprüfte API-Grenze innerhalb eines lokalen
Prozesses, keine kryptografische oder betriebssystemseitige Isolation: Der
vollständige Audit-Scan erzeugt `PreparedData` mit Record-Objekten aller
Splits. Training erhält ausschließlich die verengte `SelectionData`-Ansicht
mit Train und Validation; nur die Calibration-Matrix wird nach dem Ledger-Gate
materialisiert. Kein Stage-2-Codepfad materialisiert oder bewertet die
Testmatrix. Für einen Schutz gegen absichtlich manipulierten Prozesscode wäre
eine getrennte, extern verwaltete Ausführungsumgebung erforderlich.
