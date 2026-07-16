<div align="center">

# DATEXT Diagnostics

**Professionelles Windows-Systemdiagnosetool für IT-Administratoren und technisch versierte Anwender**

![Version](https://img.shields.io/badge/Version-0.99.11.11-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%2010%2F11-lightgrey)
![Framework](https://img.shields.io/badge/.NET-8.0-purple)
![License](https://img.shields.io/badge/License-Proprietary-red)

[Funktionen](#funktionen) · [Installation](#-installation) · [Systemvoraussetzungen](#systemvoraussetzungen) · [Optionale Komponenten](#-optionale-komponenten) · [Export-Formate](#-export-formate) · [DATEXTAgent](#-datextagent) · [Support](#-support--kontakt)

</div>

---

## 📋 Übersicht

DATEXT Diagnostics ist eine umfassende Windows-Diagnoseanwendung die Systemadministratoren und IT-Professionals eine schnelle, strukturierte Übersicht über alle relevanten System-, Netzwerk- und Sicherheitsinformationen eines Windows-Rechners bietet — ohne Kommandozeile, ohne Scriptaufwand.

---

## Funktionen

### Übersicht
Der oberste Menüpunkt und zugleich Startbildschirm der Anwendung — fasst die beim Programmstart automatisch ermittelten Kernwerte des Systems zusammen, bevor überhaupt ein einzelnes Werkzeug aufgerufen wird.
- Anzeige relevanter lokaler Systeminformationen (System, Energie, Ressourcen, Festplatten, Netzwerk, IT-Sicherheit, Internet) mit farblich codierten Badges zur schnellen Übersicht
- Ermittlung und visuelle Anzeige von erkanntem CG-NAT-ISP-Anschluss und/oder erkannter aktiver VPN-Verbindung
- 💡 *Direkt-Aktionen:* IP-Adressen lassen sich per Symbol direkt an Ping, Traceroute, DNS-Auflösung oder Port-Scan übergeben — ohne sie manuell abzutippen. Bei erkannten Wortmann Systemen kann die Seriennummer an die Webseite zur Wortmann Seriennummernsuche übergeben werden (Seitenaufruf und Seriennummer über Zwischenablage einfügen).

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/fbb1054d-b0f3-4428-bbbd-b16f44546b98" />
<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/374594d9-8324-4da3-9248-f0946cd46543" />

### System Informationen
- **System-Health** — Ampel-Bewertung von Hardware, Sicherheit, Speicherplatz, Netzwerk und Ereignissen; schnellster Weg zur Ersteinschätzung
- **UEFI-Boot-Zertifikat** prüfen — inkl. Erkennung von physischer vs. virtueller Maschine und plattformspezifischer Handlungsempfehlung
- **Autorun-Analyse** über Registry-Autostart, geplante Aufgaben, Autostart-Ordner und automatisch startende Dienste
- **Ereignis-Log-Auswertung**, auch mit Schnellfiltern zu gängigen Problemen und Ermittlung von zusammenhängenden Events
  - 💡 *Kontext-Korrelation:* verwandte Ereignisse im Umkreis von ±5 Minuten um einen ausgewählten Eintrag anzeigen
  - 💡 *Anonymisierter Export* für Support-Tickets, ohne sensible Nutzerdaten preiszugeben
- **Installierte Software** Übersicht mit Such- und Exportfunktion
- **System-Verwaltungsprogramme** im Schnellzugriff (services.msc, devmgmt.msc, compmgmt.msc, msconfig u. v. m.)
- **Troubleshooting** für Windows Komponentenüberprüfung (DISM / SFC), Windows Update Reparatur, Netzwerk-Stack Resets, diverse weitere Werkzeuge

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/96ea53d2-8351-4bdc-b790-295ece8ac79f" />
<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/b8b0505c-cc90-44bf-9ab1-bc6b9ac5f85e" />

### Netzwerk
- Netzwerk- und Gerätescan mit Gerätetyp-Erkennung via PING, ARP, DNS, NetBIOS, LLMNR, mDNS, SSDP und SNMP-Info, dazu Topologie-Visualisierung
- **Ping** mit fortlaufender Statistik, Jitter, Paketverlust und automatischer MTU-Erkennung
- **Traceroute** mit Hop-für-Hop-Latenz, Reverse-DNS und Erkennung von CGNAT-/Doppel-NAT-Situationen im Pfad
- **DNS-Analyse** — A/AAAA/CNAME/NS/SOA/MX/TXT/SRV/PTR, Propagationsprüfung gegen mehrere öffentliche DNS-Server, IP-Zusatzinfos
- **Port-Scan** mit automatischer Diensterkennung, unterscheidet geschlossen (abgelehnt) von gefiltert (Firewall)
- **Netzwerk & Verbindungen** — Hub mit sechs Reitern (Netzwerkkarten, Routing-Tabelle, Verbindungen, Netzwerk-Analyse, Netzwerklast, Diagnose)
- **DHCP-Discovery** IPv4/IPv6 — erkennt alle antwortenden DHCP-Server, deckt Rogue-DHCP zuverlässig auf

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/d1833c8e-1502-43b9-b3f1-6fbb31a02126" />

**Verbindungsanalyse mit Risikobewertung** — jede aktive TCP-/UDP-Verbindung wird dem auslösenden Prozess zugeordnet (inkl. Authenticode-Signaturprüfung), auffällige Ziel-IPs werden automatisch gegen VirusTotal und Shodan abgeglichen.

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/f23955bd-87d1-4ba6-b872-9ed2200aa92a" />

### Active Directory
- mit installiertem RSAT kann ein Active Directory nach Benutzern/Computern durchsucht werden (automatische Domain-Controller-Erkennung, keine manuelle Serverangabe nötig)
- Analyse der zugewiesenen Group-Policies inkl. Verarbeitungsreihenfolge und Vererbung — deckt auf, warum eine erwartete Richtlinie auf einem Rechner nicht ankommt

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/39984655-d296-4e49-af89-422742c661ef" />

### MS-SQL
- SQL-Instanz-Erkennung (Registry, SQL Browser, Broadcast)
- Datenbankanalyse mit Windows-Authentifizierung und Impersonation
- 💡 Deckt neben der reinen Erreichbarkeit auch Aktivität (laufende Abfragen, Sperren), Performance-Kennzahlen (u. a. Page Life Expectancy) und Konfiguration ab

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/6ee1c87b-3652-4f8e-9882-12bccc3fcee3" />

### Packet Capture
- Paket-Mitschnitt via PKTMON (direkt über den Windows-Kernel, ganz ohne Drittanbieter-Treiber) oder NetSH-Traces (ETW-basiert, häufig vom Microsoft-Support angefordert)
- Konvertierung der Mitschnitte von .etl nach .pcap zur Auswertung via WireShark

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/c93510c4-c6a3-4b1e-90d1-70d83d0b4e48" />

### Performance
- Messung der Internet-Bandbreite mit dem <a href="https://www.speedtest.net/apps/cli" target="_blank">Ookla Speedtest CLI</a> (Download, Upload, Ping, Jitter, Paketverlust). Erfordert Internetzugriff auf Port TCP 8080 für die Bandbreitenmessung!
- Manuelle Berechnung der Download- und Uploadzeiten für individuell angegebene Bandbreiten und Datenmengen
- Messung der netzwerkinternen (LAN-)Bandbreite — praktisch, um eine langsame Dateiübertragung zwischen Netzwerk und Zielsystem einzugrenzen
- Messung der Festplattengeschwindigkeit via WinSat und/oder DiskSpd.exe (sequenziell und 4K-Random, inkl. IOPS)
- 💡 Bis zu 10 Messungen der aktuellen Sitzung werden zum direkten Vorher-/Nachher-Vergleich vorgehalten

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/7b0f4a76-2052-4359-8db9-88abdaf0afe7" />

### E-Mail & Domäne
- Analyse zuständiger MX-Mailserver einer E-Mail-Adresse und Test der Erreichbarkeit (echter SMTP-Verbindungsaufbau inkl. TLS und vollständigem Protokollmitschnitt), um Zustellungsprobleme zu analysieren
- Header-Analyse von E-Mail-Kopfzeilen — Routing-Pfad mit Zeitstempeln je Hop, SPF-/DKIM-/DMARC-/BIMI-Prüfung, Spam-Score
- E-Mail-Domain-Analyse nach MX-Server, SPF, DKIM, DMARC, BIMI und Auswertung der Zustellbarkeit zwischen zwei Mail-Domains aufgrund von SPF, DKIM, DMARC, inkl. Blacklist-Prüfung

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/bf53d8c3-a7a6-4c21-9fea-e5a59d1cd9c0" />

### Updates
- Aktualisierung von installierten Anwendungen via Winget
- Suche und Installation von Anwendungen via Winget
- 💡 PIN-Schutz für Pakete, die von einer automatischen Sammel-Aktualisierung ausgenommen werden sollen

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/7cf2641c-148e-43ae-9ebc-25eca669bba3" />
<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/de31f36d-6fd6-4636-b268-81936368288f" />

### Inventar
- Hardware- und Software-Inventarisierung des lokalen Rechners — ganz ohne vorherige Agent-Installation
- Erstellung eines Inventarisierungs-Agenten (siehe [DATEXTAgent](#-datextagent)) zur Verteilung im lokalen Netzwerk und zentrales Sammeln von Systeminformationen in einer verschlüsselten .json-Datei. Auswertung der gesammelten Geräteinformationen.
- Export nach PDF, XLSX, CSV, JSON

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/8b253484-bf84-4d04-a955-593c81f3f9da" />

---

## 🚀 Installation

DATEXT Diagnostics wird als **Self-Contained Single-File Executable** ausgeliefert — keine Installation erforderlich, keine .NET-Laufzeit auf dem Zielsystem notwendig.

```
DATEXT.Diagnostics.exe
```

### Systemvoraussetzungen

| Komponente      | Anforderung                                                        |
|-----------------|--------------------------------------------------------------------|
| Betriebssystem  | Windows 10 (1903+) / Windows 11                                    |
| Architektur     | x64                                                                |
| .NET Runtime    | Nicht erforderlich (Self-Contained)                                |
| Rechte          | Standard-Benutzer (einige Funktionen erfordern Admin-Rechte)       |
| Speicherplatz   | ca. 250 MB (inkl. Extraktionsordner)                               |

---

## 🔧 Optionale Komponenten

| Komponente            | Funktion                              | Bezug                        |
|-----------------------|----------------------------------------|-------------------------------|
| **Ookla Speedtest CLI** | Internet-Speedtest                  | Automatisch über App installierbar |
| **RSAT (AD-Tools)**   | Active Directory Abfragen             | Windows Features             |
| **DATEXTAgent**       | Remote-Inventarisierung über HTTP-API | Separat deploybar             |
| **DiskSpd**           | Disk-/Festplatten-Performance         | Separat deploybar             |

---

## 📤 Export-Formate

Alle Dialoge unterstützen den Export in folgende Formate:

- 📄 **PDF** — druckfertige Berichte mit DATEXT-Branding
- 📊 **XLSX** — Excel-kompatible Tabellen
- 📋 **CSV** — für Weiterverarbeitung in anderen Tools
- `{}` **JSON** — für API-Integration und Scripting
- 📝 **TXT** — einfache Textausgabe

---

## Stack

| Komponente         | Technologie                            |
|---------------------|-----------------------------------------|
| UI-Framework        | WPF / .NET 8 (Self-Contained, win-x64) |
| Windows-APIs        | WMI, P/Invoke, SetupAPI                |
| PDF-Export          | PDFsharp 6.x                           |
| Excel-Export        | Eigene, abhängigkeitsfreie OOXML-Generierung (ZIP + XML) |
| JSON                | Newtonsoft.Json                        |
| Aufgabenplanung     | TaskScheduler Managed Wrapper          |
| DNS-Abfragen        | DnsClient.NET                          |
| E-Mail (SMTP/MIME)  | MailKit / MimeKit                      |
| SNMP                | #SNMP Library (SharpSnmpLib)           |
| Netzwerk-Topologie  | GraphX for .NET                        |

<details>
<summary>Vollständige Paketliste inkl. transitiver Abhängigkeiten (ermittelt via <code>dotnet list package --include-transitive</code>)</summary>

**Direkte Pakete**

| Paket | Version |
|---|---|
| DnsClient | 1.8.0 |
| GraphX | 3.0.0 |
| Lextm.SharpSnmpLib | 12.5.7 |
| MailKit | 4.17.0 |
| MimeKit | 4.17.0 |
| Newtonsoft.Json | 13.0.4 |
| PdfSharp | 6.2.4 |
| System.Data.SqlClient | 4.9.1 |
| System.Management | 10.0.9 |
| System.ServiceProcess.ServiceController | 10.0.9 |
| System.Text.Encoding.CodePages | 10.0.9 |
| TaskScheduler | 2.12.2 |

**Transitive Pakete** (automatisch mitgezogen, im Self-Contained-Build enthalten)

| Paket | Version |
|---|---|
| BouncyCastle.Cryptography | 2.6.2 |
| Microsoft.Extensions.DependencyInjection.Abstractions | 8.0.2 |
| Microsoft.Extensions.Logging.Abstractions | 8.0.3 |
| Microsoft.Win32.Registry | 5.0.0 |
| QuickGraphCore | 1.0.0 |
| runtime.native.System.Data.SqlClient.sni (win-x64/x86/arm64) | 4.4.0 |
| System.CodeDom | 10.0.9 |
| System.Diagnostics.EventLog | 10.0.9 |
| System.Formats.Asn1 | 8.0.1 |
| System.Security.AccessControl | 6.0.1 |
| System.Security.Cryptography.Pkcs | 8.0.1 |
| System.Security.Principal.Windows | 5.0.0 |

Alle Pakete inkl. Lizenz, Copyright-Hinweis und konkretem Verwendungszweck innerhalb der Anwendung sind über **Hilfe → Open-Source-Lizenzen** im Programm einsehbar.

</details>

---

## 🤝 DATEXTAgent

Für die Remote-Inventarisierung steht der **DATEXTAgent** zur Verfügung — ein schlanker Windows-Dienst, der auf dem Zielsystem installiert wird und Hardware- sowie Softwareinformationen über eine lokale HTTP-API bereitstellt. DATEXT Diagnostics kann diese Daten abrufen und in den Inventar-Export integrieren. Die gesammelten Geräteinformationen werden verschlüsselt als .json-Datei zentral abgelegt und lassen sich anschließend im Inventar-Bereich der Hauptanwendung auswerten.

---

## 📞 Support & Kontakt

- 🌐 [www.datext.de](https://www.datext.de)
- 📧 [info@datext.de](mailto:info@datext.de)

---

## ⚖️ Lizenz

Copyright © 2024–2026 DATEXT Beratungsges. mbH. Alle Rechte vorbehalten.
Dieses Projekt ist nicht Open Source. Die Verwendung, Vervielfältigung oder Weitergabe
ohne ausdrückliche schriftliche Genehmigung der DATEXT Beratungsges. mbH ist nicht gestattet.

Eingesetzte Open-Source-Komponenten unterliegen ihren jeweiligen Lizenzen (siehe **Hilfe → Open-Source-Lizenzen** in der Anwendung).

---

*Entwickelt mit ❤️ in Hagen, NRW*
