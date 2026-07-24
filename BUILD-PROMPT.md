```json
{
  "board_name": "PulseWristTrack",
  "one_liner": "Ultrakompakte, runde Smartwatch-Trägerplatine für Seeed XIAO nRF52840 Sense und den optischen Sensor MAX30102 zur Erfassung von Gesten- und Vitaldaten.",
  "market_gap": "Mangel an tragbaren, handgelenkstauglichen Trägerboards mit integrierter 6-Achsen-IMU und Haut-Kontaktsensorik, die ohne Kabelsalat direkt getragen werden können.",
  "confidence": "high",
  "price_eur": 24,
  "target_enclosure": "3D-gedrucktes Rundgehäuse (38mm Durchmesser) mit integrierter 22mm Armband-Führung",
  "injection_notes": "keine"
}
```

## ⚠️ KRITISCHE BUILD-HINWEISE (ZUERST LESEN — dieses Board ist KEIN Standard-Bluepill-Carrier)
Dieses Board ist ein **rundes Wearable** mit HF-Modul. Platzierung ist funktionskritisch — halte dich strikt an die unten genannten EXAKTEN Koordinaten. Falls die Pipeline eine automatische Reihen-Platzierung (`suggest_placement`) erzwingt, ist das hier FALSCH: platziere anschliessend die kritischen Bauteile ZURÜCK auf die vorgegebenen Koordinaten. Konkret:
1. **Umriss NICHT rechteckig mit 4× M3:** Der Umriss ist ein **KREIS Ø 36,0 mm** (Zentrum 0,0) mit 2 Armband-Schlitzen (Edge.Cuts) + **2× M2**-Bohrungen — NICHT das Standard-100×80-Rechteck mit 4× M3.
2. **ANTENNEN-KEEPOUT (BLE-Reichweite!):** Zone 18×6 mm an der Oberkante (Y=11,5…17,5 mm) muss auf **ALLEN 4 Lagen kupferfrei** sein (keine Tracks, keine GND/Power-Plane-Füllung). Das XIAO-Modul (U1) so platzieren, dass seine Chip-Antenne in dieser Zone liegt. Verletzung = Bluetooth funktioniert nicht.
3. **MAX30102 (U2) exakt zentriert bei X=0,Y=0 auf der UNTERSEITE (B.Cu)** — Hautkontakt. Nicht in eine Reihe schieben.
4. Bauteile sind bewusst auf **Oberseite (F.Cu)** und **Unterseite (B.Cu)** aufgeteilt (siehe Liste) — diese Seiten-Zuordnung beibehalten.
5. Die GND-/Power-Planes (In1/In2) müssen den Antennen-Keepout **aussparen**.

Nur 16 Bauteile → nach korrekter Platzierung gut autoroutbar (Signale F/B, Planes innen).

## BUILD-PROMPT

### Projekt-Spezifikation: PulseWristTrack (Wearable Wrist-Tracker)

Setze ein ultrakompaktes, 4-lagiges Trägerboard für das **Seeed Studio XIAO nRF52840 Sense** und den optischen Herzfrequenz-/SpO2-Sensor **MAX30102** um. Die Platine trennt funktional die **Serviceseite (oben)** und die **Hautkontaktseite (unten)**.

---

### 1. Harte DFM- & Routing-Vorgaben
*   **Bauteilanzahl:** Maximal 16 Komponenten (äußerst übersichtlich, garantiert autoroutbar).
*   **Leiterbahnen & Abstände:** 
    *   Signale (I2C, INT): ≥ 0.35 mm Breite, Clearance ≥ 0.3 mm.
    *   Power (3.3V, 1.8V, GND, BAT): ≥ 0.5 mm Breite.
*   **Lagen-Stackup (4 Lagen):**
    *   **F.Cu (Top):** Signale, SMT-Pads für XIAO, Schalter, JST-Buchse.
    *   **In1.Cu (Inner 1):** Durchgehende GND-Massefläche (Solid Ground Plane).
    *   **In2.Cu (Inner 2):** Power-Flächen (aufgeteilt in 3.3V und 1.8V).
    *   **B.Cu (Bottom):** SMT-Pads für MAX30102, LDO, Pegelwandler und Signale zur Unterseite.

---

### 2. Mechanik & Geometrie
*   **Platinenform:** Kreisrund, exakt **36.0 mm Durchmesser** (Zentrum bei X=0, Y=0).
*   **Armband-Schlitze (Edge.Cuts):** Zwei Schlitze (22.0 mm × 3.0 mm) für Standard-20mm/22mm-Uhrenarmbänder:
    *   *Schlitz 1 (oben):* Zentriert bei `X=0, Y=14.0 mm`.
    *   *Schlitz 2 (unten):* Zentriert bei `X=0, Y=-14.0 mm`.
*   **Befestigungsbohrungen:** 2× M2-Bohrungen (Ø 2.1 mm) zur Gehäusemontage:
    *   Platziert bei `X=-12.0 mm, Y=0` und `X=12.0 mm, Y=0` (mit 1.0 mm kupferfreiem Keepout-Ring).
*   **Antennen-Keepout (Sehr Wichtig!):** Rechteckiges Keepout-Areal (18.0 mm × 6.0 mm) an der Oberkante des Kreises (`Y = 11.5 mm` bis `Y = 17.5 mm`). In dieser Zone darf sich auf **keiner der 4 Lagen** Kupfer oder eine Leiterbahn befinden, um die Bluetooth-Sendeleistung der XIAO-Chipantenne nicht zu dämpfen.

---

### 3. Bestückungsliste & Platzierung (Top vs. Bottom)

#### OBERSEITE (F.Cu) – Bedienung, Akku & Host
1.  **U1 (Seeed Studio XIAO nRF52840 Sense):** SMD-Pads für castellated Montage. Platziert zentriert bei `X=0, Y=3.0 mm`. Die USB-C-Buchse zeigt nach oben (Richtung Y+), die Chip-Antenne liegt exakt im Antennen-Keepout-Bereich an der Oberkante.
2.  **J1 (JST-PH 2-Pin SMT/THT rechtwinklig):** Akku-Anschluss für 3.7V LiPo. Platziert bei `X=-8.0 mm, Y=-7.0 mm`.
3.  **SW1 (SMT-Schiebeschalter, z.B. JS202011MQN):** Ein-/Ausschalter in Reihe mit dem Pluspol des Akkus (`BAT+` zu `BAT_SW`). Platziert bei `X=8.0 mm, Y=-7.0 mm`.

#### UNTERSEITE (B.Cu) – Sensorik zur Hautseite (Plan aufliegend)
4.  **U2 (MAX30102 Pulse Oximeter):** OLGA-14-Gehäuse (`Sensor_Optical:MAX30102`). Platziert exakt im Zentrum bei `X=0, Y=0`.
5.  **U3 (AP2112K-1.8TRG1):** 1.8V LDO Regler (SOT-23-5) zur Versorgung des MAX30102-Cores. Platziert bei `X=-4.0 mm, Y=-4.0 mm`.
6.  **Q1, Q2 (BSS138):** N-Kanal-MOSFETs (SOT-23) zur Pegelwandlung der I2C-Leitungen (SDA/SCL) von 1.8V (Sensor-Seite) auf 3.3V (XIAO-Seite). Platziert nahe U2.
7.  **C1 (10µF 0805):** Eingangskondensator für U3 (an 3.3V).
8.  **C2 (1µF 0805):** Ausgangskondensator für U3 (an 1.8V).
9.  **C3 (10µF 0805):** Stützkondensator für MAX30102 `V_LED` (an 3.3V).
10. **C4 (1µF 0805):** Stützkondensator für MAX30102 `VDD` (an 1.8V).
11. **R1, R2 (4.7k 0603):** I2C-Pullups für SDA/SCL auf der 1.8V-Seite.
12. **R3, R4 (4.7k 0603):** I2C-Pullups für SDA/SCL auf der 3.3V-Seite.
13. **R5 (10k 0603):** Pullup für den Interrupt-Pin (`INT`) des MAX30102 an 3.3V (geht an XIAO D3).

---

### 4. Verbindungs- & Netzliste (Schematic-Leitfaden)
*   `BAT+` von J1 geht über den Schalter SW1 als Netz `BAT_SW` an den Laderegler-Eingang `BAT+` auf der Unterseite des XIAO (U1). `BAT-` geht direkt an `GND`.
*   U1 `3V3` versorgt U3 (LDO-Eingang), die Pullups R3, R4, R5 und den Drain der Pegelwandler.
*   U3 `VOUT (1.8V)` versorgt U2 `VDD` sowie die Pullups R1, R2.
*   `I2C-Bus (SDA/SCL)` wird über Q1/Q2 pegelgewandelt:
    *   3.3V-Seite (Source an XIAO): U1 `D4` (SDA) und `D5` (SCL).
    *   1.8V-Seite (Drain an Sensor): U2 `SDA` und `SCL`.
*   `Interrupt (INT)` geht von U2 `INT` direkt an U1 `D3`.

---

### 5. Arbeitsaufträge für die MCP-Werkzeuge
1.  **Projekt anlegen:** Erstelle das KiCad-Projekt `PulseWristTrack`.
2.  **Schaltplan zeichnen:** Setze die obige Netzliste sauber um. Verwende aussagekräftige Labels für die Busse und Spannungsversorgungen.
3.  **PCB-Layout erstellen:** 
    *   Definiere die kreisrunde Platine (Ø 36.0 mm) und fräse die Armband-Schlitze und M2-Löcher auf `Edge.Cuts`.
    *   Platziere die Bauteile strikt nach der topologischen Aufteilung (Top vs. Bottom).
    *   Richte den 4-Lagen-Stackup ein und weise In1.Cu der Masse zu.
    *   Zeichne die Antennen-Keepout-Zonen auf allen Lagen.
4.  **Routing:** Führe das Autorouting mit den definierten Leiterbahnbreiten (0.35 mm Signal, 0.5 mm Power) durch.
5.  **DRC & Export:** Starte die Design-Rule-Check-Prüfung, bereinige etwaige Fehler und exportiere sämtliche Gerber- und Bohrdaten nach `./gerbers`.

*Schließe den Prozess ab und fasse ehrlich zusammen, was erfolgreich generiert wurde.*
