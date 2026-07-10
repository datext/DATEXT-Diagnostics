# Changelog DATEXT-Diagnostics

Alle nennenswerten Änderungen an DATEXT Diagnostics werden in dieser Datei dokumentiert.
Format angelehnt an [Keep a Changelog](https://keepachangelog.com/de/1.0.0/).

## [Unreleased]


## [0.99.11.9] - 2026-07-10

### Added
- **Troubleshooting-Dialog** (System-Bereich): Windows-Update-Reparaturschritte als auswählbare Checkliste (Update-Ordner-Reset, BITS-Warteschlange leeren, Komponenten neu registrieren, Winsock/WinHTTP-Reset, Update-Richtlinien zurücksetzen, Komponentenspeicher-Bereinigung, gpupdate), funktional angelehnt an wureset-tools/script-wureset (nativ nachgebaut, kuratiert auf Windows-10/11-relevante Schritte)
- **Troubleshooting**: neue Sektion "Weitere Werkzeuge" – Systemschutz-Einstellungen öffnen, Internet-Explorer-Optionen öffnen, temporäre Windows-Dateien löschen (mit Scan/Bestätigung/Ergebnis), Windows Store zurücksetzen, nach Windows-Updates suchen, PC neu starten (inkl. "Neustart abbrechen"). Bewusst als Einzel-Buttons statt Checkliste, da der Dialog nicht auf WU-Reparatur begrenzt ist
- **WindowsComponentCheckDialog**: DISM CheckHealth als neue Phase 2 ergänzt (schneller Beschädigungs-Flag-Check vor dem eigentlichen ScanHealth) – Phasen 2–3 (ScanHealth) und 3–4 (RestoreHealth) entsprechend verschoben
- **SystemOverviewDialog**: Windows-Produktschlüssel-Zeile unter der Betriebssystem-Ausgabe (BIOS/UEFI-hinterlegter OEM-Key via `SoftwareLicensingService`, inkl. Copy-Button; zeigt „–" wenn nicht auslesbar)
- **HelpDialog**: neue Handbuchseite "Troubleshooting" (System-Gruppe) – beschreibt die WU-Reparatur-Checkliste und die Sektion "Weitere Werkzeuge", inkl. PDF-Export-Einbindung

### Changed
- **Troubleshooting**: Winsock/WinHTTP-Reset an letzte Stelle (Schritt 7) verschoben, standardmäßig deaktiviert und optisch als eigener, härterer Abschnitt mit Warnhinweis abgegrenzt (kein Bestandteil des eigentlichen WU-Resets, kann fest hinterlegte IP-/Proxy-Konfigurationen beeinträchtigen)
- **Troubleshooting**: Nav-Eintrag und Dialog-Header-Icon ergänzt (🛠️, das ursprüngliche Stethoskop-Symbol wurde von der Segoe-UI-Emoji-Schrift nicht zuverlässig dargestellt)

### Fixed
- **Troubleshooting**: Dienst-Stopp-Timeout für wuauserv von 30s auf 60s erhöht und Fehlschläge werden jetzt korrekt als Schritt-Fehler gemeldet statt stillschweigend als Erfolg (Sandbox-Test zeigte sonst kaskadierende Folgefehler: gesperrter SoftwareDistribution-Ordner, blockierte DLL-Registrierung)
- **Troubleshooting**: `wuaueng.dll`/`qmgr.dll` aus der Neu-Registrierungsliste entfernt – exportieren unter Windows 10/11 kein `DllRegisterServer` mehr und lieferten unabhängig vom Dienststatus einen irreführenden Fehler
- **Troubleshooting**: „Update-Komponentenspeicher bereinigen" (DISM StartComponentCleanup) wirkte eingefroren, da DISM seinen Fortschritt per `\r` auf derselben Zeile aktualisiert statt per Zeilenumbruch – `ReadLineAsync()` lieferte dadurch minutenlang nichts. Prozess-Reader liest jetzt zeichenweise mit `\r`/`\n` als Segmentgrenze; der Schritt zeigt zusätzlich einen echten Fortschrittsbalken (aus der geparsten Prozentangabe) statt nur eines rotierenden Indikators
- **Troubleshooting**: stderr von Prozessaufrufen (netsh/dism/gpupdate) wird jetzt mitgelesen statt verworfen (vermeidet Pipe-Deadlock-Risiko bei umfangreicher Fehlerausgabe)
- **SystemOverviewDialog**: Produktschlüssel fehlte in `BuildExportData()` und war dadurch weder in "Bericht kopieren" noch in CSV/JSON enthalten; zusätzlich in der PDF-Sektions-Whitelist (SYSTEM) ergänzt, da diese unabhängig vom Dictionary gepflegt wird
- **SystemOverviewDialog**: Antivirus-Erkennung fiel nicht mehr durch die tieferen Detection-Layer durch, wenn SecurityCenter2 nur einen inaktiven Defender-Eintrag zurücklieferte
- **SystemOverviewDialog**: Firewall-Erkennung dedupliziert jetzt korrekt, mehrfache Norton-Einträge werden nicht mehr angezeigt

### Known Issues
- Kompilierte EXE löst trotz Code-Signing die Norton-Heuristik `IDP.HELU.PSE80` als False Positive aus (Symantec-Meldung ausstehend, ggf. EV-Zertifikat)

---

## [0.99.11.8] - 2026-07-08

### Added
- `UpdateCheckDialog` neu eingeführt
- `HelpDialog` komplett überarbeitet (31 Seiten)
- `OpenSourceLicensesDialog` um alle 15 transitiven NuGet-Abhängigkeiten erweitert
- Umfassende `README.md` erstellt
- `SystemOverviewDialog`: Copy-Buttons für Systemkennungen
- `SystemOverviewDialog`: Netzwerk-Aktionsbuttons (Ping/Traceroute/DNS/Port-Scan)
- Wortmann AG Seriennummer-Lookup (herstellerabhängig)
- `DhcpDiscoveryDialog`: Dual-Socket-Ansatz als Workaround für vom Windows-DHCP-Client belegten Port 68
- `DhcpDiscoveryDialog`: Neu gestaltetes Badge-Row-Layout, `WrapPanel`-Server-Karten, dreistufige MAC-Vendor-Erkennung
- `ConnectionsDialog`: Klickbare Stat-Badges
- `ConnectionsDialog`: WinVerifyTrust P/Invoke zur Authenticode-Prüfung
- `ConnectionsDialog`: `QueryFullProcessImageName` zur Prozesspfad-Auflösung
- `ConnectionsDialog`: Shodan-InternetDB-Reputationsspalte mit Caching
- `ConnectionsDialog`: Kontextmenü-Einträge für VirusTotal-/MalwareBazaar-Hash-Lookup
- `WingetDialog`: PIN-Schutz für Sammel-Updates, Kontextmenüs, GitHub-API-basiertes Winget-Self-Update
- `EventLogDialog`: Gruppierte Szenario-Filter, Drucker-Event-Unterstützung, Kontext-Korrelationsmodus, Support-Ticket-Export mit Anonymisierung

### Changed
- `SystemOverviewDialog`: Energieeinstellungen in die System-Karte verschoben
- `BuildExportData` und PDF-Sektions-Whitelist um Akku-, Windows-Update-, Proxy/VPN- und Netzwerkzonen-Felder erweitert

### Fixed
- **Kritischer Lifecycle-Bug**: Endlos-Reload-Schleife bei schnellem Dialog-Wechsel behoben (`_isLoading`-Guard bleibt über Unload/Load-Zyklen erhalten)
- Interne Zeitsynchronisation: w32tm, Cloudflare und worldtimeapi laufen jetzt parallel als Race statt sequenziell
- Monitor-Erkennung: `EnumDisplayDevices` P/Invoke-Fallback für treiberlose "Generic Monitor"-Fälle
- DMARC-Anzeige: Vergleich nutzt jetzt `StartsWith` statt exakten Match
- **EventLogDialog**: Abgeschnittene Felder in der Anzeige korrigiert

---

## [0.99.11.6]

### Changed
- System, Ereignis-Viewer: Erweiterung und Gruppierung der Schnellfilter Auswahl

- **System & Stabilität**
  - Abstürze/Hänger, Bluescreen, Dienst-Abstürze, Festplatten-Warnungen, Rechner-Neustarts, Ressourcenengpässe, Treiberfehler, UEFI CA 2023 Update
- **Sicherheit & Konten**
  - Anmeldefehler, Erfolgreiche Anmeldungen, Kontoverwaltung, Kontosperrungen
- **Netzwerk**
  - DNS-Fehler, Netzwerkprobleme, RDP-Verbindungen
- **Anwendungen & Updates**
  - App-Abstürze, GPO-Anwendungsfehler, Update-Fehler
- **Peripherie & Hardware**
  - Druckprobleme, USB-Geräteprobleme, Zeitsync-Probleme

- **Kontextmenü auf Ereigniseintrag erweitert**
 - 💡 Kontext-Korrelation: verwandte Ereignisse lassen sich im Umkreis von ±5 Minuten um einen ausgewählten Eintrag anzeigen
 - 💡 Anonymisierter Export für Support-Tickets, ohne sensible Nutzerdaten preiszugeben


<img width="1586" height="901" alt="image" src="https://github.com/user-attachments/assets/cd7468ef-b942-425e-9da9-2a92ab58c038" />

### Fixed
-Systemübersicht: Diverse kleine Bugfixes



---

## [0.99.11.5]

### Changed
- Optimierung des PDF-Exports der Systeminformationen aus der Systemübersicht.

### Fixed
-Diverse kleine Bugfixes

---

## [0.99.11.3]

### Changed
- Systemübersicht Optimierung beim App-Start, Anzeige eines Laufbalken um die aktiven Hintergrundabfragen zu visualisieren.
- Update, Winget: Optimierung der App Aktualisierung über den Update / Winget Bereich.
> - Hinweis auf fehlende lokale Administratorberechtigungen die zur Installation / Aktualisierung von Apps erforderlich sein können und Möglichkeit die App mit erhöhten Rechten neu zu starten.
> - Apps mit dem PIN Flag sind von dem Update aller Apps ausgeschlossen und können jetzt gezielt aktualisiert werden
> - Die einzelnen App Updates können jetzt über ein Kontextmenü bedient werden und z.B. gezielt mit PIN vor weiteren Updates exkludiert werden oder PIN aufgehoben werden.

<img width="1586" height="901" alt="image" src="https://github.com/user-attachments/assets/cf1069e6-1f76-4822-91ee-e51c9efb46fc" />

---

## [0.99.11.1]

### Added
- Netzwerk, Verbindungen: neue "Risiko" Spalte:
> - RiskScore berechnet sich aus 5 Signalen: Shodan-Rep (+3), unsignierter Prozess mit ESTABLISHED (+2), bekannter C2-Port (+3), unbekannter Prozess (+1), Hochrisiko-Land (+1)
> - RiskLabel: ✅ OK / 🔵 Gering / 🟡 Mittel / 🔴 Hoch
> - RiskColor: transparent / hellblau / hellgelb / hellrot als Zellhintergrund
> - RiskTooltip: listet alle aktiven Signale auf
> - Score wird automatisch neu berechnet wenn ShodanRepColor gesetzt wird (via OnPropertyChanged)
- Netzwerk, Verbindungen: Filterung der Verbindungen nach Top-Prozessen oder über die Badges über der Ausgabetabelle.
- Netzwerk, Verbindungen: Manuelle bzw. automatische Aktualisierungen erzeugen eine Verbindungshistorie auf die Zugegriffen werden kann

<img width="1586" height="901" alt="image" src="https://github.com/user-attachments/assets/e32e5198-5b0f-4bce-8252-b9f9a30671a9" />

- Sytermübersicht: Erkennung von CGNAT ISP Internetanbindung auf Basis der RFC 6598, 100.64.0.0/10 
- Sytermübersicht: Hinweis auf doppeltes-NAT, wenn Router hinter Router skaliert wurde

---


## [0.99.11.0]

### Fixed
- Systemübersicht: Bugfixing bei der Erkennung einer aktiven VPN Verbindung und Badge Anzeige
- Systemübersicht: Bugfixing bei der Erkennung einer IPv4 Internetanbindung mit CGNAT ISP, wie Vodafone etc.

<img width="1336" height="867" alt="image" src="https://github.com/user-attachments/assets/a93ca3ac-aad7-4e92-86f5-76f74901e5de" />

---

## [0.99.10.2]

### Fixed
- Bugfixing im DHCP-Discovery. Zuverlässigere Erkennung von IPv4 und IPv6 Rogue DHCP Servern

<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/f67b78b6-9e05-4b21-9670-1043e5652296" />

---

## [0.99.10.1]

### Added
- Systemüberblick: Ergänzung in der Internet Verbindungsinformation in der Übersicht um mögliche öffentliche IP aus bekannten Cloudhosting Umgebungen, wie AWS, Azure.

<img width="1586" height="901" alt="image" src="https://github.com/user-attachments/assets/cf3c4a7d-f5c0-4c52-89b6-1cde8f76e8db" />

---

## [0.99.10.0]

### Added
- Internet Speedtest ermöglicht jetzt die benutzerdefinierte Transferzeit-Berechnung mit individueller Dowload- und Uploadgeschwindigkeit in Mbit/s.

### Changed
- Änderung des Standard-Speicherpfades für Dritt-Tools wie ooka.exe oder diskspd.exe auf %USERPROFILE%\Documents\DATEXT-Diagnostics\Tools
- Änderung des Standard-Speicherpfades für Paketmitschnitte von PKTMon oder NetSH Trace auf %USERPROFILE%\Documents\DATEXT-Diagnostics\NetworkTraces

### Fixed
- Systemüberblick: Erkennung installierter Antivirensoftware optimiert.

---

## [0.99.09.2]

### Fixed
- Systemüberblick: Erkennung installierter Antivirensoftware optimiert.

---

## [0.99.09.0]

### Fixed
- Erkennungsroutine der zu löschenden veralteter Programm-Ordner unter %APPDATA%/local/temp/.net/DATEXT-Diagnostics nach Updates.
- Exportfunktion in der "Übersicht" wieder hergestellt

---

## [0.99.09.0]

### Added
- Korrektur der ermittelten Durchschnittswerte im Internet-Speedtest
- App extrahiert die .Net Komponenten jetzt nur noch einmal pro Version in einen eigenen Ordner unter %LOCALAPPDATA%\Temp\.net\DATEXT-Diagnostics\<xxx> und bietet bei einer App Aktualisierung die Löschung von alten Unterordnern an.

---

## [0.99.08.4]

### Added
- Portscan Dialog erweitert um UDP Unterstützung. Manuelle Porteingabe kann jetzt gezielt nach UDP, TCP, oder UDP+TCP abgefragt werden.
- Beim Start einer neuen Version ab 0.99.08.04 wird das Löschen von temporären Ordnern unter %userprofile%\AppData\Local\Temp\.net\DATEXT-Diagnostics
angeboten, die vorherige Programmversionen erstellt haben.

---

## [0.99.08.3]

### Added
- Kleinere Bugfixes, insbesondere die Aktualisierung von Statusinformationen in der Systemübersicht.
- Erweiterte Auswertung der E-Mail & Domain Abfragen.

### Fixed
- Email-Check: schnellere Ermittlung des zuständigen SMTP Servers einer Domain
- Email-Check: ausführlichere Verbindungsinformationen
- Email-Check: zusätzliche Auswertung der Sicherheitsinformationen einer SMTP-Verbindung
- Email-Check: Testmail senden ermöglicht auch einen MX-Direkttest um MX zu MX Verbindungsinformationen zu liefern
- Header-Analyse: zusätzliche Verlaufshistorie von unterschiedlichen Mailheader-Auswertungen integriert
- E-Mail-Analyse: zusätzliche Verlaufshistorie von unterschiedlichen Mailheader-Auswertungen integriert
- E-Mail-Analyse: BIMI Abfrage integriert
- E-Mail-Analyse: Kommunikationsauswertung zwischen zwei Domains auf Basis der ermittelten SPF, DKIM, DMARC Werte und damit verbundene Erfolgsaussicht auf erfolgreiche Zustellung
- PDF Exportfunktion optisch optimiert.

---

## [0.99.006]

### Added
- `SystemOverviewDialog`: Virtualisierungserkennung (WMI/SMBIOSBIOSVersion/Registry)
- `SystemOverviewDialog`: Physische Datenträgererkennung via `MSFT_PhysicalDisk` mit `Win32_DiskDrive`-Fallback, inkl. Typ-Badges und VM-spezifischen Labels
- Multi-Monitor-Unterstützung via `EnumDisplayMonitors` P/Invoke mit `QueryDisplayConfig` für Anzeigenamen
- `UefiBootCertDialog`: Ausrichtung auf Microsofts Februar-2026-Playbook — Windows Server 2022, Hyper-V-Gen1-Ausschluss, 14 Empfehlungsszenarien, hypervisor-spezifische VM-Anleitung (VMware `.nvram`-Löschung, VirtualBox `VBoxManage`, QEMU `qm enroll-efi-keys`)

### Fixed
- `SqlAnalysisDialog`: Kompilierfehler behoben (fehlende XAML-Elemente, TwoWay-Bindings auf Read-Only-Properties auf `Mode=OneWay` korrigiert, `DataGridCell`/`TextBlock`-Style-Konflikte in separate Styles aufgeteilt)
- `SqlAnalysisDialog`: Export-Vollständigkeit über TXT/XLSX/PDF für alle 27 Datensammlungen verifiziert
- `SystemOverviewDialog`: Netzwerkzonen-Erkennung nutzt jetzt direkt `NLM.GetCategory()`
- `w32tm /stripchart`-Parsing der Internet-Zeitabweichung korrigiert

---

## [0.99.004]

### Added
- `ConnectionsDialog`: Shodan-InternetDB-Reputation und Hash-Lookup-Einträge

### Fixed
- `SystemHealthDialog`: 30–90 Sekunden Einfrieren durch `Get-WindowsOptionalFeature` behoben (ersetzt durch CBS-Registry-Abfrage)
- AD-Dialog: RSAT-Erkennung und Umlaut-Encoding-Probleme behoben
- Winget-Dialog: PIN-Erkennung gefixt (las zuvor falsche Spalte), Detail-Panels für Katalogsuche und installierte Pakete ergänzt

---

## Hinweise zur Pflege

- Neue Einträge während der Entwicklung direkt unter `[Unreleased]` ergänzen
- Beim Release: `[Unreleased]` in `[Versionsnummer] - Datum` umbenennen, Abschnitt 1:1 in die GitHub-Release-Notes übernehmen, neue leere `[Unreleased]`-Sektion oben ergänzen
- Kategorien: `Added`, `Changed`, `Fixed`, `Removed`, `Security`, optional `Known Issues`
