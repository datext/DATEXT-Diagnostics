# DATEXT Diagnostics

**Professionelles Windows-Systemdiagnosetool für IT-Administratoren und technisch versierte Anwender**

![Version](https://img.shields.io/badge/Version-0.99.11.7-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%2010%2F11-lightgrey)
![Framework](https://img.shields.io/badge/.NET-8.0-purple)
![License](https://img.shields.io/badge/License-Proprietary-red)

---

## 📋 Übersicht

DATEXT Diagnostics ist eine umfassende Windows-Diagnoseanwendung die Systemadministratoren und IT-Professionals eine schnelle, strukturierte Übersicht über alle relevanten System-, Netzwerk- und Sicherheitsinformationen eines Windows-Rechners bietet — ohne Kommandozeile, ohne Scriptaufwand.

---

## Funktionen

## Übersicht
- Anzeige relevanter lokaler Syteminformationen mit farblich codierten Bagdes zur schnellen Übersicht
- Ermittlung und visuelle Anzeige von erkannten CG-NAT ISP Anschluß und / oder erkannter aktiver VPN Verbindung
<img width="1404" height="761" alt="image" src="https://github.com/user-attachments/assets/18d8ca00-f392-41e4-bf4c-478bec91bcac" />

## System Informationen
- gezielte Auswertung von verschiedenen Systeminformationen und Analysen
<img width="1324" height="762" alt="image" src="https://github.com/user-attachments/assets/e306945f-efad-4b96-b59d-806d8719b86b" />

## Netzwerk
- Netzwerk- und Gerätescan mit Gerätetyp-Erkennung via PING, ARP, DNS, NetBIOS, LLMNR, mDNS, SSDP und SNMP-Info, dazu Topologie Visualisierung
- diverse Tools zur Auswertung und Analyse von Netzwerkkarten, Netzwerkverbindungen, DHCP-Discovery ipV4/ipV6, DNS, Portscans u.v.m.
<img width="1324" height="762" alt="image" src="https://github.com/user-attachments/assets/ea35dbe8-26f5-4549-9a63-33f665c79441" />

## Active Directory
- mit installiertem RSAT kann ein Active Directory nach Benutzern / Computern durchsucht werden
- Analyse der zugewiesenen Group-Policies
<img width="1324" height="762" alt="image" src="https://github.com/user-attachments/assets/dce125ed-d2c1-4eb3-8fbb-1a7d04bf3e1b" />

## MS-SQL
- SQL-Instanz-Erkennung (Registry, SQL Browser, Broadcast)
- Datenbankanalyse mit Windows-Authentifizierung und Impersonation
<img width="1324" height="762" alt="image" src="https://github.com/user-attachments/assets/fbf5affa-b7a0-42a0-b3dc-bc391ffa5c9e" />

## Packet Capture
- Paket-Mitschnitt via PKTMON oder NetSH-Traces. Konvertierung der Mitschnitte von .etl nach .pcap zur Auswertung via WireShark
<img width="1324" height="762" alt="image" src="https://github.com/user-attachments/assets/35c34e8d-a6c0-4226-8cd0-199b30bcb075" />

## Performance
- Messung der Internet-Bandbreite mit dem <a href="https://www.speedtest.net/apps/cli" target="_blank">Ookla Speedtest CLI</a>. Erfordert Internetzugriff auf Port TCP 8080 für die Bandbreitenmessung!
- Manuelle Berechnung der Download- und Uploadzeiten für individuell angegebene Bandbreiten und Datenmengen
- Messung der netzwerkinternen Bandbreite
- Messung der Festplattengeschwindigkeit via WinSat und oder DiskSpd.exe
<img width="1324" height="762" alt="image" src="https://github.com/user-attachments/assets/e356e055-0d78-45c4-9018-46978e96f6dc" />

## E-Mail & Domäne
- Analyse zuständiger MX Mailserver einer Email-Adresse und Test der Erreichbarkeit um Zustellungsprobleme zu analysieren
- Header-Analyse von Email-Kopfzeilen
- E-Mail Domain Analyse nach MX-Server, SPF, DKIM, DMarc, BiMi und Auswertung der Zustellbarkeit zwischen zwei Mail-Domains aufgrund von SPF, DKIM, DMARC
<img width="1324" height="760" alt="image" src="https://github.com/user-attachments/assets/363f0f48-2a6e-4efa-8821-f5977f2f96fb" />

## Updates
- Aktualiserung von installierten Anwendungen via Winget
- Suche und Installation von Anwendungen via Winget
<img width="1324" height="760" alt="image" src="https://github.com/user-attachments/assets/7141c7d9-9eb6-4467-8b1b-f387b45254eb" />

## Inventar
- Hardware- und Software-Inventarisierung des lokalen Rechners
- Erstellung eines Inventarisierungs-Agenten zur Verteilung im lokalen Netzwerk und zentrales Sammeln von Systeminformationen in einer verschlüsselten .json Datei. Auswertung der gesammelten Geräteinformationen.
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
|-----------------------|---------------------------------------|------------------------------|
| **Ookla Speedtest CLI** | Internet-Speedtest                  | Automatisch über App installierbar |
| **RSAT (AD-Tools)**   | Active Directory Abfragen             | Windows Features             |
| **DATEXTAgent**       | Remote-Inventarisierung über HTTP-API | Separat deploybar            |
| **DiskSpd**           | Disk/Festplatten Performance          | Separat deploybar            |

---

## 📤 Export-Formate

Alle Dialoge unterstützen den Export in folgende Formate:

- 📄 **PDF** — druckfertige Berichte mit DATEXT-Branding
- 📊 **XLSX** — Excel-kompatible Tabellen
- 📋 **CSV** — für Weiterverarbeitung in anderen Tools
- `{}` **JSON** — für API-Integration und Scripting
- 📝 **TXT** — einfache Textausgabe

---

**Stack:**

| Komponente         | Technologie                          |
|--------------------|--------------------------------------|
| UI-Framework       | WPF / .NET 8 (Self-Contained, win-x64) |
| Windows-APIs       | WMI, P/Invoke, SetupAPI              |
| PDF-Export         | PDFsharp 6.x                         |
| Excel-Export       | ClosedXML                            |
| JSON               | Newtonsoft.Json                      |
| Aufgabenplanung    | TaskScheduler                        |

---

## 🤝 DATEXTAgent

Für die Remote-Inventarisierung steht der **DATEXTAgent** zur Verfügung — ein schlanker Windows-Dienst der auf dem Zielsystem installiert wird und Hardware- sowie Softwareinformationen über eine lokale HTTP-API bereitstellt. DATEXT Diagnostics kann diese Daten abrufen und in den Inventar-Export integrieren.

---

## 📞 Support & Kontakt

- 🌐 [www.datext.de](https://www.datext.de)
- 📧 [info@datext.de](mailto:info@datext.de)

---

## ⚖️ Lizenz

Copyright © 2024–2026 DATEXT GmbH. Alle Rechte vorbehalten.  
Dieses Projekt ist nicht Open Source. Die Verwendung, Vervielfältigung oder Weitergabe  
ohne ausdrückliche schriftliche Genehmigung der DATEXT GmbH ist nicht gestattet.

---

*Entwickelt mit ❤️ in Hagen, NRW*
