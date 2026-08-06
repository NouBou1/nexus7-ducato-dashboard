# Nexus 7 → Digitales Dashboard für einen Fiat Ducato von 1986

Ein Google Nexus 7 von 2012 als zentraler Touchscreen in einem Fiat Ducato Baujahr 1986: Offline-Navigation, Live-Überwachung der Bordbatterien per Bluetooth und Mediensteuerung — auf Hardware, die mit Android 4.1 ausgeliefert wurde und seit Jahren aus dem Support ist.

*[English version](README.md) · [Flashing-Anleitung](docs/flashing.md)*

![Agama Car Launcher auf dem Nexus 7](images/car-launcher.jpg)

---

## Warum

Der Ducato ist von 1986. Kein Radioschacht mit Bordcomputer, kein CAN-Bus zum Anzapfen, kein Werksdisplay — nur ein Armaturenbrett mit einem Loch darin. Ein modernes Doppel-DIN-Android-Gerät hätte funktioniert, aber ich hatte ein Nexus 7 (2012) in der Schublade, und ein 7-Zoll-IPS-Panel mit 1280×800 ist ein besserer Bildschirm als das, was in den meisten günstigen Head Units steckt.

Der Haken: Android 4.1 auf einem Tegra 3 war schon damals langsam und ist heute unbenutzbar — keine Sicherheitsupdates, keine aktuellen Apps, dazu die bekannte eMMC-Degradation, die diese Tablets nach ein paar Jahren zäh macht.

Das Projekt war also weniger „App installieren" als: ein wartbares System auf veraltete Hardware bekommen und es danach wie ein Gerät und nicht wie ein Tablet verhalten lassen.

## Was es kann

- **Offline-Navigation** — Kartenmaterial liegt auf dem Gerät, Routing funktioniert ohne Mobilfunk (im Wohnmobil relevanter als in der Stadt).
- **Live-Batterieüberwachung** — die beiden Bordbatterien haben Bluetooth-BMS-Module. Das Tablet verbindet sich mit beiden und liest den Batteriezustand aus. Die Bordelektrik ist damit vom Fahrersitz aus sichtbar statt nur über eine Bodenklappe.
- **Medien** — Internetradio und lokale Musik, bedienbar über eine Oberfläche, die für ein fahrendes Fahrzeug dimensioniert ist.
- **Verhalten wie ein Festeinbau** — startet selbstständig, sobald die Zündung Strom liefert, und schläft ein, wenn der Strom weg ist. Kein Power-Knopf, kein Sperrbildschirm, kein „wo ist der Launcher hin".

Jede Funktion läuft über ihren eigenen Kanal: **GPS** für die Ortung, **Bluetooth** für die Batteriemodule, und WLAN oder Handy-Hotspot nur für Internetradio und Updates. Nichts, was während der Fahrt zählt, hängt an einer Verbindung.

## Hardware & Software

| | |
| :--- | :--- |
| Gerät | Google Nexus 7 (2012) Wi-Fi — Codename `grouper`, Tegra 3 |
| Fahrzeug | Fiat Ducato, Baujahr 1986 |
| ROM | AOSP Android 7.1.2 (Nougat), AndDiSa-Build für `grouper` |
| Recovery | TWRP 3.x (`grouper`) |
| Google-Apps | OpenGApps `arm / 7.1 / pico` |
| Launcher | Agama Car Launcher |
| Batterieüberwachung | ECO-WORTHY BMS (2×) über Bluetooth LE |
| Automatisierung | Tasker / MacroDroid |

## Die Entscheidungen, auf die es ankam

### Android 7.1.2 statt etwas Neuerem

Für `grouper` gibt es Community-ROMs bis zu deutlich neueren Android-Versionen, und der Reflex ist, die höchste verfügbare Nummer zu nehmen. Auf diesem Gerät ist das die falsche Entscheidung. Der Tegra 3 hat vier Cortex-A9-Kerne und 1 GB RAM, und der eigentliche Flaschenhals ist der eMMC-Speicher — neuere Android-Versionen mit größeren Runtimes und mehr Hintergrundarbeit warten vor allem auf den Speicher.

Ich habe mich für AndDiSas AOSP-7.1.2-Build entschieden, weil er speziell für dieses Gerät gepflegt und auf diese Grenzen abgestimmt ist. Dazu das `pico`-GApps-Paket statt eines vollen — jeder installierte Google-Dienst ist ein Hintergrundprozess, der um denselben langsamen Speicher konkurriert.

Das Ergebnis ist ein Tablet, das die Karte sofort öffnet. Ein neueres Android auf derselben Hardware hätte auf dem Datenblatt besser ausgesehen und sich an der Kreuzung schlechter angefühlt.

### Einschalten mit der Zündung

Ein Dashboard, das einen Tastendruck braucht, ist kein Dashboard. Der Bootloader des Nexus 7 hat ein Off-Mode-Charge-Flag, das steuert, was bei anliegendem Strom im ausgeschalteten Zustand passiert — standardmäßig zeigt das Gerät eine Ladeanimation und bleibt aus. Deaktiviert man das Flag, bootet es stattdessen:

```bash
fastboot oem off-mode-charge 0
```

Am geschalteten Stromkreis angeschlossen startet das Tablet damit beim Drehen des Zündschlüssels.

### Schlafverhalten

Das Gegenstück zum Booten bei Strom ist, danach nicht dauerhaft wach zu bleiben. *Entwickleroptionen → Wach bleiben bei Stromversorgung* hält den Bildschirm während der Fahrt an, die Übergänge übernehmen Tasker-Regeln: Strom verbunden → Display an, Bluetooth an, Launcher in den Vordergrund; Strom getrennt → Display aus und Deep Sleep, damit über Nacht nichts an der Starterbatterie zieht.

## Screenshots

| | |
| :--- | :--- |
| ![Offline-Navigation](images/offline-navigation.jpg) | ![Medienwiedergabe](images/media-player.jpg) |
| Offline-Navigation — Karten liegen auf dem Gerät | Medienwiedergabe |

![ECO-WORTHY BMS über Bluetooth](images/bms-bluetooth.jpg)

*Beide BMS-Module in der ECO-WORTHY-App gekoppelt. Hier getrennt dargestellt — die Fotos sind am Schreibtisch entstanden, nicht im Fahrzeug.*

## Ehrliche Einschränkungen

- **Der Launcher ist eine Testversion.** Agama Car Launcher ist kostenpflichtig, und die Testphase ist auf dieser Installation abgelaufen — daher das „Buy now"-Overlay im Screenshot. Entweder lizenzieren oder auf eine offene Alternative wechseln.
- **Micro-USB als Stromanschluss** ist die Schwachstelle des ganzen Aufbaus — dieser Stecker ist nicht für dauerhafte Vibration gebaut. Ein ausgeleierter Port wird am ehesten zuerst ausfallen.
- **Die Fotos sind Werkbank-Aufnahmen**, keine Einbaufotos. Sie zeigen das laufende System, nicht die fertige Einbaulösung.

## Flashing-Anleitung

Der komplette Ablauf — Bootloader entsperren, TWRP, Wipe, Sideload — steht in **[docs/flashing.md](docs/flashing.md)**.

> ⚠️ Das Entsperren des Bootloaders löscht das Gerät vollständig. Alles auf dem internen Speicher ist danach weg.

---

*Umgesetzt im eigenen Wohnmobil. Dokumentation nachträglich aufbereitet — aus Notizen und vom Gerät selbst.*
