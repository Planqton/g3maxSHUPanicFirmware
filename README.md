# G3 Max — Panickey-PFW + QuickConfig-App

Gepatchte VCU-Firmware („PFW") für den **Ninebot G3 Max**, als SHU-Paket zum Flashen über die
**ScooterHacking Utility**, plus eine Android-App zum Konfigurieren per Bluetooth.

Basis ist die 1dragon-CFW (V10.10.0.3) mit drei Patches:

- **Panic-Taste** — frei belegbare Brems-/Tasten-Sequenz („Unlock-Code") am Roller.
- **no-UID-lock** — die originale CFW ist an die Werks-Unique-ID *eines* VCU gebunden und läuft
  sonst ohne Funktionen. Der Check ist entschärft → läuft auf jedem G3 Max.
- **no-flash-lock** — die originale CFW blockt fremde Firmware beim OTA („Error 7 – firmware may be
  restricted"). Entfernt → SHU kann jederzeit wieder etwas anderes draufflashen.

Der SHU-Token (`fe801cb2…`) wird vom Image sauber mitgeschrieben, d. h. **SHU bleibt nach dem Flash
dauerhaft nutzbar** — der x3utils-Patch muss nur einmal gemacht werden.

## Dateien

| Datei | Tuning-Cap | meldet sich als | Zweck |
|---|---|---|---|
| `PFW_G3_MAX_Panickey.zip` | 45 km/h | **V9.9.9** | Standard-Build |
| `PFW_G3_MAX_Panickey-99kmh.zip` | **99 km/h** | **V9.9.8** | angehobener Cap im Tuning-Modus |
| `G3MaxQuickConfig-debug-unsigned.apk` | — | — | Android-App (BLE-Konfiguration, Demo-Modus) |

Die Versionsnummer ist ein Erkennungsmerkmal, keine echte Firmware-Version — daran unterscheidet die
App, welcher Build läuft (`reg 0x17`), und setzt den Regler-Maximalwert entsprechend.

Der **Standard-Modus bleibt bei beiden Builds unangetastet** (~25 km/h). Angehoben wird nur der
Maximalwert, den man im **Tuning-Modus** einstellen kann.

> **99 km/h ist das Maximum.** Ein Build mit Cap 100 wird zwar geschrieben, scheitert aber beim
> ersten Boot und wird vom Bootloader auf den alten Slot zurückgerollt (Symptom: „flasht durch",
> danach steht wieder die alte Version drin). Am Gerät verifiziert — deshalb 99.

## Flashen

1. **Einmalig:** [`ztakis/x3utils`](https://github.com/ztakis/x3utils) → *„Flash SHU compatible"*
   per ST-Link/SWD. Setzt den Distributions-Key, damit die VCU SHU-Pakete annimmt.
2. Gewünschtes `.zip` über die **ScooterHacking Utility** flashen (Typ VCU).
3. Roller neu starten, in der App die Version prüfen (V9.9.9 bzw. V9.9.8).

Ein Wechsel zwischen den beiden Builds ist jederzeit in beide Richtungen möglich.

**Vorher ein SWD-Backup der VCU ziehen.** Das ist der Fallback, falls ein Flash schiefgeht — die
VCU ist per ST-Link immer wiederherstellbar, solange man ein Image hat.

## Android-App

<p align="center">
  <img src="docs/app-unlock-code.png" alt="Unlock-Code-Editor der G3MaxQuickConfig-App" width="330">
</p>

Verbindet per BLE mit dem Roller und liest/schreibt die Konfiguration:

- **Unlock-Code** — die Brems-/Tasten-Sequenz, mit der der Tuning-Modus am Roller aktiviert wird
  (siehe Screenshot). Bis zu **20 Schritte**, in beliebiger Reihenfolge, sortier- und löschbar.
  Verfügbare Eingaben: `Bremse L`, `Bremse R`, `M-Taste`, `Power`, `Blinker L`, `Blinker R`, `Boost`.
- **Profile** — Sport-Geschwindigkeit für Standard- und Tuning-Modus, Unterbodenbeleuchtung,
  vordere Lichtleiste, Tempomat (je Profil getrennt).
- **Einstellungen** — Panik-Taste, Piepton im Tuning-Modus.
- **Demo-Modus** — zeigt alle Karten mit Beispieldaten, ohne dass ein Roller in der Nähe ist.

Die APK ist ein Debug-Build (unsigniert) — Android verlangt beim Installieren „Unbekannte Quellen".

## Hinweise

Auf eigene Gefahr. Ein E-Scooter mit angehobenem Limit ist im öffentlichen Straßenverkehr nicht
zulassungsfähig; Betriebserlaubnis und Versicherungsschutz erlöschen. Bremsen, Reifen und Rahmen
des G3 Max sind nicht für 99 km/h ausgelegt — der Wert ist eine Softwaregrenze, kein Fahrversprechen.
