# Projekt 2 – Gewächshaus: Temperatursignal (Live Zero) und Schwellen

Stand: 24.11.2026 · Hugo Collienne

## Idee: Live Zero

Das simulierte Temperatursignal (HW-140) liefert 0,5 V bei −10 °C und 10 V bei +50 °C. Es beginnt bewusst nicht bei 0 V.

Misst die LOGO! fast 0 V, kann das keine gültige Temperatur sein. Dann ist die Leitung unterbrochen oder der Sensor ausgefallen, und die Anlage geht in Störung.

Das Prinzip heißt Live Zero und wird in der Industrie z. B. mit 4–20 mA statt 0–20 mA verwendet.

## Umrechnung Temperatur und Spannung

1 °C entspricht 0,158 V: Der Messbereich von −10 bis +50 °C (60 °C) verteilt sich auf 9,5 V (10 V minus 0,5 V).

```math
U = 0{,}5\,\mathrm{V} + (T + 10\,^\circ\mathrm{C}) \cdot 0{,}1583\,\mathrm{V}/^\circ\mathrm{C}
```

Umgekehrt: T = (U − 0,5 V) × 6,316 °C/V − 10 °C.

| Temperatur (°C) | Spannung am HW-140 (V) |
| --- | --- |
| −10 | 0,50 |
| 0 | 2,08 |
| 10 | 3,67 |
| 18 | 4,93 |
| 20 | 5,25 |
| 22 | 5,57 |
| 24 | 5,88 |
| 25 | 6,04 |
| 30 | 6,83 |
| 35 | 7,63 |
| 50 | 10,00 |

## Vorschlag: Sollwert 22 °C mit Totzone ±2 K

Die Anlage heizt unter 20 °C (5,25 V), kühlt über 24 °C (5,88 V) und schaltet dazwischen alles aus. Die Zahlen sind ein Vorschlag und lassen sich ändern.

Die Totzone von 4 K zwischen 20 und 24 °C verhindert, dass Relais und Motoren ständig ein- und ausschalten, wenn die Temperatur um den Sollwert schwankt.

| Zustand | Temperatur | Spannung (V) | Lüfter 1 (Heizungslüfter) | Lüfter 2 (Vakuumlüfter) | Heizung (rote Leuchte) |
| --- | --- | --- | --- | --- | --- |
| Zu kalt | unter 20 °C | unter 5,25 | an | aus | an |
| Sollbereich | 20–24 °C | 5,25–5,88 | aus | aus | aus |
| Zu warm | über 24 °C | über 5,88 | an | an | aus |

Die Heizung leuchtet nur, wenn Lüfter 1 läuft.

## Fühlerbruch: unter 0,3 V gilt als Störung

Fällt das Signal unter 0,3 V (rechnerisch etwa −11,3 °C, also außerhalb des Messbereichs), erkennt die LOGO! einen Fühlerbruch oder Leitungsbruch.

1. Alle Ausgänge gehen aus (Lüfter, Heizungsleuchte, Pumpe).
2. Die TDE zeigt eine Störmeldung.
3. Die Störung wird mit dem echten Taster quittiert, erst danach ist ein Neustart möglich.

Intern rechnet die LOGO! den Bereich 0–10 V als 0–1000 (im LOGO!-Handbuch zu prüfen). Dann entspricht die Schwelle 0,3 V dem Wert 30.

## Offene Fragen an den Lehrer

- [ ] Darf der HW-140 als „Temperatursimulation“ im Stromlaufplan und in der Stückliste stehen?
- [ ] Zählen die TDE-Funktionstasten zur Grenze von 12 Eingängen?
- [ ] Reicht die Heiz- und Kühlregelung mit Totzone als Schwierigkeitsgrad für den Bonus?
- [ ] Ist die rote Leuchte für die Heizung in Ordnung, wenn die Störung gelb gemeldet wird?

## Funktionsbeschreibung (Entwurf)

Die Anlage regelt Temperatur und Bewässerung eines Modell-Gewächshauses. Sie wird von einer Siemens LOGO! 12/24RCE mit Erweiterungsmodul DM8 und TDE-Display gesteuert.

**Betriebsarten (Wahlschalter, 3 Stellungen):**

- Aus: Alle Ausgänge sind aus.
- Manuell: Lüfter 1, Lüfter 2 und Pumpe werden einzeln über die TDE bedient. Die Verriegelung bleibt aktiv, die Heizung darf nur bei laufendem Lüfter 1 leuchten.
- Auto: Die Anlage regelt selbst.

**Temperaturregelung (Auto):** Das Temperatursignal wird mit einem HW-140 simuliert (0,5–10 V entsprechen −10 bis +50 °C). Der Sollwert ist 22 °C, die Totzone ±2 K.

- Unter 20 °C: Die rote Leuchte (Heizung) und Lüfter 1 sind an, Lüfter 2 ist aus.
- 20 bis 24 °C: Alles ist aus.
- Über 24 °C: Lüfter 1 und Lüfter 2 sind an, die Heizung ist aus.

**Bewässerung (Auto):** Die Bodenfeuchte wird mit einem zweiten Spannungssignal simuliert (0,5–10 V entsprechen 0–100 %). Unter 40 % [Vorschlag] schaltet die Pumpe ein, bis 60 % [Vorschlag] erreicht sind. Danach gilt eine Sperrzeit [Vorschlag: 30 s], damit nicht sofort wieder gegossen wird. Die Wasserversorgung kommt aus einem Becken, das über ein mechanisches Schwimmerventil an die Leitung angeschlossen ist.

**Überwachung und Störungen:**

- Läuft die Pumpe länger als [Vorschlag: 3 s] ohne Durchfluss-Signal, schaltet sie ab, die gelbe Leuchte 2 ("Kein Durchfluss") leuchtet und die TDE zeigt eine Meldung. Der Durchfluss wird mit einem Schalter simuliert.
- Fällt ein Analogsignal unter 0,3 V (Fühlerbruch), oder löst ein Motorschutzschalter aus, gehen alle Ausgänge aus. Die gelbe Leuchte 1 ("Störung") leuchtet.
- Jede Störung wird mit dem Quittier-Taster bestätigt, erst danach ist ein Neustart möglich.

**Not-Halt:** Der Not-Halt-Taster (3SU1801-0NB00-2AC2, 2 Öffner, zweikanalig) liegt am Sicherheitsrelais. Es nimmt die Spannung von den Ausgängen weg, unabhängig vom LOGO!-Programm. Die LOGO! erhält nur die Rückmeldung am Eingang I4 und zeigt auf der TDE mit rotem Hintergrund "ALARM!". Nach dem Entriegeln ist ein Quittieren nötig.

**Anzeigen:** Die TDE zeigt Temperatur, Sollwert, Bodenfeuchte, Betriebsart und Meldungen.

## Bauteilliste (Entwurf)

Schütze und Motorschutzschalter kommen von Siemens, die Typen richten sich nach deinem Beispielprojekt. Die Bezeichnungen (BMK) sind ein Vorschlag. Typen mit „prüfen“ sind im Katalog zu kontrollieren.

| BMK | Bauteil | Typ | Menge | Hinweis |
| --- | --- | --- | --- | --- |
| Q1 | Hauptschalter | 3LD3054-0TL53 | 1 | wie Beispielprojekt |
| F1, F2 | Leitungsschutzschalter | 5SY6202-7 / 5SY6102-7 | 2 | wie Beispielprojekt |
| T1 | Netzteil 24 V DC | SITOP PSU100L 24 V/10 A (6EP1334-1LB00) | 1 | wie Beispielprojekt |
| A1 | LOGO! | 6ED1052-1MD08-0BA2 | 1 | 12/24RCE |
| A2 | Erweiterungsmodul | 6ED1055-1MB00-0BA2 (DM8) | 1 | für Q5 und Q6 |
| A3 | Display | 6ED1055-4MH08-0BA1 (TDE) | 1 | Netzwerkkabel zur LOGO! |
| A4 | Sicherheitsrelais | 3SK1111-1AB30 | 1 | zweikanalig, mit Rückführkreis |
| S0 | Not-Halt | 3SU1801-0NB00-2AC2 | 1 | 2 Öffner, Gehäuse mit Pilztaster |
| S1 | Wahlschalter Aus/Manuell/Auto | 3SU1, 3 Stellungen, 2 Schließer | 1 | Typ nach Schulbestand prüfen |
| S2 | Quittier-Taster (blau) | 3SU1100-0AB50-1BA0 | 1 | prüfen |
| S3 | Schalter Durchfluss-Simulation | 3SU1, 2 Stellungen | 1 | Typ nach Schulbestand |
| K1, K2, K3 | Schütz 24 V DC | 3RT2015-2BB42 | 3 | Lüfter 1, Lüfter 2, Pumpe |
| F3, F4, F5 | Motorschutzschalter mit Hilfsschalter | F3, F4: Siemens 3VU1300-1MC00 (0,16–0,24 A, Lüfter 0,12 A, auf 0,16 A einstellen, mit Lehrer abgesprochen); F5: Moeller PKZM0-1 (0,63–1 A, Pumpe 0,76 A, auf 0,76 A einstellen, Herstellermix mit Lehrer klären) | 3 | Bereich nach Typenschild |
| M1, M2 | Lüftermotor | 4IK25GN-UT4 mit 4GN50K (400 V Y, 3~, 0,12 A) | 2 | Typenschild lesen |
| M3 | Pumpenmotor | 1LE1001-0CB22-2AA4 (400 V Y, 0,76 A) | 1 | Typenschild lesen |
| P1 | Leuchtmelder rot (Heizung) | 3SU1102-6AA20-3AA0 | 1 | prüfen |
| P2, P3 | Gelbe Leuchten Störung und Kein Durchfluss | Eaton SL7-L24-Y | 2 | 24 V, 54 mA |
| B1, B2 | Spannungsquelle Simulation (Temperatur, Bodenfeuchte) | HW-140 | 2 | 0,5–10 V einstellen, vorher ohne Last messen |
| X1 | Reihenklemmen | PT 2,5, PT 2,5-PE, Abschluss D-ST 2,5 | nach Plan | wie Beispielprojekt |
| U1… | Tragschiene und Kabelkanal | TS 35×7,5, Rittal TS.8800752 | nach Plan | wie Beispielprojekt |

Wähle den MSS-Bereich so, dass der Nennstrom vom Typenschild im Bereich liegt, und stelle ihn auf diesen Strom ein. Die Fühler und Endschalter an der Anlage brauchen Zugentlastung und PG-Verschraubungen.
