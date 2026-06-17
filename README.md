# DATEXT Diagnostics

**Professionelles Windows-Systemdiagnosetool für IT-Administratoren und technisch versierte Anwender**

![Version](https://img.shields.io/badge/Version-0.99.11.1-blue)
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
<img width="1566" height="937" alt="image" src="https://github.com/user-attachments/assets/caa119ff-9d95-4e2a-85ec-4652efea4e0f" />

## System Informationen
- gezielte Auswertung von verschiedenen Systeminformationen und Analysen
<img width="1566" height="937" alt="image" src="https://github.com/user-attachments/assets/f9874692-a399-47b5-804e-69f111f8d414" />

## Netzwerk
- Netzwerk- und Gerätescan mit Gerätetyp-Erkennung via PING, ARP, DNS, NetBIOS, LLMNR, mDNS, SSDP und SNMP-Info, dazu Topologie Visualisierung
- diverse Tools zur Auswertung und Analyse von Netzwerkkarten, Netzwerkverbindungen, DHCP-Discovery ipV4/ipV6, DNS, Portscans u.v.m.
<img width="1566" height="937" alt="image" src="https://github.com/user-attachments/assets/a0c09fbd-2a70-48d4-9bbe-29d76e9ff447" />

## Active Directory
- mit installiertem RSAT kann ein Active Directory nach Benutzern / Computern durchsucht werden
- Analyse der zugewiesenen Group-Policies
<img width="1566" height="937" alt="image" src="https://github.com/user-attachments/assets/4ada3c37-22e6-4fd0-a5dd-f0883b575760" />

## MS-SQL
- SQL-Instanz-Erkennung (Registry, SQL Browser, Broadcast)
- Datenbankanalyse mit Windows-Authentifizierung und Impersonation
<img width="1566" height="937" alt="image" src="https://github.com/user-attachments/assets/063c7b69-8065-4c68-b782-e4183f528134" />

## Packet Capture
- Paket-Mitschnitt via PKTMON oder NetSH-Traces. Konvertierung der Mitschnitte von .etl nach .pcap zur Auswertung via WireShark
<img width="1566" height="937" alt="image" src="https://github.com/user-attachments/assets/0347cbd4-9f29-4c9d-afa6-5e2a5cb86e58" />

## Performance
- Messung der Internet-Bandbreite mit dem <a href="https://www.speedtest.net/apps/cli" target="_blank">Ookla Speedtest CLI</a>. Erfordert Internetzugriff auf Port TCP 8080 für die Bandbreitenmessung!
- Manuelle Berechnung der Download- und Uploadzeiten für individuell angegebene Bandbreiten und Datenmengen
- Messung der netzwerkinternen Bandbreite
- Messung der Festplattengeschwindigkeit via WinSat und oder DiskSpd.exe
<img width="1566" height="937" alt="image" src="https://github.com/user-attachments/assets/9ed107db-91bd-4357-9e82-0904160e1490" />

## E-Mail & Domäne
- Analyse zuständiger MX Mailserver einer Email-Adresse und Test der Erreichbarkeit um Zustellungsprobleme zu analysieren
- Header-Analyse von Email-Kopfzeilen
- E-Mail Domain Analyse nach MX-Server, SPF, DKIM, DMarc, BiMi und Auswertung der Zustellbarkeit zwischen zwei Mail-Domains aufgrund von SPF, DKIM, DMARC
<img width="1566" height="937" alt="image" src="https://github.com/user-attachments/assets/2e8114bb-72b5-400e-9e0d-1a0f40e1be21" />

## Updates
- Aktualiserung von installierten Anwendungen via Winget
- Suche und Installation von Anwendungen via Winget
<img width="1566" height="937" alt="image" src="https://github.com/user-attachments/assets/592790bd-2012-41a0-bb1e-038c69b3af23" />

### Inventar
- Hardware- und Software-Inventarisierung des lokalen Rechners
- Erstellung eines Inventarisierungs-Agenten zur Verteilung im lokalen Netzwerk und zentrales Sammeln von Systeminformationen in einer verschlüsselten .json Datei. Auswertung der gesammelten Geräteinformationen.
- Export nach PDF, XLSX, CSV, JSON
<img width="1566" height="937" alt="image" src="https://github.com/user-attachments/assets/cf8e662f-b9ae-43be-b970-4750b970c7fb" />

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
