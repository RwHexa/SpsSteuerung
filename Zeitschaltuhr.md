# Projekt-Konzept: Autarke Hutschienen-Zeitschaltuhr auf ESP32-Basis
**Modulares SPS-Steuerungssystem via OpenPLC V4.2.9**

---

## 1. Projektübersicht
Das Ziel dieses Projekts ist die Entwicklung und Umsetzung einer industrietauglichen, vollkommen autarken Zeitschaltuhr für den Schaltschrank- oder Gehäuseeinbau. Im Gegensatz zu klassischen IoT-Lösungen verzichtet dieses System bewusst auf eine permanente Internet- oder Cloud-Verbindung. 

Die Konfiguration der täglichen Ein- und Ausschaltzeiten erfolgt direkt am Gerät. Das System basiert auf modernen SPS-Standards (IEC 61131-3) und bietet maximale Zuverlässigkeit, Wartungsfreundlichkeit sowie eine visuelle Echtzeit-Diagnose.

---

## 2. Erforderliche Systemkomponenten
Das Gesamtsystem gliedert sich in eine performante Steuereinheit, ein präzises Zeitmessmodul und eine intuitive Mensch-Maschine-Schnittstelle (HMI).

* **Zentrale Steuereinheit (SPS-Kern):** *ESP32 WROOM Mikrocontroller*
  Übernimmt die zyklische Abarbeitung der Programmlogik, steuert die Ein- und Ausgänge und verarbeitet die Benutzereingaben in Echtzeit.
* **Echtzeituhr (RTC):** *DS3231 Uhrenmodul (inkl. Pufferbatterie)*
  Sorgt für eine hochpräzise Zeithaltung im Sekundenbereich. Dank der integrierten Knopfzelle bleibt die Uhrzeit auch bei einem kompletten Stromausfall über Jahre hinweg exakt gesichert.
* **Anzeigeeinheit (HMI-Display):** *SSD1306 OLED-Display (128x64 Pixel)*
  Dient der visuellen Darstellung der aktuellen Uhrzeit sowie der Menüführung bei der Zeiteinstellung.
* **Bedienelemente:** *3 physische Industrie-Taster*
  Ermöglichen die vollständige Bedienung und Programmierung direkt am Gerät (Taster-Belegung: *Menü*, *Plus*, *Minus*).
* **Aktoren / Ausgänge:** *LED-Statusanzeigen / Relaisstufen*
  Visualisieren den aktuellen Schaltzustand der Zeitschaltuhr (z. B. Last EIN / AUS).

---

## 3. Technisches Zusammenspiel (Systemarchitektur)
### Der Datenfluss und Steuerungsablauf:
1. **Zeiterfassung:** Das *DS3231 RTC-Modul* sendet die aktuelle Stunde und Minute über den digitalen **I2C-Bus** (GPIO 21 & 22) permanent an den *ESP32*. OpenPLC synchronisiert die interne Systemzeit automatisch mit diesem Hardware-Takt.
2. **Benutzerinteraktion:** Wird der *Menü-Taster* betätigt, wechselt die interne Programmlogik schrittweise durch die Einstellmodi (Zustandsmaschine). Über die Taster *Plus* und *Minus* werden die Zielwerte für die Schaltzeiten im flüchtigen Speicher verändert.
3. **Daten-Visualisierung:** Der *ESP32* sendet die darzustellenden Informationen (aktuelle Uhrzeit im Normalbetrieb oder die blinkenden Einstellwerte im Programmiermodus) ebenfalls über den *I2C-Bus* an das *OLED-Display*.
4. **Schalt-Logik (Kernprozess):** Das SPS-Programm rechnet alle Uhrzeiten (RTC-Zeit, Einschaltzeit, Ausschaltzeit) intern in eine fortlaufende **Gesamt-Minute des Tages** um. Ein mathematischer Vergleicher prüft im Millisekundentakt, ob die aktuelle Minute innerhalb des definierten Fensters liegt. Die Logik unterscheidet dabei vollautomatisch zwischen *Tagbetrieb* und *Nachtbetrieb (über Mitternacht)*.
5. **Aktion:** Entspricht die Uhrzeit den Vorgaben, schaltet der *ESP32* den physischen Digitalausgang (GPIO 32) gegen High und aktiviert das Relais bzw. die Last-LED.

---

## 4. Vorteile dieser Erarbeitung (Projektnutzen)
* **Kein Versions-Mismatch mehr:** Durch den Einsatz des aktuellen *OpenPLC Editors V4.2.9* ist die PC-Simulation nativ integriert. Funktionstests können ohne externe Software direkt auf dem Entwicklungsrechner durchgeführt werden.
* **100% Autarkie:** Keine Abhängigkeit von WLAN-Netzwerken, Routern oder NTP-Servern im Internet. Perfekt für abgelegene Einsatzorte oder abgeschirmte Schaltschränke.
* **Hardware-Schonung & Entprellung:** Die Tastereingänge werden programmtechnisch über Flankenerkennung (*R_TRIG*) und Ausschaltverzögerungen (*TOF*) abgesichert. Ein unkontrolliertes "Durchspringen" der Menüs durch mechanisches Tasterprellen ist ausgeschlossen.
* **Effizientes Live-Debugging:** Über die integrierte Diagnose-Schnittstelle (das "Auge"-Symbol im Editor) können alle Signalwege, Timer und Register im laufenden Betrieb am PC visuell überwacht werden, was die Inbetriebnahmezeit drastisch verkürzt.
