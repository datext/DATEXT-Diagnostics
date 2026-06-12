# DATEXT Diagnostics

**Professionelles Windows-Systemdiagnosetool für IT-Administratoren und technisch versierte Anwender**

![Version](https://img.shields.io/badge/Version-0.99.10.2-blue)
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
<img width="1421" height="864" alt="image" src="https://github.com/user-attachments/assets/377c6e4d-6f9c-479d-867b-a733ba8806b1" />

## System Informationen
- gezielte Auswertung von verschiedenen Systeminformationen und Analysen
<img width="1421" height="864" alt="image" src="https://github.com/user-attachments/assets/11c189b8-e50f-4adb-913e-e0f4b0d02dee" />

## Netzwerk
- Netzwerk- und Gerätescan mit Gerätetyp-Erkennung via PING, ARP, DNS, NetBIOS, LLMNR, mDNS, SSDP und SNMP-Info, dazu Topologie Visualisierung
- diverse Tools zur Auswertung und Analyse von Netzwerkkarten, Netzwerkverbindungen, DHCP-Discovery ipV4/ipV6, DNS, Portscans u.v.m.
<img width="1421" height="864" alt="image" src="https://github.com/user-attachments/assets/4fc9fe5b-07a2-4e92-b42c-fd08beee6ecc" />

## Active Directory
- mit installiertem RSAT kann ein Active Directory nach Benutzern / Computern durchsucht werden
- Analyse der zugewiesenen Group-Policies
<img width="1421" height="864" alt="image" src="https://github.com/user-attachments/assets/349cda09-65f0-4c7e-9e1d-d2364d4562cb" />

## MS-SQL
- SQL-Instanz-Erkennung (Registry, SQL Browser, Broadcast)
- Datenbankanalyse mit Windows-Authentifizierung und Impersonation
<img width="1421" height="864" alt="image" src="https://github.com/user-attachments/assets/a5f4d60d-05b6-4449-bf41-c488d0407b4a" />

## Packet Capture
- Paket-Mitschnitt via PKTMON oder NetSH-Traces. Konvertierung der Mitschnitte von .etl nach .pcap zur Auswertung via WireShark
<img width="1421" height="864" alt="image" src="https://github.com/user-attachments/assets/d51d2bee-f9f8-4205-a195-54e145fa1327" />

## Performance
- Messung der Internet-Bandbreite mit dem Ookla Speedtest CLI
- Messung der netzwerkinternen Bandbreite
- Messung der Festplattengeschwindigkeit via WinSat und oder DiskSpd.exe
<img width="1421" height="864" alt="image" src="https://github.com/user-attachments/assets/12f93131-b60d-4e31-92fc-cc209f172198" />

## E-Mail & Domäne
- Analyse zuständiger MX Mailserver einer Email-Adresse und Test der Erreichbarkeit um Zustellungsprobleme zu analysieren
- Header-Analyse von Email-Kopfzeilen
- E-Mail Domain Analyse nach MX-Server, SPF, DKIM, DMarc, BiMi und Auswertung der Zustellbarkeit zwischen zwei Mail-Domains aufgrund von SPF, DKIM, DMARC
<img width="1421" height="864" alt="image" src="https://github.com/user-attachments/assets/bde929d8-7f9a-4547-924d-b5ca813caf51" />

## Updates
- Aktualiserung von installierten Anwendungen via Winget
- Suche und Installation von Anwendungen via Winget

<img width="1421" height="864" alt="image" src="https://github.com/user-attachments/assets/641d1eb1-8d41-4668-9dbe-64d7cb385da8" />

### Inventar
- Hardware- und Software-Inventarisierung des lokalen Rechners
- Erstellung eines Inventarisierungs-Agenten zur Verteilung im lokalen Netzwerk und zentrales Sammeln von Systeminformationen in einer verschlüsselten .json Datei. Auswertung der gesammelten Geräteinformationen.
- Export nach PDF, XLSX, CSV, JSON
<img width="1421" height="864" alt="image" src="https://github.com/user-attachments/assets/6eee47fd-5e6b-4a10-8598-a74b8673059e" />

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
