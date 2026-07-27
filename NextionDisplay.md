# Projekterweiterung: Integration des Nextion Touch-Displays (NX3224T024_011)
**HMI-Schnittstelle und Datenkommunikation via Modbus RTU mit OpenPLC V4.2.9**

---

## 1. System-Vorteile durch den Wechsel auf Nextion
Der Umstieg vom klassischen OLED-Display auf das **Nextion NX3224T024_011 (2,4 Zoll Touch)** bringt signifikante architektonische und funktionale Vorteile:
* **Entlastung des ESP32:** Das Nextion-Display besitzt einen eigenen Prozessor. Die grafische Oberfläche (Pages, Buttons, Texte, Schriften) wird komplett autark berechnet. Der ESP32 tauscht nur noch reine Nutzdaten aus.
* **Wegfall mechanischer Komponenten:** Die 3 physischen Hardware-Taster entfallen komplett. Die Bedienung erfolgt verschleißfrei direkt über das Touch-Display.
* **Keine Software-Entprellung nötig:** Da die Signale digital über die serielle Schnittstelle übertragen werden, entfällt das mechanische Tasterprellen. Die `R_TRIG`- und `TOF`-Netzwerke im OpenPLC-Editor werden überflüssig, was den Kontaktplan massiv verschlankt.

---

## 2. Hardware-Architektur & Verkabelung
Das Nextion-Display nutzt zur Kommunikation nicht mehr den I2C-Bus, sondern eine serielle **UART-Verbindung (Universal Asynchronous Receiver-Transmitter)**.

### Anschlussbelegung am ESP32 WROOM:
* **VCC:** An **5V** des ESP32 (Zwingend erforderlich für die Hintergrundbeleuchtung des Displays).
* **GND:** An einen freien **GND**-Pin des ESP32.
* **TX (Nextion):** An **GPIO 16 (RX2)** des ESP32 (Empfangsleitung).
* **RX (Nextion):** An **GPIO 17 (TX2)** des ESP32 (Sendeleitung).

*Hinweis:* Das DS3231 RTC-Uhrenmodul bleibt weiterhin unberührt an den I2C-Pins (GPIO 21 & 22) angeschlossen. Beide Kommunikationsbusse laufen parallel und unabhängig voneinander auf dem ESP32.

---

## 3. Datenübertragungskonzept (Modbus RTU)
Die Kommunikation zwischen der OpenPLC-Runtime und dem Nextion-Display wird über das industrielle Standardprotokoll **Modbus RTU** abgewickelt. Der ESP32 fungiert hierbei als *Modbus-Server*.

Im OpenPLC-Editor werden den Variablen feste Speicherbereiche (**Holding Register / `%MW`**) zugewiesen, auf die das Nextion-Display direkt zugreift:

| Variablenname | OpenPLC-Adresse | Modbus-Register | Richtung (aus Sicht des Displays) |
| :--- | :--- | :--- | :--- |
| `RTC_Hour` | `%MW0` | **40001** | **Lesen** (Aktuelle Stunde anzeigen) |
| `RTC_Minute` | `%MW1` | **40002** | **Lesen** (Aktuelle Minute anzeigen) |
| `On_Hour` | `%MW2` | **40003** | **Schreiben & Lesen** (Einschalt-Stunde) |
| `On_Minute` | `%MW3` | **40004** | **Schreiben & Lesen** (Einschalt-Minute) |
| `Off_Hour` | `%MW4` | **40005** | **Schreiben & Lesen** (Ausschalt-Stunde) |
| `Off_Minute` | `%MW5` | **40006** | **Schreiben & Lesen** (Ausschalt-Minute) |
| `Selected_Weekday`| `%MW6` | **40007** | **Schreiben & Lesen** (Wochentag 0–7) |

---

## 4. GUI- & Layout-Design (Nextion Editor)
Das Display-Layout wird mit dem kostenlosen *Nextion Editor* auf dem PC erstellt und per SD-Karte oder USB-Adapter auf das Display geflasht. Es gliedert sich in zwei funktionale Seiten:

### Seite 0: Hauptanzeige (`page0`)
Läuft im regulären Automatisierungsbetrieb.
* Zeigt die aktuelle Uhrzeit der SPS (aus Register 40001 und 40002) im Format `HH:MM` groß an.
* Visualisiert den aktuellen Schaltzustand der Last-LED an Pin 32 (EIN / AUS).
* Enthält einen Touch-Button **"Einstellungen"**, der zur Konfigurationsseite wechselt.

### Seite 1: Einstellungsmenü (`page1`)
Ermöglicht die intuitive Programmierung der Zeitschaltuhr direkt auf dem Glas.
* **Numerische Felder (`n0` bis `n3`):** Zeigen die aktuell gespeicherten Schaltzeiten für EIN und AUS.
* **Touch-Buttons `[ + ]` und `[ - ]`:** Erhöhen oder verringern den Wert direkt im Modbus-Register. Interne Nextion-Skripte begrenzen die Werte logisch (Stunden: 0–23 / Minuten: 0–59).
* **Wochentagsauswahl (`[ < ]` und `[ > ]`):** Schaltet die Variable `Selected_Weekday` von 0 bis 7 durch. Ein Textfeld übersetzt die Zahl live in Text (z. B. `0` = "Jeden Tag", `1` = "Montag", etc.).
* **Button "Speichern & Zurück":** Schließt das Menü, sichert die Werte im ESP32 und springt zur Hauptseite zurück.

---

## 5. Software-Konfiguration in OpenPLC
Damit der Datenaustausch reibungslos funktioniert, wird in der Web-Oberfläche der OpenPLC-Runtime unter **Industrial Protocols** das Protokoll **Modbus RTU** aktiviert:
* **Port:** `/dev/ttyS2` (entspricht UART2 auf dem ESP32).
* **Baudrate:** **9600** oder **115200** bps (muss mit den Einstellungen im Nextion-Projekt exakt übereinstimmen).
* **Data Bits:** 8, **Stop Bits:** 1, **Parity:** None.
