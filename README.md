
# DATEXT Diagnostics

**Windows-Diagnosetool für IT-Administratoren und technisch versierte Anwender**

![Version](https://img.shields.io/badge/Version-0.99.12.01-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%2010%2F11-lightgrey)
![Framework](https://img.shields.io/badge/.NET-8.0-purple)
![License](https://img.shields.io/badge/License-Proprietary-red)

[Auf einen Blick](#auf-einen-blick) · [Installation](#installation) · [Funktionen](#funktionen) · [Berichte und Export](#berichte-und-export) · [DATEXTAgent](#datextagent) · [Hinweise](#hinweise) · [Support](#support)

</div>

---

## Auf einen Blick

DATEXT Diagnostics bündelt die Werkzeuge, die man bei der Fehlersuche an Windows-Rechnern und im Netzwerk ständig braucht, in einer Anwendung. Ohne Kommandozeile und ohne Skripte.

- **Übersicht und Bewertung:** Systemzustand, Sicherheit und Netzwerk auf einen Blick, mit farbiger Bewertung
- **Netzwerk:** Geräte-Scan, Ping, Traceroute, DNS, Port-Scan, DHCP-Discovery, Verbindungsanalyse
- **E-Mail:** Verbindungs- und Sicherheitsprüfung von Mailservern, Domain- und Header-Analyse
- **Active Directory und MS-SQL:** Suche, Gruppenrichtlinien, Datenbankanalyse
- **Inventar:** Hardware und Software lokal erfassen und bewerten, auf Wunsch zentral über den DATEXTAgent
- **Messungen:** Internet-, LAN- und Festplattengeschwindigkeit
- **Einheitliche PDF-Berichte** aus allen Bereichen

---

## Installation

Aktuell gibt es eine selbstextrahierende EXE die ohne installierte .NET-Laufzeit auskommt.

| Variante | Beschreibung |
|---|---|
| **Portable EXE** (`DATEXT-Diagnostics.exe`) | Eine einzelne Datei, keine Installation. Beim ersten Start wird die Laufzeit nach `%TEMP%\.net\DATEXT-Diagnostics\<Hashwert>` entpackt. Geeignet für Admin-Stick und einmalige Einsätze. |

Die Programmdatei ist digital signiert.

### Systemvoraussetzungen

| Komponente | Anforderung |
|---|---|
| Betriebssystem | Windows 10 (1903+) / Windows 11, auch aktuelle Windows-Server-Versionen |
| Architektur | x64 |
| .NET-Laufzeit | Nicht erforderlich (Self-Contained) |
| Rechte | Standardbenutzer; einzelne Funktionen (BitLocker, Überwachungsrichtlinie, Paketmitschnitt u. a.) brauchen Administratorrechte. Die Anwendung weist darauf hin und startet auf Wunsch als Administrator neu. |
| Speicherplatz | ca. 250 MB |

### Optionale Komponenten

| Komponente | Funktion | Bezug |
|---|---|---|
| Ookla Speedtest CLI | Internet-Speedtest | Lässt sich aus der Anwendung installieren |
| DiskSpd | Festplattentest mit IOPS | Lässt sich aus der Anwendung installieren |
| RSAT (AD-Tools) | Active-Directory-Abfragen | Windows-Feature |
| DATEXTAgent | Remote-Inventarisierung | Wird aus der Anwendung erstellt und verteilt |

---

## Funktionen

### Übersicht
Startbildschirm mit den beim Programmstart ermittelten Kernwerten des Systems.

- Anzeige von System, Energie, Ressourcen, Festplatten, Netzwerk, IT-Sicherheit und Internet, mit farbigen Badges
- Erkennung von CG-NAT-Anschlüssen und aktiven VPN-Verbindungen
- **Bewertung:** Auf Wunsch wird das System inventarisiert und bewertet, Probleme stehen zuerst
- *Direkt-Aktionen:* IP-Adressen lassen sich per Symbol an Ping, Traceroute, DNS oder Port-Scan übergeben. Bei Wortmann-Systemen kann die Seriennummer an die Seriennummernsuche übergeben werden.

<img width="1423" height="758" alt="image" src="https://github.com/user-attachments/assets/1f2f9d60-3eba-4ced-bd36-0c500064e4ea" />
<img width="1423" height="758" alt="image" src="https://github.com/user-attachments/assets/af007db6-dfe6-4f3f-8e2a-ac7c505810b2" />


<details>
<summary><b>System-Informationen</b></summary>

- **System-Health:** Ampel-Bewertung von Hardware, Sicherheit, Speicherplatz, Netzwerk und Ereignissen, der schnellste Weg zur Ersteinschätzung
- **UEFI-Boot-Zertifikat:** Prüfung der Zertifikatsumstellung 2011 → 2023, mit Erkennung physischer oder virtueller Maschine und passender Handlungsempfehlung
- **Autorun-Analyse:** Registry-Autostart, geplante Aufgaben, Autostart-Ordner und automatisch startende Dienste, mit Signaturprüfung und Risikobewertung
- **Ereignislog-Viewer** mit Schnellfiltern für gängige Probleme
  - *Kontext-Korrelation:* verwandte Ereignisse im Umkreis von ±5 Minuten um einen Eintrag
  - *Anonymisierter Export* für Support-Tickets
- **Installierte Software** mit Such- und Exportfunktion
- **System-Verwaltungsprogramme** im Schnellzugriff (services.msc, devmgmt.msc, compmgmt.msc, msconfig u. v. m.)
- **Troubleshooting:** Komponentenprüfung (DISM / SFC), Windows-Update-Reparatur, Netzwerk-Resets und weitere Werkzeuge


<img width="1423" height="758" alt="image" src="https://github.com/user-attachments/assets/4b0bfc02-5e67-49ec-8b21-0b4ba4601fd2" />

</details>

<details>
<summary><b>Netzwerk</b></summary>

- **Geräte-Scan** mit Typ-Erkennung über Ping, ARP, DNS, NetBIOS, LLMNR, mDNS, SSDP, WS-Discovery und SNMP. Zusätzlich werden Dienste-Banner (HTTP/HTTPS, SSH, FTP) gelesen und Windows-Informationen über SMB ermittelt, ohne Anmeldung. Dazu eine Topologie-Ansicht.
- **Ping** mit fortlaufender Statistik, Jitter, Paketverlust und automatischer MTU-Erkennung
- **Traceroute** mit Hop-für-Hop-Latenz, Reverse-DNS und Erkennung von CGNAT-/Doppel-NAT im Pfad
- **DNS-Analyse:** A/AAAA/CNAME/NS/SOA/MX/TXT/SRV/PTR, DNSSEC und CAA, MTA-STS, TLS-RPT, SPF mit Zählung der DNS-Lookups, Propagation gegen mehrere öffentliche Server, IP-Zusatzinfos, Bewertung und Vergleich mit der letzten Abfrage
- **Port-Scan** mit Diensterkennung; unterscheidet geschlossen (abgelehnt) von gefiltert (Firewall)
- **Netzwerk und Verbindungen:** Netzwerkkarten, Routing-Tabelle, Verbindungen, Netzwerk-Analyse, Netzwerklast und Diagnose
- **DHCP-Discovery** für IPv4 und IPv6: erkennt alle antwortenden DHCP-Server und deckt Rogue-DHCP auf

<img width="1420" height="754" alt="image" src="https://github.com/user-attachments/assets/f45092d0-1fa3-46fc-9a2f-59c6bd5e0b7c" />

**Verbindungsanalyse mit Risikobewertung:** Jede aktive TCP-/UDP-Verbindung wird dem auslösenden Prozess zugeordnet (inkl. Authenticode-Prüfung). Auffällige Ziel-IPs werden gegen VirusTotal und Shodan abgeglichen.

<img width="1420" height="754" alt="image" src="https://github.com/user-attachments/assets/a1b4bad6-5180-4bd0-ab38-30da1a6bde5a" />

</details>

<details>
<summary><b>E-Mail und Domäne</b></summary>

- **E-Mail-Check:** Ermittelt den zuständigen Server, prüft Verbindung, TLS, Zertifikat und Anmeldeverfahren, testet alle Ports (587, 465, 25) auf einmal und versendet auf Wunsch eine Testmail. Die Sicherheitsanalyse bewertet SPF, DMARC, DKIM, MX, PTR, Blacklist, TLS und DANE. Vollständige Protokollmitschnitte.
- **E-Mail-Domain-Analyse:** MX, SPF, DKIM, DMARC, BIMI, Blacklist-Prüfung und Auswertung der Zustellbarkeit zwischen zwei Domains
- **Header-Analyse:** Routing-Pfad mit Zeitstempeln je Hop, SPF-/DKIM-/DMARC-/BIMI-Prüfung, Spam-Score

<img width="1420" height="754" alt="image" src="https://github.com/user-attachments/assets/b6dc0ac6-85a0-4a23-bb00-1700639e798e" />

</details>

<details>
<summary><b>Active Directory</b></summary>

- Suche nach Benutzern und Computern (mit installiertem RSAT), automatische Domain-Controller-Erkennung, Abgleich mit dem Netzwerk-Scan
- Analyse der Gruppenrichtlinien inkl. Verarbeitungsreihenfolge und Vererbung: zeigt, warum eine erwartete Richtlinie auf einem Rechner nicht ankommt

<img width="1420" height="754" alt="image" src="https://github.com/user-attachments/assets/267474c7-37be-4309-83d2-f417e65c06b7" />

</details>

<details>
<summary><b>MS-SQL</b></summary>

- Instanz-Erkennung über Registry, SQL Browser und Broadcast
- Analyse mit Windows-Authentifizierung und Impersonation
- Neben der Erreichbarkeit auch Aktivität (laufende Abfragen, Sperren), Performance-Kennzahlen (u. a. Page Life Expectancy), Konfiguration, Sicherheitsprüfung, Backups, Indizes und Agent-Jobs

<img width="1420" height="754" alt="image" src="https://github.com/user-attachments/assets/dc7d534c-5b37-480f-8a26-747c24d40ea8" />

</details>

<details>
<summary><b>Paketmitschnitt</b></summary>

- Mitschnitt über **PKTMON** (direkt über den Windows-Kernel, ohne Drittanbieter-Treiber) oder **netsh trace** (ETW-basiert, wird häufig vom Microsoft-Support angefordert)
- Konvertierung von .etl nach .pcap zur Auswertung mit Wireshark

<img width="1420" height="754" alt="image" src="https://github.com/user-attachments/assets/bdd28c6a-6472-4cf0-b0f5-8f34d23e83af" />

</details>

<details>
<summary><b>Performance</b></summary>

- **Internet-Speedtest** mit dem Ookla Speedtest CLI: Live-Messung (Download, Upload, Ping, Jitter, Paketverlust, Latenz unter Last), Serverauswahl, Ergebnislink und Messverlauf mit CSV-Export. Benötigt Internetzugriff auf TCP 8080.
- **Transferzeiten:** Berechnung für Standardgrößen oder eigene Bandbreiten und Datenmengen
- **Netzwerk-Speedtest (LAN):** Messung zu einer SMB-Freigabe mit automatischer Adapterwahl, Latenz, Min/Max über mehrere Läufe und Bewertung relativ zur Linkgeschwindigkeit, praktisch zum Eingrenzen langsamer Dateiübertragungen
- **Disk-Speedtest** mit WinSAT und/oder DiskSpd (sequenziell und 4K-Random, inkl. IOPS)

<img width="1420" height="754" alt="image" src="https://github.com/user-attachments/assets/3516f320-4576-4d0e-a7f2-d8a1d9632abe" />

</details>

<details>
<summary><b>Updates</b></summary>

- Aktualisierung installierter Anwendungen über Winget
- Suche und Installation von Anwendungen
- PIN-Schutz für Pakete, die von der Sammel-Aktualisierung ausgenommen werden sollen

<img width="1420" height="754" alt="image" src="https://github.com/user-attachments/assets/5b53d21e-755e-4ce5-9a63-71f8cc429a33" />

</details>

<details>
<summary><b>Inventar</b></summary>

- **Lokale Inventarisierung** ohne vorherige Agent-Installation: Hardware, Software, Updates, Dienste, Autostart, Netzwerk, Freigaben, Drucker, Benutzer und Gruppen, Sicherheit (BitLocker, NTLM, SMB-Signierung, Credential Guard, Überwachungsrichtlinie, bei Domänenmitgliedern auch Gruppenrichtlinien und LAPS)
- **Bewertung** der Ergebnisse, Probleme zuerst
- **Vergleich** mit einem gespeicherten Inventar: neue oder entfernte Software, Updates, Benutzer, Dienste
- Speichern verschlüsselt oder als JSON, ausführlicher PDF-Bericht
- **Agent-Verwaltung:** Erstellung und Verteilung des DATEXTAgent, zentrales Sammeln und Auswerten (siehe [DATEXTAgent](#datextagent))

<img width="1420" height="754" alt="image" src="https://github.com/user-attachments/assets/30fabe16-d885-46ed-ad66-d53e4191c4ef" />

</details>

---

## Berichte und Export

Alle Berichte sind in einem einheitlichen PDF-Layout aufgebaut: DATEXT-Kopf, Titelblock, Werte in zwei Spalten, echte Tabellen und farbige Bewertungsblöcke. Weitere Formate gibt es je nach Dialog.

| Format | Verwendung |
|---|---|
| **PDF** | Druckfertige Berichte |
| **XLSX / CSV** | Tabellen zur Weiterverarbeitung (z. B. Active Directory, Software, Messverläufe) |
| **JSON** | Für Skripte und den Import in andere Systeme, auch verschlüsselt für Inventare |
| **TXT / Zwischenablage** | Schnelles Weitergeben, z. B. in Tickets |

---

## DATEXTAgent

Für die Remote-Inventarisierung gibt es den **DATEXTAgent**, einen schlanken Windows-Dienst. Er wird aus DATEXT Diagnostics erstellt und im Netzwerk verteilt und erfasst Hardware und Software des Zielsystems. Die gesammelten Geräteinformationen werden verschlüsselt als .json-Datei zentral abgelegt und lassen sich im Inventar-Bereich auswerten und vergleichen.

---

## Hinweise

- **Virenscanner:** Einzelne Virenscanner melden die EXE trotz digitaler Signatur als Fehlalarm. Ursache ist eine generische Heuristik für Self-Contained-.NET-Programme, die Netzwerk- und WMI-Abfragen durchführen. Ein tatsächlicher Schadcode-Treffer liegt nicht vor.
- **Geräte-Scan und Windows 11 24H2 / Server 2025:** Die SMB-Erkennung zeigt für diese und neuere Versionen immer „Build 26100“. Das ist eine Eigenheit von Windows, kein Fehler.
- **Gruppenrichtlinien über VPN:** Die Abfrage der Gruppenrichtlinien braucht Zugriff auf den Domänencontroller. Über langsame Verbindungen kann sie nach 90 Sekunden abbrechen; das Inventar weist dann in den Warnungen darauf hin.

---

## Stack

| Komponente | Technologie |
|---|---|
| UI | WPF / .NET 8 (Self-Contained, win-x64) |
| Windows-APIs | WMI, P/Invoke, SetupAPI |
| PDF-Export | PDFsharp 6.x |
| Excel-Export | Eigene, abhängigkeitsfreie OOXML-Erzeugung (ZIP + XML) |
| JSON | Newtonsoft.Json |
| Aufgabenplanung | TaskScheduler Managed Wrapper |
| DNS | DnsClient.NET |
| E-Mail (SMTP/MIME) | MailKit / MimeKit |
| SNMP | #SNMP Library (SharpSnmpLib) |
| Netzwerk-Topologie | GraphX for .NET |
| Installer | WiX Toolset 5 |

<details>
<summary>Vollständige Paketliste inkl. transitiver Abhängigkeiten</summary>

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

**Transitive Pakete** (im Self-Contained-Build enthalten)

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

Alle Pakete mit Lizenz, Copyright-Hinweis und Verwendungszweck sind in der Anwendung unter **Hilfe → Open-Source-Lizenzen** einsehbar.

</details>

---

## Support

- Web: [www.datext.de](https://www.datext.de)
- E-Mail: [info@datext.de](mailto:info@datext.de)
- Änderungen je Version: siehe [CHANGELOG.md](CHANGELOG.md)

---

## Lizenz

Copyright © 2024–2026 DATEXT Beratungsges. mbH. Alle Rechte vorbehalten.
Dieses Projekt ist nicht Open Source. Die Verwendung, Vervielfältigung oder Weitergabe ohne ausdrückliche schriftliche Genehmigung der DATEXT Beratungsges. mbH ist nicht gestattet.

Eingesetzte Open-Source-Komponenten unterliegen ihren jeweiligen Lizenzen (siehe **Hilfe → Open-Source-Lizenzen** in der Anwendung).

---

*Entwickelt in Hagen, NRW*
