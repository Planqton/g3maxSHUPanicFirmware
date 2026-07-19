# Testbuilds — Diagnose "100 km/h (V9.9.8) fällt nach dem Flash auf V9.9.9 zurück"

Symptom: `PFW_G3_MAX_Panickey-100kmh.zip` flasht scheinbar durch, danach meldet die VCU
wieder **V9.9.9** = der 45er-PFW aus Slot A läuft weiter → A/B-Rollback, das neue Image
wurde nicht übernommen.

Das Paket selbst ist nachweislich in Ordnung (Struktur, `md5.enc`, NinebotTEA-Checksum,
no-UID-lock/no-flash-lock/`fe801cb2`-Token alle vorhanden). Gegenüber dem funktionierenden
45er unterscheiden sich **genau 4 Bytes**:

| Offset (Roh-FW) | 45er | 100er | Bedeutung |
|---|---|---|---|
| `0xE116` / `0xE4C2` | `2D` (45) | `64` (100) | Tuning-Cap |
| `0xE486` / `0xE56A` | `99` | `98` | Versions-Spoof `movw #0x0999` → `#0x0998` |

Diese beiden Testbuilds isolieren, welche der zwei Änderungen den Rollback auslöst.

## TEST-A — `PFW_G3_MAX_Panickey-TEST-A-v998-45kmh.zip`
Nur der **Versions-Patch** (V9.9.8), Cap bleibt **45**. (Diff zum 45er: `0xE486`, `0xE56A`)

- Zeigt die App danach **V9.9.8** → Versions-Patch ist unschuldig, Ursache ist der Cap-Wert 100.
- Fällt es wieder auf **V9.9.9** zurück → die Version selbst blockt (Downgrade-Check 0x998 < 0x999).

## TEST-B — `PFW_G3_MAX_Panickey-TEST-B-60kmh-v999.zip`
Nur der **Cap-Patch auf 60 km/h**, Version bleibt **V9.9.9**. (Diff zum 45er: `0xE116`, `0xE4C2`)

Die Version bleibt hier absichtlich gleich — Erfolgskontrolle deshalb über **BLE Slot 01, Byte 7**
(`f1 01` lesen): steht dort **60**, ist der Cap-Patch übernommen worden.

- Byte 7 = 60 → Cap-Patch ok, nur der Wert 100 ist zu hoch (dann hochtasten: 60 → 80 → 100).
- Byte 7 = 45 / Rollback → schon 60 crasht beim Boot, Limit liegt tiefer.

## Reihenfolge
Erst TEST-A, dann TEST-B, dazwischen jeweils zurück auf `PFW_G3_MAX_Panickey.zip` (45er) —
das ist der bekannte funktionierende Stand.

Ergebnis beider Tests bitte notieren, daraus folgt der Fix für das 100er-Image.
