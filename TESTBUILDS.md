# Diagnose: warum der 100-km/h-Build zurückrollt (abgeschlossen)

Symptom war: `PFW_G3_MAX_Panickey-100kmh.zip` flasht scheinbar durch, danach meldet die VCU
wieder **V9.9.9** = der 45er-PFW aus Slot A läuft weiter → A/B-Rollback, das neue Image wurde
beim ersten Boot verworfen.

Das Paket selbst war in Ordnung (Struktur, `md5.enc`, NinebotTEA-Checksum, no-UID-lock /
no-flash-lock / `fe801cb2`-Token alle vorhanden). Gegenüber dem 45er unterscheiden sich
**genau 4 Bytes**:

| Offset (Roh-FW) | 45er | 100er | Bedeutung |
|---|---|---|---|
| `0xE116` / `0xE4C2` | `2D` (45) | `64` (100) | Tuning-Cap |
| `0xE486` / `0xE56A` | `99` | `98` | Versions-Spoof `movw #0x0999` → `#0x0998` |

## Testreihe am Gerät

| Build | Änderung | Ergebnis |
|---|---|---|
| TEST-A | nur Version `0x0998`, Cap bleibt 45 | **V9.9.8** → übernommen, Versions-Patch unschuldig |
| TEST-B | nur Cap 60, Version bleibt 9.9.9 | nicht eindeutig ablesbar (Version by design gleich) |
| TEST-C | Cap **99** + Version `0x0998` | **V9.9.8** → übernommen, läuft |

## Ergebnis

Der Rollback kommt **allein vom Cap-Wert `0x64` (100)**. **99 (`0x63`) ist das Maximum**, das die
VCU annimmt — irgendwo in der Verarbeitung des Caps sitzt eine Zweistelligkeits-Grenze
(`cmp #100` bzw. BCD-/Anzeige-Konvertierung); 100 lässt den ersten Boot scheitern, der
Bootloader fällt auf Slot A zurück.

**Deliverable daraus:** `PFW_G3_MAX_Panickey-99kmh.zip` (Cap 99, meldet sich als V9.9.8).
Firmware-Payload byte-identisch mit dem getesteten TEST-C.
`PFW_G3_MAX_Panickey-100kmh.zip` ist damit als **nicht funktionsfähig** belegt.

Die Testbuilds (TEST-A/B/C) wurden nach Abschluss entfernt.
