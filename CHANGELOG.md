# Changelog DATEXT-Diagnostics

Alle nennenswerten Änderungen an DATEXT Diagnostics werden in dieser Datei dokumentiert.
Format angelehnt an [Keep a Changelog](https://keepachangelog.com/de/1.0.0/).

## [0.99.12.02] - 2026-10-09
### Added
- **DATEXTAgent 1.1.2**: Neu gebaut und signiert, in Diagnostics eingebettet. Enthält die neue Gruppenrichtlinien-Logik des Inventars (lokaler Cache zuerst, Weg zum DC, Felder `gpoSource` und `dcPath`, siehe „Changed“). Der Agent kann nicht nachladen: Bei VPN wartet er wie bei lokalem DC bis zu 90 s und liefert bei Zeitüberschreitung die lokalen Daten. Agent-Inventare von 1.1.1 bleiben lesbar.

### Changed
- **Inventar – Gruppenrichtlinien**: Die angewendeten Computer-GPOs werden zuerst aus dem lokalen Cache gelesen (`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Group Policy\History`): sofort, ohne Netz, auch bei Rechnern ohne Verbindung zum Domänencontroller. Danach wird der Weg zum DC geprüft (DNS, TCP 389, Schnittstelle bzw. Subnetz): **lokal** → `gpresult` liefert die frischen Daten samt Sicherheitsgruppen; **nur über VPN** (Tunnel-/PPP-Schnittstelle oder bekannter VPN-Adaptername) → die lokalen Daten erscheinen sofort mit dem Hinweis „Abfrage beim DC läuft“, `gpresult` läuft bis zu 180 s im Hintergrund und aktualisiert die Anzeige danach selbst (der geöffnete Reiter bleibt erhalten); **nicht erreichbar** → nur lokale Daten mit Hinweis auf den Stand des letzten Refresh. Liefert `gpresult` nichts, bleiben die lokalen Daten, und es steht eine Warnung im Inventar; vorher fehlten die GPOs in diesem Fall ganz. Im Agent (kein Nachladen möglich) wird bei VPN wie bei lokalem DC bis zu 90 s gewartet. Neue Felder im Inventar: `gpoSource` (Registry/gpresult) und `dcPath` (Lokal/VPN/Nicht erreichbar). Außerdem wird bei allen gestarteten Werkzeugen die Standardeingabe geschlossen (ein GUI-Prozess hat keine Konsole).

---
## [0.99.12.01] - 2026-10-08
### Added
- **DATEXTAgent 1.1.1**: Der signierte Agent ist in Diagnostics eingebettet (`DATEXTAgent\bin\Release\…\publish\win-x64\DATEXTAgent.exe` → `Resources\DATEXTAgent.exe`). Agent 1.1 richtet die Firewall selbst ein, deshalb entfällt die Sperre der täglichen und wöchentlichen Bereitstellung. 1.1.1 enthält zusätzlich die geänderte Sicherheitsprüfung des Inventars (siehe „Changed“).

### Changed
- **Lokale Inventarisierung – Statusanzeige**: Während der Erfassung zeigt die Statuszeile, welche Bereiche noch laufen, und die verstrichene Zeit (z. B. „Schritt 12 von 13 · läuft noch: Sicherheit · 47 s“). Vorher wirkte ein langsamer letzter Bereich wie ein Hänger.
- **PDF-Berichte aller Dialoge**: E-Mail-Check, E-Mail-/Header-Analyse, DNS, Routing, DHCP, Geräte-Scan, Verbindungen, Netzwerk-Analyse, Active Directory, Gruppenrichtlinien, System-Health, UEFI, Software, Ereignislog, Autorun, Winget, MS-SQL, Speedtests und die Log-Dialoge (Ping, Traceroute, Port-Scan, Pktmon, Netsh, Troubleshooting) nutzen die Bausteine des Inventar-Berichts (`InventoryPdfExporter.ExportReport/ExportLogReport`: Titelblock, zweispaltige Werte, Tabellen, Bewertungsblöcke). `ExportService.ExportDiagnosticToPdfWithHeader` hat keine Aufrufer mehr.

## [0.99.12.00] - 2026-10-05
### Added
- **Geräte-Scan / mDNS**: Die mDNS-Suche fragt nicht mehr nur `ANY local.` ab, auf das viele Geräte nicht antworten, sondern zusätzlich das Standard-Diensteverzeichnis (`_services._dns-sd._udp.local`), die PTR-Einträge der wichtigen Diensttypen (mit einmaligem Nachlauf für gemeldete Typen) und `_device-info._tcp.local`. Aus den Diensten je Gerät wird der Gerätetyp abgeleitet: `_ipp`/`_ipps`/`_printer`/`_pdl-datastream` → Drucker, `_googlecast` → Chromecast/Google TV, `_airplay` → Apple TV, `_raop` → AirPlay-Lautsprecher, `_hap`/`_homekit` → HomeKit, `_smb`/`_afpovertcp`/`_nfs` → NAS/Dateiserver. Bei Apple-Geräten liefert `_device-info` (TXT `model=`) das Modell, z. B. "MacBookPro18,3". Die Dienste erscheinen als `mDNS:ipp`, `mDNS:googlecast` usw. in der Dienstespalte; Geräte ohne Hostnamen übernehmen den Instanznamen. Hörzeit 3 → 4 s. Der mDNS-Typ ersetzt eine bestehende Erkennung nur, wenn der Typ unbekannt ist oder ein Apple-Modell gemeldet wurde.
- **Geräte-Scan / Banner-Grabbing**: Prüft nicht mehr nur, ob ein Port offen ist, sondern liest Banner: HTTP (`Server`-Header, `<title>`, `WWW-Authenticate`-Realm, z. B. "FRITZ!Box 7590", "HP LaserJet", "Synology DiskStation"), HTTPS (Zertifikatsname und Aussteller, zusätzlich Titel/Server; Zertifikatsprüfung für den Scan abgeschaltet, damit selbstsignierte Zertifikate lesbar sind), SSH (Versionszeile, z. B. "OpenSSH_8.9p1 Ubuntu") und FTP (Begrüßungszeile). Die Banner stehen in der Dienstespalte und liefern einen Typ-Hinweis (FRITZ!Box, Drucker, Synology, QNAP, UniFi, MikroTik, Cisco, pfSense/OPNsense, IP-Kameras, Proxmox/ESXi, IIS), der nur übernommen wird, wenn der Typ noch unbekannt ist. Lesezeit je Port mindestens 800 ms.
- **Geräte-Scan / SMB**: Über Port 445 werden Windows-Infos ohne Zugangsdaten gelesen (SMB2-Negotiate, danach Session-Setup mit NTLMSSP-Negotiate; die Challenge-Antwort enthält Rechnername, Domäne und Windows-Build). Es findet keine Anmeldung statt, im Zielsystem entsteht also kein fehlgeschlagener Anmeldeversuch. Funktioniert auch bei blockiertem Ping. Anzeige in der Dienstespalte als "SMB: Windows 11 23H2 (Build 22631)" und "Domäne: …" bzw. "Arbeitsgruppe/ohne Domäne"; der Rechnername ersetzt einen fehlenden Hostnamen, der OS-Hinweis "Windows" fließt in die Typ-Erkennung ein. Da Windows-Client und -Server dieselben Builds nutzen, werden bei gemeinsamen Builds beide Namen genannt (z. B. "Windows 10 1809 / Server 2019"). Läuft mit der Option "Banner-Grabbing"; Geräte, die nur SMB1 sprechen, werden übersprungen. **Grenze:** Ab Windows 11 24H2 / Server 2025 meldet Windows in der NTLM-Challenge dauerhaft Build 26100 (gegengeprüft an einem Gerät mit Build 26300/26H2, das per SMB trotzdem 26100 liefert). **Samba/Linux:** Ein antwortender SMB-Dienst gilt nicht mehr automatisch als Windows (FRITZ!Box, NAS, Linux-Server mit Samba wurden vorher als "Windows" ausgewiesen). Die Anzeige wird erst am Scan-Ende nach der Typ-Erkennung entschieden: Windows nur bei plausibler Version (echtes Windows meldet einen Build, Samba/ksmbd Build 0 oder nichts), und wenn weder MAC-Hersteller (AVM, Synology, QNAP, Drucker- und Netzwerkhersteller …) noch Gerätetyp (Router, NAS, Drucker, Kamera …) dagegen sprechen. Sonst steht "SMB-Dienst (Samba/Linux, kein Windows)" in der Dienstespalte, und ein nur aus SMB stammender Windows-Hinweis wird zurückgenommen. Neuere Versionen sind darüber nicht unterscheidbar; die Anzeige lautet deshalb "Windows 11 24H2 oder neuer / Server 2025 (NTLM-Build 26100)".
- **Geräte-Scan / WS-Discovery**: Dritter Vorab-Schritt neben mDNS und SSDP (UDP 3702, Multicast 239.255.255.250). Ein SOAP-Probe wird je lokaler IPv4-Schnittstelle zweimal gesendet; geantwortet wird per Unicast. Erkennt Windows-PCs mit aktivierter Netzwerkerkennung (`pub:Computer`), WSD-Drucker/-Scanner (`PrintDeviceType`/`ScanDeviceType`) und ONVIF-Kameras (`NetworkVideoTransmitter`; Name und Hardware-Modell aus den ONVIF-Scopes). Treffer außerhalb des Ping-Bereichs erscheinen als Discovery-Einträge; auf Online-Geräten werden fehlende Hostnamen und unbekannte Typen ergänzt, Modell und "WSD" stehen in der Dienstespalte. Statuszeile zeigt zusätzlich "WSD: n".
- **DNS-Lookup – Sicherheit (DNSSEC · CAA)**: Neue Card. DNSSEC: DS-Record der Zone beim Parent = Zone signiert; zusätzlich das AD-Flag der Antwort = der genutzte Resolver validiert (viele Heimrouter tun das nicht – wird als Hinweis genannt). CAA (RFC 8659): erlaubte Zertifizierungsstellen mit Erklärung (issue / issuewild / iodef), gesucht vom Namen aufwärts bis zur Zone ("geerbt von …"). Die Zone wird über den SOA ermittelt (bei www.datext.de → datext.de).
- **DNS-Lookup – Mail-Prüfungen**: MTA-STS (`_mta-sts`, RFC 8461) und SMTP TLS Reporting (`_smtp._tls`, RFC 8460) der Zone. SPF: mehrere SPF-Records (PermError), "+all"/"all", "?all", fehlendes abschließendes "all", veraltetes "ptr" und die Grenze von 10 DNS-Lookups (RFC 7208 §4.6.4, rekursiv über include/redirect gezählt, Makros ausgenommen) – Anzeige "SPF: 7 von max. 10 DNS-Lookups". Fehlender SPF/DMARC bei Domains mit MX wird gemeldet. DKIM wird bewusst nicht im DNS-Dialog gesucht (Selektoren sind nicht aus DNS ermittelbar) – dafür gibt es die E-Mail-Analyse mit Selektorliste, BIMI und Blacklists; die Card weist darauf hin.
- **DNS-Lookup – Antwortzeit**: Antwortzeit des gewählten DNS-Servers in der Zusammenfassung und als Vergleichszeile unter den öffentlichen Resolvern der Propagation.
- **DNS-Lookup – Bewertung**: Abschnitt "Bewertung" in der Zusammenfassung sammelt alle Auffälligkeiten (vorher über die Panels verstreut): NXDOMAIN/SERVFAIL/Timeout, fehlender bzw. nicht zurückzeigender PTR (fehlende PTR mehrerer Adressen in einer Zeile), fehlender SPF/DMARC, DMARC `p=none`, SPF-Warnungen, widersprüchliche bzw. unterschiedliche Propagation, kein DNSSEC / Resolver validiert nicht, kein CAA. Fehler rot, Warnungen orange, Empfehlungen blau; Klick springt zum Bereich.
- **DNS-Lookup – Abfrage-Vergleich**: Nach jeder Abfrage Abgleich mit der letzten Abfrage desselben Hosts über denselben Server – neue/entfallene Adressen, Nameserver, MX, PTR, geänderter SOA-Serial, NXDOMAIN ↔ auflösbar, Propagation-Status; Hinweis oben in der Zusammenfassung und in der Statuszeile ("unverändert seit #3"). Zeigt z. B. nach einer DNS-Umstellung, ob die Änderung schon sichtbar ist.

### Changed
- **Internet-Speedtest – Live-Messung**: Die Messung läuft im jsonl-Format mit Zwischenständen alle 250 ms. Download und Upload laufen sichtbar hoch, die Phase steht im Kartenkopf („Download 58 %“); vorher blieb bis zum Ende alles bei „--“. Die ersten Messwerte nach wenigen Millisekunden sind Ausreißer (gemessen: 2,8 GB/s nach 7 ms) und werden übersprungen.
- **Internet-Speedtest – Ergebnis**:
  - **Latenz unter Last:** Sie wird angezeigt, getrennt für Download und Upload, mit Tooltip zu Bufferbloat.
  - **Ergebnislink:** Der Link von Ookla erscheint mit „Öffnen“ und „Link kopieren“.
  - **Server und Anbieter:** Server, ISP und externe IP werden angezeigt; die IP lässt sich ausblenden, auch für Kopieren und Export.
  - **Bewertung:** Die Kacheln sind neutral, die Farbe des Werts trägt die Bewertung (Ping, Jitter, Paketverlust).
  - **Formate:** Unter 10 ms hat der Ping eine Nachkommastelle; Transferzeiten unter einer Sekunde stehen in ms.
- **Internet-Speedtest – Serverwahl**: Neu ist ein Auswahlfeld mit den nächstgelegenen Servern (Name, Ort, Entfernung) bzw. „Automatisch“. Die Auswahl wird gemerkt.
- **Internet-Speedtest – Messverlauf**:
  - Der Verlauf wird gespeichert (höchstens 50 Messungen) und als Tabelle angezeigt, mit Durchschnitt und „Verlauf löschen“.
  - Er lässt sich als CSV exportieren.
  - Bisher ging er beim Schließen verloren und bestand aus 9-pt-Spalten.
- **Internet-Speedtest – Transferzeiten und eigene Berechnung**: Der „Transfer-Zeit-Rechner“ ist in zwei Karten geteilt:
  - **Transferzeiten:** Kacheln für 100 MB (Mailanhang), 1 GB (Software-Update), 10 GB (Video/Spiel) und 100 GB (Datensicherung), statt Textzeilen für 1 MB / 1 GB / 5 GB; 1 MB war immer „< 0,1 s“. Die Zeiten stehen in 16 pt, dazu ein Vergleichsbalken zwischen Download und Upload.
  - **Eigene Berechnung:** Eingaben in einer Zeile, Platzhalter „Messwert 512“ in den Bandbreitenfeldern, Ergebnis als zwei große Werte. Eine Zeile nennt die Grundlage der Rechnung (Messwerte, eigene Werte oder gemischt). Ohne Messung lässt sich mit eigener Bandbreite rechnen.
  - **Zeitangaben:** Sie sind lesbarer, z. B. „2 min 48 s“ statt „2,8 min“, und gelten auch im Export.
- **Internet-Speedtest – Layout passt sich der Breite an**: Die Verbindungsklasse steht rechts neben Messverlauf und Protokoll, bündig mit der Karte unter den Reitern; Transferzeiten und eigene Berechnung stehen nebeneinander. In schmalen Fenstern liegen die Bereiche untereinander, die Verbindungsklasse vor dem Verlauf. Die Kennzahlen springen auf 3 Spalten, die Transferkacheln auf 2. Die Eignung zeigt Name und Begründung untereinander, damit lange Begründungen umbrechen statt abgeschnitten zu werden.
- **Internet-Speedtest – Darstellung**:
  - **Emojis:** 🔥 〰️ 🎯 ↺ ✅ ⚠️ ❌ 🌐 💡 📊 ⭐ sowie die Rahmengrafik im Protokoll sind entfernt bzw. durch Fluent-Icons ersetzt.
  - **Zentrale Stile:** Statt eigener Button-Templates gelten die zentralen Stile, Toolbar-Höhe 30 px. Die Eingabefelder nutzen `Input.TextBox`.
  - **Statuszeile und CLI-Info:** Neu sind eine Statuszeile und die Angabe zu Version und Pfad der CLI; der englische Ookla-Werbetext entfällt.
  - **Export:** TXT und PDF entstehen aus den Messwerten, mit Ergebnislink, Eignung und Verlauf.
  - **Code:** Er ist auf `.Ookla` und `.Export` aufgeteilt.
- **Netzwerk-Speedtest – Adapter automatisch**: Die manuelle Adapterauswahl entfällt. Sie hatte keinen Einfluss auf die Messung, vorgewählt war der „schnellste“ Adapter, oft Hyper-V oder VPN. Jetzt wird der Adapter ermittelt, über den Windows den Server tatsächlich erreicht (Routing-Tabelle). Angezeigt werden Linkgeschwindigkeit und, bei echten Adaptern, die gemessene Auslastung; virtuelle Adapter werden als solche gekennzeichnet.
- **Netzwerk-Speedtest – Latenz und Wiederholungen**:
  - **Latenz:** Neue Kachel mit dem Verbindungsaufbau zu SMB (TCP 445), gemessen zur aufgelösten IPv4 (bester von 4 Versuchen nach einem Aufwärmversuch).
  - **Wiederholungen:** Bei mehreren Läufen werden Mittelwert sowie Minimum und Maximum angezeigt; vorher nur der Durchschnitt.
- **Netzwerk-Speedtest – Bewertung**:
  - **Bezug zum Adapter:** Bewertet wird relativ zur Linkgeschwindigkeit des genutzten Adapters (z. B. „1 Gbit/s-Link optimal ausgelastet (94 %)“) statt nach festen Klassen.
  - **Hinweise:** 100-Mbit/s-Link, WLAN, hohe Latenz und große Unterschiede zwischen Lesen und Schreiben werden erklärt.
  - **Eignung:** Sie zeigt eine Begründung (z. B. „1 TB in ca. 49 min“) statt nur ✅/❌. Neu ist der Punkt „Office-/Datenbankdateien“, der die Latenz berücksichtigt.
- **Netzwerk-Speedtest – Freigaben und Testdateien**:
  - **Zuletzt genutzt:** Das Pfadfeld ist eine Auswahl der zuletzt erfolgreich gemessenen Freigaben (höchstens 8). Testgröße und Anzahl der Läufe werden gemerkt.
  - **Liegengebliebene Testdateien:** `datext_speedtest_*.tmp` aus Abstürzen oder früheren Abbrüchen wird erkannt und lässt sich löschen.
  - **Ordnerauswahl:** „Durchsuchen“ öffnet eine echte Ordnerauswahl statt des Datei-Dialogs mit dem Trick „Ordner auswählen“ im Dateinamen.
- **Netzwerk-Speedtest – Messverlauf**: Der Verlauf ist eine Tabelle mit Hinweis „Noch keine Messungen“, Mittelwertzeile und CSV-Export.
- **Netzwerk-Speedtest – Darstellung**:
  - **Emojis:** Emojis sind durch Fluent-Icons ersetzt.
  - **Zentrale Stile:** Es gelten die zentralen Stile mit Toolbar-Höhe 30 px. Die lokalen Button-Aliase und die globalen ItemsControl-/StackPanel-Stile sind entfernt.
  - **Kacheln und Platzhalter:** Die Kacheln sind neutral, die Farbe trägt die Bewertung. Das Pfadfeld hat einen echten Platzhalter; vorher wurde die Textfarbe per Code fest auf `Brushes.Black` gesetzt.
  - **Status und Export:** Fehler erscheinen in der Statuszeile und im Protokoll statt zusätzlich als Meldungsfenster. „Kopieren“ bestätigt kurz. TXT und PDF entstehen aus den Messwerten, ohne festes „DATEXT“ im Kopf.
  - **Code:** Er ist auf `.Measure` und `.Export` aufgeteilt.
- **E-Mail-Check – Neuaufbau**: Der Dialog ist jetzt ein UserControl unter `Dialogs\Mail` wie alle anderen. Vorher war er ein Window, dessen Inhalt MainWindow herauslöste; der Code ist auf `.Tests`, `.Analysis`, `.Export` und `.Kim` aufgeteilt (vorher 3.241 Zeilen in einer Datei).
- **E-Mail-Check – Bedienung**:
  - **Toolbar:** E-Mail-Feld, „Server ermitteln“, „Verbindung testen“, „Alle Ports“, „Testmail …“, „Sicherheitsanalyse“ und „Stopp“ ersetzen die rote Schritt-Leiste, die zwei Button-Sätze vor und nach Autodiscover und die doppelten Test-Buttons in den erweiterten Optionen.
  - **Server und Optionen:** Server, Port, Verschlüsselung, Zeitlimit, Zertifikatfehler ignorieren und DNS-Server stehen in einer einklappbaren Zeile. Bei Port 25 prüft „Automatisch“ jetzt STARTTLS, falls angeboten (vorher unverschlüsselt).
  - **Gemerkt:** Letzte Adresse, letzter Server, Port und Zeitlimit werden gespeichert.
- **E-Mail-Check – Verbindungskarte**: Vier Statuszeilen für Verbindung, Verschlüsselung (TLS-Version, Cipher, Schlüsselaustausch), Zertifikat (gültig bis, Aussteller, Fehler, „Anzeigen“) und Anmeldung (angebotene AUTH-Verfahren, Warnung bei AUTH ohne Verschlüsselung) ersetzen die Ampel-Ellipsen und den dreizeiligen TLS-Fließtext. Die Karte zeigt das Ergebnis des gewählten Protokoll-Reiters, bei Reitern ohne eigenes Verbindungsergebnis das letzte derselben Domain. Bei Wechsel der Domain wird sie zusammen mit der Port-Übersicht geleert. MailKit-Fehlertexte erscheinen gekürzt auf Deutsch, z. B. „Servername passt nicht zum Zertifikat (ausgestellt für …) – Servernamen statt IP verwenden“.
- **E-Mail-Check – Port-Übersicht**: „Alle Ports“ prüft 587 (STARTTLS), 465 (TLS) und 25 gleichzeitig und zeigt je Port eine Statuszeile mit Ergebnis, TLS, Zertifikat, Anmeldeverfahren und Dauer, inklusive Hinweisen wie „kein TLS-Dienst auf diesem Port“ und „Port 25 oft vom Provider gesperrt“.
- **E-Mail-Check – Sicherheitsanalyse**:
  - **Zusammenfassung:** SPF, DMARC, DKIM, MX, PTR, Blacklist, TLS nach BSI TR-02102, DANE und IMAP/POP3 erscheinen als Statuszeilen mit Bewertung, statt nur im Protokolltext.
  - **Kompakt:** Probleme stehen zuerst. Der Kartenkopf zählt „1 Problem · 1 Hinweis · 3 OK“, die Details sind einzeilig, der volle Text steht im ToolTip. Ein Klick auf eine Zeile springt an die Stelle im Protokoll und hebt sie kurz hervor.
  - **Je Reiter:** Das Ergebnis wird am Protokoll-Reiter gespeichert. Beim Reiterwechsel zeigt die Karte die Analyse des gewählten Reiters, bei anderen Reitern die letzte Analyse derselben Adresse. Bei Änderung der E-Mail-Adresse wird die Karte geleert.
  - **DNS-Sicht:** Zeigt der MX über das System-DNS auf eine private Adresse, weist eine eigene Zeile auf die interne DNS-Sicht hin und empfiehlt einen öffentlichen DNS-Server. Auf dem Testrechner lieferte das System-DNS für datext.de „SPF fehlt“, Google-DNS den tatsächlichen SPF-Eintrag.
  - **Abbruch:** Er ist zwischen allen Schritten möglich.
  - **Blacklist:** Die Prüfung der öffentlichen IP gehört jetzt zur Analyse.
  - **TLS-Bewertung:** Sie nutzt nur erfolgreiche Verbindungstests; vorher bewertete sie auch abgebrochene Verbindungen mit „Note F, None, 0 Bit“.
- **E-Mail-Check – Testmail**: Die doppelte Logik (normal und erweitert, nur eine davon mit IMAP-Prüfung) ist zusammengeführt. Erfolg, Relay-Test, MX-Direkttest und IMAP-Empfang erscheinen in der Statuszeile statt in Meldungsfenstern; Bestätigungsfragen vor dem Versand bleiben. Gemerkte Anmeldedaten werden als SecureString statt als Klartext-String gehalten.
- **E-Mail-Check – Protokolle**:
  - **Emojis:** Emojis der Dienste werden als `[OK]` / `[!]` / `[X]` / `[i]` ausgegeben, Rahmenlinien gekürzt, Tastenkappen-Ziffern (1️⃣) als „1.“.
  - **Farbig:** Die Marker erscheinen grün, orange, rot und blau, Abschnittsüberschriften fett. Im SMTP-Mitschnitt stehen Client-Befehle (`C:`) in Akzentfarbe, Server-Fehlercodes 4xx/5xx in Rot, Trennlinien gedämpft. Zeilenumbruch ist umschaltbar und wird gemerkt.
  - **Reiter:** Sie nutzen den zentralen Reiterstil mit kurzen Titeln („#3 Analyse“, voller Titel im ToolTip), Schließen-Glyphe und Kontextmenü (kopieren, Bericht als PDF/TXT, schließen). Die Reiterzeile bleibt einzeilig: Sie scrollt mit dem Mausrad, ein Überlauf-Menü listet alle Protokolle. Vorher brachen viele Reiter in mehrere gestreckte Zeilen um.
  - **Export:** Bericht mit Ergebnis-Übersicht (Verbindung, Port-Übersicht, Sicherheitsanalyse) vor dem Protokoll, für den gewählten oder alle Reiter, über `ExportFile`. Ohne festes „DATEXT“ im Kopf, Werte aus den Ergebnissen statt aus Anzeigefeldern.
- **E-Mail-Check – Darstellung**:
  - **Emojis:** 🔍 ⟲ ⚡ ✉️ 🔒 ⏹ ⚙️ 📡 📋 🟢 ✓ ❌ 🔵 ✕ sind durch Fluent-Glyphen ersetzt.
  - **Zentrale Stile:** `ctl:Card`, `ctl:StatusRow`, zentrale Buttons und Eingaben mit Höhe 30 ersetzen GroupBox-Template, `ModernButton`/`PrimaryButton`/`ModernTextBox`, `Label`-Controls, `CompactStyles.xaml` und die festen Farben aus `ApplyTheme`/Statusbadge; der Dunkelmodus wird nicht mehr übersteuert.
  - **Zwei Spalten:** Links stehen die Ergebnisse (Verbindung, Sicherheitsanalyse, Port-Übersicht) mit eigener Scrollleiste, rechts die Protokolle über die volle Höhe statt fester 440 px unter den Karten. Die Trennlinie ist verschiebbar; das Verhältnis wird gemerkt, ein Doppelklick setzt 40/60. Die Karten lassen sich einklappen, der Zustand wird gemerkt.
  - **Responsive:** Unter 1.100 px Inhaltsbreite schaltet ein Umschalter „Ergebnisse | Protokolle“ zwischen den Spalten um, statt zu stapeln. Nach einer Prüfung springt er auf „Ergebnisse“. Die Toolbar-Buttons brechen um; vorher feste linke Spalte von 320 px bei `MinWidth 1100`.
- **Stil-Wörterbuch – `ctl:StatusRow`**: Neue Option `SingleLineDetail="True"`: Die Detailzeile wird einzeilig mit „…“ gekürzt statt umzubrechen, für lange Listen in schmalen Spalten.
- **E-Mail-Header-Analyse – Neuaufbau**: Der Dialog liegt jetzt in `Dialogs\Mail` und ist aufgeteilt in `.Parse` (Header, Received, Authentication-Results, Spam), `.Assessment` (Bewertung, Online-Prüfungen) und `.Export`. Der Verlauf speichert das Analyseobjekt statt eines nachgebauten Anzeigezustands.
- **E-Mail-Header-Analyse – Bewertung**: Statuszeilen mit Zähler, Probleme zuerst:
  - **Authentifizierung:** SPF, DKIM (alle Signaturen mit `d=`/`s=`, Ausrichtung zur From-Domain), DMARC mit Richtlinie bzw. Aktion, ARC und Microsoft compauth.
  - **Phishing-Indizien:** Abweichung von From und Return-Path, Antwortadresse an fremde Domain, Anzeigename mit einer anderen Adresse („service@paypal.com“ &lt;…@fremd&gt;), Message-ID-Domain.
  - **Spam:** SpamAssassin (Schwelle aus `required=`), Microsoft SCL/BCL/SFV aus `X-Forefront-Antispam-Report` und rspamd.
  - **Zustellung und Transport:** Zustellzeit mit langsamster Station und Hinweis auf Uhrzeit-Abweichungen; Übergaben ohne TLS zwischen öffentlichen Servern.
  - **Absender-IP:** Herkunft wird angegeben.
- **E-Mail-Header-Analyse – Online-Prüfungen** (abschaltbar, gemerkt):
  - **Blacklist** der Absender-IP mit Bedeutung des Eintrags.
  - **SPF-Gegenprüfung:** Erlaubt der aktuelle SPF-Eintrag der Envelope-Domain diese IP?
  - **BIMI-Eintrag** mit Hinweis auf ein vorhandenes VMC-Zertifikat.

  Die Prüfungen laufen nach der sofortigen Offline-Auswertung, mit Laufbalken und „Stopp“.
- **E-Mail-Header-Analyse – Zustellweg**: Spalten für IP und Protokoll/TLS je Station (`ESMTPS`, `version=TLS1_2`, Postfix „using TLSv1.3“), Zeiten in Ortszeit, Verzögerung farbig, die langsamste Station fett. Ein Klick zeigt den vollständigen Received-Header.
- **E-Mail-Header-Analyse – Eingabe**:
  - „Datei öffnen …“ (Strg+O) sowie Ziehen und Ablegen für `.eml` und `.txt`; bei `.msg` ein Hinweis auf „Speichern unter .eml“.
  - „Analysieren“ übernimmt bei leerem Feld die Zwischenablage; Strg+Enter startet die Analyse.
  - Der Platzhalter erklärt, wo Outlook, Outlook im Web, Gmail und Thunderbird die Header anzeigen.
  - „Alle Header“ hat einen Filter, Adress- und Nachrichtenfelder ein Kopier-Symbol.
- **E-Mail-Header-Analyse – Darstellung**:
  - **Layout wie beim E-Mail-Check:** Links Eingabe und Verlauf, rechts das Ergebnis; Trennlinie verschiebbar und gemerkt, Doppelklick setzt 35/65. Unter 1.000 px schaltet ein Umschalter „Eingabe | Ergebnis“ um, nach der Analyse automatisch auf „Ergebnis“. Adress- und Nachrichtenkarte stehen nebeneinander, schmal untereinander.
  - **Zentrale Stile:** `Button.*` mit Höhe 30 statt drei Buttons mit eigenem Template (rotes „Analysieren“, Höhe 34), `Tab.Item`/`DataGrid.Default` statt lokaler Reiter-, Kopf- und Zellstile mit roter Unterstreichung. Statuszeile in der Toolbar.
  - **Emojis:** ⚠ ❌ ✅ ⚠️ ℹ️ ✕ sind entfernt.
  - **Export:** Bericht mit Bewertung, Adressen, Nachricht, Authentication-Results, Zustellweg und allen Headern aus dem Analyseobjekt. „Kopieren“ kopiert den Bericht mit kurzer Bestätigung, „Rohe Header kopieren“ steht im Menü.
- **Firmenname „DATEXT GmbH“**: Statt „DATEXT Beratungsges. mbH“ an allen Stellen der Anwendung:
  - **Oberfläche:** Dialog „Über …“, Hilfe, beide Handbücher (`Dokumentation\`).
  - **Assembly-Angaben:** Firma und Copyright, sichtbar in den Dateieigenschaften und im Handbuch-PDF.
  - **Berichte:** Kopf, Fußzeile und Dokumenteigenschaften aller PDF-Berichte (Diagnosen, Systembericht, Inventar, MS-SQL, Systeminfo) sowie der MS-SQL-Textbericht.
  - **Sonstiges:** Signatur der SMTP-Testmail und der KIM-Anbietereintrag.
  - **MSI-Setup:** Hersteller, Installationsordner `C:\Program Files\DATEXT GmbH\…`, Registry-Schlüssel `Software\DATEXT GmbH\DATEXT Diagnostics` und Lizenztext. Der Pfad war noch nicht ausgeliefert, eine Umstellung bestehender Installationen ist nicht nötig.
  - **Unverändert:** Das Code-Signing-Zertifikat lautet weiter auf den bisherigen Firmennamen.
- **Navigation**: „Header-Analyse“ heißt jetzt „E-Mail Header-Analyse“, „E-Mail-Analyse“ heißt „E-Mail Domain-Analyse“ (Navigation, Hilfe, Export; im DNS-Lookup heißt der Button „Domain-Analyse“).
- **E-Mail-Analyse – Neuaufbau**: Der frühere `DomainAnalysisDialog` ist jetzt `EmailAnalysisDialog` in `Dialogs\Mail` (UserControl, aufgeteilt in `.Checks` und `.Export`). Die Analyse liegt im neuen Dienst `MailDomainAnalyzer`, der ein Berichtsobjekt liefert. Das MainWindow hält eine einzige Instanz: Wechselt die Adresse im E-Mail-Check, bleiben Ergebnis und Verlauf erhalten, die neue Domain wird nur in ein leeres Feld übernommen.
- **E-Mail-Analyse – Bewertung**: Statuszeilen mit Zähler, Probleme zuerst; ein Klick springt zum passenden Abschnitt im Protokoll. Neue bzw. erweiterte Prüfungen:
  - **SPF:** Auswertung nach RFC 7208 statt Textsuche, mit Zähler der DNS-Abfragen (max. 10, rekursiv über include/redirect), Qualifier von `all`, veraltetem `ptr`, zu weiten Netzen und erkannten Versanddiensten.
  - **DKIM:** Alle Selektoren werden geprüft, mit Schlüssellänge; ein leeres `p=` gilt als widerrufener Schlüssel.
  - **DMARC:** `p`, `sp`, `pct` und `rua` aus den Tags; ohne eigenen Eintrag wird die Richtlinie der Organisationsdomain übernommen („geerbt von …“).
  - **Neu:** MTA-STS (TXT und Richtlinie per HTTPS mit Zertifikatsprüfung), TLS-RPT, DANE/TLSA für die ersten zwei MX und Null-MX (RFC 7505).
  - **Reputation:** Die Versand-IPs aus SPF (`ip4` bis /29, `a`, `mx`) und die MX-IPs werden auf Blacklists geprüft, die Domain gegen Spamhaus DBL und SURBL. Die Abfragen laufen immer über das System-DNS; ist Spamhaus darüber nicht erreichbar (Testeintrag 127.0.0.2), erscheint ein Hinweis statt „sauber“.
  - **DNS-Sicht:** Liefert der DNS-Server für den MX eine private Adresse (Split-DNS), weist eine Zeile auf die interne Sicht hin. Fehlende SPF-/DMARC-Einträge gelten dann als Warnung „Fehlt (intern)“.
- **E-Mail-Analyse – Mail-Fluss**: Bewertet Absender → Empfänger anhand der DNS-Daten: Empfänger-MX bzw. Null-MX, strenge Anbieter (Microsoft 365, Google), SPF-Gültigkeit, DKIM, DMARC-Richtlinie und Reputation des Absenders.
- **E-Mail-Analyse – Darstellung**: Layout wie beim E-Mail-Check mit Bewertung, Mail-Fluss und Verlauf links und farbigem Protokoll rechts; Trennlinie verschiebbar und gemerkt, schmal mit Umschalter. Das farbige Protokoll (`MailLogFormat`) nutzt jetzt auch der E-Mail-Check. Zentrale Stile statt eigener Buttons, Emojis entfernt, Meldungen in der Statuszeile statt als MessageBox. Ein Klick im Verlauf zeigt den gespeicherten Bericht ohne neue Abfrage.
- **E-Mail-Analyse – Export**: Bericht mit Bewertung, Mail-Fluss (falls geprüft) und Protokoll statt nur des Protokolltexts. „Kopieren“ bestätigt kurz.
- **Inventarisierung – Gemeinsame Erfassung**: Die lokale Inventarisierung hatte eine eigene Kopie des Agent-Collectors (rund 1.500 Zeilen), die auseinandergelaufen war. Die Erfassung liegt jetzt einmal in `DATEXT.Shared\Inventory\InventoryCollector.cs`; der Agent nutzt sie über einen schmalen Wrapper. Die Bereiche laufen parallel und melden ihren Fortschritt. Auf dem Testrechner dauert die Erfassung 19–22 statt 63 Sekunden. Windows-Features kommen über WMI (`Win32_OptionalFeature`) in rund 2 statt 60 Sekunden, PowerShell, DISM und Registry bleiben als Fallback.
  - **Neu erfasst:** Patch-Stand (UBR, z. B. Build 26300.9457), Zustand und Anschluss der Datenträger (NVMe/SATA/USB), Benutzer-Installationen aus den geladenen Profilen (HKU, z. B. Teams, Zoom, VS Code) mit Konto, Autostart aus `WOW6432Node\Run`.
  - **Sicherheit auch ohne Domäne:** NTLM-Stufe, SMB-Signierung, Credential Guard und Überwachungsrichtlinie werden auf jedem Rechner geprüft, vorher nur bei Domänenmitgliedern. Domänenteile (DC, GPOs, LAPS, Kerberos) bleiben Mitgliedern vorbehalten. Windows LAPS wird zusätzlich über Intune/CSP erkannt.
  - **Abbruch:** gpresult, PowerShell, DISM, auditpol und nltest werden bei „Abbrechen“ (oder Esc) beendet; jedes Werkzeug hat ein Zeitlimit.
  - **Weniger Fehlalarme:** Keine Warnung mehr für leere privilegierte Gruppen und für jedes Administratorkonto außer „Administrator“. Gewarnt wird stattdessen bei aktivem integriertem Administratorkonto, außer es wird von LAPS verwaltet.
- **Lokale Inventarisierung – Neuaufbau**: Ergebnisse direkt auf der Seite statt nur im Extrafenster. Toolbar mit Statuszeile, „Vergleichen …“ und Menü „Speichern“, schmale Leiste für fehlende Administratorrechte, vor dem ersten Scan eine kompakte Übersicht statt der großen Karte mit Emojis. Fortschritt mit Schritten („Schritt 7 von 13 · BitLocker fertig“) statt Endlosbalken, Dauer je Bereich im Reiter „Protokoll“.
- **Inventar-Ansicht (`InventoryDetailView`)**: Gemeinsame Ansicht für die lokale Inventarisierung und das Detailfenster der Agent-Inventarisierung, das sie nur noch einbettet.
  - **Übersicht:** Kacheln (Windows-Version, RAM, Datenträger, Programme, letztes Update, aktive Konten, Mitgliedschaft) und eine Bewertung als Statuszeilen, Probleme zuerst; ein Klick öffnet den Bereich. Bewertet werden Update-Alter (Warnung ab 45, Fehler ab 90 Tagen), Firewall, BitLocker (Systemlaufwerk als Fehler), TPM, Datenträgerzustand, Konten, riskante Features, Sicherheitsbefunde und unvollständige Bereiche. Ersetzt den Reiter „Compliance“.
  - **Reiter:** einzeilig mit Überlauf-Menü, zentrale Stile statt eigener Button-Templates, Hinweis-Symbol statt ⚠️/✅/🔑/🌐. Software mit Suche über Name, Hersteller und Benutzer sowie Filter „für alle / nur Benutzer“, Datum lesbar („08.09.2026“ statt „20260908“). Dienste mit Filter „Automatisch, aber beendet“. „Domäne“ heißt „Sicherheit“ und erscheint auf jedem Rechner. Schlüssel/Wert-Bereiche sind kopierbar und unter 700 px einspaltig.
- **System-Übersicht – Inventar & Bewertung**: Die lokale Inventarisierung ist jetzt die zweite Ansicht der System-Übersicht.
  - **Umschalter:** Unter dem Kopf der Übersicht steht „Live-Übersicht | Inventar & Bewertung“.
  - **Navigation:** „Inventar → Lokale Inventarisierung“ öffnet die Übersicht in dieser Ansicht, mit Status-Punkt bei Problemen. Umschalter und Navigation bleiben synchron.
  - **Erfassung nur auf Klick**, nicht beim Öffnen der Übersicht.
  - **Chip „Bewertung“:** In der Live-Übersicht neben dem Gesamtstatus. „Jetzt prüfen“ startet die Erfassung, danach z. B. „2 Probleme · 3 Hinweise“ mit den wichtigsten Befunden als Tooltip; ein Klick öffnet die Bewertung. Der Chip zählt nicht zum Gesamtstatus der Live-Werte.
  - **Bedienelemente:** Die Inventarisierung hat keinen eigenen Titelkopf mehr; „Vergleichen …“ und „Speichern“ stehen in ihrer Toolbar. Hilfe entsprechend angepasst.
- **System-Übersicht – Gemeinsamer Systembericht**: Ist unter „Inventar & Bewertung“ ein Inventar erfasst, enthalten die Exporte der Live-Übersicht auch Bewertung und Inventar. Im Export-Menü lässt sich das über „Inventar & Bewertung einbeziehen“ abwählen; ohne Erfassung ist der Eintrag gesperrt und nennt den Grund. Datei und PDF-Titel heißen dann „Systembericht“ statt „System-Übersicht“.
  - **PDF:** Die Bewertung steht als Zusammenfassung vorne. Nach den Live-Werten folgen Software (mit Benutzer-Installationen), Updates, Benutzer, Gruppen mit Mitgliedern, automatisch startende, aber beendete Dienste, Autostart, Features, BitLocker, Freigaben, Drucker, Sicherheitskonfiguration, GPOs und Erfassungshinweise. Die Softwareliste der Seite „Installierte Software“ entfällt dann, weil das Inventar sie vollständiger enthält.
  - **CSV:** Dieselben Inhalte als Zeilen „Kategorie, Eigenschaft, Wert“.
  - **JSON:** Zusätzlich `bewertung` (Bereich, Status, Ergebnis, Details) und `inventar` mit dem vollständigen Inventar im Agent-Format.
  - **Kopieren:** Übernimmt die Bewertung.
  - **Ein Export-Menü für beide Ansichten:** „Kopieren“ und „Exportieren“ stehen im Kopf der Übersicht und gelten in Live- und Inventar-Ansicht gleich; nur „Aktualisieren“ und das Datenalter sind der Live-Ansicht vorbehalten. Das Menü enthält zusätzlich „Inventar-Detailbericht (PDF, ausführlich)“, den Tabellenbericht der Agent-Inventarisierung. Das Menü „Speichern“ der Inventar-Ansicht enthält nur noch die Datendateien (verschlüsselt, unverschlüsselt) und keinen eigenen PDF-Bericht mehr.
  - **Technik:** Eine gemeinsame Quelle (`InventoryReport`) für alle Formate.
- **System-Übersicht – PDF im Stil des Inventar-Berichts**: Der PDF-Export der Live-Übersicht entsteht nicht mehr als Textausgabe über `ExportService`, sondern mit den Bausteinen des Inventar-Berichts (`InventoryPdfExporter.SystemReport`):
  - **Aufbau:** Titelblock mit Rechner, Domäne, Hersteller, Modell und Seriennummer. Die Live-Werte stehen als zweispaltige Tabellen (System, Ressourcen, Grafik, Netzwerk, IT-Sicherheit, Windows Update, Energie, Internet, Zeit).
  - **Tabellen:** Datenträger mit Partitionen; Belegung ab 90 % rot. Bildschirme mit Auflösung und Skalierung.
  - **Mit erfasstem Inventar:** Die Bewertung steht als farbige Blöcke vorne, danach folgen dieselben Tabellen wie im Inventar-Detailbericht (Hardware, Software, Updates, Dienste, Features, Benutzer, Gruppen, Autostart, BitLocker, Freigaben, Drucker, WSUS, Domäne).
  - **Ohne Inventar:** Die Softwareliste der Seite „Installierte Software“ erscheint als Tabelle.
  - **Unverändert:** CSV, JSON und „Kopieren“. Der „Inventar-Detailbericht“ bleibt im Menü.
- **Inventar-PDF – Bewertung**: Der PDF-Bericht der Inventarisierung (lokal und Agent) zeigt die Bewertung aus der Anzeige direkt nach der Titelseite, statt einer eigenen, abweichenden Compliance-Liste am Ende. Gute Befunde stehen grün darin.
- **Inventory Agent – Neuaufbau des Dialogs**: Aufgeteilt in `.Deploy`, `.Scripts`, `.Agents`, `.Import`. Kopf, Statuszeile und Reiter wie in den anderen überarbeiteten Dialogen, zentrale Stile statt lokaler Button-Stile, keine Emojis, Meldungen in der Statuszeile statt MessageBox (nur die Deinstallation fragt nach).
  - **Bereitstellen:**
    - Modus als Chips „Einmalig | Täglich | Wöchentlich“, nur passende Felder sichtbar.
    - Unter 1000 px stehen die Formular-Karten untereinander.
    - Die vier Wege (ZIP-Paket, Freigabe, GPO-Paket, Remote) stehen als Karten mit Erklärung nebeneinander.
    - Der Ablageort zeigt, ob ein Netzwerkpfad genutzt wird, mit Hinweis auf das Schreibrecht für Computerkonten.
    - Kunde, Ablageort und Port werden gemerkt.
  - **API-Key:**
    - Pflicht für geplante Agents; er wird automatisch erzeugt (32 Zeichen).
    - Gespeichert je Kunde mit DPAPI (aktueller Windows-Benutzer).
    - Für Suche und Verwaltung wählbar; der Key wird nur an den eingestellten Port gesendet.
  - **Agents im Netzwerk:**
    - Auswahl-Spalte und Status als Badge (Online, Erfasst gerade, Key falsch, Ohne Key).
    - Spalten „Letzte Erfassung“ und „Läuft ab“.
    - Neu „Jetzt erfassen“ (`POST /collect`); Abruf, Prüfung und Deinstallation für die markierten Agents.
    - Abgerufene Inventare werden verschlüsselt in einem eigenen Abrufordner gespeichert.
    - Der Suchbereich darf mehrere /24-Netze umfassen, bis 4096 Adressen.
  - **Auswertung:**
    - Bewertungs-Spalte aus der gemeinsamen Bewertung (z. B. „2 P · 3 H“).
    - Kacheln als Filter: Geräte, mit Problemen, ohne BitLocker, Updates älter als 45 Tage, Windows 10.
    - Software-Suche über alle Geräte („„Chrome“ auf 3 von 4 Geräten“) und Suchfeld für Rechner, Kunde, Betriebssystem, CPU.
    - „Details“ öffnet das Inventar, dort auch der Vergleich mit einem früheren Stand.
    - Import liest auch JSON-Listen.
    - Export-Menü: CSV (mit Bewertung, Build, letztem Update), PDF, verschlüsselt (eine Datei je Gerät) oder unverschlüsselt (Liste für den Datenbank-Import).
- **DATEXTAgent 1.1**:
  - **Netzwerk-Einrichtung:** Der Dienst richtet URL-Reservierung und Firewall-Regel beim Start selbst ein. Remote-Installation und GPO kopieren nur noch, legen den Dienst an und starten ihn.
  - **Absicherung:** Ohne API-Key sind nur lesende Abfragen erlaubt. Der Key wird zeitkonstant verglichen, die Kopfzeile `Access-Control-Allow-Origin: *` entfällt.
  - **Neu in `/status`:** Ablaufdatum, Key-Pflicht und Zeitpunkt der letzten Erfassung.
  - **Ablaufdatum:** wird auch beim Start und stündlich geprüft.
  - **Installationsskript:**
    - Idempotent: Gleiche Version und Konfiguration wird übersprungen, „Einmalig“ läuft nur einmal je Agent-ID.
    - Prüft Administratorrechte und .NET 8 über die installierten Runtimes.
    - Setzt kein netsh mehr ein und prüft die Rückgabe von `sc create`.
    - Das Paket enthält zusätzlich Deinstallationsskript und LIESMICH.
- **Inventar speichern**: Kein automatischer Verlauf, gespeichert wird nur manuell. Standard ist „Inventar speichern (verschlüsselt)“ im Agent-Format (AES-256-GCM, nur mit DATEXT Diagnostics lesbar). Optional „Inventar als JSON (unverschlüsselt, für Import)“ als Klartext, z. B. für den Import in eine Datenbank. Beide lassen sich über „Vergleichen …“ wieder laden. Gilt auch für das Detailfenster der Agent-Inventarisierung.
- **Inventar-Vergleich (neu)**: „Vergleichen …“ lädt ein früheres Inventar, also eine gespeicherte Datei (verschlüsselt oder Klartext) oder eine Agent-Datei, und zeigt die Unterschiede zur angezeigten Erfassung: Software (neu, entfernt, Version), Updates, Dienste (Starttyp, Konto, Pfad), Benutzer (aktiviert, Administrator), Gruppenmitglieder, Autostart, Features, Freigaben, Drucker, BitLocker, Sicherheitsbefunde, Hardware und Systemwerte (Build, BIOS, RAM, Firewall). Filter nach Art (Neu/Entfernt/Geändert) und Bereich; Hinweis, wenn die Datei von einem anderen Rechner stammt. Flüchtige Werte (IP-Adressen, Dienststatus, letzte Anmeldung) bleiben außen vor.
- **E-Mail-Check – KIM/TI**: Konnektor-Suche, TI-DNS und Routing-Prüfung bleiben als Basis für eine spätere Version in `.Kim` erhalten (Oberfläche ausgeblendet). Die Routing-Prüfung sucht jetzt nach den TI-Netzen 100.102.0.0/15 bzw. 10.30.0.0/15 statt nach 100.64.0.0/100.30.0.0.
- **Disk-Speedtest – Messverfahren**: DiskSpd misst jetzt in vier Teilmessungen nach dem Vorbild von CrystalDiskMark:
  - **Teilmessungen:** Sequenziell schreiben und lesen (1 MB oder 64 KB, QD8), 4K zufällig lesen mit QD32 (4 Threads × 8) und mit QD1.
  - **Ohne Windows-Cache:** Gemessen wird ohne Windows-Cache (`-Su`) und mit Latenz (`-L`); vorher lief DiskSpd mit Cache, das Ergebnis hing damit auch vom RAM ab.
  - **Testdatei:** Größe wählbar, 256 MB / 1 GB / 4 GB, Standard 1 GB statt fest 256 MB.
  - **Random 4K:** Vorher ein Mix aus 70 % Lesen / 30 % Schreiben, angezeigt als „Random 4K“. Neu ist QD1 – das, was Programmstarts spüren – mit mittlerer Latenz.
- **Disk-Speedtest – Laufwerke**:
  - **Erkennung:** Typ, Modell und Anschluss werden je Laufwerksbuchstabe ermittelt (HDD, SATA-SSD, NVMe über BusType, USB, virtuell).
  - **USB:** Externe USB-Platten, die sich als Wechseldatenträger melden, sind wählbar.
  - **Freier Platz:** Er wird gegen die gewählte Dateigröße geprüft.
  - **Liegengebliebene Testdateien:** Sie werden erkannt und lassen sich löschen.
- **Disk-Speedtest – Bewertung**: Sie bezieht sich auf den Laufwerkstyp, z. B. „NVMe erreicht nur SATA-Niveau“ mit Hinweisen zu Anbindung, Treiber und Drosselung, statt auf feste Klassen nach dem Lesewert. Die Eignung zeigt eine Begründung. In virtuellen Maschinen erscheint ein Hinweis, dass die Werte den Host-Speicher widerspiegeln; WinSAT ist dort nicht mehr gesperrt.
- **Disk-Speedtest – Fortschritt**: Die Phase steht mit Restzeit im Kartenkopf, dazu ein Fortschrittsbalken; vorher liefen 3 × 30 s ohne Anzeige.
- **Disk-Speedtest – Messverlauf**: Eine Tabelle mit „Noch keine Messungen“-Hinweis und CSV-Export ersetzt die ItemsControl mit 9-pt-Spalten. Der Hinweis zum Methodenunterschied erscheint als Zeile statt als Banner.
- **Disk-Speedtest – Darstellung**:
  - **Emojis:** 🪟 📥 ✅ ⏳ 🎲 🏆 🔴🟣🟢🟡🟠 ⚠️ ⚡ ℹ ✕ sind entfernt bzw. durch Fluent-Icons ersetzt.
  - **Methodenwahl:** Eine Auswahlbox ersetzt die zwei großen Methodenkarten.
  - **Zentrale Stile:** Es gelten die zentralen Stile mit Höhe 30 px. Entfernt sind die lokalen Button-Aliase (Höhe 34), der Download-Button mit eigenem Template und die globalen ItemsControl-/StackPanel-Stile.
  - **Kacheln:** Sie sind neutral.
  - **Status und Export:** Fehler erscheinen in der Statuszeile statt als MessageBox. „Kopieren“ bestätigt kurz. TXT, PDF und CSV entstehen aus den Messwerten, ohne festes „DATEXT“ im Kopf.
  - **Code:** Er ist auf `.Measure` und `.Export` aufgeteilt.
- **MS-SQL – Code-Aufteilung**: Die Code-Datei hatte 6.969 Zeilen. Sie ist ohne Verhaltensänderung in Partial-Dateien aufgeteilt: `.Models`, `.Connection`, `.Health`, `.Activity`, `.Databases`, `.Analysis`, `.Procedures`, `.Maintenance`, `.Security`, `.LiveMonitor` und `.Export`. Die Hauptdatei hat noch rund 300 Zeilen.
- **MS-SQL – Abfragetexte**:
  - In den SQL-Spalten werden Zeilenumbrüche und Leerraum zusammengezogen. Vorher stand oft nur „SELECT“, die erste Zeile.
  - Der Volltext erscheint als umbrechender Tooltip in Consolas.
  - Alle Tabellen haben das Kontextmenü „Markierte Zeilen kopieren“ (mit Spaltenköpfen).
- **MS-SQL – Wartetypen**: Die Wartezeit wird lesbar dargestellt („3,3 min“ statt „8.670.384“, das abgeschnitten wurde). Die Spalte „Wartetyp“ ist breiter, die Kopfzeilen sind deutsch.
- **MS-SQL – Leere Tabellen**: Sie zeigen „Keine Einträge“. Die abgeschnittenen Platzhalterzeilen („Keine Linked Server…“) sind entfallen.
- **MS-SQL – Express-Edition**: Statt „Max. Serverspeicher: Unbegrenzt“ erscheint die tatsächliche Grenze von 1.410 MB, mit Tooltip zu den Express-Grenzen. Die Auslastung wird darauf bezogen.
- **MS-SQL – Bedienung**:
  - Enter in Server, Port, Benutzer oder Kennwort verbindet.
  - Nach erfolgreicher Anmeldung werden Server, Port, Anmeldeart, Benutzer, Verschlüsselung und Zertifikatsoption gemerkt, **ohne Kennwort**.
  - Der Status oben zeigt Icon und Text getrennt.
- **MS-SQL – Darstellung**:
  - Emojis sind durch Fluent-Icons bzw. reinen Text mit Statusfarbe ersetzt, rund 70 Stellen. In den Zellen „Status“ (sp_WhoIsActive) und „Warnungen“ (sp_BlitzCache) stehen jetzt Glyphen.
  - Der Reiter „Analyse“ hat ein Icon wie die übrigen.
  - Installations-Buttons nutzen `Button.Secondary`, Ausführen-Buttons `Button.Primary`, ohne lokale Paddings.
  - Eingabe- und Kennwortfeld sind 30 px hoch.
  - 76 Hex-Farben im Code (`#333`, `#555`) sind durch Farb-Tokens ersetzt.
- **Gruppenrichtlinien – Neue Auswertungen**:
  - **Reiter „Erweiterungen“:** Status jeder clientseitigen Erweiterung mit letzter Ausführung, Dauer und Fehlercode.
  - **Reiter „Sicherheitsgruppen“:** Die Gruppen, die gpresult für die Filterung berücksichtigt hat, mit Suche. Daneben die durchsuchten Container mit „Vererbung blockiert“.
  - **Detailbereich:**
    - Alle Verknüpfungen mit Position, Verarbeitung, deaktiviert und erzwungen
    - Sicherheitsfilter und WMI-Filter
    - Version AD/SYSVOL; eine Abweichung weist auf ein Replikationsproblem hin
    - Beteiligte Erweiterungen
    - „GUID kopieren“
- **Gruppenrichtlinien – Kennzahlen und Aktionen**:
  - **Kacheln:** GPOs, Angewendet, Nicht angewendet, Verknüpfung deaktiviert, Verwaiste Verknüpfung (als Filter klickbar), Erweiterungsfehler (öffnet den Reiter) und Langsame Verbindung.
  - **Infozeilen:** Für Computer und Benutzer mit Konto, OU, Standort und letzter Verarbeitung.
  - **Neue Aktionen:** „gpupdate /force“ mit anschließender Neuanalyse; Rückfragen zur Abmeldung werden mit „Nein“ beantwortet. „HTML-Bericht“ öffnet `gpresult /H` im Browser.
  - **Export:** Er enthält alle Bereiche, neu sind „GPO-Liste als CSV“ und „gpresult-XML speichern“.
  - **„Kopieren“:** Meldet „Kopiert“ statt einer MessageBox.
- **Gruppenrichtlinien – Darstellung**:
  - Der Titel heißt „GRUPPENRICHTLINIEN“ (vorher „GROUP POLICY ANALYSE“). Der Untertitel nennt Benutzer, Computer und Zeitpunkt.
  - Status als Badge mit Text statt Emojis (✅ ⛔ 🚫 ⚫ ▶ ⏹ 🗒️).
  - Zentrale `Button.*`-, `Tab.Item`- und `Badge`-Stile; lokale Button-Templates und DataGrid-Höhen entfallen.
  - Die Rohdaten werden erst beim Öffnen des Reiters geladen.
  - Statuszeile mit Fehler- und Warnfarbe.
  - Der Code ist auf `.Parse`, `.Run` und `.Export` aufgeteilt.
- **Active Directory – Exakte letzte Anmeldung**: Neu „Exakt ermitteln“ im Detailbereich und „Letzte Anmeldung exakt ermitteln (alle DCs)“ im Kontextmenü beider Tabellen; ein Rechtsklick markiert dabei die Zeile.
  - **Warum:** Die Spalte zeigt `lastLogonTimestamp`; es wird nur alle 9–14 Tage repliziert. Das exakte `lastLogon` wird nicht repliziert, jeder DC kennt nur die Anmeldungen, die er selbst verarbeitet hat.
  - **Wie:** Für das gewählte Konto wird jeder DC der Domäne einzeln abgefragt. Vorher wird er mit 1,5 s Timeout auf ADWS (TCP 9389) geprüft, damit ein nicht erreichbarer DC nicht lange blockiert. Angezeigt wird der neueste Wert.
  - **Anzeige:** Das Detail zeigt den exakten Wert mit liefernden DC, den Wert je DC und den replizierten Wert. Nicht abfragbare DCs werden genannt („Wert evtl. zu alt“).
  - **Liste:** In der Liste steht danach „TT.MM.JJJJ HH:MM (exakt)“, farblich hervorgehoben. Sortierung, Inaktivitätsfilter, Kacheln und Export verwenden den neueren der beiden Werte.
  - **Bewusst nur gezielt:** Der Wert wird nur für einzelne Konten ermittelt, nicht für die ganze Liste.
  - **Beispiel:** Testkonto repliziert 28.09., exakt 05.10. 15:11 (srv-dc01), Abfrage über 2 DCs in ~2–3 s.
- **Active Directory – Kennzahlen**: Kacheln unter der Toolbar, per Klick als Filter nutzbar.
  - Computer: Gesamt, Aktiv, Deaktiviert, Inaktiv > 180 Tage, Nie angemeldet.
  - Benutzer: Gesamt, Gesperrt, Deaktiviert, Inaktiv > 180 Tage, Kennwort läuft nie ab, Kennwort abgelaufen bzw. Änderung bei Anmeldung.
  - Das Badge zeigt „x von y“.
- **Active Directory – Benutzerdaten**: Neue Spalte „Kennwort“:
  - „Läuft nie ab (Konto)“
  - „Kein Ablauf (Richtlinie)“, wenn die Kennwortrichtlinie kein Höchstalter hat
  - „Änderung bei Anmeldung“ (pwdLastSet = 0; AD meldet das ebenfalls als „abgelaufen“)
  - „Abgelaufen“ oder „Läuft ab TT.MM.JJJJ“, laut `msDS-UserPasswordExpiryTimeComputed`

  Im Detailbereich stehen außerdem UPN, Kennwort zuletzt gesetzt, Erstellungsdatum, Beschreibung und der vollständige OU-Pfad (von außen nach innen). „Letzte Anmeldung“ zeigt die Tage seitdem; ein Tooltip und eine Fußnote weisen auf die Replikationsverzögerung von `lastLogonTimestamp` (9–14 Tage) hin.
- **Active Directory – Details**: Der Detailbereich ist in der Höhe veränderbar und zeigt beim Laden der Gruppen eine Ladeanzeige. Die Gruppen werden je Objekt zwischengespeichert, statt bei jedem Klick ~1,5 s PowerShell zu starten. Neu sind „DN kopieren“ und „Vorgesetzter“ (springt zum Konto des Vorgesetzten). Die Werte sind markier- und kopierbar.
- **Active Directory – Export**:
  - Der Kopf enthält Domäne, DC, Suchbegriff, aktive Filter und Trefferzahl.
  - Exportiert wird der aktive Reiter, auch der Netzwerk-Abgleich.
  - CSV nutzt Semikolon und BOM für deutsches Excel.
  - JSON enthält Kennwortdaten und bereits geladene Gruppen; das immer leere `Groups` fällt weg.
  - TXT beschränkt sich auf lesbare Spalten.
  - „Kopieren“ meldet „Kopiert“.
- **Active Directory – Darstellung**:
  - Der Objekttyp ergibt sich aus dem Reiter; die doppelte Auswahl in der Seitenleiste ist entfallen.
  - Suchfeld mit Lupe und Platzhalter, Vergleichsart als Dropdown, DC-Feld mit Platzhalter „DC automatisch“.
  - Der Kopf zeigt Domäne und DC; Statuszeile für Ergebnisse und Fehler; leere Tabellen zeigen einen Hinweis.
  - Statusplaketten in den Tabellen über die zentralen `Badge`-Stile.
  - Emojis durch Fluent-Icons ersetzt.
  - Lokale Button-Templates und DataGrid-Stile entfernt; zentrale `Button.*`-, `Input.TextBox`- und `Tab.Item`-Stile.
  - Der Status der Anmeldedaten erscheint als Icon und Text.
  - Der nie sichtbare Platzhalter-Button „AD suchen“ ist entfernt.
  - Der Code ist auf `.Model`, `.Query` und `.Export` aufgeteilt.
- **Troubleshooting – Darstellung**:
  - Emojis sind durch Fluent-Icons ersetzt, und Buttons nutzen die zentralen Stile (keine lokalen Hover-Stile mehr).
  - Status und „Abbrechen“ stehen im Kopfbereich; die untere Aktionsleiste und die Karte „Gesamtfortschritt“ sind entfallen.
  - Das Protokoll ist immer sichtbar, mit „Kopieren“, „Speichern“ und „PDF“.
  - Die Gruppen haben Auf-/Zuklapp-Pfeile.
  - Schritt-Karten von Update-Reparatur und Netzwerk teilen sich eine Vorlage.
  - Der PDF-Untertitel lautet allgemein „Reparaturprotokoll“.
- **Troubleshooting – Vorab-Prüfungen**:
  - WinHTTP-Proxy-Reset zeigt die aktuelle Einstellung.
  - Update-Ordner-Reset weist auf vorhandene alte Sicherungsordner und deren Größe hin.
  - Ein Abbruch der DISM-Bereinigung wird als unvollständig erklärt.
- **Troubleshooting – Weitere Werkzeuge**: Neu sind:
  - Druckwarteschlange leeren
  - Uhrzeit synchronisieren (`w32tm /resync`)
  - Alte Update-Sicherungen löschen (`SoftwareDistribution.bak_*`, `Catroot2.bak_*`)
  - Windows-Problembehandlungen

  Entfallen sind Zuverlässigkeitsverlauf, Leistungsüberwachung, Systeminformationen und Internetoptionen; sie liegen in der System-Verwaltung.
- **System-Verwaltung – Werkzeuge**: 45 statt 21 Werkzeuge, gruppiert:
  - Verwaltung: neu u. a. Aufgabenplanung, Lokale Benutzer und Gruppen, Gruppenrichtlinie, Sicherheitsrichtlinie, Zertifikate für Computer und Benutzer, Freigaben, Druckverwaltung.
  - Überwachung & Diagnose: Ressourcenmonitor, Leistungsüberwachung, Zuverlässigkeitsverlauf, Systeminformationen.
  - Sicherheit & Updates: Windows-Sicherheit, Firewall (erweitert), Windows Update.
  - Netzwerk: Netzwerk-Einstellungen, Internetoptionen, Remote-Einstellungen.
  - System & Einstellungen: Umgebungsvariablen, Programme und Features, Windows-Features, Windows-Wiederherstellung, Datenträgerbereinigung.
  - Benutzer & Konten: Konten (Einstellungen).
  
  Die Liste ist datengetrieben im Code statt 21 kopierter XAML-Blöcke. Beim Öffnen wird die Verfügbarkeit geprüft: Fehlt eine Datei oder ist das Tool in Windows Home nicht enthalten (Gruppenrichtlinie, Sicherheitsrichtlinie, Lokale Benutzer, Druckverwaltung, BitLocker), erscheint die Kachel ausgegraut mit Begründung. Ein Schild-Symbol markiert Tools, die Administratorrechte anfordern. Per Rechtsklick gibt es „Als Administrator starten“ und „Befehl kopieren“. Neu ist ein Suchfeld (Name, Befehl, Beschreibung; Eingabetaste öffnet den ersten Treffer). Die Kopfzeile ist fest wie in den anderen Dialogen, Rückmeldungen erscheinen in der Statuszeile statt per MessageBox, und die Emojis 💼/🚀 sowie doppelte Icons sind ersetzt.
- **Suchfelder (Autorun, Ereignislog, Installierte Software, Winget, System-Verwaltung)**: Eingegebener Text beginnt jetzt bündig mit dem Platzhalter. Die zentrale Vorlage `Input.TextBox` wendet `Padding` doppelt an (Inhaltsbereich und Textansicht), der Text stand deshalb gut 25 px rechts vom Platzhalter. Behoben in den betroffenen Suchfeldern; die zentrale Vorlage ist unverändert, weil eine Änderung alle Eingabefelder der App verschieben würde.
- **Winget-Pakete – Ergebnis je Paket**: Aktualisieren, Installieren und Deinstallieren laufen über einen gemeinsamen Ablauf. Jeder winget-Exit-Code wird ausgewertet: erfolgreich, Neustart nötig, nicht anwendbar oder fehlgeschlagen, jeweils mit Klartext (u. a. Anwendung läuft, Datenträger voll, Richtlinie, Prüfsumme, UAC abgelehnt). Am Ende steht eine Zusammenfassung, z. B. „19 erfolgreich, 1 mit Neustartbedarf, 1 fehlgeschlagen“, mit Liste der Ausnahmen. Das Neustart-Banner richtet sich nach dem Exit-Code statt nach der Zeichenfolge „3010“ in der Ausgabe. Alle Aufrufe laufen mit `--exact` und `--disable-interactivity`, sodass winget nicht auf unsichtbare Rückfragen wartet. Pakete mit blockierendem Pin werden mit Hinweis übersprungen.
- **Winget-Pakete – Darstellung**:
  - Kennzahl-Kacheln: Updates (davon mit Pin), installiert, über winget, ohne Quelle (ARP/MSIX), mit Pin, zuletzt geprüft mit winget-Version. Ein Klick wechselt in den Tab bzw. Filter. Sie ersetzen Badge und Zeitstempel in der Kopfzeile.
  - „Alle Pakete“ mit Spalten Status (farbiger Punkt), Verfügbar, Quelle und Pin-Art sowie Filter „Nur mit Update / Über winget verwaltbar / Ohne Quelle / Mit Pin“.
  - Suchergebnisse zeigen die Übereinstimmung getrennt und ob das Paket bereits installiert ist.
  - Emojis ersetzt: Schaltflächen, Tab-Kopf, Pin- und Update-Badges, Detailzeilen, 45 Katalog-Symbole (jetzt Gruppen-Icons), Protokollzeilen mit „[OK]/[!]/[X]“.
  - Die eigenen Button-Templates in den Bannern sind auf zentrale Stile umgestellt, der Paketfilter nutzt `Input.TextBox`.
  - „Kopiert“ als kurze Rückmeldung.
  - Katalog: Python-ID `Python.Python.3.12` → `3.13`.
- **Winget-Pakete – Export**: Neu ist CSV aller Pakete (Name, ID, installiert, verfügbar, Update, Quelle, Pin). TXT und PDF enthalten Computer, Abfragezeit, Kennzahlen, Quelle und Pin-Art. Die Detaildaten aus `winget show` (2–3 s) werden je Paket zwischengespeichert.
- **Installierte Software – Darstellung und Bedienung**:
  - Kennzahl-Kacheln: Programme (mit Anzahl Hersteller), letzte 30 Tage, nur dieser Benutzer, 32 Bit, verwaist, Gesamtgröße. Ein Klick filtert. Sie ersetzen Badge, Statistik-Untertitel und das Leer-Panel „Jetzt laden“.
  - Neue Spalten „Bereich“ (Alle Benutzer 64/32 Bit, nur dieser Benutzer) und „Hinweis“. Programmsymbol aus `DisplayIcon`, im Hintergrund nachgeladen.
  - Größen in MB/GB statt „4096,0 MB“. Version wird numerisch sortiert, das Datum nach Datum.
  - Detailbereich für den gewählten Eintrag: Installationsordner, Deinstallationsbefehl, MSI-Produktcode, Registry-Schlüssel und Website.
  - Kontextmenü: Deinstallieren, Ändern/Reparieren, Installationsordner, In Registry öffnen, Hersteller-Seite, Websuche, Name bzw. Deinstallationsbefehl kopieren.
  - Schalter „Systemkomponenten und Updates anzeigen“. Herstellerfilter mit Platzhalter statt Eintrag „— Alle Hersteller —“, „Filter zurücksetzen“, Suche (auch im Installationsordner) mit 200 ms Verzögerung.
  - Emojis aus den Spaltenköpfen entfernt; „Kopiert“ als kurze Rückmeldung statt MessageBox.
- **Installierte Software – Deinstallieren und Ändern**: „Deinstallieren“ öffnete nur `appwiz.cpl`. Jetzt startet nach Bestätigung das Deinstallationsprogramm des Eintrags, bei MSI `msiexec /x {Produktcode}`. Der Assistent bleibt sichtbar; eine stille Deinstallation wird bewusst nicht angeboten. „Ändern/Reparieren“ nutzt `ModifyPath` bzw. `msiexec /i`. `NoRemove`/`NoModify` werden beachtet, verwaiste Einträge lassen sich nicht starten. Nach Abschluss wird die Liste neu gelesen.
- **Installierte Software – Export**: Kopieren, PDF, CSV und JSON exportieren die aktuelle Ansicht; der Kopf nennt die aktiven Filter. Vorher wurde immer alles exportiert, auch bei gesetztem Filter. CSV mit allen Feldern (Bereich, Produktcode, Installationsordner, Deinstallation, Systemkomponente, verwaist, Registry) und BOM. JSON mit ISO-Datum und denselben Feldern. Die System-Übersicht übernimmt nur Programme (ohne Systemkomponenten) und schreibt keinen Emoji mehr in den Abschnittskopf.
- **Ereignislog-Viewer – Abfrage direkt an Windows**: Schweregrad, Schnellfilter (Anbieter + IDs), Einschränkung auf eine Quelle und Zeitraum werden als XPath beim Lesen gefiltert. Der Umfang „Letzte N“ gilt damit für die Treffer, nicht für die Rohdaten. Neue Zeiträume: letzte 24 Stunden, 7 Tage und 30 Tage; „Von/bis“ jetzt mit Datumsauswahl statt freier Texteingabe. Zeitraum-Abfragen sind auf 50.000 Treffer begrenzt, die Statuszeile meldet die Begrenzung. Gemessen: 0,1–0,5 s je Schnellfilter.
- **Ereignislog-Viewer – Protokolle**: Neu direkt wählbar sind Setup, Windows PowerShell, PowerShell (Operational), Defender, Aufgabenplanung, RDP-Sitzungen, RDP-Verbindungen und Druckdienst. „Weitere …“ durchsucht alle Ereignisprotokolle des Computers (hier 1.311). Bei einem deaktivierten Protokoll wird die Aktivierung angeboten, ohne Adminrechte mit Neustart als Administrator.
- **Ereignislog-Viewer – Darstellung**:
  - Kennzahl-Kacheln statt linker Kartenspalte, die Liste hat jetzt die volle Breite. Die Kacheln Kritisch, Fehler, Warnungen, Informationen und „Überwachung gescheitert“ sind zugleich der Schweregrad-Filter: Ein Klick blendet den Schweregrad aus und lädt neu. Dazu kommen „Häufigste Quelle“ und „Zeitraum der Treffer“.
  - Neue Zeile „Häufigste Fehler/Warnungen“ (Quelle + ID + Anzahl); ein Klick lädt nur diese. Die Einschränkung erscheint als entfernbares Merkmal, ebenso per Rechtsklick „Nur diese Quelle (und ID) laden“.
  - Schweregrad als farbiger Punkt mit Text statt Emojis und voll eingefärbter Zeilen.
  - Banner und Schaltflächen nutzen die zentralen Stile statt eigener Button-Templates; vier Toolbar-Zeilen wurden auf zwei reduziert.
  - Schnellfilter mit Icons und Gruppen. Ein Tooltip zeigt Protokoll, Anbieter und IDs, daneben steht eine kurze Beschreibung.
  - „Kopiert“ als kurze Rückmeldung statt MessageBox; „Keine Daten“ als Statushinweis.
- **Ereignislog-Viewer – Detail**: Zusätzlich zur Nachricht zeigt das Detail Kategorie, Schlüsselwörter, Benutzer (aufgelöst aus der SID), Computer, Prozess-ID und Datensatz-Nr. Die Werte werden für den gewählten Eintrag nachgeladen. Umschaltbar auf „XML“ (Rohdaten des Ereignisses). Die Recherche-Links Google, MS Learn, eventid.net, Bing und answers.microsoft.com stehen in einem Untermenü.
- **Ereignislog-Viewer – Export**: Neu sind CSV (alle Felder, vollständige Nachricht, BOM) und EVTX. EVTX ist eine echte Ereignisprotokoll-Datei inklusive Meldungstexten, die sich in der Windows-Ereignisanzeige öffnen lässt; bei begrenzter Anzahl gilt der Zeitraum der geladenen Treffer, für das Sicherheitsprotokoll nur mit Admin. TXT und Zwischenablage enthalten den vollständigen Text mit Abfragekopf. PDF umfasst bis zu 1.000 Einträge und weist auf den Rest hin. Der Support-Auszug anonymisiert wahlweise Computer- und Domänenname, Benutzername, Konten-SIDs sowie IPv4- und IPv6-Adressen.
- **Autorun-Analyse – Prüfumfang**:
  - Neue Orte: `Policies\Explorer\Run` (HKCU und HKLM), HKLM `WOW6432Node\…\RunOnce`, Debugger-Umleitungen (Image File Execution Options), `AppInit_DLLs` (mit `LoadAppInit_DLLs`), `BootExecute` (ohne den Windows-Standard `autocheck autochk *`), Active Setup und WMI-Ereignisabonnements mit Befehls- oder Skript-Consumer (dateilose Persistenz; nur mit Admin lesbar).
  - Winlogon zeigt nur noch die Autostart-Werte `Shell`, `Userinit`, `Taskman` und `AppSetup` und meldet Abweichungen vom Windows-Standard.
  - Der Zustand aus StartupApproved wird gelesen, also derselbe wie in Task-Manager → Autostart.
  - Bei geplanten Tasks erscheinen alle Tasks mit allen Aktionen, jeweils ein Eintrag je Aktion. COM-Handler werden bis zur DLL aufgelöst (InprocServer32 bzw. AppID → Dienst-DLL). Auslöser und Zustand stehen auf Deutsch, dazu der Ordner des Tasks.
  - Dienste unterscheiden „Automatisch (verzögert)“; bei svchost-Diensten wird die eigentliche Dienst-DLL geprüft.
- **Autorun-Analyse – Deaktivieren wie im Task-Manager**: Run-Einträge (Benutzer, Computer, 32 Bit) und Autostart-Ordner lassen sich per Rechtsklick über StartupApproved deaktivieren und wieder aktivieren. Das ist umkehrbar, der Eintrag selbst bleibt erhalten, und das Byte-Format ist identisch mit dem des Task-Managers. Für HKLM-Einträge und den gemeinsamen Autostart-Ordner wird angeboten, die App als Administrator neu zu starten. Früher per Umbenennung in `.disabled` deaktivierte Verknüpfungen werden erkannt und lassen sich wiederherstellen. „Löschen“ verschiebt Ordner-Einträge jetzt in den Papierkorb, statt sie endgültig zu löschen.
- **Autorun-Analyse – Darstellung**:
  - Kennzahl-Kacheln: Einträge, Gefahr, Warnung, nicht signiert, Datei fehlt, deaktiviert. Ein Klick filtert die Tabellen.
  - Suchfeld (Name, Befehl, Herausgeber, Ort, Hinweis) und Filter „Microsoft-Einträge ausblenden“, standardmäßig an wie bei Sysinternals Autoruns. Auf dem Testsystem sind dann 51 statt 410 Einträge zu sehen.
  - Einheitliche Spalten: Status mit farbigem Punkt, Herausgeber laut Signatur und „Hinweis“ mit dem Grund der Einstufung. Deaktivierte Einträge sind abgeblendet, der Tooltip zeigt den vollen Ort und die geprüfte Datei.
  - Tab-Zähler zeigen Gesamt und Auffällige getrennt.
  - Hinweisleisten sind neutral statt gelb, Emojis sind durch Segoe-Fluent-Icons ersetzt.
  - „Kopiert“ erscheint als kurze Rückmeldung statt als MessageBox.
  - Der Scan lässt sich abbrechen, die Statuszeile zeigt den Fortschritt der Signaturprüfung.
  - Ohne Admin erscheint ein Hinweis, was fehlt.
- **Autorun-Analyse – Export**: CSV mit Bereich, geprüfter Datei, Signatur, Herausgeber, Risiko als Text, Hinweis, Auslöser, Ort und Aktiv-Zustand, mit BOM. PDF und Zwischenablage mit „[X]/[!]/[OK]“ statt Emojis und mit der Liste nicht geprüfter Bereiche.
- **UEFI Boot-Zertifikat – ohne PowerShell**: Die Zertifikate in db, KEK und PK werden direkt aus der Firmware gelesen (`GetFirmwareEnvironmentVariableEx` mit `SeSystemEnvironmentPrivilege`) statt über `Get-SecureBootUEFI`. Der Boot-Manager wird über den Volume-GUID-Pfad der EFI-Partition geprüft (`X509Certificate.CreateFromSignedFile`) statt mit mountvol und `Get-PfxCertificate`; als Admin läuft die Prüfung automatisch. TPM wird über die TBS-API (`Tbsi_GetDeviceInfo`) ermittelt, BitLocker über die Shell-Eigenschaft `System.Volume.BitLockerProtection` (beides ohne Admin) und der Task über die Taskplaner-COM statt `schtasks`. Die Ereignisse kommen per `EventLogReader` (Quelle Microsoft-Windows-TPM-WMI, letzte 30 Tage) statt aus den letzten 500 System-Einträgen beliebiger Quelle. Ohne Admin wird der Status aus dem Servicing-Status abgeleitet (z. B. Boot-Manager aus `WindowsUEFICA2023Capable = 2`) und als „abgeleitet“ gekennzeichnet.
- **UEFI Boot-Zertifikat – Darstellung**:
  - Neue Kennzahl-Kacheln: Secure Boot, Umstellung 2023, KEK 2023, Windows UEFI CA 2023, Boot-Manager, Tage bis zum Ablauf von PCA 2011.
  - Neue Karte „Zertifikatsumstellung 2011 → 2023“. Je Rolle (KEK, Windows-Boot, Drittanbieter, Option-ROMs, Boot-Manager) zeigt sie, ob der 2023-Nachfolger vorhanden ist.
  - Neue Karte „Windows-Servicing“ mit Umstellungsstatus, Rollout-Steuerung (AvailableUpdates/-Policy, HighConfidenceOptOut, MicrosoftUpdateManagedOptIn, ConfidenceLevel) und Task mit letzter Ausführung. Dazu kommen die Fehlercodes inklusive `KEKLastUpdateError`/`-Reason` und eine Ereignisliste mit Bedeutung, Anzahl und Zeitpunkt.
  - Zertifikate werden nach Rolle bewertet: Ein ersetztes 2011-Zertifikat erscheint als „Ersetzt“, ein abgelaufener PK oder ein fremdes abgelaufenes Zertifikat als „Ohne Wirkung“, denn die Firmware prüft Ablaufdaten nicht.
  - Empfehlungen sind neu formuliert und brechen fließend um; vorher waren die Zeilen fest umbrochen und mit Emojis versehen. Buttons und Icons nutzen Segoe Fluent.
  - Das Ergebnis der Boot-Manager-Prüfung steht in der Zeile statt in einer MessageBox.
  - Der Admin-Hinweis erscheint nur noch einmal.
- **UEFI Boot-Zertifikat – Export**: Speicherort ist jetzt per Dialog wählbar, mit einheitlichem Dateinamen (`ExportFile`) statt fest im Netzwerk-Trace-Ordner und eigenem Erfolgsbanner. JSON wird mit `System.Text.Json` erzeugt statt von Hand zusammengesetzt; neu enthalten sind die 2011→2023-Rollen, Ereignisse, Task und Boot-Manager. CSV enthält die Rollentabelle und wird mit BOM geschrieben, PDF und Zwischenablage nutzen denselben Bericht ohne Emojis. Standardformat ist PDF.
- **System-Health – weitgehend ohne PowerShell**: Von 13 PowerShell-Skripten sind 2 übrig (Hyper-V-Switch-Bindung nur auf Hosts mit vEthernet-Adapter, Server-Rollen nur auf Windows Server). Ersetzt durch Windows-API, Registry, WMI und COM: Defender (`MSFT_MpComputerStatus`), Firewall-Profile (Registry inkl. Gruppenrichtlinie), SMB1/PowerShell 2.0 (`Win32_OptionalFeature`), RDP/NLA (inkl. Richtlinie), VBS/Credential Guard (`Win32_DeviceGuard`), NIC-Statistik/Link/Offload (`NetworkInterface` + Treibereinstellungen), Routing (`GetIpForwardTable`), ARP (Nachbartabelle), Netzwerkzone (NetworkListManager), Firewall-Regeln (`HNetCfg.FwPolicy2`), Energieplan (powrprof), Commit (`GetPerformanceInfo`), DPC/ISR (PDH), Stabilitätsindex (WMI), Auslagerungsdatei (WMI), Aufgaben (Taskplaner-COM), .NET-Laufzeiten (Installationsordner statt `dotnet --version`, das ohne SDK "Nicht installiert" meldete). Sprachunabhängig, schneller und mit kleinerem AV-Heuristik-Fußabdruck (vgl. Malwarebytes-Vorfall). Vollständiger Scan getestet: 72 s.
- **System-Health – neue Prüfungen**: Defender-Signaturen älter als 7 Tage (Warnung), Manipulationsschutz aus (Hinweis), letztes Update älter als 45 Tage (Warnung), Neustart ausstehend auch nach Komponentenwartung (CBS), UAC-Stufe im Klartext ("ohne Rückfrage hochstufen" als Warnung), RAM-Typ inkl. LPDDR und eingestelltem Takt, MEMORY.DMP neben den Minidumps.
- **System-Health – Darstellung** (angeglichen an Port-Scan, Traceroute, DNS-Analyse, Ping):
  - Bereichs-Kacheln unter der Werkzeugleiste (Hardware, Betriebssystem, Speicher, Sicherheit, Netzwerk, Performance, Dienste, Ereignisse) mit Anzahl Probleme/Warnungen; Klick öffnet den Tab.
  - Gesamtbewertung mit Punktzahl ("65 / 100": 100 − 15 je Problem − 4 je Warnung) statt farbiger Emoji-Kreise; bei Abbruch als unvollständig gekennzeichnet.
  - Problemliste: Fehler vor Warnungen, Klick auf ein Problem öffnet den zugehörigen Tab. Neue Karte "Nicht geprüft" für Bereiche, die mangels Rechten nicht ausgewertet werden konnten (fließen nicht in die Bewertung ein).
  - Emojis durch Segoe-Fluent-Icons in Theme-Farben ersetzt (zentraler Converter für alle Status-Icons; Tab "Betriebssystem" mit Icon statt "⊞", "Bluescreens" statt "💥 BSODs", "Lösung:" statt "🔧"). Feste Farben `#333`/`#555` durch Theme-Farben ersetzt. Exporte ohne Emojis ("[OK]", "[!]", "[X]", "[i]") und mit Liste der nicht geprüften Bereiche.
  - Performance-Karten: "Auswirkung" bricht um (vorher horizontales StackPanel mit Pin-Symbol).
- **Ping – Optionen und Auswertung**:
  - Auswahl des Prüflaufs: "Ping-Test" statt "Standard" (neben "MTU-Erkennung"), damit erkennbar ist, dass hier die Art der Prüfung gewählt wird; Tooltip erklärt beide Prüfläufe.
  - Neue Optionen: Intervall (Standard 1 s wie Windows-ping statt fest 0,5 s), Paketgröße (32–1472 Bytes), Timeout (1–5 s), IP-Version (Auto/IPv4/IPv6), 100 Pakete.
  - Statistik zusätzlich mit Median, 95. Perzentil, längster Ausfallserie und Zeitpunkt des letzten Verlusts.
  - VoIP-Bewertung: geschätzter MOS (1–5, vereinfachtes E-Modell nach ITU-T G.107 aus Ø-Laufzeit, Jitter und Verlust) mit Einstufung (sehr gut … unbrauchbar) und Prüfung der Teams/VoIP-Grenzwerte (Laufzeit < 100 ms, Jitter < 30 ms, Verlust < 1 %); bei weniger als 10 Paketen als Richtwert gekennzeichnet.
  - Vergleich mit dem letzten Ping zum selben Host (Adresse, Ø, Jitter, Verlust) im Protokoll und in der Statuszeile.
  - Mehrere Adressen: Löst ein Name auf mehrere Adressen auf, zeigt die Kachel "Ziel (+n)" mit Tooltip aller Adressen (gepingte markiert) und Hinweis, wie eine bestimmte Adresse geprüft wird (IP-Version, direkte Eingabe, Schnellaktion der DNS-Analyse); das Protokoll nennt die nicht geprüften Adressen. Bewusst keine eigene DNS-Server-Auswahl im Ping – dafür ist die DNS-Analyse mit Schnellaktion "Ping" je Adresse da.
- **Ping – Darstellung** (angeglichen an Port-Scan, Traceroute und DNS-Analyse):
  - Statistikleiste oben statt Kartenspalte links: Kacheln Ziel, Verlust, Latenz Ø (min/max/95 %), Jitter, VoIP (MOS), Ausfallserie bzw. MTU; Historien-Hinweis links in der Kachel-Reihe, Kacheln linksbündig, Fortschritt rechts. Ergebnisse in voller Breite.
  - Tabs "Tabelle · Protokoll · Historie": Tabelle je Paket (Nr., Zeit, Antwort von, Laufzeit, Latenzbalken, TTL, Bytes, farbiger Status); darüber der Latenz-Verlauf mit ms-Skala, gestrichelter Ø-Linie und roten Markierungen für verlorene Pakete (vorher unsichtbar).
  - Protokoll ohne Emojis; Historie mit Spalten Pakete, Verlust, Ø, Jitter und Status mit farbigem Punkt (OK · MOS / Verlust / keine Antwort / MTU / Abgebrochen). Ein Historien-Eintrag stellt Tabelle, Diagramm, Kacheln und Protokoll wieder her.
- **Port-Scan / Traceroute / DNS-Analyse – Statistikleiste einheitlich**: Der Historien-Hinweis ("Historie #n · Host", "Zum letzten …") steht in allen drei Dialogen links in der Kachel-Reihe; Traceroute und DNS-Analyse blendeten dafür vorher eine zusätzliche Zeile ein. Die Ergebniskacheln stehen linksbündig direkt dahinter (vorher im Port-Scan und Traceroute rechtsbündig); Ziel-Adresse und Fortschritt stehen rechts. DNS-Analyse: Detailtext des Hinweises gekürzt (Einträge · Dauer), damit die Kacheln in eine Reihe passen. Traceroute: Ziel-Badge "Erreicht" / "Nicht erreicht" ohne Emojis.
- **DNS-Lookup – Darstellung** (angeglichen an Port-Scan und Traceroute):
  - Statistikleiste oben mit Kacheln statt der Baum-Navigation links (die immer nur eine Card zeigte und deren Einträge die Card-Überschriften wiederholten): Adressen (IPv4/IPv6), Nameserver (mit DNSSEC), Mail (MX, SPF/DMARC), Dienste, Propagation (OK/CDN/!), Hinweise; bei IP-Eingabe PTR. Farbe nach Bewertung, Klick springt zur Card. Alle Cards stehen untereinander, die Ergebnisse haben die volle Breite.
  - Emojis durch Segoe-Fluent-Icons in Theme-Farben ersetzt (Schnellaktionen Kopieren/Ping/Traceroute/Port-Scan mit denselben Glyphen wie die Dialogköpfe, Prüfzeilen, SRV, Reverse, IP-Info, E-Mail-Analyse-Button, Lade- und Dauer-Anzeige); Flaggen-Emojis (unter Windows nur Buchstabenpaare) durch Ländername mit Code. Exporte mit "[OK]"/"[!!]" statt Emojis. Ungenutzte Klasse `IpActionItem` entfernt.
  - Lange Werte (TXT, SPF, DMARC, CAA) brechen um statt rechts abgeschnitten zu werden – die Zeilen lagen in horizontalen StackPanels, in denen `TextWrapping` nie greift; der SPF in der Zusammenfassung wird vollständig statt nach 67 Zeichen gekürzt gezeigt. Lange Werte zusätzlich als Tooltip.
  - Historie: neue Spalte "Status" mit farbigem Punkt (OK / n Hinweise / NXDOMAIN / Abgebrochen).
- **DNS-Lookup – Laufzeit**: SRV-Dienste parallel (max. 4 gleichzeitig) statt nacheinander mit je bis zu 5 s Timeout; SRV, Reverse-Lookup, IP-Info, Propagation und die neuen Prüfungen laufen gleichzeitig, Reverse-Lookups untereinander parallel. Gemessen: 0,2–0,7 s statt 1–2 s bei identischen Ergebnissen.
- **DNS-Lookup – Exporte**: CSV mit allen Abschnitten (Spalten Abschnitt;Typ;Wert;Zusatz;TTL;DNS-Server, korrekt maskiert) – vorher fehlten SRV, Reverse, Propagation und IP-Info. TXT/PDF/Zwischenablage mit Status (NXDOMAIN, Abbruch), Antwortzeit, SPF-Auswertung, MTA-STS/TLS-RPT und Abschnitt Sicherheit.
- **DNS-Lookup – IP-Informationen per HTTPS, abschaltbar**: Statt ip-api.com (ohne Schlüssel nur unverschlüsseltes HTTP) jetzt ipinfo.io per HTTPS: Land (ausgeschrieben), Region, Stadt, Provider/Organisation, ASN, Hostname, Anycast-Kennung ("Standort nicht eindeutig"). Die Einstufung Hosting/Proxy/Mobil entfällt – ipinfo.io bietet sie nur im Bezahltarif. Abfragen laufen parallel über einen gemeinsamen HttpClient, User-Agent "DATEXT-Diagnostics/<Version>" statt "ConnectionTest/2.8". Neuer Schalter "IP-Info online" in der Eingabezeile (gespeichert als `DnsOnlineLookups`, Standard: an); aus = nur DNS-Abfragen, keine Adressen an externe Dienste, Hinweis im Panel. Nicht öffentliche Adressen werden nie online abgefragt.
- **Traceroute – Laufzeit**: Timeout je Versuch 1,5 s statt 3 s (ein stiller Hop kostete 9 s); Abbruch nach 5 Hops ohne Antwort in Folge mit Hinweis "Ziel oder Firewall beantwortet vermutlich keine Pings" (vorher bis zur maximalen Hop-Zahl, oft Minuten). Reverse-DNS läuft im Hintergrund mit 1,5 s Zeitgrenze statt je Hop nacheinander ohne Zeitgrenze; die Namen werden am Ende eingesetzt. Bewusst keine parallele Prüfung mehrerer Hops: Router begrenzen ihre "TTL abgelaufen"-Antworten, gleichzeitige Pakete würden Verlust vortäuschen.
- **Traceroute – Darstellung** (angeglichen an den Port-Scan):
  - Statistikleiste oben statt Karten in einer linken Spalte: Ziel (Adresse, erreicht/nicht erreicht), Hops, max. Latenz, Hops ohne Antwort, Fortschritt ("Hop 7 von max. 30"); die Tabelle hat die volle Breite.
  - Tabelle mit IP (bei Lastverteilung "+n" mit Tooltip), Hostname, Netz (LAN/öffentlich – zeigt, wo die Strecke das eigene Netz verlässt), min / Ø / max, Verlust, Latenzbalken, Sprung, Status; Zielzeile hervorgehoben.
  - Latenzsprünge: Steigt die Laufzeit um mindestens 30 ms gegenüber dem vorigen Hop, wird der Hop markiert ("▲ +87 ms") – zeigt, wo die Verzögerung entsteht. Auch im Protokoll und CSV.
  - Routen-Vergleich: nach jedem Lauf Abgleich mit dem letzten Trace zum selben Host – "Route ab Hop n geändert", Änderung von Hop-Zahl, Ziel-Laufzeit und Erreichbarkeit; im Protokoll und in der Statuszeile.
  - Historie: Spalte "Ziel" mit farbigem Punkt und Text statt Emojis ✅/❌.
  - CSV-Export mit allen neuen Werten (weitere Router, Netz, min/Ø/max, Verlust, Sprung, Status).
- **Port-Scan – Darstellung**:
  - Eine Ergebnisansicht: immer die Tabelle (vorher Karten unter 25 Ports, ohne Antwortzeit und Info), schon während des Scans sichtbar; sortiert nach Status (offen, keine Antwort, gefiltert, geschlossen), innerhalb nach Port; Latenzbalken bei offenen Ports (60 px = 300 ms); Status- und Antwortzeit-Spalte sortieren nach Reihenfolge bzw. numerisch.
  - Klickbare Kacheln OFFEN / GEFILTERT / GESCHLOSSEN / KEINE ANTWORT filtern die Tabelle (zweiter Klick hebt auf, aktive Kachel hervorgehoben ohne Layout-Sprung, Filterhinweis in der Statuszeile).
  - Port-Auswahl: Vorlagen (Web, Mail, Windows/AD, Alle, Keine), aufklappbare Gruppen, Protokoll-Abzeichen je Dienst (z. B. "TCP 443", "UDP 53"); Live-Vorschau unter der eigenen Port-Angabe ("→ 101 Ports (TCP)", bei TCP + UDP mit Anzahl Prüfungen; ungültige Teile rot).
  - Statistikleiste: geprüfte Adresse und Ping-Laufzeit; Fortschrittsbalken mit Restzeit dorthin verlegt (vorher klein unten links).
  - Scan-Vergleich: nach jedem Scan Abgleich mit dem letzten Scan desselben Hosts – "neu offen", "nicht mehr offen", "Status geändert" im Protokoll und kurz in der Statuszeile.
  - Spaltenköpfe der Historie mit farbigen Punkten statt Emojis.
- **Port-Scan – parallele Prüfung**: Ports wurden streng nacheinander geprüft – gefilterte Ports warten jeweils auf den Timeout, große Bereiche dauerten Stunden. Jetzt parallel mit Begrenzung: TCP 32 gleichzeitig, UDP 4 (Router/Firewalls drosseln ICMP-Antworten). Gemessen an 16 überwiegend gefilterten Ports: 24,4 s → 1,6 s bei identischen Ergebnissen. Die Live-Anzeige wird gebündelt alle 250 ms aktualisiert statt je Port; Tabelle, Karten und Protokoll werden am Ende nach Port sortiert.
- **DHCP-Discovery – IPv6-Router (Router Advertisements, passiv)**: Nach der Suche werden die IPv6-Router des Adapters aus dem Zustand gelesen, den Windows aus empfangenen Router Advertisements aufgebaut hat – es werden keine Pakete gesendet: IPv6-Standard-Gateways, im NDP-Cache als Router markierte Nachbarn (`NeighborTable.ReadAllIPv6`, Router-Kennzeichen) und per Router Advertisement gebildete Präfixe (SLAAC, `PrefixOrigin.RouterAdvertisement`), jeweils mit MAC und Hersteller. Bewertung: mehrere IPv6-Router auf einem Adapter → Warnung (fremdes Gerät kann per Router Advertisement auch in reinen IPv4-Netzen Gateway und Adressen verteilen; harmlose Ursache Ausfallsicherheit wird genannt); Router ist dasselbe Gerät wie ein gefundener DHCP-Server → Hinweis; anderes Gerät als der DHCPv4-Server → Hinweis zum Prüfen; SLAAC-Adressen ohne bekannten Router → Hinweis auf veraltete Adressen. Anzeige als eigene Karte "IPv6-RA", im Protokoll, im Warnfeld, im Status "IPv6-Router prüfen" (sofern nicht bereits Rogue-DHCP) und in TXT/PDF-Export. Nicht verfügbar (stehen nur im Paket selbst): M/O-Flags und DNS-Angaben aus dem Router Advertisement.
- **DHCP-Discovery – passive Überwachung**: Neue Option "Danach passiv auf DHCP-Server achten für …" mit frei einstellbarer Dauer (Zahl + Minuten/Stunden/Tage, 1 Minute bis höchstens 30 Tage; größere Werte werden auf 30 Tage begrenzt, ungültige Eingaben vor dem Start gemeldet; Dauer und Einheit werden gespeichert). Nach der aktiven Suche achtet der Dialog auf UDP 68 weiter auf Antworten von DHCP-Servern (OFFER und ACK, nur BOOTREPLY) und findet so Server, die nur zeitweise aktiv sind. Ein bisher unbekannter Server wird sofort mit Zeitstempel gemeldet, als Karte ergänzt ("Passiv gesehen"), ergänzt um Hostname/MAC/Hersteller und neu bewertet; Status "Neuer DHCP-Server!". Ausgewertet werden nur Server-Angaben – die angebotene Client-Adresse wird verworfen, Daten anderer Clients werden weder angezeigt noch protokolliert. Statuszeile mit Restzeit und Zählern (alle 15 s), am Ende Zusammenfassung je Server. Beenden jederzeit mit Stopp. Die Paketauswertung akzeptiert dafür optional DHCP-ACK und verwirft generell alles außer BOOTREPLY.
- **DHCP-Discovery – MAC des DHCPv6-Servers aus der DUID**: Die Server-ID (Option 2) wird ausgewertet. Bei DUID-LLT und DUID-LL mit Hardwaretyp Ethernet enthält sie die MAC des Servers; fehlt der Eintrag im Nachbar-Cache, wird diese für Hersteller und Gruppierung verwendet (Kennzeichnung "aus Server-DUID"). Damit fasst die Gruppierung Dual-Stack-Server auch ohne NDP-Eintrag zusammen. Die DUID (Typ + Hex) erscheint bei den Angebotsdaten. DUID-EN/-UUID enthalten keine MAC. Einschränkung: Manche Geräte bilden die DUID aus der MAC einer anderen eigenen Schnittstelle (z. B. WAN) – dann bleibt die Gruppierung getrennt.
- **DHCP-Discovery – Relay und Absender**: Das BOOTP-Feld `giaddr` wird ausgewertet (gesetzt = Angebot über ein DHCP-Relay, Server in einem anderen Netz → Hinweis). Die Absender-IP jedes Angebots wird festgehalten; weitere Angebote derselben Server-ID werden nicht mehr verworfen, sondern ihre Absender gesammelt. Weicht der Absender ohne Relay von der Server-ID ab → Warnung (Server mit mehreren Adressen oder ein Gerät, das sich als dieser Server ausgibt); kommen Angebote derselben Server-ID von verschiedenen MACs → Warnung (gefälschte Server-ID). Die Absender-MAC stammt aus dem Nachbar-Cache zur Absender-IP (ein UDP-Socket liefert die MAC nicht); eine Fälschung mit identischer IP ist daher nicht erkennbar. Relay und abweichende Absender erscheinen bei den Angebotsdaten.
- **DHCP-Discovery – Angebote vergleichen**: Die DHCPv4-Angebote werden untereinander und mit der vor der Suche festgehaltenen Adapter-Konfiguration (IP/Maske, Gateway, DNS, DHCP an/aus) verglichen. Unterschiedliche Gateways, DNS-Server, Subnetze oder Domänen verschiedener Server ergeben eine Warnung (starkes Rogue-Indiz). Weicht das Angebot eines anderen als des genutzten Servers von der aktuellen Konfiguration ab, ebenfalls eine Warnung (anderes Subnetz besonders hervorgehoben); weicht der genutzte Server selbst ab, ein Hinweis (z. B. geänderte Server-Konfiguration oder statisch eingetragene DNS-Server); bei statisch konfiguriertem Adapter nur Hinweise. Ergebnis im Protokoll (mit aktueller Konfiguration), im Warnfeld, im Status "Abweichende Angebote" (sofern nicht bereits "Rogue-DHCP!") und in TXT/PDF-Export.
- **DHCP-Discovery – sicherheitsrelevante Optionen**: Fragt zusätzlich klassenlose Routen (Option 121, RFC 3442, sowie die ältere Microsoft-Variante 249) und WPAD (Option 252) an und bewertet sie. Routen, deren Bereich öffentliche Adressen umfasst, ergeben eine Warnung (TunnelVision, CVE-2024-3661: DHCP-Routen sind spezifischer als die VPN-Route und leiten Verkehr unverschlüsselt am VPN vorbei ins LAN – z. B. `0.0.0.0/1` + `128.0.0.0/1`). Geprüft wird der gesamte Bereich, nicht nur die Startadresse (`192.168.0.0/15` umfasst auch öffentliche Adressen). Routen ins eigene Subnetz bzw. in andere private Netze und die Standardroute sind Hinweise, mit Vermerk, wenn deren Gateway von Option 3 abweicht. WPAD ist ein Hinweis (in Firmennetzen oft gewollt, von einem Rogue-Server missbrauchbar). Anzeige als eigener Abschnitt auf der Karte, im Protokoll mit Zusammenfassung, in TXT/PDF-Export; Status "Sicherheitswarnung", sofern nicht bereits "Rogue-DHCP!" gilt. Fehlerhafte Routen (Präfixlänge > 32, abgeschnitten) werden verworfen.
- **DHCP-Discovery – alle Angebotsdaten**: Bisher wurden nur Server, angebotene IP, Maske, Gateway und Lease-Zeit ausgewertet, obwohl DNS-Server (6), Domäne (15) und NTP-Server (42) bereits angefragt wurden. Jetzt werden diese angezeigt und zusätzlich Domain-Suchliste (119, inkl. Komprimierung und Aufteilung auf mehrere Optionen), PXE-Boot mit TFTP-Server (66) und Bootdatei (67) sowie Hersteller-Optionen (43, Länge und Hex-Vorschau) angefragt; für klassisches PXE auch die BOOTP-Felder Next-Server (siaddr) und Bootdatei (file). Bei DHCPv6 werden die bereits angefragten DNS-Server (23) und die Domain-Suchliste (24) ausgewertet. Anzeige auf den Karten, im Protokoll und in TXT-, PDF- und CSV-Export (CSV mit neuen Spalten). Anfragepuffer 576 statt 300 Bytes (DHCP-Mindestgröße).
- **DHCP-Discovery – CSV-Export**: In der DHCPv6-Zeile stand die angebotene Adresse durch ein überzähliges Semikolon in der Spalte "Subnetzmaske".
- **DHCP-Discovery – Client-Kennung umschaltbar**: Neue Option "Lokal verwaltete MAC verwenden (empfohlen)", Standard an, Zustand wird gespeichert (`DhcpUseRandomMac`). An: DISCOVER und SOLICIT nutzen pro Suche eine zufällige, lokal verwaltete Unicast-MAC und den Hostnamen "DATEXT-Scanner" – die eigene IP-Vergabe und der Gerätename dieses PCs im Router bleiben unberührt. Aus: echte Adapter-MAC mit dem eigenen Rechnernamen (vorher echte MAC zusammen mit "DATEXT-Scanner", wodurch Router den Gerätenamen des PCs überschreiben konnten). Die verwendete Kennung steht im Protokoll. Antwortet im Zufallsmodus kein Server, nennt der Hinweis MAC-Filter und DHCP-Snooping als mögliche Ursache und empfiehlt, die Option auszuschalten.
- **DHCP-Discovery – MAC-Ermittlung**: Über die Windows-Nachbartabelle (`NeighborTable.FindMac`, `GetIpNetTable2`, IPv4 und IPv6) statt Textauswertung von `arp -a` bzw. `netsh interface ipv6 show neighbors` mit Prozessstart je Server. Fehlt der Eintrag, löst ein kurzer Ping die ARP-/NDP-Auflösung aus (auch bei verworfenem Ping). Nicht freigegebenes `Ping`-Objekt entfällt. `NeighborTable` hat dafür `ReadAllIPv6()` (mit Router-Kennzeichen) und `FindMac()` erhalten.
- **DHCP-Discovery – Hersteller**: Aus der zentralen MAC-Hersteller-Datenbank (`LocalDeviceIdentificationService`, ~59.000 Einträge) statt einer eigenen Tabelle mit ~40 Präfixen (teils fragwürdig, z. B. `06:00:00` = "Hyper-V (lokal)") und Online-Abfrage bei macvendors.com (Limit, Daten nach außen). Unbenutzte `LookupMacVendorOnlineAsync` entfernt.
- **DHCP-Discovery – Ablauf**: Hostname, MAC und Hersteller werden erst nach der Wartezeit für alle Server parallel ermittelt. Vorher lief das je Server mitten in der Empfangsschleife (Reverse-DNS bis 2 s), kurz vor Ablauf konnten dabei Antworten verloren gehen.
- **DHCP-Discovery – Wartezeit**: 5 s statt fest 10 s (DHCP-Server antworten meist in unter einer Sekunde), Wiederholung der Anfrage nach 2 statt 4 s. Ein gemeinsamer Countdown statt zwei, die abwechselnd in dieselbe Statuszeile schrieben.
- **Verbindungen – Datenquelle**: Statt `netstat -ano` (Prozessstart alle 5 s bei Auto-Refresh, Textausgabe in Codepage 850 mit lokalisierten Statusnamen) jetzt `GetExtendedTcpTable`/`GetExtendedUdpTable` (IPv4 + IPv6, Zustand als Code, PID). Gegengeprüft: gleiche Einträge wie netstat, ca. 80 ms (20 ms mit Pfad-Cache). Prozessnamen aus einer einzigen Prozess-Momentaufnahme je Aktualisierung, Prozesspfade über Aktualisierungen hinweg gecacht (Schlüssel PID + Startzeit, erkennt wiederverwendete PIDs).
- **Verbindungen – Aktualisierung**: Die Tabelle wird nicht mehr vorab geleert (kein Flackern), bekannte Werte für Land, Signatur und Reputation werden sofort aus den Caches gesetzt statt bei jedem Refresh leer zu bleiben; nur Fehlendes wird nachgeladen. Ein laufender Ergänzungslauf wird bei neuem Refresh abgebrochen. Caches sind threadsicher (`ConcurrentDictionary`). Signaturprüfung läuft mit 4 parallelen Prüfungen.
- **Verbindungen – Online-Abfragen**: ip-api.com über den Batch-Endpunkt (bis 100 IPs in einer Anfrage statt bis zu 30 Einzelanfragen – Limit 45/min wurde mit Auto-Refresh überschritten), nach HTTP 429 eine Minute Pause, Fehlschläge 5 min gemerkt (vorher bei jedem Refresh erneut abgefragt). Shodan-Netzwerkfehler 10 min pro IP pausiert. JSON-Auswertung mit `System.Text.Json` statt Regex. Neuer Schalter im Aktualisieren-Menü "Online-Abfragen (Land, Reputation)" (gespeichert in den Einstellungen, Standard: an): aus = keine Remote-Adressen an externe Dienste.
- **Verbindungen – Filter/Suche**: Tabelle wird in einem Schritt ersetzt (ein Reset-Event statt eines Events je Zeile, Sortierung bleibt erhalten), Suche filtert erst 200 ms nach dem letzten Tastendruck. Doppelten Code zusammengefasst (Top-Prozesse, SHA256-Berechnung), toten Code entfernt (`UpdateHistoryPanel`, netstat-Statusübersetzung, `RunCmd`, netsh-Parser).
- **Netzwerk-Analyse – TCP-Einstellungen**: Per WMI `MSFT_NetTCPSetting` (Vorlage "Internet") und `MSFT_NetOffloadGlobalSetting` (root\StandardCimv2, Grundlage von `Get-NetTCPSetting`) statt netsh-Textauswertung – sprachunabhängig, ohne Prozessstart. Werte-Codes aus der Windows-Moduldefinition (`NetTCPIP\*.cdxml`) übernommen. Die Verbindungsauswertung läuft im Hintergrund statt auf dem UI-Thread und nutzt den dort gelesenen Port-Bereich (vorher zusätzlicher netsh-Aufruf auf dem UI-Thread).
- **MAC-Hersteller-Datenbank** (`Resources\MacVendorDatabase\mac-vendors-export.json`) aktualisiert: Stand 03.10.2026 (vorher 13.02.2026), 59.005 statt 56.906 Einträge (+2.099, keine entfallen, 207 umbenannt). Quelle unverändert maclookup.app (IEEE-Daten, gleiches Format); Direktdownload `https://maclookup.app/downloads/json-database/get-db`. Neues Skript `Resources\MacVendorDatabase\Update-MacVendorDatabase.ps1` für künftige Updates: lädt die Datei, prüft sie (JSON-Array, Felder, keine doppelten Präfixe – der Ladecode bricht sonst ab –, nicht über 10 % kleiner als bisher), sichert die alte als `mac-vendors-export.<Datum>.json.bak` und aktualisiert `liesmich-mac-database.txt`; `-WhatIf` lädt und prüft nur. Danach neu bauen bzw. publishen.
- **PktMon Capture – Neuaufbau**: Der Dialog ist wie der E-Mail-Check aufgebaut, mit Einstellungen links, Protokoll rechts und verschiebbarer Trennlinie. Das Verhältnis wird gespeichert, Doppelklick setzt 42/58 zurück; schmale Fenster schalten mit „Einstellungen | Protokoll“ um.
  - **Optionen:** Komponenten (alle / nur Netzwerkkarten, `--comp nics`), Pakete (alle / nur verworfene, `--type drop`), Paketgröße (`--pkt-size`), maximale Dateigröße in MB (`-s`) und Protokollmodus (Ringpuffer, mehrere Dateien, Speicher). Die Werte werden gespeichert.
  - **Live-Kacheln:** Laufzeit, Pakete, verworfene Pakete und Dateigröße aus `pktmon counters --json`, alle 2 s.
  - **Filter:** Eingabe mit Prüfung, Vorlagen (DNS, HTTPS, SMB, RDP, DHCP, Ping) und Anzeige der aktiven Filter aus `pktmon filter list`. Ein Hinweis erklärt, dass Filter erst für die nächste Aufzeichnung gelten.
  - **Aufzeichnungen:** Die letzten 15 ETL-/PCAPNG-Dateien aus dem neuen und dem alten Ordner, mit „PCAPNG“ (umwandeln), „Wireshark“ und „Im Ordner zeigen“. Wireshark wird über „App Paths“ in der Registry gefunden.
  - **Umwandlung:** Nach dem Stoppen wird die Datei automatisch in PCAPNG umgewandelt (abschaltbar).
  - **Abfragen:** „Paketverluste“ (`counters --type drop -r`, mit letztem Grund je Komponente), „Zähler“ und „Komponenten“ schreiben ins Protokoll.
  - **Protokoll und Export:** Das Protokoll ist farbig ([OK]/[!]/[X]/[i]), mit Zeilenumbruch-Schalter. Kopieren und Export als TXT oder PDF enthalten Rechner, Speicherort, Zähler und Filter. Statt MessageBoxen gibt es eine Statuszeile, die Emojis sind entfernt.
- **Speicherort der Netzwerkmitschnitte**: PktMon und Netsh-Trace speichern unter `%LocalAppData%\DATEXT\NetworkTraces` statt unter „Dokumente\DATEXT-Diagnostics\NetworkTraces“. Bei einer OneDrive-Umleitung von „Dokumente“ wurden ETL-Dateien bis 512 MB mitsynchronisiert. Vorhandene Dateien im alten Ordner zeigt der PktMon-Dialog weiter an. Der Battery-Report bleibt unter „Dokumente“. Der Netsh-Dialog nennt den Pfad jetzt dynamisch statt fest „Documents\…“.
- **Netsh-Trace – Neuaufbau**: Der Dialog ist wie PktMon aufgebaut: Einstellungen links, Protokoll rechts, verschiebbare Trennlinie (gespeichert, Doppelklick 42/58) und Umschalter für schmale Fenster.
  - **Szenarien:** Liste mit Mehrfachauswahl und Beschreibung, die Auswahl wird gespeichert. „Anbieter“ zeigt die ETW-Anbieter der gewählten Szenarien statt der kompletten, tausendzeiligen Anbieterliste.
  - **Paketmitschnitt:** an/aus, Schnittstellen (physisch / virtueller Switch / beides), Filter nach IP-Adressen und Protokoll (geprüft; netsh kennt keine Portfilter).
  - **Datei:** maximale Größe, „Wenn voll“ (überschreiben / anhalten), Zusatzbericht (.cab) abschaltbar (`report=disabled`, Stoppen deutlich schneller), `persistent=yes` für Aufzeichnungen über einen Neustart.
  - **Live-Kacheln:** Laufzeit, Dateigröße, Szenarien, Paketmitschnitt.
  - **Aufzeichnungen:** ETL, CAB, PCAPNG und Berichte mit „PCAPNG“, „Bericht“, „Wireshark“, „Öffnen“ und „Im Ordner zeigen“.
  - **Protokoll und Export:** farbiges Protokoll, Statuszeile statt MessageBoxen, Export als TXT oder PDF. Die Emojis sind entfernt. „Firewall“ und die volle Anbieterliste sind entfallen, „Schnittstellen“ ist neu.
- **Gemeinsame Bausteine für PktMon und Netsh-Trace**: `TraceFileList` (Liste „Aufzeichnungen“ mit Wireshark-Suche über „App Paths“), `TraceTools` (Aufruf der Konsolenprogramme in UTF-8 mit Zeitlimit, Neustart als Administrator) und `ICaptureSession` für die Nachfrage beim Beenden. Sie gilt jetzt für beide Dialoge zusammen.
- **etl2pcapng**: `Etl2PcapngService` ohne MessageBoxen. Das Tool liegt jetzt in `Tools\etl2pcapng\` statt `Tools\Tools\`; ein geprüftes Tool aus dem alten Ordner wird übernommen. Die Ausgabe heißt `.pcapng` statt `.pcap`.
- **Routing-Tabelle – Neuaufbau**: Die Daten kommen aus `MSFT_NetRoute` (aktiver und persistenter Speicher) statt aus geparstem `route print`. Dadurch gibt es Adapternamen, Routen- und Schnittstellenmetrik getrennt (Tooltip „Wirksam 271 = Route 256 + Schnittstelle 15“) und die Herkunft (Lokal, Konfiguriert, DHCP, Router-Ankündigung), für IPv4 und IPv6 im gleichen Format.
  - **Bewertung:** Karte mit StatusRows; Klick filtert die Tabelle auf die betroffenen Routen. Geprüft werden:
    - fehlende IPv4-Default-Route
    - mehrere Default-Routen (gleiche Metrik = Warnung)
    - Default-Gateway mit APIPA-Adresse oder außerhalb der Netze des Adapters
    - gleiches Netz auf mehreren Adaptern (typisch VPN plus Heimnetz)
    - persistente Routen, die nicht aktiv sind
    - manuell eingetragene Routen (Hinweis)
  - **Filter:** IPv4 / IPv6 / beide als Chips, Systemrouten (Loopback, Multicast, Broadcast, Host-Routen, Link-Local) standardmäßig ausgeblendet, Suche nach Ziel, Gateway und Adapter. Die Einstellungen werden gespeichert.
  - **Kacheln:** Default-Gateway, Routen (angezeigt/gesamt), Adapter, persistent (davon inaktiv), Auffälligkeiten.
  - **Route hinzufügen:** Formular mit Ziel/Präfix, Gateway, Adapter (Vorauswahl: Adapter der genutzten Default-Route), Metrik und „persistent“ (Standard aus). Eingaben werden geprüft (Hostbits, IP-Version, Metrik), Hinweise erscheinen bei einem Gateway außerhalb des Adapternetzes, einer zusätzlichen Default-Route oder einem getrennten Adapter. Eine Vorschau zeigt den genauen `netsh`-Befehl.
  - **Kontextmenü:** „Zeile kopieren“, „Befehl zum Anlegen kopieren“ (mit Routenmetrik), „Als Vorlage für neue Route“, „Route löschen“ (auch mit Entf).
  - **Ohne Adminrechte:** Die Tabelle wird nur angezeigt, mit Hinweisleiste und „Als Administrator neu starten“.
  - **Kopieren und Export:** Bericht (Bewertung und angezeigte Routen) zum Kopieren und als TXT/PDF. Zeilen werden nur noch bei Auffälligkeiten eingefärbt (vorher sechs Farben nach Kategorie, mit Legende); Emojis entfernt.

### Fixed
- **Routing-Tabelle – Hinzufügen und Löschen ohne Wirkung**: `Verb = "runas"` mit `UseShellExecute = false` wird von Windows ignoriert. Ohne Adminrechte scheiterten beide Aktionen, beim Löschen ohne Meldung (Exit-Code nicht geprüft, Tabelle neu geladen, als hätte es geklappt). Jetzt laufen sie nur mit Adminrechten, und das Ergebnis von netsh steht in der Statuszeile.
- **Routing-Tabelle – Löschen zu breit**: `route delete <Ziel>` entfernte **alle** Routen zu diesem Ziel, auf allen Adaptern und mit allen Gateways. Jetzt wird gezielt gelöscht (`netsh interface ipv4|ipv6 delete route prefix=… interface=… nexthop=…`), auch IPv6-Routen (vorher leere Argumente) und persistente Routen. Vor dem Löschen kommt eine Rückfrage mit dem genauen Befehl, bei einer aktiven Default-Route mit zusätzlicher Warnung. Von Windows aus der Adapteradresse angelegte Routen werden nicht gelöscht, weil sie sofort zurückkämen.
- **Routing-Tabelle – „Route hinzufügen“ führte freien Text aus**: Das erste Wort der Eingabe wurde als Programm gestartet. Vorgabe war eine neue Default-Route, „persistent“ war vorab angehakt. Jetzt gibt es ein Formular mit fester Befehlszeile und ausdrücklichem `store=active` (der Standard von netsh ist persistent).
- **Routing-Tabelle – „Route kopieren“**: Der Befehl übernahm die wirksame Metrik (Route plus Schnittstelle), sodass eine nachgebaute Route eine falsche Metrik bekam, und „Auf Verbindung“ als Gateway. Danach kam eine MessageBox.
- **Routing-Tabelle – doppelte Zeilen**: Laden beim Öffnen, IPv6-Haken und Aktualisieren liefen parallel in dieselbe Liste. Jetzt zählt nur das Ergebnis des letzten Ladevorgangs.
- **Routing-Tabelle – persistente Routen**: Inaktive persistente Routen fehlten, weil der Abschnitt „Ständige Routen“ übersprungen wurde. Die Persistenz kam aus der Registry, verglich nur das Ziel und kannte keine mit netsh oder über eine feste IP-Konfiguration angelegten Routen (Testrechner: Registry 0, tatsächlich 2, davon eine inaktive Default-Route).
- **Routing-Tabelle – „Konflikt“**: Gleiches Ziel mit verschiedenen Gateways galt als Konflikt; das ist bei zwei Adaptern normal. Die echten Fehlerbilder werden jetzt in der Bewertung geprüft (siehe Changed).
- **Netsh-Trace – Szenario und Paketmitschnitt wirkungslos**: Der Start-Befehl war fest `trace start capture=yes …`. Das gewählte Szenario und der Haken „Paket-Capture“ wurden nie übergeben, jede Aufzeichnung war ein reiner Paketmitschnitt ohne Systemereignisse.
- **Netsh-Trace – „ETL analysieren“**: Der Befehl `trace diagnose input=… output=…` existiert so nicht (`diagnose` startet eine Diagnosesitzung). Jetzt `trace convert … dump=TXT report=yes`, erreichbar über „Bericht“ in der Liste. Die Schaltflächen „ETL analysieren“ und „Wireshark“ wurden außerdem nie aktiviert, „PDF“ verwies nur auf „System-Infos exportieren“.
- **Netsh-Trace – Stoppen**: Bei einem Fehler galt der Trace trotzdem als beendet, ein zweiter Versuch war nicht möglich. Jetzt wird danach `trace show status` geprüft. Das Stoppen mit Zusatzbericht (oft Minuten) zeigt einen Hinweis und hat ein Zeitlimit von 30 min statt keinem.
- **Netsh-Trace – laufender Trace**: Ein nach einem Neustart der App oder außerhalb gestarteter Trace wird beim Öffnen erkannt und kann gestoppt werden. Beim Beenden der App wird nachgefragt.
- **Netsh-Trace – Kodierung**: Umgeleitet schreibt netsh UTF-8 (nachgeprüft). Die Szenarien wurden als OEM 850 gelesen und per Ersetzungstabelle „repariert“, die übrigen Befehle mit der Konsolen-Codepage. Umlaute waren deshalb falsch.
- **Netsh-Trace – Navigation**: Beim erneuten Öffnen wurden Protokollkopf und Szenarien jedes Mal neu geschrieben, und die Auswahl sprang zurück, auch während eines laufenden Traces.
- **Netsh-Trace – Neustart als Administrator**: Die Sperre für die Einzelinstanz wurde nicht freigegeben, die App schloss sich ganz.
- **etl2pcapng – Signatur**: Die heruntergeladene EXE wurde ohne Prüfung ausgeführt. Jetzt wird erst nach Rückfrage in eine Zwischendatei geladen und nur übernommen, wenn die Authenticode-Signatur gültig und der Herausgeber „Microsoft Corporation“ ist.
- **PktMon Capture – falsche Parameter**: Die Paketgröße wurde mit `-s` übergeben; das ist aber die Dateigröße in MB. „128 Bytes“ ergab also eine Datei von höchstens 128 MB mit vollen Paketen. Jetzt gilt `--pkt-size` für die Paketgröße und `-s` für die Dateigröße. Der „Echtzeit“-Modus mit `--etw` ist entfallen; er schrieb keine auswertbare Datei.
- **PktMon Capture – Zähler**: Die Anzeige zeigte nur „Aktualisiert hh:mm:ss“. Jetzt wird `counters --json` ausgewertet: Pakete der größten Komponente (ein Paket durchläuft mehrere Komponenten), verworfene Pakete summiert.
- **PktMon Capture – laufende Aufzeichnung**:
  - **Nach Neustart der App:** Eine laufende Aufzeichnung wurde nicht erkannt und ließ sich nicht stoppen. Beim Öffnen fragt der Dialog jetzt `pktmon status` ab und übernimmt sie.
  - **Beim Beenden:** Die Aufzeichnung lief unbemerkt weiter. Jetzt fragt die App: Ja = stoppen, Nein = weiterlaufen lassen, Abbrechen. Das Menü „Beenden“ läuft dafür über `Close()` statt direkt `Shutdown()`.
- **PktMon Capture – Verfügbarkeit**: Die Prüfung suchte nach „is not recognized“ in der Ausgabe. Ein fehlendes pktmon.exe löst aber eine Win32Exception aus, die stumm abgefangen wurde. Jetzt wird sie erkannt, und die Schaltflächen werden gesperrt.
- **PktMon Capture – Kodierung**: Umlaute in der pktmon-Ausgabe waren falsch dekodiert. Umgeleitet schreibt pktmon UTF-8 (nachgeprüft), und so wird die Ausgabe jetzt gelesen.
- **PktMon Capture – Ausgabeformat**: `etl2pcap` erzeugt PCAPNG; die Datei hieß aber `.pcap`. Jetzt `.pcapng`.
- **PktMon Capture – Filter**: Port (1–65535, höchstens zwei), IP/CIDR, Filtername und ICMP mit Port werden vor dem Aufruf geprüft. Vorher landeten Fehleingaben ungeprüft bei pktmon.
- **PktMon Capture – Neustart als Administrator**: Die Sperre für die Einzelinstanz wurde nicht freigegeben. Die neue Instanz beendete sich sofort, danach die alte. Jetzt `App.ReleaseSingleInstance()`; bei Abbruch der Benutzerkontensteuerung (1223) läuft die App weiter.
- **Windows-Komponenten und DHCP-Discovery – Neustart als Administrator**: Derselbe Fehler wie bei PktMon: Die neue Instanz beendete sich sofort, danach schloss sich die alte, und es blieb kein Fenster offen. Ein Abbruch der Benutzerkontensteuerung wurde außerdem als Fehler „Neustart nicht möglich“ gemeldet. Beide nutzen jetzt die zentrale Methode `App.RestartAsAdministrator()`, ebenso PktMon und Netsh-Trace über `TraceTools`. Damit geben alle Dialoge mit „Als Administrator neu starten“ die Sperre frei.
- **PktMon Capture – PDF-Schaltfläche**: Sie verwies nur auf „System-Infos exportieren“. Jetzt erzeugt der Export echte PDF- und TXT-Berichte.
- **Internet-Speedtest – Transferrechner zehnfach falsch**: Die Messwerte wurden aus den Textfeldern zurückgelesen. Die deutsche Anzeige „123,4“ wurde mit InvariantCulture gelesen, also als 1234 Mbit/s.
  - **Betroffen:** Transferzeiten (10 GB bei 123,4 Mbit/s: „1,1 min“ statt 11 min), manuelle Eingaben („100,5“ → 1005), die Verbindungsklasse im Export und die Ping-Bewertung.
  - **Lösung:** Die Werte werden jetzt als Zahlen geführt; Eingaben werden in Landeskultur gelesen (Komma oder Punkt).
- **Internet-Speedtest – Abbrechen**:
  - **Fehlermeldung nach „Stopp“:** Nach dem Abbrechen folgte die MessageBox „Fehler beim Speedtest … TCP 8080 blockiert“.
  - **Prozessbaum:** Es wurde nur der Prozess beendet, nicht der Prozessbaum.
- **Internet-Speedtest – Lizenz und Datenschutz**: Der Dialog schrieb ungefragt eine Zustimmungsdatei mit fest hinterlegtem Lizenz-Hash in zwei AppData-Ordner und startete mit `--accept-license --accept-gdpr`. Die Ookla-Bedingungen bekam der Anwender nie zu sehen. Jetzt wird vor der ersten Messung einmalig nachgefragt, mit Links zu Lizenz und Datenschutz und dem Hinweis auf die Datenweitergabe durch Ookla. Die Zustimmung wird gemerkt.
- **Internet-Speedtest – Download der CLI**:
  - **Prüfsumme:** Die heruntergeladene ZIP-Datei wird per SHA-256 geprüft; Ookla signiert `speedtest.exe` nicht.
  - **Tote Adresse:** Die generische Ersatzadresse liefert inzwischen 403 und ist entfernt.
  - **Suche im PATH:** Akzeptiert wird nur noch die Ookla-CLI. Die Python-Variante „speedtest-cli“ heißt ebenfalls `speedtest.exe`, kennt die Parameter aber nicht.
  - **Kein Blockieren:** Die Prüfung läuft im Hintergrund statt synchron im UI-Thread bei jedem Öffnen.
- **Internet-Speedtest – Firewall-Prüfung**:
  - **Server:** Geprüft wurden 5 zufällige Server weltweit aus der statischen Liste. Jetzt sind es die nächstgelegenen Server, die die Messung tatsächlich nutzt (`speedtest -L`), dazu speedtest.net:443.
  - **Dauer:** Die Prüfung läuft parallel, rund 4 s statt bis zu 50 s; Port 8080 wird nicht mehr doppelt geprüft.
  - **Ausgabe:** Eine Ergebnistabelle ersetzt Rahmengrafik und Emojis.
- **Internet-Speedtest – Kleinere Fehler**:
  - **Paketverlust:** Liefert der Server keinen Wert, erscheint „nicht gemessen“ statt „0 % – Kein Verlust“.
  - **Einheitenauswahl:** Sie zeigte nur „…“ (44 px breit).
  - **Debug-Ausgabe:** Der Block „=== DEBUG INFO ===“ im Protokoll ist entfernt.
  - **Export:** Die Eignung im Export entspricht jetzt der Anzeige (vorher nur nach Download bewertet).
  - **Toter Code:** Die ungenutzte `SpeedtestDialog.xaml(.cs)` ist entfernt.
- **E-Mail-Check – Tastenkürzel ohne Funktion**: Enter, Strg+T, Strg+R, Strg+D und Esc hingen am Window, das nie angezeigt wurde, und funktionierten nicht, obwohl das Banner sie nannte. Ebenso liefen Startfokus und Initialisierung nie, und das Zeitlimit wurde nie gespeichert (nur in `OnClosing`).
- **E-Mail-Check – Abbrechen**: „Abbrechen“ gab die Oberfläche sofort frei, obwohl der Test weiterlief. Ein zweiter Test überschrieb dann den Abbruch-Token, beide schrieben in dasselbe Protokoll, und die Abbruchmeldung erschien doppelt. Jetzt endet die Prüfung wirklich, bevor die Buttons frei werden; im Test sofort statt nach 4 s (die Port-Vorprüfung war nicht abbrechbar). Ein Abbruch erscheint als „Abgebrochen.“ statt als Fehler „The operation was canceled“. Autodiscover lässt sich abbrechen und zeigt sein Protokoll live statt erst am Ende. Beim Verlassen des Dialogs wird eine laufende Prüfung beendet.
- **E-Mail-Check – Alter Server nach Adresswechsel**: Nach Änderung der E-Mail-Adresse testeten „Verbindung testen“ und die Testmail weiter gegen den für die vorherige Domain ermittelten Server. Außerdem wurde bei jedem Tastendruck der Benutzername überschrieben und im Hauptfenster die Domain-Analyse verworfen.
- **E-Mail-Check – Geratener Server**: Ohne ermittelten Server testeten die Schnelltests stillschweigend `mail.<domain>`. Jetzt ist „Verbindung testen“ erst mit eingetragenem Server aktiv.
- **E-Mail-Check – Protokoll im falschen Reiter**: Meldungen der Dienste landeten im gerade ausgewählten Reiter. Klickte man während eines Tests auf einen anderen, wurde dieser fortgeschrieben.
- **E-Mail-Check – KIM-Hinweis**: Bei `.telematik`-Servern verwies eine Meldung auf die ausgeblendeten TI-Optionen. Jetzt steht ein Hinweis im Protokoll, und der Test läuft.
- **E-Mail-Check – IMAP-Empfangsprüfung**:
  - **Zustellzeit:** Sie zählte ab dem Schließen der Erfolgsmeldung und hing damit von der Reaktionszeit des Anwenders ab. Jetzt zählt sie ab der Annahme durch den SMTP-Server.
  - **Suche:** Statt alle 5 s bis zu 50 komplette Mails herunterzuladen, wird serverseitig nach der Message-ID gesucht.
- **E-Mail-Check – IMAP/POP3-Ermittlung**: Autodiscover V2 übernahm den REST-Endpunkt von Exchange als IMAP- und POP3-Server. Jetzt wird das gemeldete Protokoll geprüft.
- **E-Mail-Check – Verbindungstest langsam**: Jeder Verbindungstest ermittelte vorab die öffentliche IP und fragte DNS-Blacklists ab. Das ist jetzt Teil der Sicherheitsanalyse.
- **E-Mail-Check – Anzeige fehlgeschlagener Verbindungen**: Bei abgebrochener TLS-Aushandlung zeigte die Verschlüsselung „TLS“ als in Ordnung. Das Statusbadge blieb bei laufenden Tests grau, weil „läuft“ case-sensitiv mit „Läuft“ verglichen wurde. Fehler des Anmeldedialogs erschienen als MessageBox mit Stacktrace.
- **E-Mail-Check – Anmeldedaten im Protokoll**: Das SMTP-Protokoll schwärzt AUTH-Daten jetzt ausdrücklich (`RedactSecrets`).
- **SPF-Prüfung (E-Mail-Check, Header-Analyse)**:
  - **redirect=:** Es wurde nicht ausgewertet; gmail.com galt als „kein 'all'“ und eine Google-IP als „nicht erlaubt“.
  - **Qualifizierer:** `-ip4:`/`~include:` zählten als Treffer.
  - **IPv6:** Für `a`/`mx` wurde nur IPv4 geprüft.

  Jetzt erfolgt die Auswertung nach RFC 7208 (erster Treffer entscheidet, include/redirect bis 10 Ebenen, a/mx mit IPv4/IPv6 und CIDR).
- **Blacklist-Prüfung**: Treffer zeigen die Bedeutung bei Spamhaus (SBL Spamquelle, XBL kompromittiert, PBL Endkunden-Adressbereich). Ein reiner PBL-Eintrag ist in der Header-Analyse ein Hinweis, kein Problem.
- **E-Mail-Header-Analyse – Zeitstempel**: Die Received-Zeiten wurden mit `DateTime.TryParse` in deutscher Kultur gelesen. Das scheiterte am Standardformat mit Wochentag und `(CEST)`, die Verzögerungen standen fast immer auf „—“. Jetzt werden sie mit dem RFC-5322-Parser von MimeKit gelesen.
- **E-Mail-Header-Analyse – Absender-IP**: Geprüft wurde die erste IPv4-Zahl im ältesten Received-Header, meist ein interner Client (10.x/127.0.0.1) oder der Heimanschluss des Absenders; IPv6 fehlte. Jetzt gilt der Reihe nach: Received-SPF `client-ip`, Microsoft `CIP`, „sender IP is“ bzw. „designates … as permitted sender“, sonst der erste Hop von außerhalb der Empfängerdomain mit öffentlicher IP.
- **E-Mail-Header-Analyse – Eigene IP**: Bei jeder Analyse wurden die eigene öffentliche IP ermittelt und auf Blacklists geprüft; das hat keinen Bezug zur Mail und entfällt.
- **E-Mail-Header-Analyse – Eingefügte .eml**: Textzeilen mit Doppelpunkt („Hinweis: …“) wurden als Header gelesen, weil der Parser an der Leerzeile nicht aufhörte. Kodierte Betreffs und Namen (`=?utf-8?B?…?=`) blieben unlesbar. Vorangestellte Zeilen wie „Microsoft Mail Internet Headers Version 2.0“ werden jetzt übersprungen.
- **E-Mail-Header-Analyse – Werte der vorigen Analyse**: Fehlte im neuen Header SPF, DKIM oder DMARC, behielt das Badge die Farbe der vorigen Mail. Auth-Details blieben stehen und bekamen das Blacklist-Ergebnis angehängt. Beim Wiederherstellen aus dem Verlauf passierte dasselbe.
- **E-Mail-Header-Analyse – DKIM**: Bei mehreren Signaturen zählte nur die erste. Eine gültige Signatur des Versanddienstes überdeckte eine fehlgeschlagene der Absenderdomain.
- **E-Mail-Header-Analyse – BIMI-Logo**: Das Logo wurde ohne Zertifikatsprüfung von der Adresse aus dem DNS der *angeblichen* Absenderdomain geladen. Bei Phishing-Mails rief der PC damit einen Server des Angreifers auf. Angezeigt werden konnte es ohnehin nie, weil BIMI nur SVG erlaubt. Jetzt werden nur Eintrag und VMC angezeigt.
- **E-Mail-Header-Analyse – Parallele Analysen**: Während der Netzabfragen blieb „Analysieren“ ohne Fortschrittsanzeige aktiv; ein zweiter Klick schrieb parallel in dieselben Felder. Meldungen erscheinen in der Statuszeile statt als MessageBox.
- **Geteilte TXT-Einträge (E-Mail-Analyse, E-Mail-Check, Header-Analyse)**: TXT-Einträge über 255 Zeichen bestehen aus mehreren Teilstrings, gelesen wurde nur der erste. Lange SPF-Einträge und DKIM-Schlüssel ab 2048 Bit waren dadurch abgeschnitten. Die Teilstrings werden jetzt zusammengesetzt.
- **E-Mail-Analyse – Fehlgeschlagene Abfragen**: Eine fehlgeschlagene DNS-Abfrage galt als „kein Eintrag“. Bei microsoft.com über Google-DNS meldete der Dialog „SPF fehlt“, weil die Antwort zu groß für UDP und TCP-Port 53 gesperrt war. Jetzt steht „Nicht ermittelbar“ mit Hinweis im Protokoll.
- **E-Mail-Analyse – DMARC**: `sp=` wurde als `p=` gelesen, weil nach dem Text „p=“ gesucht wurde.
- **E-Mail-Analyse – Blacklists**: Geprüft wurde die öffentliche IP dieses PCs statt der Server der Domain, über öffentliche Resolver. Diese beantworten Spamhaus-Abfragen nicht, das Ergebnis war deshalb immer „sauber“.
- **E-Mail-Analyse – Mail-Fluss**: Bewertet wurde die DMARC-Richtlinie des Empfängers statt der des Absenders.
- **E-Mail-Analyse – SPF**: Die Textsuche wertete „mx“ auch innerhalb eines include-Namens als Mechanismus; mehrere SPF-Einträge (PermError) wurden nicht erkannt.
- **E-Mail-Analyse – DKIM**: Die Suche endete beim ersten gefundenen Selektor und zeigte keine Schlüssellänge.
- **E-Mail-Analyse – MX**: Null-MX (`0 .`) galt als Server ohne Adresse. Ein MX mit IP-Adresse statt Namen (interne Zonen) wurde als Hostname aufgelöst und als „ohne Adresse“ gemeldet.
- **E-Mail-Analyse – Verlauf**: Ein Adresswechsel im E-Mail-Check erzeugte den Dialog neu; Verlauf und Ergebnis gingen verloren.
- **E-Mail-Analyse – Sonstiges**: Das BIMI-Logo wird nicht mehr von fremden Servern geladen. Fehler erscheinen in der Statuszeile statt als MessageBox. Der TXT-Export schreibt UTF-8 mit BOM, Umlaute sind in Editoren dadurch lesbar.
- **Lokale Inventarisierung – Neustart als Administrator**: Die Sperre für die Einzelinstanz wurde vorher nicht freigegeben. Die neue Instanz beendete sich sofort, danach schloss sich die alte, und es blieb kein Fenster offen. Ein Abbruch der Benutzerkontensteuerung zeigt keine Fehlermeldung mehr.
- **Inventarisierung und System-Übersicht – Computer-SID**: Immer leer, in der System-Übersicht stand „–“. Gesucht wurde die Gruppe „Administrators“, die auf deutschem Windows „Administratoren“ heißt; selbst dann wäre es die BUILTIN-SID S-1-5-32 gewesen. Jetzt die SID des lokalen Administrator-Kontos (RID 500) ohne RID.
- **Inventarisierung – Datenträgertyp**: Jede SSD galt als „HDD“, weil `Win32_DiskDrive` als MediaType immer „Fixed hard disk media“ meldet. Jetzt über `MSFT_PhysicalDisk` (SSD/HDD, Bus, Zustand); virtuelle Datenträger heißen „Virtuell“.
- **Inventarisierung – Update-Datum**: WMI liefert „M/d/yyyy“. Mit deutscher Einstellung wurde „9/10/2026“ als 9. Oktober gelesen und „10/16/2025“ gar nicht; die Sortierung im Reiter und im PDF war falsch. Das Datum wird jetzt kulturunabhängig gelesen und als ISO-Datum gespeichert; ältere Inventare werden ebenfalls richtig gelesen.
- **Inventarisierung – Grafikspeicher**: `AdapterRAM` ist ein 32-Bit-Wert; Karten ab 4 GB zeigten 4095 MB oder 0. Jetzt aus der Registry (`HardwareInformation.qwMemorySize`).
- **Inventarisierung – OEM-Codepage**: `auditpol`, `nltest` und der DISM-Fallback lesen in Codepage 850, deren Provider im Agent nie registriert war. Die Abfragen scheiterten dort still, die Überwachungsrichtlinie blieb leer; in Diagnostics hing es davon ab, welcher Dialog vorher geöffnet war.
- **Inventarisierung – Credential Guard**: `Win32_DeviceGuard` wurde im Namespace `root\cimv2` gesucht statt in `root\Microsoft\Windows\DeviceGuard`. Credential Guard galt dadurch immer als „nicht konfiguriert“. Ein von Windows 11 ohne Konfiguration aktivierter Credential Guard gilt jetzt als aktiv.
- **Inventarisierung – Firewall**: Gelesen wurden nur die lokalen Registry-Werte; per Gruppenrichtlinie gesetzte Profile wurden nicht berücksichtigt, ein fehlender Wert galt als „Deaktiviert“. Jetzt der wirksame Status über die Firewall-API, Fallback Richtlinie vor lokaler Einstellung.
- **System-Übersicht – „wird ermittelt …“ im Export und in der Anzeige**:
  - **Fehler:** CPU, Arbeitsspeicher und Grafik standen im Bericht auf „wird ermittelt …“, Bildschirm und DPI-Skalierung waren leer.
  - **Ursache:** Wurde die Übersicht während des Ladens kurz aus dem Fenster genommen, verwarf der Ladevorgang alle Werte, die in dieser Zeit eintrafen. Der Lauf galt trotzdem als vollständig, beim nächsten Anzeigen wurde nicht neu geladen. Seit „Übersicht“ und „Lokale Inventarisierung“ dieselbe Seite zeigen, trat das schon beim Wechsel zwischen beiden Einträgen auf.
  - **Hauptfenster:** Eine bereits angezeigte Seite wird nicht mehr entfernt und neu eingesetzt.
  - **Übersicht:** Ein unterbrochener Ladevorgang gilt nicht als geladen und wird beim Wiederanzeigen neu gestartet.
  - **Export und Kopieren:** Läuft das Laden noch, fragen beide nach; noch fehlende Werte erscheinen als „nicht ermittelt“. Betroffen waren auch Energiesparplan, NTP-Quelle und Netzwerkzone, die im selben Ladeablauf erst gegen Ende kommen.
  - **Antivirus und Firewall:** Noch nicht ermittelte Werte wurden als „Kein Antivirenprogramm gefunden / Nicht geschützt“ bzw. „Keine Firewall gefunden / Nicht aktiv“ exportiert – eine falsche Aussage statt eines fehlenden Werts. Jetzt „nicht ermittelt“.
- **PDF-Export – „You must not change font resolver after it was once used“**:
  - **Fehler:** Nach einem Inventar-PDF schlug in derselben Sitzung jeder weitere PDF-Export über `ExportService` fehl, z. B. Systemübersicht/Systembericht, E-Mail-Analysen und Diagnosen.
  - **Ursache:** PDFsharp erlaubt nur einen Schriften-Resolver pro Prozess. Der Inventar-Export setzte einen eigenen, `ExportService` prüfte nur ein eigenes Merkerfeld und wollte danach seinen setzen.
  - **Lösung:** Alle PDF-Exporte (Inventar, Systeminfo, Hilfe, MS-SQL, Netzwerk-Topologie, `ExportService`) nutzen jetzt `ExportService.EnsureFontResolver()` mit dem Windows-Resolver.
  - **Nebeneffekt:** Der entfernte Inventar-Resolver kannte nur „Arial normal“; Überschriften im Inventar-PDF sind jetzt echt fett.
- **Inventory Agent – RunOnce löschte das eigene Ergebnis**: Der Standard-Ablageort war `C:\DATEXTAgent\Data`, die Selbst-Deinstallation löschte `C:\DATEXTAgent` komplett. Neuer Standard `C:\ProgramData\DATEXT\Inventar`; die Deinstallation entfernt nur noch den Programmordner und alte Datenordner nur, wenn sie leer sind.
- **Inventory Agent – Wochentag um einen Tag verschoben**: Gespeichert wurde die Listenposition (Montag = 0), der Agent las sie als `DayOfWeek` (0 = Sonntag). „Montag“ ergab Sonntag.
- **Inventory Agent – Abruf lieferte alte Daten**: Der HTTP-Cache wurde nur beim Dienststart gefüllt, die geplanten Erfassungen schrieben nur die Datei. „Inventar abrufen“ zeigte den Stand vom Dienststart, auch nach Wochen.
- **Inventory Agent – Port offen bei „Einmalig“**: Der Agent startete den HTTP-Server auch im RunOnce-Modus, und das Skript öffnete URL-Reservierung und Firewall. Der Dialog meldete dagegen „kein HTTP-Dienst“.
- **Inventory Agent – Remote-Installation meldete immer Erfolg**: Rückgabewerte von `sc.exe` und `netsh` wurden nicht geprüft. Das Kennwort stand im Klartext auf der `netsh`-Kommandozeile und war damit in der Prozessliste sichtbar; `netsh -r` für den Kontext `http` funktioniert remote nicht. Ein vorhandener Dienst wird jetzt vorher entfernt.
- **Inventory Agent – GPO**: Das Startskript installierte bei jedem Neustart neu; bei „Einmalig“ folgten Erfassung und Deinstallation bei jedem Start. Der SYSVOL-Pfad in der Anleitung enthielt den eigenen Rechnernamen statt der Domäne.
- **Inventory Agent – .NET-Prüfung**: `dotnet --version` liefert die SDK-Version, der Registry-Fallback den Host. Mit .NET 9 oder 10 wurde .NET 8 bei jeder Installation erneut geladen.
- **Inventory Agent – Suche**:
  - Agents mit Key wurden nach einem Neustart nicht mehr gefunden, weil Port und Key nur aus dem ungespeicherten Formular kamen. Ein abgelehnter Key galt als „kein Agent“.
  - Jeder 172.x-Bereich galt als „virtuell“, das betraf auch Firmennetze. Agents werden jetzt über ihre ID zusammengeführt.
  - „Ausgewählte abrufen“ lief immer auf alle Agents.
- **Inventory Agent – Auswertung**: Gleiche Rechnernamen bei verschiedenen Kunden überschrieben sich. Die Datei „GPO-Einrichtung“ und die Versionsanzeige legten bei jedem Öffnen eine leere `.tmp`-Datei in %TEMP% ab.
- **Inventory Agent – CLI `--run-once`** schrieb unverschlüsselt und mit anderem Dateinamen als der Dienst.
- **Inventar-PDF – Spaltenbreiten**:
  - **Vertauschte Breiten:** In mehreren Tabellen war die Spalte „#“ flexibel und die Namensspalte nur 25 pt breit, Texte überlagerten sich. Betroffen waren Bildschirme, angewendete GPOs (dort zusätzlich „Bereich“ 300 pt) und Computer-Sicherheitsgruppen.
  - **Zeichen:** ⚠ und ✓ fehlen in der PDF-Schrift Arial. Sie sind durch „!“, „Ja“ bzw. Text ersetzt; die Spalte „⚠“ der Feature-Tabelle heißt jetzt „Hinweis“.
- **Inventarisierung – Kleinere Fehler**: Die Agent-Erfassung schrieb aus parallelen Bereichen ungeschützt in die Warnungsliste. Ein Fehler beim JSON-Export beendete die App, jetzt erscheint eine Meldung in der Statuszeile. Die letzte Anmeldung wurde über PowerShell mit kulturabhängigem Datum gelesen, jetzt über die Netzwerkverwaltungs-API. Die Administratoren-Gruppe wird sprachunabhängig über ihre SID erkannt, die Installationssprache über die LCID statt einer festen Liste. Die gpresult-Datei bekommt einen eindeutigen Namen je Prozess.
- **Netzwerk-Speedtest – Lesewert durch Client-Cache geschönt**: Die gerade geschriebene Datei wurde gepuffert gelesen und kam aus dem RAM des eigenen PCs. Testfreigabe: 1.528 statt real 941 Mbit/s. Geschrieben wurde mit WriteThrough, das jeden Block einzeln auf die Serverplatte zwingt: 1.000 statt 2.190 Mbit/s. Beide Richtungen laufen jetzt ohne Client-Cache (`FILE_FLAG_NO_BUFFERING`, 1-MB-Blöcke).
- **Netzwerk-Speedtest – Oberfläche fror ein**: Die Pfadprüfung (`Directory.Exists`) lief im UI-Thread, auch nach jeder Tipp-Pause. Bei nicht erreichbarem Server stand das Programm je Prüfung 13–47 s (gemessen). Jetzt läuft die Prüfung im Hintergrund mit 8 s Zeitlimit und prüft auch die Schreibrechte; die längste UI-Pause im Test lag bei 65 ms.
- **Netzwerk-Speedtest – Einheiten**: MiB/s × 8 wurde als „Mbit/s“ ausgegeben, rund 5 % zu niedrig. Jetzt wird dezimal gerechnet (MB/s = 10⁶ Byte/s).
- **Netzwerk-Speedtest – Testdatei blieb liegen**: Bei Abbruch oder Fehler blieb die bis zu 1 GB große Testdatei auf der Freigabe. Sie wird jetzt in jedem Fall gelöscht; der Abbruch greift innerhalb von rund 0,1 s.
- **Disk-Speedtest – Falsches Laufwerk gemessen**: Die Testdatei lag in `X:\Windows\Temp`; den Ordner gibt es nur auf dem Systemlaufwerk. Für D: fiel der Test still auf `%TEMP%` zurück und maß damit C:. Jetzt liegt die Datei auf dem gewählten Laufwerk: in `%TEMP%`, falls es dort liegt, sonst im temporären Ordner `X:\DATEXT-Disktest`, der danach wieder entfernt wird. Ist das nicht möglich, gibt es eine Fehlermeldung.
- **Disk-Speedtest – Abbruch**: DiskSpd lief nach „Stopp“ weiter, hielt die Testdatei offen, und sie blieb liegen. Jetzt wird der Prozess beendet und die Datei gelöscht. Als Administrator gestartetes WinSAT wird ebenfalls beendet; über UAC gestartetes kann nicht beendet werden, das Protokoll weist darauf hin.
- **Disk-Speedtest – „Random I/O“ ohne Wirkung**: Die Option wurde nur ins Protokoll geschrieben. Sie entfällt; zufällige Zugriffe werden eigens gemessen.
- **Disk-Speedtest – Laufwerkstyp**: Angezeigt wurde immer der Typ des ersten physischen Datenträgers. MediaType 5 (SCM) wurde als „NVMe“ gedeutet.
- **Disk-Speedtest – DiskSpd-Download**: Geladen wurde zuerst die nicht existierende Version 2.2.1, das ergab immer 404, und dann v2.1. Jetzt wird das aktuelle Release über die GitHub-API geladen, mit passender Architektur (amd64/ARM64). Übernommen wird es nur mit gültiger Microsoft-Signatur. Eine vorhandene diskspd.exe ohne gültige Signatur wird nicht ausgeführt.
- **Disk-Speedtest – WinSAT**:
  - **Random 16K:** Der zufällige Wert stammt aus einer 16-KB-Messung und wurde als „Random 4K“ angezeigt.
  - **Codepage:** Die OEM-Codepage war fest 850; jetzt gilt die des Systems.
  - **UAC abgelehnt:** Es erscheint ein Hinweis in der Statuszeile statt zweier Meldungen.
- **Disk-Speedtest – „Log löschen“**: Es löschte auch Messverlauf und Ergebnisse. Jetzt leert es nur das Protokoll.
- **MS-SQL – Verbindungs-String**: Er wurde per Verkettung gebaut. Jetzt über `SqlConnectionStringBuilder`.
  - **Semikolon im Kennwort:** Ein `;` im Kennwort brach die Anmeldung ab („Keyword not supported“).
  - **Fremde Identität:** Das Kennwort `x;Integrated Security=True` meldete trotz SQL-Anmeldung mit nicht existierendem Benutzer als laufender Windows-Benutzer an.
  - **master-Verbindung:** Auch `ToMasterConnStr` zerlegte den String an `;` und nutzt jetzt den Builder.
- **MS-SQL – Verschlüsselung „Streng“**: Die Option funktionierte nie; `System.Data.SqlClient` meldete „Invalid value for key 'encrypt'“. Sie ist entfernt; die Umstellung auf `Microsoft.Data.SqlClient` steht unter „Planned“.
- **MS-SQL – Benannte Instanz**: Bei `SERVER\INSTANZ` mit Standardport fragt der Dialog jetzt zuerst den SQL Browser. Vorher wurde 5 s lang Port 1433 versucht; lauschte dort die Standardinstanz, landete die Verbindung bei der falschen Instanz. Verbinden zur lokalen Testinstanz: 0,7 statt 6,9 s.
- **MS-SQL – „Windows (anderer Benutzer)“**: Das Anmelde-Token wurde bei jedem Aufruf in ein neues, besitzendes `SafeAccessTokenHandle` gepackt, das es beim Aufräumen schloss. Der Rückgabewert von `ImpersonateLoggedOnUser` wurde nicht geprüft, sodass Abfragen danach still unter dem eigenen Konto liefen. Jetzt gibt es ein einziges SafeHandle je Sitzung, und alle Abfragen laufen über `WindowsIdentity.RunImpersonated`.
- **MS-SQL – Fremd-Prozeduren (sp_WhoIsActive, sp_Blitz, sp_BlitzCache)**:
  - **Installation:** Vorher wurde das Skript ohne Rückfrage vom ungepinnten `main`-Zweig auf GitHub geladen und sofort in `master` ausgeführt. Jetzt wird das letzte **Release-Tag** geladen und der Inhalt geprüft (enthält die Prozedur). Es folgt eine Bestätigung mit Server, Datenbank, Objekt, Quelle, Release, Version, Lizenz (wie vom Repository gemeldet, z. B. GPL-3.0 bei sp_WhoIsActive) und Umfang. Alternativ lässt sich das Skript **nur speichern**.
  - **Updateprüfung:** Sie verglich die Version („8.34“) mit dem Release-Tag („20260708“), es hieß deshalb immer „Update verfügbar“. Jetzt vergleicht sie die Versionsnummer aus dem Skript.
- **MS-SQL – Live-Monitor und Auto-Refresh**:
  - **Beim Trennen:** Sie liefen weiter, weil `Disconnect` zuerst `_isConnected` zurücksetzte und der Reiterwechsel-Handler danach sofort abbrach.
  - **Beim Verlassen des Dialogs:** Sie fragten weiter alle 3 bzw. 10 s den Server ab. Jetzt pausieren sie bei `Unloaded` und laufen bei `Loaded` weiter.
- **MS-SQL – Reiter wurden ungewollt neu geladen**: `SelectionChanged` blubbert aus Tabellen und Auswahlfeldern hoch. Schon das Markieren einer Zeile oder eine andere Sortierung lud deshalb den ganzen Reiter neu und brach laufende Abfragen ab.
- **MS-SQL – Fehlalarme in der Diagnose**:
  - **Wartetypen:** Die harmlosen Leerlauf-Waits wurden nur unvollständig gefiltert, z. B. fehlten `SOS_WORK_DISPATCHER`, `QDS_*`, `PWAIT_*`, `XE_*`, `DIRTY_PAGE_POLL` und `PREEMPTIVE_XE_*`. Auf der ruhenden Testinstanz stand `SOS_WORK_DISPATCHER` an erster Stelle und ergab „1× Kritisch“. Jetzt gilt die Liste nach Paul Randal, ergänzt für SQL 2019/2022.
  - **I/O-Latenz:** Sie wurde schon bei wenigen Vorgängen bewertet; 36 ms auf der msdb-Datendatei galten als kritisch. Jetzt erst ab 100 Vorgängen, mit Schwellen je Dateityp: Lesen 20/50 ms, Log-Schreiben 5/20 ms, Daten-Schreiben 50/100 ms.
- **MS-SQL – Blocking-Ketten (Aktivität)**:
  - **Abgeschnittene Zeilen:** Die Zeilen waren unten abgeschnitten. Die feste Zeilenhöhe von 28 px reichte nicht für 12-pt-Text mit Rand und Zell-Padding; jetzt ist die Höhe automatisch (mindestens 28 px).
  - **Phantom-Blocker:** `blocking_session_id = 0` (nicht blockiert) wurde als „blockiert von SPID 0“ gewertet. Die Karte erschien deshalb auch ohne Blocking, mit dem Eintrag „SPID 0 (Blocker)“, und jede laufende Abfrage war rot hinterlegt.
  - **Sonderfälle:** Negative Kennungen werden benannt: −2 verwaiste DTC-Transaktion, −3 verzögerte Wiederherstellung, −4 Latch. Ein Blocker ohne laufende Abfrage wird als mögliche offene Transaktion gekennzeichnet.
  - **Wartezeit:** „wartet 0ss“ zeigte ein doppeltes s.
- **MS-SQL – Erhöhte DB-Berechtigungen**: Die Abfrage las `sys.database_principals` ohne Datenbankkontext und prüfte damit für jede Datenbank nur die Rollen aus `master`. Mitglieder von db_owner und anderen Rollen in Benutzerdatenbanken wurden so nie gefunden. Jetzt wird jede erreichbare Datenbank einzeln abgefragt.
- **MS-SQL – Zeilenhervorhebung**: Die Hervorhebung über `BgFarbe` (an 18 Stellen gesetzt) erschien nie. Die DataTrigger hingen am DataGrid statt an der Zeile und verglichen mit alten Hex-Werten, die die Farb-Tokens nie liefern. Jetzt bindet der Zeilenstil direkt.
- **MS-SQL – Darstellung**:
  - Button-Texte mit Unterstrich erschienen ohne ihn („spBlitzCache ausführen“), weil WPF `_` als Zugriffstaste wertet.
  - Der Verbindungstest zeigte leere Kästchen statt Status-Icons: Emojis in der Icon-Schrift.
  - „Verbinden“ und „Trennen“ waren unten abgeschnitten (lokales `Padding 16,8` bei 30 px Höhe).
  - Der Platzhalter zeigte „SERVER\\\\INSTANZ“ mit doppeltem Backslash.
- **MS-SQL – Export und Kopieren**:
  - **Export:** Sitzungsspeicher, Indexnutzung, Ring-Buffer und Buffer Pool wurden vor dem Export nur angestoßen und fehlten, wenn sie nicht rechtzeitig fertig waren. Jetzt werden sie abgewartet.
  - **„Kopieren“:** Es zeigt Rückmeldung und fängt Fehler ab.
  - **Erneutes Verbinden:** Der Zustand wird jetzt auch vor der Datenbankwahl zurückgesetzt; vorher blieben Server-Info und Datenbankliste stehen.
  - **Ladefehler:** Fehler beim Laden eines Reiters erscheinen im Status oben statt nur im Debug-Log.
- **Datumsfelder (Active Directory, Ereignislog-Viewer)**: Das gewählte Datum wurde abgeschnitten (AD-Suche, 33 px Platz für 52 px Text) bzw. gar nicht angezeigt (Ereignislog, 0 px).
  - **Ursache:** Der implizite Button-Stil in `App.xaml` (`MinWidth 120`, `Margin 5`, `Padding 15,8`) griff auf den Kalender-Button im Standard-Template des DatePickers. Er blähte ihn auf 120 px auf, ebenso die Monats-Buttons im aufgeklappten Kalender.
  - **Lösung:** Neuer zentraler Stil `Input.DatePicker` (Themes/Controls.xaml).
    - Eingabefeld im Look von `Input.TextBox`, 30 px hoch, mit Platzhalter „TT.MM.JJJJ“.
    - Kalender-Button 30 × 30 mit Fluent-Glyphe außerhalb des Feldes, in einer festen Grid-Spalte und mit explizitem Stil, nach dem Muster aus `Dokumentation/WPF-IconButtons-DynamischeZeilen.md`.
    - Der Kalender öffnet bündig unter dem Feld; `Input.Calendar` setzt den App-Button-Stil darin zurück.
  - **Ohne Code-Änderung:** `SelectedDate`, die Eingabe per Tastatur und die Filter-Ereignisse bleiben unverändert.
  - **Ereignislog:** Die Datumsfelder sind dort jetzt 150 statt 120 px breit. Die Uhrzeitfelder daneben („00:0…“) waren durch das doppelte Padding von `Input.TextBox` ebenfalls abgeschnitten.
- **Gruppenrichtlinien – Analyse brach ab**: `gpresult` braucht in Domänen oft 20–40 s (Testgerät: 31 s), der Dialog wartete aber fest 30 s.
  - **Zeitgrenze:** Danach las er den Exit-Code des noch laufenden Prozesses, und die Analyse schlug fehl. Jetzt gilt eine Zeitgrenze von 180 s, mit Laufzeitanzeige.
  - **Stopp:** „Stopp“ beendet `gpresult` jetzt; vorher lief es weiter.
  - **Pfadlänge:** `gpresult` lehnt Ausgabepfade über 127 Zeichen ab. Bei langen %TEMP%-Pfaden wird deshalb der 8.3-Kurzpfad genutzt.
  - **Temp-Dateien:** Die XML-Ausgabe (~1 MB, mit Gruppenmitgliedschaften) bleibt nicht mehr in %TEMP% liegen.
- **Gruppenrichtlinien – Status**:
  - **Falsch „angewendet“:** Nicht angewendete GPOs galten als angewendet, z. B. „Richtlinien der lokalen Gruppe“ im Benutzerbereich. Jetzt entscheidet `AppliedOrder`.
  - **Deaktivierte Verknüpfung:** Sie hieß „Deaktiviert“, als wäre das GPO selbst aus.
  - **Neue Statuswerte:** „Verknüpfung deaktiviert“, „GPO deaktiviert“, „Sicherheitsfilter“, „WMI-Filter“, „Kein Lesezugriff“, „Nicht angewendet (keine Einstellungen für diesen Bereich)“ und „Verwaiste Verknüpfung“, jeweils mit Begründung.
- **Gruppenrichtlinien – Namen und verwaiste Verknüpfungen**: Nicht angewendete GPOs erschienen nur als `{GUID}`. Ihre Namen werden jetzt per LDAP nachgeladen; das braucht kein RSAT. GPOs, die im AD nicht mehr existieren, werden als „Verwaiste Verknüpfung“ (gelöschtes GPO) markiert. Testgerät: 4 Namen aufgelöst, 2 verwaiste Verknüpfungen gefunden.
- **Gruppenrichtlinien – Werte**:
  - **GUID und Standort:** Sie waren immer leer, weil falsche XML-Elemente gelesen wurden.
  - **Letzte Aktualisierung:** Sie zeigte immer „–“. Jetzt erscheint der Zeitpunkt der letzten Verarbeitung je Computer und Benutzer, aus der Gruppenrichtlinieninfrastruktur; bei älter als 24 h in Warnfarbe.
  - **Reihenfolge:** Sie zeigte die Position innerhalb der OU (`LinkOrder`) statt der Verarbeitungsreihenfolge (`AppliedOrder`).
  - **Mehrere Verknüpfungen:** Bei GPOs mit mehreren Verknüpfungen erschien nur die erste.
- **Gruppenrichtlinien – Sicherheitsrichtlinien**: Der Reiter sammelte beliebige XML-Blattknoten und zeigte auf dem Testgerät zwei sinnlose Zeilen („Blocked = true“). Jetzt werden die gewinnenden Sicherheitseinstellungen ausgewertet, mit Wert und Quell-GPO:
  - Kennwort- und Kontosperrrichtlinie, deutsch benannt, Werte mit Einheit („30 Minuten“, „Läuft nie ab“)
  - Kerberos und Überwachungsrichtlinie
  - Sicherheitsoptionen
  - Systemdienste
  - Benutzerrechte
  - Eingeschränkte Gruppen
  - Ereignisprotokoll
- **Gruppenrichtlinien – Kleinere Fehler**:
  - **Einlesefehler:** Sie werden in der Statuszeile gemeldet statt verschluckt.
  - **„Als Administrator neu starten“:** Der Button gibt die Einzelinstanz-Sperre frei, und ein Abbruch der UAC-Abfrage ist kein Fehler mehr.
  - **Kachel „Verweigert“:** Sie zählte Deaktiviertes mit.
  - **Absturz beim Öffnen:** Der Reiterwechsel während `InitializeComponent` ließ den Dialog abstürzen.
- **Active Directory – Benutzersuche**: Gesucht wurde nur im CN (`Name`). Bei den meisten Konten weicht der CN vom Anmeldenamen ab („Mustermann, Max“ statt „mustermann-ma“), daher fand die Suche nach dem Anmeldenamen nichts. Benutzer werden jetzt über Name, Anmeldename, Anzeigename, E-Mail und UPN gesucht, Computer über Name und DNS-Name.
- **Active Directory – Abfragen**:
  - **Apostroph:** Ein Apostroph im Suchbegriff („O'Brien“) brach den AD-Filter mit einem Syntaxfehler ab.
  - **Fehler sichtbar:** Fehler wurden verschluckt (stderr wurde nie gelesen), ein nicht erreichbarer DC oder falsche Anmeldedaten sahen deshalb aus wie „keine Treffer“. Jetzt erscheint die Fehlermeldung in der Statuszeile, mit Klartext für „DC nicht erreichbar (ADWS, TCP 9389)“ und „Anmeldung abgelehnt“.
  - **Stopp:** „Stopp“ hatte keine Wirkung. Jetzt wird der PowerShell-Prozess beendet.
- **Active Directory – Anmeldedaten**:
  - **Übergabe:** Benutzer, Passwort, DC und Suchbegriff gehen jetzt als Umgebungsvariablen an den Kindprozess. Die Passwort-Variable wird direkt nach dem Einlesen entfernt.
- **Active Directory – Filter**:
  - **Gesperrte Konten:** Sie waren standardmäßig ausgeblendet, weil das leere Häkchen „Gesperrt“ sie herausfilterte. Jetzt gibt es einen optionalen Filter „Nur gesperrte Konten“.
  - **Schnellfilter:** Die Schaltflächen „30T/90T/180T“ waren missverständlich, „Nie“ war ohne Funktion. Ersetzt durch eine Auswahl „Inaktiv seit mehr als 30/90/180/365 Tagen“, „Nie angemeldet“ und „Angemeldet in den letzten 30 Tagen“.
  - **Dropdowns:** Betriebssystem und OU springen nach einer Suche nicht mehr auf „Alle“ zurück.
  - **OU-Vergleich:** Der OU-Filter vergleicht exakt statt per Teilstring („Server“ traf auch „Server › Alt“).
- **Active Directory – Netzwerk-Abgleich**:
  - **Absturz:** Ein Hostname, der im Geräte-Scan mehrfach vorkam (LAN + WLAN), brachte die Anwendung zum Absturz. Jetzt werden beide IPs angezeigt.
  - **Vergleichsbasis:** Abgeglichen wurde gegen das letzte Suchergebnis. Nach einer Suche „srv“ erschienen deshalb alle übrigen Rechner als „Nur im Scan“. Jetzt werden für den Abgleich immer alle Computer geladen.
  - **Ohne Geräte-Scan:** Die Schaltfläche ist dann gesperrt, mit Hinweis.
- **Active Directory – Details**:
  - Die primäre Gruppe (Domänen-Benutzer/-Computer) fehlte in den Mitgliedschaften.
  - Beim schnellen Wechsel zwischen Zeilen konnte eine ältere Gruppenabfrage die Details überschreiben.
  - Gibt man eine IP-Adresse als DC ein, wird sie nicht mehr als fremde Domäne gewertet.
  - Die Navigation „In AD suchen“ aus anderen Dialogen startet die Suche jetzt auch.
  - Ping, Traceroute und Port-Scan nutzen den DNS-Namen.
- **Troubleshooting – Neustart**: „Vollständiger Neustart“ rief `shutdown /q` auf. Diesen Schalter gibt es nicht, es passierte also nichts, und die Karte „Neustart abbrechen“ erschien trotzdem. Jetzt gibt es „In 15 Sekunden“ (`/r /t 15`; Windows schließt dabei alle Programme ohne Rückfrage, was in der Bestätigung steht) und „Jetzt neu starten“ (`/r /t 0`). Der Exit-Code von `shutdown` wird geprüft; die Abbrechen-Karte erscheint nur bei einem tatsächlich geplanten Neustart.
- **Troubleshooting – Update-Richtlinien**: Der Schlüssel `HKLM\…\Policies\Microsoft\Windows\WindowsUpdate` wurde ohne Vorschau und ohne Sicherung gelöscht. Jetzt gilt:
  - Die Bestätigung listet die betroffenen Werte auf, darunter den WSUS-Server.
  - Bei Domänenmitgliedern weist sie darauf hin, dass gpupdate die Werte der Domänen-GPOs zurückbringt.
  - Vorher wird eine `.reg`-Sicherung unter `%ProgramData%\DATEXT-Diagnostics\Backups` angelegt; schlägt sie fehl, wird nichts gelöscht.
- **Troubleshooting – TCP/IP-Reset**: Die Bestätigung zeigt jetzt alle Adapter mit statischer IPv4-Adresse, die dabei verloren geht, und die erkannte Hyper-V-Rolle.
- **Troubleshooting – Komponenten neu registrieren**: 20 DLLs wurden blind per `regsvr32` registriert. Auf aktuellem Windows fehlt die Hälfte davon, fünf haben keine `DllRegisterServer`-Funktion, und die Liste enthielt `shell32`, `ole32` und `oleaut32`, die mit Windows Update nichts zu tun haben. Jetzt sind es 13 Update- und Krypto-DLLs; fehlende DLLs und DLLs ohne Registrierungsfunktion werden übersprungen und im Ergebnis getrennt gezählt. Das Ergebnis des Dienst-Stopps fließt in die Gesamtbewertung ein.
- **Troubleshooting – Administratorrechte**: Ohne Administratorrechte schaltete ein beendeter Schritt die gesperrten Schritte wieder frei. Werkzeuge, die Administratorrechte brauchen, sind jetzt ohne diese Rechte gesperrt und markiert. „Als Administrator neu starten“ gibt die Einzelinstanz-Sperre frei; sonst beendete sich die neue Instanz sofort.
- **Troubleshooting – Werkzeuge**: Die Ausgabe der Werkzeuge wurde verworfen, es erschien nur „fehlgeschlagen – siehe Protokoll“. Jetzt landet sie im Protokoll, und das Ergebnis steht in der Karte. Die Temp-Bereinigung zeigt statt bis zu vier MessageBoxen eine einzige Bestätigung mit Anzahl und Größe; `%TEMP%\.net` bleibt weiterhin ausgenommen. Die Empfehlung zum Neustart erscheint nur noch nach TCP/IP- oder Winsock-Reset.
- **System-Verwaltung – Mail (Outlook-Profile)**: `control mlcfg32.cpl` fand die Datei bei Office Click-to-Run nicht; sie liegt im Office-Ordner, nicht in System32. Der Ersatzweg wurde nie erreicht, weil `control.exe` immer erfolgreich startet. Er hätte auch nicht funktioniert: `rundll32 mapi32.dll,ShowNewAcctWiz` ruft eine Funktion auf, die es in `mapi32.dll` nicht gibt. Jetzt wird die CPL im Office-Ordner direkt geöffnet (Office 15/16, 32 und 64 Bit), sonst `outlook.exe /profiles`. Ohne klassisches Outlook ist die Kachel ausgegraut.
- **System-Verwaltung – UAC-Abbruch**: Wurde die Abfrage nach Administratorrechten abgebrochen (z. B. bei `msconfig` oder `rstrui`), erschien „Konnte nicht starten“. Jetzt meldet die Statuszeile „Abfrage nach Administratorrechten abgebrochen“.
- **System-Verwaltung – Beschriftung**: „Windows Wiederherstellung“ öffnete `rstrui.exe`, also die Systemwiederherstellung (Wiederherstellungspunkte). Beide sind jetzt getrennt: „Systemwiederherstellung“ und „Windows-Wiederherstellung“ (`ms-settings:recovery`, Zurücksetzen und erweiterter Start).
- **Winget-Pakete – Tabellen falsch gelesen**: Der Parser hatte je Aufruf eine feste Spaltenliste.
  - In „Alle Pakete“ fehlte die Spalte „Verfügbar“. Die Version lautete deshalb z. B. „2026.8.11723.0 2026.09.12311“, und kein Paket zeigte ein Update (auf dem Testsystem 21).
  - In der Suche landete die Übereinstimmung in der Version („115.0.5790.110 Tag: browser“).
  - Zeilen mit doppelt breiten Zeichen (z. B. „115浏览器“) verrutschten, weil nach Zeichenposition statt nach Anzeigebreite geschnitten wurde.
  
  Jetzt gibt es einen Parser für alle Tabellen (`upgrade`, `list`, `search`, `pin list`). Die Spalten kommen aus der Kopfzeile (deutsch/englisch), geschnitten wird nach Anzeigebreite, mehrere Tabellen und Fortschrittsreste werden erkannt.
- **Winget-Pakete – Details auf deutschem Windows leer**: Der `winget show`-Parser kannte nur englische Feldnamen; Herausgeber, Beschreibung, Startseite, Lizenz und Markierungen blieben leer. Jetzt werden beide Sprachen ausgewertet, auch mehrzeilige Beschreibungen. Die Details werden nur übernommen, wenn das Paket noch gewählt ist.
- **Winget-Pakete – Pins**: Die Pin-Liste suchte die Spalte „Id“, winget schreibt auf Deutsch „ID“. Der Ersatzweg nahm das erste Wort mit Punkt; bei Namen wie „Node.js“ landete so der Name statt der ID im Ergebnis. Die Pin-Art (Pinning/Blocking/Gating) wird jetzt angezeigt. „Pin setzen“ in „Alle Pakete“ suchte per Freitext statt per ID.
- **Winget-Pakete – Stopp beendete winget nicht**: Abgebrochen wurde nur das Warten; winget und der Installer liefen weiter, und die Liste wurde während der Installation neu geladen. Jetzt wird der Prozessbaum beendet.
- **Winget-Pakete – Erfolgsmeldungen**: „Deinstallation abgeschlossen“ erschien auch bei Fehlschlag, im Updates-Kontextmenü wurde die Ausgabe doppelt protokolliert. Die Auswahl „Update prüfen“ riet die Version aus der Textausgabe; jetzt kommt sie aus `winget list`. „Als Administrator neu starten“ gibt die Einzelinstanz-Sperre frei. Beim Laden der XAML feuerte der neue Filter vor dem Anlegen der übrigen Elemente; abgesichert.
- **Installierte Software – Systemkomponenten als Programme**: Einträge mit `SystemComponent = 1` und Teilprodukte bzw. Updates (`ParentKeyName`, `ReleaseType`) erschienen wie Programme. Auf dem Testsystem waren das 299 von 416 Einträgen (Visual-C++-Zusatzlaufzeiten, .NET-Workload-Manifeste, SQL-Server-Bestandteile). Jetzt erscheinen 117 Programme wie in „Programme und Features“; der Rest ist zuschaltbar.
- **Installierte Software – Benutzerinstallationen fehlten**: Der Uninstall-Schlüssel des aktuellen Benutzers (HKCU) wurde nicht gelesen. Auf dem Testsystem fehlten so 12 Programme, u. a. Zoom Workplace, Claude, Microsoft Loop, Passbolt und CapCut.
- **Installierte Software – Datum, Version, Größe**: Ohne `InstallDate` blieb das Datum leer (40 Einträge). Jetzt wird wie in „Programme und Features“ der Zeitpunkt der letzten Änderung des Registry-Eintrags verwendet, grau-kursiv mit Tooltip. Versionen wurden als Text sortiert („0.101“ vor „0.83“), Größen ab 1 GB als MB angezeigt, und die Suche durchsuchte auch den Größentext.
- **Installierte Software – Laden blockierte die Oberfläche**: Gelesen wurde synchron im UI-Thread, die Fortschrittsanzeige erschien deshalb nie. Jetzt läuft das Lesen im Hintergrund (0,7 s). Neu gefunden werden verwaiste Einträge, deren Deinstallationsprogramm fehlt. Gleichnamige Einträge in 64 und 32 Bit werden nicht mehr zusammengelegt.
- **Ereignislog-Viewer – Schnellfilter fanden kaum Treffer**: Geladen wurden die neuesten N Einträge (Standard 200), erst danach wurde im Speicher nach IDs gefiltert. „Rechner-Neustarts“ fand so 2 von 201 Treffern, „Bluescreen“ 0 von 2. Jetzt filtert Windows bereits beim Lesen.
- **Ereignislog-Viewer – Schnellfilter ohne Anbieter**: Event-IDs sind nur je Anbieter eindeutig. Alle Schnellfilter sind jetzt an Protokoll und Anbieter gebunden und gegen die vorhandenen Anbieter geprüft. Gefundene Fehler:
  - „Festplatten-Warnungen“ lieferte 3.700 Treffer von Hyper-V-VmSwitch, FilterManager, Kernel-Processor-Power und Kernel-Boot.
  - „Netzwerkprobleme“ lieferte 436 FilterManager-Ereignisse.
  - „UEFI CA 2023“ enthielt Kernel-Boot- und Wininit-Ereignisse.
  - „Update-Fehler“ suchte im Anwendungsprotokoll (39 × ScreenConnect); Windows Update schreibt ins System-Protokoll.
  - „Treiberfehler“ enthielt 7036 (Dienststatus geändert).
  - „RDP-Verbindungen“ suchte 4624/4625 im RemoteConnectionManager-Protokoll.
  - „Abstürze/Hänger“ suchte 1002 im System-Protokoll.
  - „DNS-Fehler“ enthielt 4015 (nur DNS-Server).
  - „Druckprobleme“ enthielt 307 (Dokument gedruckt).
  
  Neu sind „Überwachungsprotokoll gelöscht“ (1102), „Defender-Erkennungen“, „RDP-Sitzungen“ und „Software-Installationen (MSI)“.
- **Ereignislog-Viewer – Zeitraum**: Der Modus lud die neuesten 50.000 Einträge und schnitt danach im Speicher zu. Ältere Zeiträume blieben dadurch leer, kurze luden trotzdem bis zu 50.000 Einträge.
- **Ereignislog-Viewer – Überwachungsereignisse**: Im Sicherheitsprotokoll haben alle Einträge Level 0, deshalb erschien jede fehlgeschlagene Anmeldung (4625) als „Information“. Der Schweregrad kommt jetzt wie in der Ereignisanzeige aus den Schlüsselwörtern: „Überwachung gescheitert“ hat eine eigene Kachel und einen eigenen Filter, „Überwachung erfolgreich“ zählt zu den Informationen, Level 5 erscheint als „Ausführlich“.
- **Ereignislog-Viewer – Kontext ±5 Min. und Sprung aus System-Health**: Der Kontext las höchstens 20.000 Einträge rückwärts; bei älteren Ereignissen blieb er leer. Jetzt wird das Zeitfenster abgefragt, inklusive Setup-Protokoll, mit Spalte „Protokoll“ und Hinweis, wenn das Sicherheitsprotokoll ohne Admin fehlt. `NavigateTo` setzte nur den Suchtext auf die letzten 200 Einträge und fiel bei unbekannten Protokollen stillschweigend auf „System“ zurück. Jetzt lädt es gezielt die Einträge der Quelle aus dem angegebenen Protokoll.
- **Ereignislog-Viewer – Suche und Exporte unvollständig**: Die Suche durchsuchte nur die ersten 120 Zeichen der Nachricht, Kopieren und TXT enthielten nur diese Kurzfassung, und das PDF brach stillschweigend nach 200 Einträgen ab. Jetzt wird überall der volle Text verwendet. Die Liste wird in einem Schritt ersetzt statt Eintrag für Eintrag, und die Suche startet 200 ms nach der letzten Eingabe.
- **Ereignislog-Viewer – fehlende Beschreibungen**: Bei fehlenden Meldungstexten des Anbieters blieb die Nachricht leer. Jetzt werden wie in der Ereignisanzeige die Ereignisdaten gezeigt.
- **Ereignislog-Viewer – Administratorrechte**: Der Hinweis erscheint jetzt, sobald das Sicherheitsprotokoll gewählt ist, statt erst nach einem Fehler; die Erkennung prüft nicht mehr englische Fehlertexte. Beim Neustart als Administrator wird die Einzelinstanz-Sperre freigegeben. Außerdem entfernt: toter Code (`queryStr` mit falschem Zeitwert, `MapEntryType`, verstecktes `txtStatus`) und fest weiße Filter-Chips, die den zentralen Stil überschrieben.
- **Autorun-Analyse – Signaturprüfung**: `X509Certificate2(Datei).Verify()` erkannte keine Katalogsignaturen. Fast alle Windows-Dateien galten deshalb als „nicht signiert“, darunter alle svchost-Dienste: 73 von 89 Diensten waren als Warnung markiert. Außerdem wurde nur geprüft, ob das Zertifikat heute gültig ist, nicht ob die Datei verändert wurde. Jetzt kommt `WinVerifyTrust` zum Einsatz, zuerst für die eingebettete Signatur, sonst über den Systemkatalog (SHA256/SHA1), ohne Sperrlistenabruf aus dem Netz. Die Ergebnisse werden je Datei zwischengespeichert und parallel geprüft. Unterschieden werden: abgelaufenes Zertifikat ohne Zeitstempel (Warnung, keine Manipulation), nicht vertrauenswürdige Stelle (Warnung), ungültig bzw. verändert (Gefahr) und Store-Apps (Paketsignatur). Auf dem Testsystem sind es 8 statt 205 Auffälligkeiten.
- **Autorun-Analyse – Pfadauflösung**: Pfade ohne Anführungszeichen wurden beim ersten Leerzeichen abgeschnitten („C:\Program“ → „Datei fehlt“). Jetzt werden wie bei CreateProcess schrittweise längere Präfixe probiert. Weitere Korrekturen:
  - Namen ohne Pfad (`explorer.exe`) werden über System32, Windows und PATH gesucht.
  - `%windir%` in Tasks wird expandiert.
  - Das Komma am Ende von `Userinit` wird ignoriert.
  - Bei `rundll32` wird die aufgerufene DLL geprüft, auch nach Schaltern wie `/d`.
  - Einträge, die nur aus Schaltern bestehen (Active Setup `/UserInstall`), gelten nicht mehr als fehlende Datei.
- **Autorun-Analyse – Winlogon-Werte und Kennwort**: Gelistet wurden alle Werte des Winlogon-Schlüssels, z. B. `CachedLogonsCount` und `ShutdownFlags` – 30 Fehlalarme. Bei eingerichteter automatischer Anmeldung wäre außerdem `DefaultPassword` im Klartext in Tabelle, Export und Zwischenablage gelandet.
- **Autorun-Analyse – Risikobewertung**: Die Prüfungen „unsigniert“ und „Datei fehlt“ endeten vor der Ortsprüfung. Eine unsignierte EXE in `%TEMP%` wurde deshalb nie als Gefahr erkannt. Jetzt gilt: unsigniert an einem verdächtigen Ort (Temp, Users\Public, Downloads, Desktop, Papierkorb, direkt in ProgramData) ist Gefahr, signiert an solchen Orten ist eine Warnung. Pauschal `\ProgramData\` gilt nicht mehr als Gefahr, denn Defender liegt dort. Neu als Gefahr erkannt werden verschleierte bzw. aus dem Internet ladende Befehle (PowerShell `-enc`, `FromBase64String`, `DownloadString`, `mshta http:`, `regsvr32 /i:http`, `certutil -urlcache`, `bitsadmin /transfer`) und doppelte Dateiendungen. Ein Microsoft-signierter Interpreter mit verdächtigem Befehl bleibt auch beim Filter „Microsoft ausblenden“ sichtbar.
- **Autorun-Analyse – `desktop.ini`**: Die Datei erschien in beiden Autostart-Ordnern als unsignierter Autostart-Eintrag. Versteckte Systemdateien werden jetzt übersprungen.
- **UEFI Boot-Zertifikat – Zertifikate nie erkannt**: In der `EFI_CERT_X509_GUID` waren zwei Bytes vertauscht (`b5 87` statt `87 b5`). Deshalb wurde kein einziges Firmware-Zertifikat erkannt, und der Dialog zeigte stattdessen passende Zertifikate aus dem Windows-Zertifikatsspeicher als „UEFI-Zertifikate“ an. Diese Ersatzquelle entfällt, ebenso der Registry-Fallback, der für „db“ den Schlüssel `SecureBoot\PK` las.
- **UEFI Boot-Zertifikat – Servicing-Werte falsch gedeutet** (gegengeprüft an Microsoft KB5068202/KB5016061):
  - `UEFICA2023Error = 0` galt als Fehler; 0 bedeutet Erfolg.
  - Fehlercodes waren negativ (`Convert.ToInt64` auf REG_DWORD); `0x80070013` wird jetzt als „Firmware verweigert das Schreiben“ erklärt.
  - `WindowsUEFICA2023Capable = 0` erzeugte die kritische Empfehlung „OEM-Firmware-Update erforderlich“; der Wert bedeutet nur „Windows UEFI CA 2023 noch nicht in der db“.
  - `Capable = 2` wurde als „Windows Server 2025“ beschrieben.
  - AvailableUpdates-Bits hatten erfundene Bedeutungen („0x4000 = SVN“, „0x1000 = DBX“). Angezeigt wird jetzt nur noch der Rohwert; 0x5944 ist als „alle Zertifikate und Boot-Manager“ benannt.
  - Ereignis 1801 bedeutet „aktualisiert, aber noch nicht in der Firmware angewendet“ (Quelle TPM-WMI, nicht Kernel-Boot), 1795 ist ein Firmware-Schreibfehler (nicht nur Hyper-V). Neu ausgewertet werden 1032, 1033, 1034, 1036, 1037, 1042–1045, 1796–1799, 1802 und 1808. Fehlerereignisse, die vor dem letzten Erfolg (1808) liegen, werden nicht mehr gemeldet.
- **UEFI Boot-Zertifikat – TPM-Hinweis**: Die Aussage „TPM 2.0 ist Voraussetzung für Secure Boot“ war falsch; ein TPM ist nur für BitLocker und Measured Boot nötig. Außerdem galt der Dienstschlüssel `Services\TPM` als Nachweis für ein TPM, den es aber auf fast jedem Windows gibt.
- **UEFI Boot-Zertifikat – Als Administrator neu starten**: Die Einzelinstanz-Sperre wird vorher freigegeben. Vorher beendete sich die neue Instanz sofort, und es blieb kein Fenster offen. Wird die UAC-Abfrage abgebrochen, läuft die App normal weiter.
- **Verbindungen / Netzwerk-Analyse – DHCP-Lease-Details**: Die Prüfung lieferte nie Daten und meldete immer "Keine DHCP-konfigurierten Adapter gefunden". Ursache: Das PowerShell-Skript endete mit `Join-String`, das es nur in PowerShell 7 gibt, aufgerufen wurde aber Windows PowerShell 5.1. Jetzt nativ ohne Prozessstart: Adapter und IPv4-Adresse aus `NetworkInterface`, DHCP-Server und Lease-Zeiten per WMI (`Win32_NetworkAdapterConfiguration`, Zuordnung über den Schnittstellenindex); ohne WMI Fallback auf DHCP-Status und -Server aus `NetworkInterface`. Neu erkannt: DHCP aktiv, aber kein Lease erhalten (APIPA 169.254.x.x / Server 255.255.255.255) – wird als "Kein DHCP-Lease" gemeldet statt als abgelaufener Lease. Bereits abgelaufene Leases werden als solche gekennzeichnet.
- **Verbindungen / Netzwerk-Analyse – IPv6-Hinweis**: "IPv6-Verbindungen aktiv" erschien praktisch auf jedem Rechner, weil jede Remote-Adresse mit "[" zählte – auch lauschende Ports (`[::]:0`). Jetzt zählen nur bestehende Verbindungen (ESTABLISHED) zu echten IPv6-Gegenstellen, ohne Loopback (`::1`) und Link-Local (`fe80::/10`). Der Text "x von y" bezieht sich damit tatsächlich auf die bestehenden Verbindungen.
- **Verbindungen / Netzwerk-Analyse – ARP-Cache**: Neu aufgebaut auf Basis der Windows-Nachbartabelle (`GetIpNetTable2` über `Services\NeighborTable.cs`, neue Methode `ReadAllIPv4`) statt `arp -a`. Vorher meldete die Prüfung falsche "IP-Konflikte" (Gruppierung nur nach IP über alle Schnittstellen – dieselbe IP an LAN und VPN galt als Konflikt) und fast immer "Veraltete ARP-Einträge" (Broadcast-MAC `ff-ff-…` gibt es normal je Schnittstelle). Echte IP-Konflikte sind im ARP-Cache nicht erkennbar (Windows hält je Schnittstelle nur einen Eintrag pro IP), die Prüfung entfällt daher. Neu: **ARP-Spoofing-Muster** (Gateway-MAC wird auf derselben Schnittstelle von einer weiteren IP genutzt, als Warnung mit harmlosen Ursachen), **Gateway nicht erreichbar** (ARP-Eintrag des Standard-Gateways unaufgelöst), **nicht erreichbare Nachbarn** als Info, **manuell gesetzte statische Einträge** als Info (ohne die von Hyper-V selbst angelegten). Multicast, Broadcast (auch Subnetz-Broadcast über die Maske, z. B. `169.254.255.255`) und eigene Adressen werden nicht als Nachbarn gezählt.
- **Verbindungen / Netzwerk-Analyse – DNS-Latenz**: Gemessen wurde bisher der Windows-DNS-Cache statt des DNS-Servers (`Dns.GetHostAddressesAsync` auf bekannte Namen, die fast immer schon lokal gecacht sind), fehlgeschlagene Abfragen gingen als Messwert in den Durchschnitt ein. Jetzt direkte Abfrage beim DNS-Server über `DnsClient` (ohne Cache, ohne Wiederholung, 2 s Timeout) mit einer Aufwärm-Abfrage, die nicht mitzählt. Zusätzlich drei zufällige, nicht gecachte Namen für die vollständige rekursive Auflösung (als Zusatzwert im Detail). Timeouts zählen nicht in den Durchschnitt, sondern werden genannt; antwortet der Server gar nicht, erscheint eine eigene Warnung. Der Resolver wird vom Adapter mit Standard-Gateway genommen statt vom ersten aktiven Adapter (konnte ein Hyper-V-/VPN-Adapter sein).
- **Verbindungen – IPv6-Adressen bei Land/Reputation**: `Split(':')[0]` machte aus `[2a00::1]:443` die Zeichenkette `[2a00`, die an ip-api.com und Shodan ging. Jetzt korrekte Zerlegung (IPv4, IPv6 mit Scope-ID, IPv4-mapped IPv6).
- **Verbindungen – Filter für private Adressen**: `172.` war pauschal ausgeschlossen (auch öffentliche Bereiche wie 172.217.x von Google), CGNAT (100.64/10), Link-Local (169.254/16, `fe80::/10`), Multicast, ULA (`fc00::/7`) gingen dagegen an externe Dienste. Jetzt werden nur öffentlich geroutete Adressen abgefragt; die Recherche-Buttons im Detailbereich nutzen dieselbe Prüfung.
- **Verbindungen – TIME_WAIT**: Das deutsche netstat schreibt TIME_WAIT als "WARTEND", die Übersetzung kannte nur "WARTEN" – TIME_WAIT-Verbindungen erschienen als "WARTEND" und die TIME_WAIT-Auswertung der Analyse zählte immer 0. Mit der Umstellung auf die Windows-API (siehe Changed) behoben.
- **Verbindungen – Risiko-Bewertung**: Die Shodan-Markierung "⛔" (schädliche Tags: Malware/Scanner/C2) floss nicht in den Risiko-Score ein, nur "⚠" (CVEs). Jetzt beide +3, Tooltip unterscheidet die Ursache.
- **Verbindungen – Handle-Leck**: `Process.GetProcessById` wurde je PID und Aktualisierung aufgerufen und nie freigegeben (mit Auto-Refresh alle 5 s stetig wachsend).
- **Netzwerk-Analyse – TCP-Einstellungen**: Auf deutschem Windows schlugen die Prüfungen für Auto-Tuning und Timestamps nie an (Suchbegriffe passten nicht zur netsh-Ausgabe, z. B. "RFC 1323-Zeitstempel" mit Bindestrich), auf anderen Sprachen blieb die Auswertung leer. Jetzt per WMI (siehe Changed). Neu bewertet: Congestion Provider DCTCP, NewReno, LEDBAT (vorher nur BBR/CTCP/CUBIC); Hinweis bei CTCP empfiehlt jetzt `template=internet congestionprovider=cubic`.
- **Netzwerk-Analyse – Port-Auslastung**: Gezählt wurden pauschal alle lokalen Ports ab 49152 inkl. lauschender Ports und UDP. Jetzt nur ausgehende TCP-Verbindungen, jeder Port einmal, innerhalb des tatsächlichen dynamischen Bereichs (Start und Anzahl); die Lösungsempfehlung nutzt den echten Startport.
- **Netzwerk-Analyse – MTU**: Ohne Erreichbarkeitstest dauerte die Messung bei gesperrtem ICMP ca. 20 s und lieferte 0; jeder Timeout galt als "Paket zu groß" (Paketverlust → zu kleine MTU). Jetzt zuerst Erreichbarkeit mit kleinem Paket (8.8.8.8, sonst 1.1.1.1), Timeouts werden einmal wiederholt, eigener Hinweis wenn kein Ziel per ICMP erreichbar ist.
- **Netzwerk-Analyse – Bufferbloat**: Ein einzelner 5-MB-Download war ab ~100 Mbit/s nach unter 1 s fertig, die meisten Pings liefen ohne Last (Ergebnis zu gut); Upload wurde nicht getestet. Jetzt je 5 s Download-Last (4 Streams, max. 250 MB) und Upload-Last (2 Streams, max. 50 MB) gegen speed.cloudflare.com, Latenz und Durchsatz je Richtung im Detail, Bewertung nach der schlechteren Richtung.
- **Netzwerk-Analyse – NetBIOS**: Geprüft wurden alle Einträge unter `NetBT\Parameters\Interfaces`, auch von alten/entfernten Adaptern – NetBIOS galt dadurch oft als aktiv. Jetzt nur aktive Adapter, betroffene Adapter werden genannt.
- **Netzwerk-Analyse – Phasen**: Statustexte widersprachen sich ("Phase 3/3" neben "Phase 3/5"). Jetzt zwei Phasen: Konfiguration (TCP, Verbindungen, DHCP, ARP, NetBIOS/LLMNR parallel), danach Messungen allein (MTU, DNS, Bufferbloat).
- **Netzwerklast**: Die Rate wurde mit fest angenommenen 2 s berechnet; der Timer läuft nicht exakt. Jetzt mit der tatsächlich vergangenen Zeit.
- **Traceroute – Latenz der Zwischen-Hops**: Bei der Antwort "TTL abgelaufen" liefert .NET immer `RoundtripTime = 0` (gemessen: tatsächlich 1–59 ms) – alle Zwischen-Hops zeigten 0 ms, Latenzbalken, Farben und "Max. Latenz" waren falsch. Jetzt eigene Zeitmessung je Versuch; beim Ziel der Windows-Wert.
- **Traceroute – Historie "Ziel erreicht"**: Galt als erreicht, sobald irgendein Hop geantwortet hatte (eine nach Hop 3 abbrechende Route stand mit ✅ in der Historie). Jetzt das tatsächliche Ergebnis des Laufs.
- **Traceroute – IPv6**: Es wurde nur eine IPv4-Adresse gesucht; bei IPv6-Zielen gab es keine Ziel-IP, "Ziel erreicht" wurde nie erkannt und der Lauf ging bis zur maximalen Hop-Zahl. Jetzt bevorzugt IPv4, sonst IPv6. Nicht auflösbarer Name: Meldung auch in der Statuszeile.
- **Traceroute – Lastverteilung, Verlust**: Antworteten bei einem Hop verschiedene Router, wurde nur der letzte angezeigt; Teilverlust (1 von 3) erschien als "OK". Jetzt alle antwortenden Router (Tabelle "IP (+n)" mit Tooltip, Protokoll, CSV), Verlust in Prozent und min / Ø / max je Hop; Status "Ziel" / "Verlust x %" / "keine Antwort" / "OK".
- **Traceroute – Stopp**: Wirkte erst nach Ende des laufenden Pings. Das Abbruch-Signal von `SendPingAsync` greift unter Windows ebenfalls erst danach (gemessen: 2933 ms) – jetzt zusätzlich abbrechbares Warten (gemessen: 333 ms).
- **Traceroute – toter Code**: Ungenutzte, doppelte `NetworkDiagnosticsService.TracerouteAsync` mit denselben Fehlern entfernt.
- **Port-Scan – Historie**: Klick auf einen Historien-Eintrag tat nichts (Befehl war in der Oberfläche nicht gebunden). Jetzt wird der Eintrag geladen.
- **Port-Scan – Beschriftung**: Die Kachel "GESCHLOSSEN" trug den Untertitel "nicht erreichbar" – das Gegenteil trifft zu (Host erreichbar, Verbindung aktiv abgelehnt). Jetzt "Host lehnt ab" mit erklärendem Tooltip.
- **Port-Scan – TCP + UDP**: Mit "TCP + UDP" wurde nur UDP geprüft, und ein eigener TCP-Port 53/123 wurde als UDP geprüft (Portliste plus UDP-Sondermenge). Jetzt wird jeder Prüfauftrag als Port + Protokoll geführt.
- **Port-Scan – Zieladresse**: Der Hostname wurde je Port neu aufgelöst; TCP versuchte IPv6 und IPv4, UDP nahm die erste Adresse – beide prüften ggf. verschiedene Adressen. Jetzt einmalige Auflösung vor dem Scan (bevorzugt IPv4), die Adresse steht im Protokoll; ein nicht auflösbarer Name ergibt eine klare Fehlermeldung statt lauter geschlossener/gefilterter Ports.
- **Port-Scan – geschlossene UDP-Ports**: Eine ICMP-"Port unreachable"-Antwort (von Windows als Fehler der Empfangs-Aufgabe gemeldet) wurde nicht ausgewertet, geschlossene UDP-Ports erschienen als gefiltert. Hinweis: Viele Ziele senden diese Antwort nicht (z. B. Windows-Firewall im Stealth-Modus).
- **Port-Scan – UDP ohne Antwort**: Wird nicht mehr als "GEFILTERT" gemeldet (unterstellt eine Sperre), sondern als neuer Zustand "KEINE ANTWORT" (blau, eigene Kachel, Legende, Erklärung) – bei UDP ist ausbleibende Antwort nicht eindeutig.
- **Port-Scan – TCP-Timeout**: Feste 500 ms ließen offene Ports über WAN/VPN als gefiltert erscheinen. Jetzt aus der Ping-Laufzeit: 3 × RTT + 300 ms, begrenzt auf 800 ms bis 3 s, ohne Ping-Antwort 1,5 s; Wert steht im Protokoll.
- **Port-Scan – TCP-Bewertung**: Jeder Fehler galt als "geschlossen", auch "Host/Netz nicht erreichbar". Jetzt zählt nur eine aktive Ablehnung (RST) als geschlossen, sonst gefiltert, mit Grund in der Info-Spalte. Zudem meldet Windows ein RST sonst erst nach interner Wiederholung (gemessen: 2059 ms) und geschlossene Ports erschienen bei kurzem Timeout als gefiltert – jetzt sofort per `SIO_TCP_INITIAL_RTO` ohne SYN-Wiederholung; als gefiltert erscheinende Ports erhalten dafür einen zweiten Versuch.
- **Port-Scan – eigene Port-Angabe**: Bereiche funktionierten nur ohne Komma – "1-100,443" prüfte still nur Port 443, ungültige Teile wurden übergangen. Jetzt sind Einzelports und Bereiche gemischt erlaubt (getrennt durch Komma, Semikolon oder Leerzeichen, z. B. "80, 443; 8000-8010"); ungültige Teile (außerhalb 1–65535, verdrehte Bereiche, Text) werden vor dem Start aufgelistet und der Scan startet nicht.
- **Port-Scan – CSV-Export**: Neue Spalte "Protokoll" (TCP- und UDP-Zeilen desselben Ports waren nicht unterscheidbar); Anführungszeichen in der Info-Spalte werden korrekt maskiert.
- **DHCP-Discovery – MAC-Zuordnung**: Die ARP-Suche verglich per `Contains` – für 10.1.1.1 traf das auch die Zeilen von 10.1.1.10/10.1.1.100, der Server bekam MAC und Hersteller eines anderen Geräts. Jetzt exakter Vergleich der IP-Spalte.
- **DHCP-Discovery – IPv6-Erkennung**: Kurze IPv6-Adressen wie `fe80::1` (typisch für Router) galten als IPv4 (Prüfung über die Zahl der Doppelpunkt-Teile) und liefen über die ARP-Suche, die MAC des IPv6-Servers fehlte. Jetzt Erkennung über die Adressfamilie.
- **DHCP-Discovery – fehlerhafte Pakete**: Die Auswertung der DHCPv4-Optionen prüfte keine Grenzen, ein abgeschnittenes Paket löste eine Ausnahme aus und brach die ganze Suche ab. Jetzt Grenzprüfung je Option, fehlerhafte Pakete (v4 und v6) werden übersprungen. Empfangspuffer 4096 statt 1024 Bytes (größere Antworten wurden abgeschnitten).
- **DHCP-Discovery – DHCPv6-Anfrage**: Der SOLICIT entsprach nicht RFC 8415: Client-Kennung mit nur 2 von 6 MAC-Bytes, ohne die Pflichtoption Elapsed Time (strenge Server verwerfen die Anfrage) und ohne IA_NA (der Server bietet dann keine Adresse an, "Angebotene Adresse" blieb leer). Jetzt DUID-LLT mit vollständiger MAC, Elapsed Time, IA_NA und Abfrage von DNS-Servern/Domain-Suchliste. Die Transaktions-ID der Antwort wird geprüft, die Wiederholung nutzt dieselbe ID.
- **DHCP-Discovery – Rogue-Bewertung**: Die Warnung erschien bei mehr als einer Gerätegruppe – auch im häufigen, harmlosen Fall "DHCPv4 vom Windows-Server, DHCPv6 vom Router". Jetzt Bewertung pro Protokoll: mehrere DHCPv4- bzw. DHCPv6-Server auf verschiedenen Geräten = Warnung, DHCPv6 auf einem anderen Gerät als DHCPv4 = Hinweis (blau statt orange). Die Heuristik, die einen IPv6-Server ohne MAC an die einzige IPv4-Gruppe hängte und so ein echtes zweites Gerät verstecken konnte, entfällt. PDF-Export übernimmt die neue Bewertung.
- **DHCP-Discovery – Registry-Abgleich**: Der gespeicherte DHCP-Server wird nur noch ausgewertet, wenn DHCP auf dem Adapter aktiv ist (`EnableDHCP = 1`). Nach Umstellung auf eine statische IP bleibt der alte Wert stehen und führte zum Fehlalarm "KRITISCH: genutzter Server hat nicht geantwortet".
- **DHCP-Discovery – Firewall-Prüfung**: Entfernt – sie öffnete nur einen beliebigen Socket, wartete 500 ms und meldete immer "verfügbar". Antwortet kein Server, nennt der Hinweis jetzt konkret die eingehenden Ports (UDP 68 / 546) als mögliche Firewall-Ursache.
- **DNS-Lookup – Fehlermeldungen unsichtbar**: Fehler (z. B. ungültiger DNS-Server) wurden angezeigt und im selben Moment wieder ausgeblendet – das Beenden des Ladezustands versteckte alle Panels, übrig blieb nur "Fehler" in der Statuszeile. Jetzt bleibt die Meldung stehen.
- **DNS-Lookup – Analysieren-Button**: Nach der ersten Abfrage wurde der Inhalt durch reinen Text ersetzt ("▶  Analysieren"), Icon und Layout aus der Oberfläche gingen verloren. Jetzt werden nur Icon und Beschriftung umgeschaltet ("Läuft…" während der Abfrage).
- **DNS-Lookup – Propagation**: Je Resolver wurde nur die erste A-Adresse verglichen. Bei mehreren A-Records (Round-Robin) oder CDN/GeoDNS lieferte jeder Resolver eine andere Reihenfolge bzw. Adresse, die Prüfung meldete "Inkonsistenz". Jetzt Vergleich der vollständigen, sortierten Adressmenge mit drei Ergebnissen: identisch (grün), unterschiedliche Adressen bei allen Resolvern – bei CDN/GeoDNS normal (blau), Resolver widersprechen sich, z. B. Adressen vs. NXDOMAIN (orange). Je Resolver Markierung, ob er der Mehrheit entspricht; alle Adressen werden mit Umbruch angezeigt statt in einer 100 px breiten Spalte abgeschnitten. Resolver ohne Antwort (Timeout/SERVFAIL) zählen nicht als abweichende Zone.
- **DNS-Lookup – "NXDOMAIN"**: Erschien bei jeder Antwort ohne A-Record, auch bei Domains nur mit IPv6 (NODATA) oder bei Serverfehlern. Jetzt Auswertung des Antwortcodes: NXDOMAIN, NODATA, SERVFAIL, REFUSED im Klartext.
- **DNS-Lookup – nicht existierende Domain**: Ergab nur leere Panels ohne Erklärung. Jetzt deutlicher Hinweis oben in der Zusammenfassung ("Domain existiert nicht (NXDOMAIN)"), ebenso bei SERVFAIL/REFUSED und bei einem nicht antwortenden DNS-Server (Timeout).
- **DNS-Lookup – Reverse-Validierung**: Der PTR-Name wurde immer per A-Record zurückaufgelöst – bei IPv6-Adressen war das Ergebnis deshalb stets "Inkonsistent". Zudem zählte nur die erste zurückgelieferte Adresse. Jetzt AAAA für IPv6, Abgleich gegen alle Adressen (als `IPAddress`, unabhängig von der IPv6-Schreibweise); die Rückauflösung zeigt alle Adressen.
- **DNS-Lookup – Propagation ohne Timeout**: Die Abfrage der öffentlichen Resolver lief ohne Zeitgrenze und mit Cache; ein nicht erreichbarer Resolver konnte den Lauf lange blockieren. Jetzt wie die Hauptabfrage: ohne Cache, 5 s Timeout, eine Wiederholung.
- **DNS-Lookup – private Adressen**: CGNAT (100.64/10), Link-Local/APIPA (169.254/16), IPv6-ULA (`fc00::/7`), `::1` und Multicast galten als öffentlich und gingen an den Online-Geo-Dienst. Jetzt erkannt und mit dem jeweiligen Bereich gekennzeichnet (vorher pauschal "RFC-1918 / RFC-4193"). Zudem wurden die ersten drei Adressen *vor* dem Filtern genommen – private Adressen konnten öffentliche verdrängen.
- **DNS-Lookup – benutzerdefinierter DNS-Server**: Ein Servername wurde synchron aufgelöst (Oberfläche blockiert); war er nicht auflösbar, fragte das Programm still den System-DNS und zeigte trotzdem den gewünschten Server an. Jetzt asynchron, bei Fehler klare Meldung; die Zusammenfassung zeigt Name und tatsächlich genutzte IP.
- **DNS-Lookup – Abbruch**: Stopp verwarf alle bis dahin gefundenen Records und zeigte nur "Abgebrochen". Jetzt wird das Teilergebnis angezeigt (mit Hinweis), ein gerade laufender PTR erscheint nicht mehr als "Fehler bei PTR-Abfrage".
- **Ping – MTU-Wert**: Angezeigt wurde die ICMP-Nutzlast als "MTU" – 28 Bytes (IPv4) bzw. 48 Bytes (IPv6) zu klein, ein normales Ethernet erschien als 1472 statt 1500. Die Suche prüfte zudem Nutzlasten bis 1500 Bytes (MTU bis 1528). Jetzt MTU = Nutzlast + Header, Suche von der Mindest-MTU (576 bzw. 1280) bis 1500, Nutzlast zusätzlich angegeben.
- **Ping – MTU bei nicht erreichbarem Ziel**: Mit Startwert 576 meldete ein Ziel ohne Antwort (oder ein nicht auflösbarer Name) "Maximale MTU auf dem Pfad: 576 Bytes". Jetzt erst Erreichbarkeitstest, sonst "MTU nicht ermittelbar".
- **Ping – MTU und Paketverlust**: Jede Zeitüberschreitung und jeder Fehler galt als "Fragment zu groß", ein einzelnes verlorenes Paket verkleinerte das Ergebnis. Jetzt zählt nur "Paket zu groß" als zu groß; Zeitüberschreitungen werden einmal wiederholt und als möglicher PMTU-Blackhole gemeldet. Bei Abbruch wird der Wert als Zwischenstand ("mindestens …") gekennzeichnet statt als Ergebnis.
- **Ping – Ziel**: Der Name wurde bei jedem Paket neu per DNS aufgelöst – bei mehreren Adressen wechselte das Ziel, ein unbekannter Name ergab "100 % Verlust". Jetzt einmalige Auflösung vor dem Start (Auto = IPv4 bevorzugt, oder fest IPv4/IPv6), klare Meldung bei nicht auflösbarem Namen; geprüfte Adresse und Reverse-DNS stehen in der Kachel "Ziel".
- **Ping – Stopp**: Wirkte erst nach Ablauf des laufenden Pings (bis 3 s). Jetzt sofort (gemessen: 15 ms).
- **Ping – doppelter Lauf**: Ein Start während eines laufenden Pings – z. B. über die Schnellaktion "Ping" der DNS-Analyse – startete einen zweiten Lauf, beide schrieben in dieselbe Anzeige. Jetzt wird der laufende Ping vorher beendet bzw. der Start ignoriert.
- **Ping – Dauerbetrieb**: Im Modus "Kontinuierlich" wuchsen Messwerte, Protokoll und Diagramm unbegrenzt, das Diagramm wurde je Paket komplett neu gezeichnet. Jetzt Tabelle mit den letzten 2.000 Paketen, Diagramm mit den letzten 300, Protokoll wird ab 5.000 Zeilen gekürzt; die Statistik läuft inkrementell über alle Pakete.
- **Ping – Fehlermeldungen**: Englische Statusnamen ("DestinationHostUnreachable", "TimedOut") ohne Absender. Jetzt deutsch ("Ziel-Host nicht erreichbar", "Zeitüberschreitung", "TTL abgelaufen" …) mit dem meldenden Router. Nach Abbruch steht "Abgebrochen" statt "Fertig" in der Statuszeile.
- **Ping – CSV-Export**: Zerlegte den Protokolltext, jede Zeile trug den Zeitpunkt des Exports. Jetzt eine Zeile je Paket mit Zeitstempel der Messung, Ziel, Antwort von, Laufzeit, TTL, Bytes, Status.
- **Ping – toter Code**: Ungenutzte `NetworkDiagnosticsService.PingAsync` entfernt.
- **System-Health – Ergebnisse unter Windows PowerShell 5.1**: Routing, Firewall-Sperrregeln, ARP-Cache, Stabilitätsindex und laufende Aufgaben nutzten `Join-String` (nur PowerShell 7) und lieferten stillschweigend immer "0 / keine / OK". Jetzt nativ (siehe Changed).
- **System-Health – Leistungsindikatoren auf deutschem Windows**: Committed Memory und DPC/ISR wurden mit englischen Zählernamen über `Get-Counter` abgefragt und standen auf deutschem Windows immer auf 0 % mit grünem Haken. Jetzt `GetPerformanceInfo` bzw. PDH mit `PdhAddEnglishCounter` (sprachunabhängig).
- **System-Health – Aktivierung**: Galt als "Aktiviert", sobald ein Produktschlüssel-Wert in der Registry stand – den gibt es auch auf nicht aktivierten Systemen. Jetzt über die Licensing-API (`SLIsGenuineLocal`): aktiviert / nicht aktiviert / manipuliert / offline; nicht aktiviert ist eine Warnung.
- **System-Health – Computer-SID**: Wurde aus der Builtin-Gruppe "Administrators" (S-1-5-32-544) abgeleitet – Ergebnis "S-1-5-32", auf deutschem Windows leer. Jetzt der Domänenanteil eines lokalen Kontos (echte Rechner-SID).
- **System-Health – TPM**: Ohne Adminrechte galt der Registry-Schlüssel `Services\TPM` (auf fast jedem Windows vorhanden) als Nachweis. Jetzt TBS-API (`Tbsi_GetDeviceInfo`, ohne Admin, mit Version 1.2/2.0); mit Admin zusätzlich, ob das TPM eingeschaltet und aktiviert ist ("vorhanden, aber deaktiviert" als Warnung).
- **System-Health – VM-Erkennung**: Hersteller "Microsoft" galt als VM-Merkmal – Surface-Geräte wurden als VM behandelt (keine Secure-Boot-/TPM-Prüfung, RAM-Typ "VM"). "A M I" (American Megatrends, echte Mainboards) entfernt.
- **System-Health – Spalte "SMART"**: Zeigte nur den Füllstand. Jetzt Spalte "Zustand" mit dem echten Zustand des physischen Datenträgers (Storage-WMI: OK/Warnung/Fehlerhaft, SSD/HDD/NVMe, mit Admin Verschleiß und Temperatur); Fehler und Warnungen erscheinen als Problem.
- **System-Health – BitLocker**: `manage-bde` braucht Adminrechte – ohne Admin stand immer "–". Jetzt Shell-Eigenschaft `System.Volume.BitLockerProtection` (ohne Admin): Aktiv / Aus / Verschlüsselt… / Angehalten (angehalten = Warnung).
- **System-Health – Netzwerkzone**: Listete alle jemals verbundenen Netze aus der Registry (jedes Hotel-WLAN als "Öffentliches Netzwerk ⚠"). Jetzt nur verbundene Netze (NetworkListManager); bei Hyper-V-vSwitch-Adaptern Kategorie und Bindung.
- **System-Health – Zeitsynchronisation**: Die `w32tm`-Ausgabe wurde nach englischen Texten durchsucht – auf deutschem Windows fehlte das Sync-Alter, die lokale CMOS-Uhr wurde nicht erkannt. Jetzt sprachunabhängig (Datumsfeld, Referenz-ID "LOCL"); Hyper-V-Zeitgeber als Hinweis statt Fehler.
- **System-Health – geplante Aufgaben**: "Letzter Lauf" stand immer auf "–" (`Get-ScheduledTaskInfo` ohne Parameter schlug fehl). Jetzt Taskplaner-COM mit Datum und Ergebnis des letzten Laufs ("Fehler 0x…" hervorgehoben, "noch nie" kein Fehler).
- **System-Health – Windows-Features**: Der DISM-Fallback suchte "Zustand" – auf deutschem Windows heißt das Feld "Status", Ergebnis "Keine aktivierten Features". Jetzt `Win32_OptionalFeature`. Exakte Namen statt Teilstring ("SMB1Protocol" traf auch "SMB1Protocol-Deprecation", die automatische SMB1-Entfernung); Media Player, DirectPlay und Legacy-Komponenten gelten nicht mehr als Risiko.
- **System-Health – "BSOD"**: Zählte auch Kernel-Power 41 (Stromausfall, Ausschaltknopf). Jetzt Bluescreens (BugCheck 1001) als Problem, unerwartete Abschaltungen getrennt als Warnung.
- **System-Health – doppelte Befunde**: Performance-Befunde (und NIC-, Routing-, NetBIOS-, Firewall-, ARP-, Software-Listen) wurden beim nächsten Scan nicht geleert – die Performance-Liste wuchs je Scan.
- **System-Health – Bewertung**: Probleme wurden verzögert eingetragen; die Gesamtbewertung konnte vor den letzten Problemen berechnet werden. Jetzt sofort auf dem UI-Thread, Fehler vor Warnungen sortiert.
- **System-Health – Leerzustände**: Nach einem Scan ohne Befund stand weiter "Noch kein Scan durchgeführt". Fehlgeschlagene Anmeldungen und Minidumps zeigten ohne Adminrechte "Keine …" mit grünem Haken, obwohl nicht prüfbar – jetzt blauer Hinweis "Nicht geprüft" und Eintrag in der neuen Liste "Nicht geprüft".
- **System-Health – Offload/Duplex**: Ungültiges Regex `'RSS|*RSS'` (RSS immer "?") und Vergleich mit dem englischen Text "Disabled"/"Half" (deutsche Treiber: "Deaktiviert"/"Halbduplex"). Jetzt Rohwerte der Treibereinstellungen, Anzeige im Treibertext.
- **System-Health – Fehlalarme**: "Unbekannt" wurde zu "aus" (Defender/Firewall bei fehlgeschlagener Abfrage als kritischer Fehler). Bedarfsgestartete Dienste (`wuauserv`, `BITS`, `W32Time`) galten gestoppt als Problem, ein deaktivierter Druckspooler (Härtung) ebenso – jetzt nur Dienste mit Starttyp Automatisch bzw. unerwartet deaktivierte. Energieplan "Ausgewogen" nur noch auf Servern eine Warnung. Handle-/Thread-Schwelle 20.000/500 statt 5.000/100. CPU-Last als 2-s-Mittel und erst ab 95 % (Scan erzeugt selbst Last); Top-Prozesse nach aktueller CPU-Last statt CPU-Sekunden seit Prozessstart. Internet erreichbar, wenn Ping 8.8.8.8, Ping 1.1.1.1 oder der Windows-HTTP-Verbindungstest klappt (vorher nur 8.8.8.8). NetBIOS nur für aktive Adapter. ARP: Spoofing-Muster statt falscher "IP-Konflikte" über alle Schnittstellen. NIC-Fehler nach Fehlerquote statt ab dem ersten Fehler seit dem Start. Laufwerke nach freiem Platz und Prozent (86 % einer 2-TB-Platte sind keine Warnung mehr). Mehrere Standard-Gateways mit aktivem VPN sind normal. PowerShell 2.0 über den Feature-Zustand statt Registry-Rest.
- **Neustart als Administrator**: Die neue Instanz fand die Einzelinstanz-Sperre der alten noch belegt und beendete sich sofort; danach schloss sich die alte – kein Fenster mehr offen. Die Sperre wird jetzt vorher freigegeben (bei abgebrochener UAC-Abfrage wieder belegt).
- **Zweite Programminstanz**: Beim Beenden einer zweiten Instanz (die die erste in den Vordergrund holt) wurde `ReleaseMutex()` auf der bereits freigegebenen Sperre aufgerufen → `ObjectDisposedException`.
- **Systemübersicht – Kopieren-Rückmeldung**: Nach Klick auf einen Kopier-Button (Computername, Seriennummer, SID, GUID, Produktschlüssel, MAC, Adressen) erschien kurz der Text "E73E" statt eines Häkchens – dem Unicode-Escape fehlte der Backslash.
- **DNS-Lookup – NXDOMAIN und Zone**: Bei einem nicht existierenden Namen lieferte die SOA-Abfrage den SOA der übergeordneten Zone (z. B. `de` mit `f.nic.de`), der als "Authoritative NS" der Domain erschien. Wird jetzt verworfen; Zonen-Prüfungen (DNSSEC, CAA, MTA-STS, SPF) entfallen für nicht existierende Namen.

---

## [0.99.11.14] - 2026-10-03
### Added
- **UI / Styles**: Zentrales Style-Wörterbuch `Themes/` (in `App.xaml` eingebunden, nur Keys mit Präfix bzw. implizit für die neuen Controls – bestehende Dialoge bleiben unverändert):
  - `Tokens.xaml`: Farben/Brushes mit fester Status-Semantik (Success/Warning/Error/Info/Neutral je Vorder-/Hintergrund/Rahmen), Auslastungsfarben, Abstands-Skala 4/8/12/16/24, Radien, Icon-Schrift (`Segoe Fluent Icons, Segoe MDL2 Assets`).
  - `Typography.xaml`: Skala `Text.Title`/`Label`/`Value`/`SubHeader`/`SectionLabel`/`Secondary`/`ChipCaption`/`Kpi`/`Mono`/`Icon`; `Text.Value` färbt Lade-/Leerzustände ("wird ermittelt …", "–", "Nicht verfügbar") automatisch dezent ein, `Text.Value.Pending` zeigt den Ladetext bis zum ersten Wert.
  - `Controls.xaml`: `Button.Primary`/`Secondary`/`Secondary.Small`/`Icon`/`Icon.Copy` (gemeinsames Template mit Hover-, Gedrückt-, Deaktiviert- und Tastaturfokus-Zustand, Icon per `ctl:ButtonAssist.Icon`, auch für ToggleButton), `ProgressBar.Usage`, `Chip`/`Chip.Success`/`.Warning`/`.Error`/`.Info`.
  - Neue wiederverwendbare Controls in `Controls/`: `Card` (Karte mit Icon, Überschrift und optionaler Kopf-Aktion), `StatusRow` (Statuszeile: Akzentbalken, Titel, Detail, Status-Pill, Aktion; bündig über `SharedSizeGroup`), `Theme` (Token-Zugriff aus Code, einheitliche Auslastungsschwellen 80/90 %).
- **SystemOverviewDialog**: als Referenz-Dialog vollständig auf das Style-Wörterbuch umgestellt (keine lokalen Styles, keine Hex-Farben mehr im XAML): Karten als `ctl:Card` ohne DropShadow (1-px-Rahmen), Fluent-Icons statt Emojis in Kopf, Kartenüberschriften, Buttons und Export-Menü; Netzwerkzone, Antivirus, Firewall, Windows Update und nicht erreichbare Netzlaufwerke als `StatusRow`; Status-Chips neutral für Informationen und farbig nur bei Auffälligkeiten, neuer Chip "Gesamtstatus" ("Keine Auffälligkeiten" / "N Hinweise"); Chip-Leiste bricht um; Auslastungsbalken normal blau, ab 80 % orange, ab 90 % rot (Laufwerke, CPU, RAM, Netzlaufwerke; Akku mit umgekehrter Skala); IP-/DNS-Zeilen nur noch mit Kopieren-Icon und "⋯"-Menü (Ping, Traceroute, DNS-Auflösung, Port-Scan) statt fünf Icon-Buttons; Platzhalter "wird ermittelt …" bis zum ersten Wert, Fehlerzustände mit "Erneut versuchen"; unter 980 px Breite einspaltiges Layout; Label-Spalten je Seite automatisch gleich breit statt fester Pixelwerte.

- **Navigation**: Kompakter Modus (nur Icons, 48 px) über ☰-Schalter oben in der Seitenleiste; Gruppen öffnen dort ein Flyout-Menü mit ihren Seiten, Tooltips zeigen die Namen. Einstellung wird gespeichert (`%AppData%\DATEXT-Diagnostics\sidebar-compact.flag`).
- **Navigation**: Status-Punkt an Einträgen (`ctl:NavAssist.Badge`); "Übersicht" zeigt einen orangen/roten Punkt, wenn der Gesamtstatus der System-Übersicht Hinweise oder Probleme meldet.
- **UI / Styles**: `Themes/Navigation.xaml` (`Nav.Item`, `Nav.Group`, `Nav.GroupContent`, `Nav.Notice`, `Nav.SearchBox`, `Nav.Focus`) und `Controls/NavAssist.cs` (Icon, Suchbegriffe, Badge, aktive Gruppe, kompakter Modus) im zentralen Style-Wörterbuch.

### Changed
- **Navigation**: Optik an das Style-Wörterbuch angeglichen: Fluent-Icons statt Emojis (vorher teils mehrfach vergeben, z. B. 📦 viermal), nur Gruppen und "Übersicht" mit Icon, Seiten eingerückt an einer dezenten Führungslinie, Pfeil als drehende Fluent-Glyphe, 1-px-Linien statt 0,5 px (Seitenleiste, Menü, Statusleiste), Farben aus den Tokens. Die Gruppe der geöffneten Seite bleibt auch zugeklappt markiert (Akzentfarbe und -balken). Suchfeld mit Fluent-Lupe, Platzhalter "Suchen (Strg+F)", ×-Button zum Leeren und Akzentrahmen bei Fokus. Update-Hinweis im gleichen Stil wie die übrigen Einträge.
- **Navigation**: Struktur vereinfacht: die Ein-Eintrag-Gruppen "MS-SQL" und "Updates" entfallen – "MS-SQL-Analyse" liegt jetzt in der Gruppe "Active Directory & SQL" (vorher "Active Directory"), "Winget-Pakete" in "System". Einheitliche Namen: "E-Mail-Check" (vorher "Email-Check"), "AD-Suche" (vorher "AD Suche"), "Gruppenrichtlinien" (vorher "Group Policies"). Die Speedtests haben erklärende Tooltips; alte Bezeichnungen bleiben über Suchbegriffe auffindbar (z. B. "Group Policies", "GPO").
- **UI / Styles – alle Dialoge**: Umstellung der übrigen 47 Dialoge/Fenster (inkl. `EmailCheckWindow`, `Resources/CompactStyles.xaml`) auf das zentrale Style-Wörterbuch:
  - ~4.000 hartcodierte Farben (XAML und Code) durch Tokens ersetzt – im XAML abhängig von der Eigenschaft (z. B. Grau als Vordergrund → `Brush.Text.*`, als Rahmen → `Brush.Border.*`, als Fläche → `Brush.Surface.*`), im Code über `Theme.Color("Color.…")` bzw. `Theme.Hex("Color.…")` (neu, liefert "#RRGGBB" für Code, der Farben als Text weiterreicht). Gruppen- und Diagrammfarben (z. B. Petrol, Lila) bleiben bewusst erhalten.
  - 676 Schriftgrößen auf die Skala gebracht (10 → 11, 13 → 12, 15 → 14, 18 → 16, 22 → 20; Kennzahlen ≥ 24 unverändert).
  - 102 lokale Style-Kopien (`Card`, `InfoCard`, `PrimaryBtn`, `SecondaryBtn`, `BtnPrimary`, `SectionLabel`, `StatLabel`, …) sind jetzt Ableitungen der zentralen Styles (`Card.Border`, `Button.Primary/Secondary/Danger/Subtle`, `Text.SectionLabel`) – Layoutwerte bleiben, eigene Templates und Schatten entfallen. Dadurch haben alle diese Buttons Hover-, Gedrückt-, Deaktiviert- und Tastaturfokus-Zustand. Hauptbuttons einheitlich in Akzentfarbe (vorher teils Gruppenfarbe, z. B. Rot in den E-Mail-Dialogen; die Gruppenfarbe bleibt im Titel).
  - 89 Buttons zeigen Fluent-Icons statt Emojis (`ctl:ButtonAssist.Icon`); Buttons, deren Text der Code zur Laufzeit mit Emoji setzt, bleiben vorerst unverändert.
  - Schatten an Karten entfernt (1-px-Rahmen statt `DropShadowEffect`), außer der Such-Hervorhebung in der Netzwerk-Topologie.
  - Neu im Wörterbuch: `Button.Danger`, `Button.Subtle`, `Card.Border`, Tokens `Brush/Color.Error.Hover/Pressed`.
- **UI / Styles – Phase 2**:
  - **Gruppen- und Diagrammfarben als Tokens**: `Brush/Color.Group.*` (System, Network, ActiveDirectory, Sql, Capture, Performance, Mail, Inventory), `Brand`, Kategoriepalette `Teal/Purple/Indigo/Pink/Brown`, `Console.Bg/Fg`; weitere 459 Farben zugeordnet. Dialog-Titel nutzen die Gruppenfarbe (E-Mail-Dialoge vorher fälschlich mit der Fehlerfarbe), die Hilfe-Palette verweist auf dieselben Tokens.
  - **Sonder-Styles**: neue zentrale Styles `Button.SplitMain/SplitArrow` (Split-Buttons, über neues `ctl:ButtonAssist.CornerRadius`), `Tab.Item`/`Tab.Radio` (Reiter im Unterstrich-Stil, Icon über `ctl:ButtonAssist.Icon`), `DataGrid.Default/ColumnHeader/Row/Cell`, `Badge`/`Badge.Success/Warning/Error/Info`, `Chip.Toggle` (Filter-Chips), `Input.TextBox`. 52 lokale Sonder-Styles leiten davon ab (datengetriebene Zeilenfarben, z. B. Routing-Tabelle, bleiben erhalten); 26 nirgends verwendete lokale Styles entfernt.
  - **Emojis in Überschriften und Texten**: 625 Stellen (Titel- und Kachel-Icons, Abschnittsüberschriften, Menüeinträge, Reiter- und Gruppen-Überschriften) zeigen Fluent-Icons. Elemente, deren Text der Code zur Laufzeit setzt, sowie Auswahllisten-Einträge, Checkboxen und Tabellen-Spaltenköpfe bleiben bewusst unverändert.
  - **Karten**: 115 Karten nach dem Muster Rahmen + Abschnittsüberschrift sind jetzt `ctl:Card` (einheitliche Kopfzeile mit Icon; Abstände unverändert), u. a. in MS-SQL-Analyse, System-Health, DNS, Ping, Traceroute, Ereignislog, E-Mail-Header, Performance-Dialogen. Im DNS-Dialog Typprüfungen von `Border` auf `FrameworkElement` erweitert.
  - **Restliche Karten**: die übrigen 91 Karten mit abweichendem Aufbau sind ebenfalls `ctl:Card`; damit nutzt kein Dialog mehr einen eigenen Karten-Rahmen. Kopfzeilen mit Zähler oder Buttons (z. B. System-Health "Aktivierte Windows-Features", "Netzwerkadapter", Troubleshooting "Gesamtfortschritt") laufen über `Header` + `HeaderAction`, bisher fett/farbig gesetzte Überschriften (Hardware, Dienste, UEFI, Inventory Agent, System-Info) über `Header`/`Icon` in Versalien. Karten ohne Überschrift (Kennzahl-Kacheln in Gruppenrichtlinien und Routing, Tabellenkarten in Winget und AD, Willkommens-/Ladekarten) sind `ctl:Card` ohne Kopf; die Kopfzeile entfällt dann automatisch (neuer Trigger im Karten-Template). 14 Karten-Überschriften mit Emoji zeigen jetzt ein Fluent-Icon. 4 dadurch unbenutzte lokale Karten-Styles entfernt.
  - **Statusanzeigen als `ctl:StatusRow`**:
    - UEFI-Boot-Zertifikat: Secure Boot (mit Boot-Modus), TPM (mit "TPM-Verwaltung"), Windows UEFI CA 2023 (mit Servicing-Status) und Boot-Manager-Signatur (mit "Prüfen") statt farbiger Einzel-Badges. Die Buttons im Secure-Boot-Warnhinweis verwenden jetzt `Button.Danger`/`Button.Secondary`.
    - Troubleshooting: Update-Reparatur- und Netzwerk-Schritte zeigen Nummer, Titel, Befehl, Status und "Ausführen" als Statuszeile. Der Schrittstatus ist jetzt ein `StatusState` statt Farbwerten.
    - Windows-Komponenten-Prüfung: die Phasen 1–4 (DISM CheckHealth/ScanHealth/RestoreHealth, SFC) als Statuszeilen; die Nummern-Badges entfallen.
    - System-Health: die Liste "Gefundene Probleme & Warnungen" zeigt je Befund eine Statuszeile mit Meldung, Bereich und Pill "Problem"/"Warnung".
  - **Kopieren-/Exportieren-Buttons**: Alle 56 Buttons in 29 Dialogen sehen jetzt aus wie in der System-Übersicht. Kopieren ist Sekundär-Button mit Kopier-Icon, Exportieren ist Primär-Button mit Speichern-Icon; öffnet er ein Formatmenü, kommt der Pfeil ▾ dazu. Neue Vorlagen im Style-Wörterbuch:
    - `Button.Copy`, `Button.Copy.Small` (in Karten, z. B. Protokoll kopieren)
    - `Button.Export` (direkter Export), `Button.Export.Menu` (mit Formatmenü)
    - `Button.Export.Split` (Hauptteil eines Split-Buttons "letztes Format erneut exportieren", zusammen mit `Button.SplitArrow`)

    Entfernt wurden eigene Inline-Templates, Emojis und abweichende Farben, z. B. dunkelgraue Kopieren-Buttons bei Ping/DNS/Portscan/Traceroute, ein blauer Kopieren- und ein oranger Export-Button in der Netzwerk-Übersicht und ein Export in Gruppenfarbe in der Domain-/E-Mail-Analyse. JSON-Menüeinträge in AD und UEFI hatten statt eines Icons den Text "{}" und zeigen jetzt ein Icon.
  - **Aktualisieren-Buttons**: Die 13 Buttons in UEFI, Autorun, Software, Verbindungen, Netzwerkkarten, Routing, Netzwerk-Übersicht, Netzwerktopologie und MS-SQL-Analyse sehen jetzt aus wie in der System-Übersicht: Sekundär-Button mit Aktualisieren-Icon statt Blau, Grün oder Grau mit 🔄-Emoji. Neue Vorlagen: `Button.Refresh`, `Button.Refresh.Small` sowie `Button.Refresh.Split` + `Button.SplitArrow.Secondary` für Aktualisieren mit Optionsmenü (Verbindungen). Aktionen wie "Alle aktualisieren" (Winget) oder "sp_Blitz aktualisieren" bleiben unverändert.
- **Menüleiste**: Emojis entfernt; Einträge mit Fluent-Icons (`MenuItem.Icon`), Hauptmenüs und ankreuzbare Einträge (Sprache, Dunkelmodus) ohne Symbol. "Update verfügbar" in Erfolgsfarbe mit Download-Icon.
- **Hilfe (HelpDialog)**: Navigation, Seitentitel und Suchindex an die neue Struktur angepasst ("Active Directory & SQL", Winget unter "System", Gruppe "Updates" entfällt, Namen "AD-Suche", "Gruppenrichtlinien", "E-Mail-Check", "MS-SQL-Analyse"). Startseite um "Navigation bedienen" (Suche, kompakter Modus, Status-Punkt) und die aktuelle Übersicht (Gesamtstatus, "⋯"-Menü, Statuszeilen) ergänzt, "Tipps & Shortcuts" um Tastaturbedienung der Navigation.
- **Dokumentation**: Technisches Handbuch (`.md`, `.docx`, `.pdf`) auf Version 0.99.11.14 / Oktober 2026 und die aktuelle Navigation umgestellt: neue Kapitel "Active Directory & SQL", "Packet Capture" (vorher unter Netzwerk) und "Inventar", Winget und Troubleshooting (ersetzt "Windows Komponenten Check") unter System, "Netzwerk & Verbindungen" mit den Registerkarten Netzwerkkarten/Routing/Verbindungen, neue Abschnitte zu Gruppenrichtlinien, MS-SQL-Analyse, Lokaler Inventarisierung, Inventory Agent sowie "Seitenleiste & zentrales Style-Wörterbuch"; Ordnerstruktur ergänzt (`Controls/`, `Themes/`, `SqlAnalysis/`, `Inventory/`, `Troubleshooting/`). Im Word-Dokument manuelle Seitenumbrüche durch Fließumbruch ersetzt (Überschriften bleiben beim Folgetext, Info-Tabellen werden nicht getrennt).
- **Navigation**: Tastatur: ↑/↓ bewegen nur den Fokus, Enter/Leertaste öffnet die Seite, →/← klappt Gruppen auf/zu, Pos1/Ende springen; Tab erreicht die Navigation als ein Element. Aus dem Suchfeld führt ↓ in die Trefferliste, Enter öffnet den ersten Treffer, Esc leert die Suche.
- **Troubleshooting**: `WindowsComponentCheckDialog` ("Windows Komponenten") als eigenständiger Nav-Punkt entfernt und 1:1 in den Troubleshooting-Dialog eingebettet (Abschnitt "Systemdatei- & Image-Integrität", auf-/zuklappbar) – inhaltlich ebenfalls ein Troubleshooting-Werkzeug, keine Logik dupliziert
- **Troubleshooting**: Checkbox-Liste + "Ausgewählte Schritte ausführen" durch unabhängige Einzel-Buttons ersetzt (analog "Weitere Werkzeuge") – jede Aktion läuft sofort bei Klick
- **Troubleshooting**: Ex-Schritte 1–3 (Update-Ordner, BITS-Queue, DLL-Reregistrierung) zu einem Button gebündelt; Update-Richtlinien, gpupdate und Update-Komponentenspeicher-Bereinigung sind jetzt eigene Buttons
- **Troubleshooting**: Update-Komponentenspeicher-Bereinigung um Option `/ResetBase` erweitert (mit Bestätigungsdialog, da nicht umkehrbar: installierte Updates werden dauerhaft fixiert, dafür mehr freigegebener Speicher)
- **Troubleshooting**: kombinierter Winsock/WinHTTP-Schritt in drei unabhängige netsh-Aktionen aufgeteilt (`netsh int ip reset` neu, `netsh winsock reset`, `netsh winhttp reset proxy`), jeweils mit Kontexthinweis zum Anwendungsfall; `netsh int ip reset` erkennt eine installierte Hyper-V-Rolle (Dienst `vmms`) und verlangt dann eine gesonderte Bestätigung mit den konkreten Risiken (vSwitch-Bindungen, NIC-Teaming, VLAN, statische IPs)
- **HelpDialog**: Inhalte der bisherigen "Windows Komponenten Check"-Seite in die Troubleshooting-Handbuchseite integriert, eigener Nav-Punkt entfernt
- **Windows Komponenten Check**: Reihenfolge auf DISM zuerst, SFC zuletzt umgestellt (CheckHealth → ScanHealth → ggf. RestoreHealth → SFC /scannow), damit SFC beim Reparieren auf einen bereits von DISM geprüften/reparierten Komponentenspeicher zugreifen kann
- **SystemHealthDialog**: acht Scan-Bereiche (Hardware, Betriebssystem, Speicher, UAC/fehlgeschlagene Anmeldungen, Netzwerkadapter/Gateway/DNS, Dienste, Ereignisse, installierte Software) laufen jetzt vollständig ohne PowerShell-Subprozess – native WMI-Abfragen (`ManagementObjectSearcher`), Registry-Zugriffe, `System.Diagnostics.Eventing.Reader.EventLogReader`, `NetworkInterface` und `Process`-Klasse statt `Get-WmiObject`/`Get-WinEvent`/`Get-NetAdapter` etc. Eliminiert für diese Bereiche das größte verbleibende AV-Heuristik-Merkmal komplett (statt es nur zu drosseln) und ist spürbar schneller. BitLocker-Status läuft jetzt über einen direkten `manage-bde.exe`-Aufruf statt eines PowerShell-Wrappers. Die nicht sauber portierbaren Bereiche (Defender/Firewall-Status, SMB1/RDP/NLA/Credential-Guard/NTP-Härtungs-Checks, Netzwerkzonen/Hyper-V-SET-Switches, Performance/Aufgaben/Windows-Features/Server-Rollen) bleiben vorerst PowerShell-basiert. Die nie funktionale "ausstehende Updates"-Anzeige (PSWindowsUpdate-Modul, praktisch nie installiert) wurde ersatzlos entfernt.
- **SystemOverviewDialog**: Typografie vereinheitlicht – feste Skala (20 Titel · 12 Label/Wert/Zwischenüberschrift · 11 Abschnitts-/Nebentext · 9 Chip-Caption) über neue Styles `SubHeader`, `SubText` und `FlBtnSmall`. Entfernt: einzeln auf 11 verkleinerte Werte (Seriennummer, BIOS, Produktschlüssel, SID, GUID, GPU-Treiber, Reverse DNS, NTP, IP-/DNS-Adressen), 13er-Werte in Ressourcen/Akku, 10er-Button und -Badge. Bildschirm-Karte nutzt jetzt `InfoLabel`/`InfoValue` statt invertierter Fett/Normal-Darstellung; AV-/Firewall-Buttons im `FlBtn`-Look statt Windows-Standardbutton; Nebentext einheitlich `#666666`; Label-Spalten links 160, rechts 130

- **SystemOverviewDialog**: Antivirus-/Firewall-Erkennung überarbeitet. Defender-Status jetzt direkt aus `MSFT_MpComputerStatus` (Client **und** Server; unterscheidet Echtzeitschutz aus, Passive Mode, EDR-Block-Mode, Signaturalter). Windows Defender Firewall wird je Profil (Domäne/Privat/Öffentlich) inkl. GPO-Vorgaben und Dienststatus `mpssvc` ausgewertet, bewertet wird das aktive Profil (`HNetCfg.FwPolicy2`) – vorher nur das Privat-Profil. Neuer dritter Status "Unbekannt"/Warnung (orange) für Einschränkungen wie veraltete Signaturen oder deaktivierte Nebenprofile; erscheint auch im Sicherheits-Chip. WMI-Abfragen mit 5-s-Timeout. Darstellung neu: je Produkt eine eigene Zeile (farbiger Akzentbalken, Produktname, Detailzeile mit Version/Hinweis wie "Aktives Profil: Domäne" oder "Signaturen veraltet", Status-Pill Aktiv/Eingeschränkt/Inaktiv/Unbekannt, einheitlicher "⚙ Verwalten"-Button mit Herstellerbezeichnung als Tooltip) statt drei lose nebeneinander gestapelter Spalten; Status- und Button-Spalte sind über alle Zeilen hinweg ausgerichtet (`SharedSizeGroup`). Export/Bericht liest die Werte jetzt aus den erkannten Daten statt aus den UI-Elementen. Dienst-Fallback über `ServiceController` statt `Win32_Service`, Liste aktualisiert (u. a. SentinelOne, Cortex XDR, Trellix, Bitdefender Endpoint, Kaspersky mit Versionsnummer im Dienstnamen; `klnagent` entfernt, da kein AV).

- **Export-Dateinamen**: Alle Berichts-Exporte (PDF, CSV, JSON, TXT, XLSX, PNG) heißen jetzt einheitlich `<datum>_<uhrzeit>_<fqdn>_<dialog>[_<detail>].ext`, z. B. `20261003_201227_pc01.domaene.de_Autorun-Analyse.pdf`.
  - Details sind z. B. Ziel-Host bei Ping, Traceroute und Port-Scan, die abgefragte Domain bei DNS- und E-Mail-Analyse oder die E-Mail-Adresse im E-Mail-Check.
  - Inventar-Exporte eines anderen Geräts tragen dessen Hostnamen statt des FQDN.
  - Vorher gab es rund ein Dutzend Schemata, teils ohne Sekunden, mit Benutzername oder ohne Rechnernamen.
  - Neue zentrale Klasse `Services/ExportFile` (`Name`, `NameFor`, `OfferOpen`).
  - Ausgenommen sind Zertifikat-Export, Handbuch-PDF und Agent-Installationspakete (ZIP).
- **Nach dem Export**: Jeder Berichts-Export fragt jetzt, ob die Datei geöffnet werden soll.
  - Nachgerüstet in Inventar (Agent, Detail, lokal), Geräte-Scan, DHCP-Discovery, Netzwerk-Übersicht, Netzwerk-Topologie, Netzwerk-Gesamtbericht, MS-SQL-Analyse, Netsh-Trace, Pktmon, System-Info (Software-CSV) und Temp-Bereinigung.
  - Vorher gab es dort nur eine OK-Meldung oder einen Hinweis in der Statuszeile.
  - Der TXT-Export in System-Health speicherte bisher ganz ohne Rückmeldung und ohne Fehlerbehandlung.
  - Alle vorhandenen Öffnen-Abfragen verwenden jetzt denselben Wortlaut: Titel "Export", Text "Export gespeichert: <Pfad> – Datei jetzt öffnen?". Zuvor gab es rund ein Dutzend Varianten ("Gespeichert … Jetzt öffnen?", "TXT-Export erfolgreich! Möchten Sie die Datei öffnen?", "Protokoll gespeichert. Datei öffnen?" ohne Pfad …). Umgestellt sind 42 Abfragen in 21 Dialogen, darunter die lokalen `AskOpen`-Helfer, sowie der E-Mail-Check über die Sprachdatei (DE/EN). Abfragen, die einen Ordner im Explorer oder einen Mitschnitt in Wireshark öffnen, sowie die Meldungen zu erzeugten Analyse-Berichten (Netsh-Trace, Windows-Berichte in der System-Übersicht) bleiben unverändert.
  - Export-Buttons, deren Beschriftung der Code während des Exports ändert, setzen sie danach wieder auf "Exportieren" zurück, nicht mehr auf "💾 Exportieren ▾".

- **Auswahlfelder (Dropdowns)**: neuer zentraler Stil `Input.ComboBox` / `Input.ComboBoxItem` im Style-Wörterbuch. Über den impliziten ComboBox-Style in `App.xaml` gilt er für alle rund 50 Auswahlfelder.
  - Feld: weiß, 1-px-Rahmen, abgerundete Ecken, 30 px hoch, Fluent-Pfeil; Hover-, Fokus- (Akzentlinie) und Deaktiviert-Zustand.
  - Liste: weiße Karte mit Schatten; Hover hellgrau, Auswahl hellblau in Akzentfarbe.
  - Editierbare Felder werden unterstützt.
  - Lokale Höhen (28–36 px), Schriftgrößen und Farben in 20 Dialogen sind entfernt.
  - Neu: `ctl:InputAssist` mit `Icon` (Glyphe im Feld bzw. am Eintrag), `Placeholder` (grauer Hinweis, solange nichts gewählt ist), `Detail` (Zusatz rechts im Eintrag), `Description` (zweite Zeile) und `StatusBrush` (Statuspunkt).
  - Listen-Zwischenüberschriften über `Input.ComboBoxHeader`, Trennlinien als `<Separator/>`.
- **Werkzeugleisten** von Ping, DNS, Traceroute, Port-Scan, Verbindungen, LAN-Bandbreite, Ereignislog und Software: Eingabefeld, Auswahlfelder und Buttons sind einheitlich 30 px hoch und stehen auf einer Linie. Vorher waren es 34 bis 38 px. Icon-Buttons (Stopp/Löschen) waren durch Innenabstand beschnitten.
- **Einträge ohne Emoji**:
  - DNS-Server-Auswahl (DNS-Analyse, Domain-/E-Mail-Analyse, E-Mail-Header, E-Mail-Check) zeigt Globus-Icon im Feld und Einträge wie "Google" mit grauer IP rechts statt "🌐 Google (8.8.8.8)".
  - Ereignislog: Protokoll- und Schnellfilter-Auswahl mit Fluent-Icons, Gruppen als Zwischenüberschriften, Trennlinie als echte Linie.
  - Ping: "Kontinuierlich" ohne ∞.
  - Platzhalter wie "Adapter wählen …", "Laufwerk wählen …" oder "Szenario wählen …" in 14 Feldern.
- **Netzwerkadapter-Auswahl** (Netzwerklast, Geräte-Scan, DHCP-Discovery, LAN-Bandbreite, Inventory Agent) über neue Hilfsklasse `ctl:AdapterList`: Typ-Icon (Ethernet / WLAN / VPN), Adaptername mit Statuspunkt, zweite Zeile mit IPv4 und Geschwindigkeit bzw. "getrennt" (bei DHCP zusätzlich das Gateway); das Feld zeigt das Typ-Icon des gewählten Adapters. Vorher reine Textzeilen wie "Ethernet – 192.168.1.20" oder "Ethernet  [1 Gbit/s]".
- **DHCP-Discovery**: Die Adapterauswahl steckte rahmenlos in einem eigenen Kasten mit 🔄-Emoji-Button. Jetzt ist sie ein normales Auswahlfeld mit Icon-Button "Adapterliste aktualisieren"; die Buttons der Leiste sind ebenfalls 30 px hoch. Das Zusatzfeld "Benutzerdefiniert" der DNS-Analyse nutzt `Input.TextBox` in derselben Höhe.

- **Geräte-Scan – Erkennung (Runde 1)**:
  - **Geräte ohne Ping-Antwort** werden gefunden, über die neue Option "Ohne Ping-Antwort" (standardmäßig an). Betroffen sind Windows-Rechner im Firewall-Profil "Öffentlich", Smartphones im Standby, Kameras und IoT-Geräte.
    - Im eigenen Subnetz wird nach dem Ping die Windows-Nachbartabelle gelesen (`GetIpNetTable2`, ohne `arp.exe`): Wer ARP beantwortet hat, ist da, auch wenn ICMP verworfen wird.
    - In gerouteten Netzen gibt es einen kurzen TCP-Check auf 445, 135, 80, 443, 22, 3389 und 62078. Auch ein abgewiesener Verbindungsaufbau zählt als Antwort.
    - Solche Geräte erscheinen als "Online" mit Antwortzeit "-" und dem Dienst-Hinweis "kein Ping (ARP)" bzw. "kein Ping (TCP …)". Die MAC kommt direkt aus der Nachbartabelle.
    - Testnetz (/24-Ausschnitt): 55 statt 49 Geräte, Scan-Dauer 4,0 statt 3,2 s.
  - **Zweiter Ping** für Geräte, die per ARP erreichbar sind, den ersten Ping aber nicht beantworten (typisch für WLAN-Stromsparmodus). Dadurch kommen Antwortzeit und TTL meist doch noch.
  - **System-Hinweis aus der TTL** der Ping-Antwort (64 → Linux/macOS/Mobil, 128 → Windows, 255 → Netzwerkgerät): neue Spalte "System", auch in CSV- und PDF-Export. Ist der Typ sonst unbekannt und die TTL zeigt Windows, lautet der Typ "💻 Computer (Windows)".
  - **Lokale Typerkennung immer aktiv:** Ohne SNMP-Treffer und ohne Fingerbank stand bisher "⚠️ Fingerbank API nicht konfiguriert" als Gerätetyp, die vorhandene lokale Heuristik lief gar nicht. Ebenso führten "Fingerbank: keine Daten" und API-Fehler zu einem Hinweis statt eines Typs. In allen drei Fällen greift jetzt die lokale Heuristik.

- **Geräte-Scan – Erkennung (Runde 2)**:
  - **Keine Firmenwerte mehr im Code:** Die fest eingebauten Namensmuster `wsha-`, `nbha-`, `nbho-`, `vmtest-`, `ad0`, `labtime` und `.datext.` sind aus der lokalen Erkennung und der Fingerbank-Nachbearbeitung entfernt.
    - Neu ist das Feld "Eigene Computer-Namen" in den Scan-Optionen. Es nimmt Präfixe (z. B. `ws-`) oder Domänen-Endungen mit Punkt (z. B. `.firma.local`) auf und wird nur lokal in den Einstellungen gespeichert.
    - Die Vorgabe ist leer.
    - Das SNMP-Codebeispiel nutzt einen neutralen Namen.
  - **Reihenfolge der Typerkennung nach Beweiskraft:**
    1. Infrastruktur-Präfixe (`sw-`, `prn-`, `ap-`, `cam-`, `fw-`, `gw-`, `ups-` …)
    2. eigene Muster
    3. allgemeine Computer-Namen (DESKTOP-…, LAPTOP-…, WIN-…)
    4. Hersteller- und Namensregeln
    5. erst zuletzt "Hostname mit Domäne → Computer (vermutlich)"

    Bisher stand die Domänen-Regel an erster Stelle. Dadurch wurden z. B. `drucker.firma.de` und jedes mDNS-Gerät `*.local` (iPhone, Drucker, Chromecast) zum "Computer". Neue Regel für Chromecast/Google TV.
  - **FRITZ!Box-Geräteliste (TR-064)**, optional, neuer Abschnitt in den Scan-Optionen:
    - Liest die Geräteliste der Box (nur lesend, HTTP-Digest-Anmeldung).
    - Ergänzt Hostnamen und MAC-Adressen und kennzeichnet "LAN/WLAN (FRITZ!Box)".
    - Nimmt aktive Geräte ohne Ping-Antwort auf ("nur FRITZ!Box"), bei "Offline-Geräte" auch der Box bekannte Offline-Geräte.
    - Adresse und Benutzer werden gespeichert, das Passwort nicht.
    - Ist die Box nicht erreichbar oder lehnt die Anmeldung ab, steht ein Hinweis in der Statuszeile; der Scan läuft weiter.
  - **Aufräumen:**
    - Die nie aufgerufene Online-Herstellerabfrage (macvendors.com) ist entfernt; sie hätte MAC-Adressen an einen externen Dienst gesendet.
    - Nur per mDNS/SSDP gefundene Geräte außerhalb des Scanbereichs erhalten jetzt MAC, Hersteller und Typ.
    - Hyper-V-Netzwerkkarten (00:15:5D) werden als "Microsoft (Hyper-V)" bzw. "Virtuelle Maschine" erkannt; die Prüfung war zuvor unerreichbar.
    - Die Abschlussmeldung zählt alle Online-Geräte inkl. "ohne Ping-Antwort".

### Fixed
- **Historie in Port-Scan, Ping, Traceroute und DNS-Analyse**: Beim Laden eines Historien-Eintrags war nicht erkennbar, welches Ergebnis angezeigt wird. Es wurde nur der Hostname ins Eingabefeld übernommen und eine Statuszeile bzw. ein Kopf-Badge "Historien-Eintrag #N" gesetzt. Bei mehreren Läufen gegen denselben Host sah man keinen Unterschied.
  - Neu ist der Hinweis-Baustein `ctl:HistoryBanner` (Port-Scan in der Statistikleiste, sonst unter der Werkzeugleiste). Er zeigt "Historie #N · Host", Uhrzeit und Kerndaten:
    - Port-Scan: Ports
    - Ping: Pakete, Verlust, Ø-Latenz
    - Traceroute: Hops, Ziel erreicht, max. Latenz
    - DNS: Server, Einträge, Dauer
  - Dazu gibt es einen Button "Zum letzten Scan / Ping / Route / Abfrage".
  - Der angezeigte Eintrag ist in der Historienliste hellblau mit Akzentbalken markiert. Nach einem neuen Lauf ist es der neueste.
  - Ein neuer Lauf oder "Zurücksetzen" blendet den Hinweis aus.
  - Gemeinsame Basisklasse `HistoryEntryBase` (IsActive) für die Einträge.
  - Die Uhrzeit ist jetzt sekundengenau statt nur Stunde:Minute.
  - Ein Export nach dem Laden eines Historien-Eintrags verwendete in Port-Scan, Ping und Traceroute bisher den Host des zuletzt gestarteten Laufs im Dateinamen und PDF-Kopf; jetzt den des angezeigten Eintrags.
- **Netzwerk & Verbindungen**: Titelzeile und Registerkarten waren vertauscht, weil jeder eingebettete Bereich seine Titelzeile unterhalb der gemeinsamen Registerleiste mitbrachte. Jetzt steht die Titelzeile des aktiven Bereichs oben (mit seinen Aktionen wie Kopieren/Exportieren/Aktualisieren), darunter die Registerkarten und dann der Inhalt; beim Tab-Wechsel wird die Titelzeile mitgewechselt. Die Einzeldialoge selbst sind unverändert. "Bericht exportieren" in der Registerleiste nutzt jetzt den zentralen Button-Stil statt eines Inline-Templates.
- **Export-Buttons**: In UEFI-Boot-Zertifikat, Autorun-Analyse, Gruppenrichtlinien, Software und Ereignislog war der Text "Exportieren" nach unten verschoben und nur halb sichtbar. Ursache war ein lokales Padding von 8 px oben/unten bei fester Höhe 30 px plus Emoji.
- **Verbindungen**: Der Hauptteil des Export-Split-Buttons hatte kein Menü, ein Klick darauf blieb ohne Wirkung. Er ist jetzt ein einzelner "Exportieren ▾"-Button mit Formatmenü. Auch der Pfeil neben "Aktualisieren" öffnete sein Optionsmenü nur per Rechtsklick; jetzt reicht ein Linksklick.
- **Navigation**: ↑/↓/Enter/Leertaste wurden fensterweit abgefangen – bei aufgeklappter Gruppe wechselte z. B. ↓ in einer Tabelle oder Auswahlliste eines Dialogs die Seite. Die Tasten wirken jetzt nur noch, wenn der Fokus in der Navigation liegt.
- **Navigation**: Die Suche zeigte Treffer in zugeklappten Gruppen nicht an (nur die Gruppenüberschrift); Gruppen mit Treffern klappen während der Suche jetzt auf, Gruppennamen werden mit durchsucht, bei null Treffern erscheint "Keine Treffer". Esc im Suchfeld war nicht verdrahtet.
- **Navigation**: Hover-Effekt war unsichtbar (Farbe `#00000010` = Alpha 0, vollständig transparent).
- **Navigation**: Update-Hinweis zeigte nie die Versionsnummer – der Text wurde per `Template.FindName` gesucht, lag aber im Content (Ergebnis immer `null`).
- **Navigation**: "Übersicht" war beim Start nicht als aktive Seite markiert (Seite wurde geladen, aber der Eintrag nicht ausgewählt).
- **SystemOverviewDialog**: Buttons "Netzwerkzone in Windows anzeigen/ändern" und "Energiebericht erstellen" hatten keinen Klick-Handler und taten nichts. Netzwerkzone öffnet jetzt die Windows-Netzwerkeinstellungen (`ms-settings:network-status`), Energiebericht ist mit `BtnSleepStudy_Click` verbunden.
- **SystemOverviewDialog**: Sicherheits-Chip zeigte während des Ladens kurz fälschlich "Keine Firewall", weil AV- und Firewall-Ergebnis nacheinander eintreffen; noch nicht ermittelte Werte gelten jetzt nicht als Problem.
- **SystemOverviewDialog**: Antivirus-Aktivstatus wurde aus dem falschen Byte von `productState` gelesen (Produkttyp statt Echtzeitstatus) – abgeschaltete Virenscanner, auch ein deaktivierter Defender, wurden als "Aktiv" angezeigt. Ausgewertet wird jetzt das Echtzeit-Nibble (Bits 12–15); verifiziert an einem G-DATA-Client (266240/270336 aktiv, Defender 393472 passiv).
- **SystemOverviewDialog**: Abfrage über die offizielle WSC-API (`IWSCProductList`) war wirkungslos (`Initialize(0)` statt Provider 4, Zustand als Text statt Enum, späte Bindung scheitert mangels registrierter Typbibliothek mit `TYPE_E_LIBNOTREGISTERED`). Jetzt über fest deklarierte COM-Interfaces, liefert korrekte Ergebnisse.
- **SystemOverviewDialog**: Registry-Fallback meldete lediglich installierte AV-Software als "Aktiv" und erzeugte Fehltreffer (z. B. Dongle-Software "Sentinel LDK", "Reset" → "eset", Hersteller-VPNs/-Browser). Treffer jetzt mit Wortgrenzen und Ausschlussliste, Status "Unbekannt · installiert".
- **SystemOverviewDialog**: Bei nicht erreichbarem Netzlaufwerk referenzierte der "Erneut versuchen"-Button den Style `BtnIpAction`, der nur in anderen Dialogen definiert ist (`ResourceReferenceKeyNotFoundException`). Nutzt jetzt den vorhandenen `ActionIconBtn`.
- **Troubleshooting**: eingebetteter Windows-Komponenten-Check ließ sich per Touch/Touchpad nicht scrollen – verschachtelter innerer `ScrollViewer` fing die Scrollgesten ab, bevor sie den äußeren ScrollViewer des Troubleshooting-Dialogs erreichten. Da der Dialog nur noch eingebettet verwendet wird, wurde der innere ScrollViewer entfernt.
- **SystemHealthDialog**: Start des Health-Checks wurde auf Systemen mit Malwarebytes als heuristisch erkanntes Trojaner-/Dropper-Verhalten eingestuft und die App beendet. Ursache: `RunPsAsync` startete PowerShell-Skripte per `-EncodedCommand` (Base64, ein stark gewichtetes AV-Heuristik-Merkmal) und der Scan feuerte beim Start ~10 dieser versteckten PowerShell-Kindprozesse gleichzeitig per `Task.WhenAll`. Skripte laufen jetzt als temporäre `.ps1`-Datei (`-File` statt `-EncodedCommand`), zusätzlich auf maximal 3 gleichzeitige PowerShell-Prozesse gedrosselt (`SemaphoreSlim`)

---

## [0.99.11.13] - 2026-09-29
### Fixed
- **SystemOverviewDialog**: Die Erkennung der CPU Cores unter virtualisierten Instanzen wurde korrigiert. Die Anzahl der vCores ist jetzt korrekt

---

## [0.99.11.10] - 2026-09-15

### Changed
- **SMTP-Test**: Die automatische SMTP Servererkennung für kundenbezogene kasserver.com Mailserver und Verweis auf smtp.all-inkl.com als Mailserver entfernt, da nur noch der kundenbezogene xxx.kasserver.com Port 465 als SMTP Server gültig ist.

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
