<div align="center">

# DATEXT Diagnostics

**Windows-Diagnosetool für IT-Administratoren und technisch versierte Anwender**

![Version](https://img.shields.io/badge/Version-0.99.12.09-blue)
![Platform](https://img.shields.io/badge/Platform-Windows%2010%2F11-lightgrey)
![Framework](https://img.shields.io/badge/.NET-8.0-purple)
![License](https://img.shields.io/badge/License-Proprietary-red)

[Auf einen Blick](#auf-einen-blick) · [Installation](#installation) · [Funktionen](#funktionen) · [Berichte und Export](#berichte-und-export) · [DATEXTAgent](#datextagent) · [Hinweise](#hinweise) · [Support](#support)

</div>

---

## Auf einen Blick

DATEXT Diagnostics bündelt die Werkzeuge, die man bei der Fehlersuche an Windows-Rechnern und im Netzwerk ständig braucht, in einer Anwendung. Ohne Kommandozeile und ohne Skripte.

- **Übersicht und Bewertung:** Systemzustand, Sicherheit und Netzwerk auf einen Blick, mit farbiger Bewertung
- **System:** System-Health,UEFI-Boot-Zertifikat, Autostart-Analyse, Ereignislog-Viewer, installierte Software, Winget-Paketverwaltung, System-Verwaltung, Troubleshooting-Werkzeuge
- **Netzwerk:** Geräte-Scan, Ping & Latenz, Traceroute, IP & DNS Analyse, Port-Scan, Verbindungsanalyse, DHCP-Discovery, Firewall-Log-Auswertung
- **E-Mail:** Verbindungs- und Sicherheitsprüfung von Mailservern, Domain- und Header-Analyse
- **Server & Dienste:** Active Directory (Objektsuche, Gruppenrichtlinien, Sicherheits- und Zustandsanalyse), Analyse von IIS (Websites, FTP, Protokolle), Hyper-V (VMs, Datenträger, Speicherorte), MS-SQL (Datenbankanalyse)
- **Packet Capture:** Mitschnitt über PKTMON oder netsh trace, Konvertierung nach .pcap
- **Speedtests:** Internet-Speedtest, LAN-Speedtest, Disk-Speedtest
- **Email & Domain:** Verbindungs- und Sicherheitsprüfung von Mailservern, Domain- und Header-Analyse
- **Inventar:** Hardware und Software lokal erfassen und bewerten, auf Wunsch zentral über den DATEXTAgent
- **Einheitliche PDF-Berichte** aus allen Bereichen

---

## Installation

Es gibt eine portable EXE. Sie kommen ohne installierte .NET 8 Laufzeit aus.

| Variante | Beschreibung |
|---|---|
| **Portable EXE** (`DATEXT-Diagnostics.exe`) | Eine einzelne Datei, keine Installation. Beim Start entpackt .NET die Laufzeit nach `%TEMP%\.net\DATEXT-Diagnostics\<Hashwert>` (Standardverhalten von Single-File-Anwendungen). Geeignet für Admin-Stick und einmalige Einsätze. |

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
| RSAT (AD-Tools) | Benutzer- und Computersuche im Active Directory (die AD-Analyse „AD-Sicherheit & Zustand“ braucht kein RSAT) | Windows-Feature |
| DATEXTAgent | Remote-Inventarisierung | Wird aus der Anwendung erstellt und verteilt |

---

## Funktionen

### Übersicht
Startbildschirm mit den beim Programmstart ermittelten Kernwerten des Systems.

- Anzeige von System, Energie, Ressourcen, Festplatten, Netzwerk, IT-Sicherheit und Internet, mit farbigen Badges
- Erkennung von CG-NAT-Anschlüssen und aktiven VPN-Verbindungen
- **Bewertung:** Auf Wunsch wird das System inventarisiert und bewertet, Probleme stehen zuerst
- *Direkt-Aktionen:* IP-Adressen lassen sich per Symbol an Ping, Traceroute, DNS oder Port-Scan übergeben. Bei Wortmann-Systemen kann die Seriennummer an die Seriennummernsuche übergeben werden.

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/fbb1054d-b0f3-4428-bbbd-b16f44546b98" />
<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/374594d9-8324-4da3-9248-f0946cd46543" />

<details>
<summary><b>System</b></summary>

- **System-Health:** Ampel-Bewertung von Hardware, Sicherheit, Speicherplatz, Netzwerk und Ereignissen, der schnellste Weg zur Ersteinschätzung
- **UEFI-Boot-Zertifikat:** Prüfung der Zertifikatsumstellung 2011 → 2023, mit Erkennung physischer oder virtueller Maschine und passender Handlungsempfehlung
- **Autorun-Analyse:** Registry-Autostart, geplante Aufgaben, Autostart-Ordner und automatisch startende Dienste, mit Signaturprüfung und Risikobewertung
- **Ereignislog-Viewer** mit Schnellfiltern für gängige Probleme, nach Themen gruppiert und einklappbar (Anwendungen, System & Stabilität, Sicherheit & Konten, Netzwerk, Active Directory; rollenabhängig Hyper-V, Cluster, SQL Server und IIS), dazu Kontextmenü zum Kopieren einzelner Angaben
  - *Kontext-Korrelation:* verwandte Ereignisse im Umkreis von ±5 Minuten um einen Eintrag
  - *Anonymisierter Export* für Support-Tickets
- **Installierte Software** mit Such- und Exportfunktion
- **Winget-Pakete:** Aktualisierung installierter Anwendungen über Winget, Suche und Installation von Anwendungen, PIN-Schutz für Pakete, die von der Sammel-Aktualisierung ausgenommen werden sollen
- **System-Verwaltungsprogramme** im Schnellzugriff (services.msc, devmgmt.msc, compmgmt.msc, msconfig u. v. m.)
- **Troubleshooting:** Komponentenprüfung (DISM / SFC), Windows-Update-Reparatur, Netzwerk-Resets und weitere Werkzeuge

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/96ea53d2-8351-4bdc-b790-295ece8ac79f" />
<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/b8b0505c-cc90-44bf-9ab1-bc6b9ac5f85e" />

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/7cf2641c-148e-43ae-9ebc-25eca669bba3" />
<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/de31f36d-6fd6-4636-b268-81936368288f" />

</details>

<details>
<summary><b>Netzwerk</b></summary>

- **Geräte-Scan** mit Typ-Erkennung über Ping, ARP, DNS, NetBIOS, LLMNR, mDNS, SSDP, WS-Discovery und SNMP. Zusätzlich werden Dienste-Banner (HTTP/HTTPS, SSH, FTP) gelesen und Windows-Informationen über SMB ermittelt, ohne Anmeldung. Die Ergebnisse stehen in einer Liste mit Kacheln je Gerätetyp, Suchfeld und Kontextmenü; Geräte mit mehreren IP-Adressen werden zusammengefasst. Dazu eine Topologie-Ansicht, die Switches, Verbindungen und Endgeräte aus SNMP-Daten (LLDP, Bridge-MIB, STP) anordnet.
- **Ping** mit fortlaufender Statistik, Jitter, Paketverlust und automatischer MTU-Erkennung
- **Traceroute** mit Hop-für-Hop-Latenz, Reverse-DNS und Erkennung von CGNAT-/Doppel-NAT im Pfad
- **IP & DNS Analyse:** A/AAAA/CNAME/NS/SOA/MX/TXT/SRV/PTR, DNSSEC und CAA, MTA-STS, TLS-RPT, SPF mit Zählung der DNS-Lookups, Propagation gegen mehrere öffentliche Server, IP-Zusatzinfos, Bewertung und Vergleich mit der letzten Abfrage; zusätzlich direkte Abfrage der autoritativen Nameserver, hängende CNAME/MX-Verweise, DNSSEC-Kette mit Signaturprüfung, SERVFAIL-Diagnose, TTL, Wildcard, Netz/AS per DNS und erkannte Dienste
- **Port-Scan** mit Diensterkennung; unterscheidet geschlossen (abgelehnt) von gefiltert (Firewall)
- **Netzwerk und Verbindungen:** Netzwerkkarten, Routing-Tabelle, Verbindungen, Netzwerk-Analyse, Netzwerklast und Diagnose
- **DHCP-Discovery** für IPv4 und IPv6: erkennt alle antwortenden DHCP-Server und deckt Rogue-DHCP auf
- **Firewall-Log-Auswertung:** liest Logs der Securepoint UTM, von pfSense/OPNsense, Linux-Diensten (sshd, fail2ban, Postfix, Dovecot) und Schlüssel=Wert-Logs (z. B. FortiGate, Sophos) oder eine reine IP-Liste. Die Quelladressen gesperrter oder fehlgeschlagener Anmeldungen werden nach Betreiber (ASN) und Netz (BGP-Präfix) gruppiert; daraus entsteht eine Sperrliste für die Firewall. VPN-Sitzungen werden mit Beginn und Ende erfasst.

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/d1833c8e-1502-43b9-b3f1-6fbb31a02126" />

**Verbindungsanalyse mit Risikobewertung:** Jede aktive TCP-/UDP-Verbindung wird dem auslösenden Prozess zugeordnet (inkl. Authenticode-Prüfung). Auffällige Ziel-IPs werden gegen VirusTotal und Shodan abgeglichen.

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/f23955bd-87d1-4ba6-b872-9ed2200aa92a" />

</details>

<details>
<summary><b>Server & Dienste</b></summary>

- **AD-Suche:** Suche nach Benutzern und Computern (mit installiertem RSAT), automatische Domain-Controller-Erkennung, Abgleich mit dem Netzwerk-Scan
- **Gruppenrichtlinien:** Analyse der Gruppenrichtlinien inkl. Verarbeitungsreihenfolge und Vererbung: zeigt, warum eine erwartete Richtlinie auf einem Rechner nicht ankommt
- **AD-Sicherheit & Zustand:** bewertet ein Active Directory rein lesend und direkt per LDAP, ohne RSAT und ohne PowerShell, mit der aktuellen Anmeldung oder gegen eine fremde Domäne mit eigenen Zugangsdaten. Die Befunde mit Stufe, Objekt und Empfehlung stammen aus vier Bereichen:
  - *Konten und Privilegien:* inaktive Konten, Kennwortprobleme, Kerberos-Risiken (Kerberoasting, AS-REP, Delegierung), `krbtgt`, LAPS, veraltete Betriebssysteme, verschachtelte Admin-Gruppen
  - *Domäne und Richtlinien:* Funktionsebenen, Kennwort- und Sperrrichtlinie, FSMO-Rollen, Vertrauensstellungen, Papierkorb, Standorte und Subnetze
  - *Domänencontroller:* Ports, DNS, Zeitabweichung, SYSVOL/DFSR, Replikation je Partition, LDAP-Signierung, Channel Binding, SMB1
  - *Sicherheitsereignisse:* Kontosperrungen und Fehlanmeldungen mit Einschätzung (Password Spraying, Brute-Force). Ist ein Controller nicht lesbar, nennt der Dialog die Ursache (Namensauflösung, RPC-Ports, Berechtigung).
- **MS-SQL-Analyse:**
  - Instanz-Erkennung über Registry, SQL Browser und Broadcast
  - Analyse mit Windows-Authentifizierung und Impersonation
  - Neben der Erreichbarkeit auch Aktivität (laufende Abfragen, Sperren), Performance-Kennzahlen (u. a. Page Life Expectancy), Konfiguration, Sicherheitsprüfung, Backups, Indizes und Agent-Jobs
- **IIS-Analyse:** liest die `applicationHost.config` (auch eine kopierte Datei) und bewertet Sites, Bindings, Zertifikate, Anwendungspools und FTP-Sites (SSL-Richtlinie, Authentifizierung, Autorisierung). Auf dem lokalen Server kommen Laufzeitstatus der Sites, Pools und Dienste dazu. Die Auswertung der Web- und FTP-Protokolle zeigt auffällige Absender (Brute-Force, Scanner, Sondierung) und die langsamsten und fehlerhaftesten URLs je Website; eine Sperrliste fasst Angreifer-Netze für die Firewall zusammen.
- **Hyper-V-Analyse:** bewertet virtuelle Maschinen, Datenträger (auch VHD-Sets, geteilte Datenträger und Pass-Through), Prüfpunkte und Speicherorte, einschließlich Cluster Shared Volumes und SMB-Freigaben, mit freiem Platz und möglichem Wachstum dynamischer Datenträger.

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/39984655-d296-4e49-af89-422742c661ef" />
<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/6ee1c87b-3652-4f8e-9882-12bccc3fcee3" />

</details>

<details>
<summary><b>Packet Capture</b></summary>

- Mitschnitt über **PKTMON** (direkt über den Windows-Kernel, ohne Drittanbieter-Treiber) oder **netsh trace** (ETW-basiert, wird häufig vom Microsoft-Support angefordert)
- Konvertierung von .etl nach .pcap zur Auswertung mit Wireshark

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/c93510c4-c6a3-4b1e-90d1-70d83d0b4e48" />

</details>

<details>
<summary><b>Speedtests</b></summary>

- **Internet-Speedtest** mit dem Ookla Speedtest CLI: Live-Messung (Download, Upload, Ping, Jitter, Paketverlust, Latenz unter Last), Serverauswahl, Ergebnislink und Messverlauf mit CSV-Export. Benötigt Internetzugriff auf TCP 8080.
  - *Transferzeiten* (im Internet-Speedtest): Berechnung für Standardgrößen oder eigene Bandbreiten und Datenmengen
- **Netzwerk-Speedtest (LAN):** Messung zu einer SMB-Freigabe mit automatischer Adapterwahl, Latenz, Min/Max über mehrere Läufe und Bewertung relativ zur Linkgeschwindigkeit, praktisch zum Eingrenzen langsamer Dateiübertragungen
- **Disk-Speedtest** mit WinSAT und/oder DiskSpd (sequenziell und 4K-Random, inkl. IOPS)

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/7b0f4a76-2052-4359-8db9-88abdaf0afe7" />

</details>

<details>
<summary><b>E-Mail & Domain</b></summary>

- **E-Mail-Check:** Ermittelt den zuständigen Server, prüft Verbindung, TLS, Zertifikat und Anmeldeverfahren, testet alle Ports (587, 465, 25) auf einmal und versendet auf Wunsch eine Testmail, bei Microsoft 365 per Anmeldung im Browser über Microsoft Graph (SMTP AUTH ist dort meist deaktiviert). Die Sicherheitsanalyse nutzt dieselbe DNS-Auswertung wie die E-Mail-Analyse und bewertet SPF, DMARC, DKIM, MX, MTA-STS, TLS-RPT, PTR, Blacklist, TLS und DANE. Vollständige Protokollmitschnitte.
- **Header-Analyse:** Routing-Pfad mit Zeitstempeln je Hop, SPF-/DKIM-/DMARC-/BIMI-Prüfung, Spam-Score
- **E-Mail-Domain-Analyse:** MX, SPF, DKIM, DMARC, BIMI, Blacklist-Prüfung und Auswertung der Zustellbarkeit zwischen zwei Domains

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/bf53d8c3-a7a6-4c21-9fea-e5a59d1cd9c0" />

</details>

<details>
<summary><b>Inventar</b></summary>

- **Lokale Inventarisierung** ohne vorherige Agent-Installation: Hardware, Software, Updates, Dienste, Autostart, Netzwerk, Freigaben, Drucker, Benutzer und Gruppen, Sicherheit (BitLocker, NTLM, SMB-Signierung, Credential Guard, Überwachungsrichtlinie, bei Domänenmitgliedern auch Gruppenrichtlinien und LAPS)
- **Bewertung** der Ergebnisse, Probleme zuerst
- **Vergleich** mit einem gespeicherten Inventar: neue oder entfernte Software, Updates, Benutzer, Dienste
- Speichern verschlüsselt oder als JSON, ausführlicher PDF-Bericht
- **Agent-Verwaltung:** Erstellung und Verteilung des DATEXTAgent, zentrales Sammeln und Auswerten (siehe [DATEXTAgent](#datextagent))

<img width="1439" height="941" alt="image" src="https://github.com/user-attachments/assets/8b253484-bf84-4d04-a955-593c81f3f9da" />

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
| Active Directory | System.DirectoryServices.Protocols (LDAP, nativ) |
| SNMP | #SNMP Library (SharpSnmpLib) |
| Netzwerk-Topologie | Eigene Anordnung und Zeichnung (WPF), ohne Zusatzpaket |
| Installer | WiX Toolset 5 |

<details>
<summary>Vollständige Paketliste inkl. transitiver Abhängigkeiten</summary>

**Direkte Pakete**

| Paket | Version |
|---|---|
| DnsClient | 1.8.0 |
| Lextm.SharpSnmpLib | 12.5.7 |
| MailKit | 4.18.1 |
| Microsoft.Identity.Client | 4.90.1 |
| MimeKit | 4.18.1 |
| Newtonsoft.Json | 13.0.4 |
| PdfSharp | 6.2.4 |
| System.Data.SqlClient | 4.9.1 |
| System.DirectoryServices.Protocols | 8.0.1 |
| System.Management | 10.0.12 |
| System.Security.Cryptography.ProtectedData | 10.0.12 |
| System.ServiceProcess.ServiceController | 10.0.12 |
| System.Text.Encoding.CodePages | 10.0.12 |
| TaskScheduler | 2.12.2 |

**Transitive Pakete** (im Self-Contained-Build enthalten)

| Paket | Version |
|---|---|
| BouncyCastle.Cryptography | 2.6.2 |
| Microsoft.Extensions.DependencyInjection.Abstractions | 8.0.2 |
| Microsoft.Extensions.Logging.Abstractions | 8.0.3 |
| Microsoft.Win32.Registry | 5.0.0 |
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

Copyright © 2024–2026 DATEXT GmbH. Alle Rechte vorbehalten.
Dieses Projekt ist nicht Open Source. Die Verwendung, Vervielfältigung oder Weitergabe ohne ausdrückliche schriftliche Genehmigung der DATEXT GmbH ist nicht gestattet.

Eingesetzte Open-Source-Komponenten unterliegen ihren jeweiligen Lizenzen (siehe **Hilfe → Open-Source-Lizenzen** in der Anwendung).

---

*Entwickelt in Hagen, NRW*
