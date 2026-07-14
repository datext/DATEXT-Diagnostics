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
- 💡 *Direkt-Aktionen:* IP-Adressen lassen sich per Symbol direkt an Ping, Traceroute, DNS-Auflösung oder Port-Scan übergeben — ohne sie manuell abzutippen

<img width="1400" height="926" alt="image" src="https://github.com/user-attachments/assets/f6a29a8b-4055-4af2-9bb5-bd247da98fb3" />

### System Informationen
- **System-Health** — Ampel-Bewertung von Hardware, Sicherheit, Speicherplatz, Netzwerk und Ereignissen; schnellster Weg zur Ersteinschätzung
- **UEFI-Boot-Zertifikat** prüfen — inkl. Erkennung von physischer vs. virtueller Maschine und plattformspezifischer Handlungsempfehlung
- **Autorun-Analyse** über Registry-Autostart, geplante Aufgaben, Autostart-Ordner und automatisch startende Dienste
- **Ereignis-Log-Auswertung**, auch mit Schnellfiltern zu gängigen Problemen und Ermittlung von zusammenhängenden Events
  - 💡 *Kontext-Korrelation:* verwandte Ereignisse im Umkreis von ±5 Minuten um einen ausgewählten Eintrag anzeigen
  - 💡 *Anonymisierter Export* für Support-Tickets, ohne sensible Nutzerdaten preiszugeben
- Auswertung installierter Software mit Such- und Exportfunktion
- **Windows-Komponenten-Prüfung und -Behebung** über SFC und DISM
- System-Verwaltungsprogramme im Schnellzugriff (services.msc, devmgmt.msc, compmgmt.msc, msconfig u. v. m.)

<img width="1404" height="761" alt="image" src="https://github.com/user-attachments/assets/b61fa338-450f-438e-bcdd-aa47736d8491" />

### Netzwerk
- Netzwerk- und Gerätescan mit Gerätetyp-Erkennung via PING, ARP, DNS, NetBIOS, LLMNR, mDNS, SSDP und SNMP-Info, dazu Topologie-Visualisierung
- **Ping** mit fortlaufender Statistik, Jitter, Paketverlust und automatischer MTU-Erkennung
- **Traceroute** mit Hop-für-Hop-Latenz, Reverse-DNS und Erkennung von CGNAT-/Doppel-NAT-Situationen im Pfad
- **DNS-Analyse** — A/AAAA/CNAME/NS/SOA/MX/TXT/SRV/PTR, Propagationsprüfung gegen mehrere öffentliche DNS-Server, IP-Zusatzinfos
- **Port-Scan** mit automatischer Diensterkennung, unterscheidet geschlossen (abgelehnt) von gefiltert (Firewall)
- **Netzwerk & Verbindungen** — Hub mit sechs Reitern (Netzwerkkarten, Routing-Tabelle, Verbindungen, Netzwerk-Analyse, Netzwerklast, Diagnose)
- **DHCP-Discovery** IPv4/IPv6 — erkennt alle antwortenden DHCP-Server, deckt Rogue-DHCP zuverlässig auf

<img width="1404" height="761" alt="image" src="https://github.com/user-attachments/assets/e54d5692-21e1-4870-906f-484d6f14b7c3" />

**Verbindungsanalyse mit Risikobewertung** — jede aktive TCP-/UDP-Verbindung wird dem auslösenden Prozess zugeordnet (inkl. Authenticode-Signaturprüfung), auffällige Ziel-IPs werden automatisch gegen VirusTotal und Shodan abgeglichen.

<img width="1404" height="761" alt="image" src="https://github.com/user-attachments/assets/b5ded776-9f07-46fb-bd29-79188f8afe12" />

### Active Directory
- mit installiertem RSAT kann ein Active Directory nach Benutzern/Computern durchsucht werden (automatische Domain-Controller-Erkennung, keine manuelle Serverangabe nötig)
- Analyse der zugewiesenen Group-Policies inkl. Verarbeitungsreihenfolge und Vererbung — deckt auf, warum eine erwartete Richtlinie auf einem Rechner nicht ankommt

<img width="1324" height="762" alt="image" src="https://github.com/user-attachments/assets/dce125ed-d2c1-4eb3-8fbb-1a7d04bf3e1b" />

### MS-SQL
- SQL-Instanz-Erkennung (Registry, SQL Browser, Broadcast)
- Datenbankanalyse mit Windows-Authentifizierung und Impersonation
- 💡 Deckt neben der reinen Erreichbarkeit auch Aktivität (laufende Abfragen, Sperren), Performance-Kennzahlen (u. a. Page Life Expectancy) und Konfiguration ab

<img width="1324" height="762" alt="image" src="https://github.com/user-attachments/assets/fbf5affa-b7a0-42a0-b3dc-bc391ffa5c9e" />

### Packet Capture
- Paket-Mitschnitt via PKTMON (direkt über den Windows-Kernel, ganz ohne Drittanbieter-Treiber) oder NetSH-Traces (ETW-basiert, häufig vom Microsoft-Support angefordert)
- Konvertierung der Mitschnitte von .etl nach .pcap zur Auswertung via WireShark

<img width="1324" height="762" alt="image" src="https://github.com/user-attachments/assets/35c34e8d-a6c0-4226-8cd0-199b30bcb075" />

### Performance
- Messung der Internet-Bandbreite mit dem <a href="https://www.speedtest.net/apps/cli" target="_blank">Ookla Speedtest CLI</a> (Download, Upload, Ping, Jitter, Paketverlust). Erfordert Internetzugriff auf Port TCP 8080 für die Bandbreitenmessung!
- Manuelle Berechnung der Download- und Uploadzeiten für individuell angegebene Bandbreiten und Datenmengen
- Messung der netzwerkinternen (LAN-)Bandbreite — praktisch, um eine langsame Dateiübertragung zwischen Netzwerk und Zielsystem einzugrenzen
- Messung der Festplattengeschwindigkeit via WinSat und/oder DiskSpd.exe (sequenziell und 4K-Random, inkl. IOPS)
- 💡 Bis zu 10 Messungen der aktuellen Sitzung werden zum direkten Vorher-/Nachher-Vergleich vorgehalten

<img width="1324" height="762" alt="image" src="https://github.com/user-attachments/assets/e356e055-0d78-45c4-9018-46978e96f6dc" />

### E-Mail & Domäne
- Analyse zuständiger MX-Mailserver einer E-Mail-Adresse und Test der Erreichbarkeit (echter SMTP-Verbindungsaufbau inkl. TLS und vollständigem Protokollmitschnitt), um Zustellungsprobleme zu analysieren
- Header-Analyse von E-Mail-Kopfzeilen — Routing-Pfad mit Zeitstempeln je Hop, SPF-/DKIM-/DMARC-/BIMI-Prüfung, Spam-Score
- E-Mail-Domain-Analyse nach MX-Server, SPF, DKIM, DMARC, BIMI und Auswertung der Zustellbarkeit zwischen zwei Mail-Domains aufgrund von SPF, DKIM, DMARC, inkl. Blacklist-Prüfung

<img width="1324" height="760" alt="image" src="https://github.com/user-attachments/assets/363f0f48-2a6e-4efa-8821-f5977f2f96fb" />

### Updates
- Aktualisierung von installierten Anwendungen via Winget
- Suche und Installation von Anwendungen via Winget
- 💡 PIN-Schutz für Pakete, die von einer automatischen Sammel-Aktualisierung ausgenommen werden sollen

<img width="1324" height="760" alt="image" src="https://github.com/user-attachments/assets/7141c7d9-9eb6-4467-8b1b-f387b45254eb" />

### Inventar
- Hardware- und Software-Inventarisierung des lokalen Rechners — ganz ohne vorherige Agent-Installation
- Erstellung eines Inventarisierungs-Agenten (siehe [DATEXTAgent](#-datextagent)) zur Verteilung im lokalen Netzwerk und zentrales Sammeln von Systeminformationen in einer verschlüsselten .json-Datei. Auswertung der gesammelten Geräteinformationen.
- Export nach PDF, XLSX, CSV, JSON

<img width="1324" height="760" alt="image" src="https://github.com/user-attachments/assets/19742715-d5be-4342-9f20-7c5284205e05" />

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
