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
