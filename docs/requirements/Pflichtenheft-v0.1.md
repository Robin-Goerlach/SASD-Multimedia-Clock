# SASD Multimedia Clock – Pflichtenheft

**Dokument-ID:** SASD-MC-PH-001  
**Version:** 0.1 (technischer Entwurf)  
**Stand:** 09.11.2026  
**Status:** Zur technischen Prüfung; **keine Implementierungs- oder Releasefreigabe**  
**Bezugsdokument:** `SASD_Multimedia_Clock_Lastenheft_V0.1.md` (08.11.2026)  
**Weiteres Referenzdokument:** `SASD_Multimedia_Uhr_AppStore_Funktionsrecherche.md`

> **Dokumentcharakter:** Das Lastenheft beschreibt, *was* benötigt wird; dieses Pflichtenheft schlägt vor, *wie* es umgesetzt, nachgewiesen und abgenommen werden soll. Bereits beschlossene Produktziele werden übernommen. Architektur, Termine, Messgrenzen und Umfang einzelner Releases sind **Vorschläge**, bis die offenen Entscheidungen bestätigt sind. Insbesondere wird kein zuverlässiges Wecken per beliebigem Streamingmedium bei gesperrter/beendeter iOS-App zugesagt.

## 0. Leseschlüssel, Verbindlichkeit und Änderungsmanagement

- **B – beschlossen:** Produktentscheidung aus dem Lastenheft; nur durch bewusste Entscheidung änderbar.
- **M – Muss V1 (vorgeschlagen):** vor Release zu implementieren oder formal aus V1 auszuklammern.
- **S – Soll:** erwünscht, ggf. nach V1 verschiebbar.
- **F – Forschungs-/Machbarkeitsauftrag:** **keine** zugesagte Funktion.
- **K – Kann/Backlog:** spätere Ausbaustufe.
- **X – ausgeschlossen:** ausdrücklich nicht Bestandteil der angegebenen Version.
- **GATE:** objektiv prüfbare Freigabebedingung.

**Normative Quellen:** Die IDs aus dem Lastenheft (`PZ-*`, `CLK-*`, `ALM-*` usw.) bleiben erhalten. Technische Umsetzungsentscheidungen dieses Dokuments tragen `DES-*`, Schnittstellen `API-*`, Datenobjekte `DATA-*`, Tests `T-*`, Risiken `R-*` und Entscheidungen `ADR-*`. Eine Abweichung wird mit Begründung, Konsequenzen, Verantwortlichem, Datum und Entscheidung dokumentiert. Keine stillschweigende Aufwertung von S/F/K zu M.

## 1. Ziel, Grenzen und Systemkontext

### 1.1 Produktziel und Nutzwert

**SASD Multimedia Clock** soll ältere iPhones und iPads als Uhr, Radiowecker, Medienplayer oder Informations-Dashboard sinnvoll weiterverwendbar machen. **Das ist ein ausdrückliches zentrales Produktziel.** Ein und dieselbe App bietet konfigurierbare Nutzungsprofile **Nachttisch**, **Schreibtisch** und **Multimedia**; sie ist keine Sammlung separater Apps. Die Einschlafhilfe ist optional und standardmäßig deaktiviert. Das Aussehen ist durch Skins variierbar, ohne Medien-, Alarm- und Konfigurationslogik zu verändern.

### 1.2 Plattformumfang

| Bereich | Festlegung | Verifikation |
|---|---|---|
| Zielgeräte | iPhone und iPad gleichberechtigt | Gerätematrix, Hoch-/Querformat |
| Mindestbetriebssystem | **Ziel iOS/iPadOS 16.0**, unter dokumentiertem Aufwandvorbehalt | Builds und Tests auf 16.x |
| Erste Implementierung | Swift + SwiftUI, native Apple-App | Xcode-Projekt, Testflight-Build |
| Media-Engine | AVFoundation/AVPlayer und unterstützte System-APIs | Format-/Zustandsmatrix |
| Alarm | OS-Funktionen, unter iOS 16 lokale Benachrichtigungen; neuere APIs nur mit Availability-Gate | Weck-/Berechtigungsmatrix |
| Zukunft | Android, Windows, Linux, macOS | Portabilitätsgrenzen dokumentiert, **nicht V1** |
| Gemeinsamer C++20-Core | **Option**, erst nach Spike/ADR | Prototyp, Kosten-Nutzen-Bewertung |

**Werkzeuggrenze:** Ein älterer Intel-Mac mit Big Sur kann für frühe Swift-Prototypen geeignet sein, ist aber keine verlässliche Basis für aktuelle App-Store-Einreichungen. Für Release-Builds sind die zum Einreichungszeitpunkt gültigen Apple-Toolchain-Anforderungen zu verifizieren; konkrete Xcode-Version und Mac-Hardware werden im Release-ADR festgelegt. Ein altes Xcode-SDK kann neue iOS-APIs nicht durch bloße `#available`-Prüfung bereitstellen.

### 1.3 Systemgrenze und externe Akteure

```text
Nutzer / iPhone / iPad
         │
         ▼
┌────────────────────────────────────────────────────┐
│ SASD Multimedia Clock (App-Sandbox)                │
│ SwiftUI UI + Profile + Skin Renderer + Scheduler   │
│ Media Catalog + Player Coordinator + Alarm Adapter│
│ Local Storage + Permission/Capability Registry    │
└───────┬────────────┬────────────┬────────────┬──────┘
        │            │            │            │
  AVFoundation   UserNotifications  Dateien    EventKit / Wetter*
        │            │            │            │
  URL-Streams,    iOS-System-     lokale      nur nach Freigabe
  lokale Medien   benachrichtig.  Medien

* Wetterdienst und weitere Integrationen erst nach separater Entscheidung.
```

**Nicht Bestandteil V1:** eigener Cloud-Account, Backendpflicht, DRM-Umgehung, ungeprüfte Einbettung kommerzieller Streamingdienste, garantierter automatischer TV-Start aus gesperrtem/beendetem Zustand, medizinische Schlafdiagnostik, Android-/Desktop-Release.

## 2. Release-Schnitt und Prioritäten

### 2.1 P0 – technische Machbarkeit (kein verkaufsfähiges Produkt)

1. SwiftUI-App auf iPhone und iPad mit iOS-16-Deployment-Target kompilieren.
2. Große Digitaluhr mit lokaler Zeit, Rotation und Nachtmodus.
3. AVPlayer mit direktem MP3-Stream (SWR3-Test-URL), Fehleranzeige und Stop.
4. Lokale Audiodatei via Dokumentenauswahl importieren und offline abspielen.
5. Lokale Weckbenachrichtigung anlegen, Status anzeigen, Testalarm durchführen.
6. Verhaltensmatrix für Vordergrund, Hintergrund, gesperrt, beendet, Neustart und Audio-Unterbrechung protokollieren.
7. Skinwechsel zwischen Retro und modernem Layout demonstrieren.
8. Auf mindestens einem realen älteren iPhone und iPad testen; Simulator allein genügt nicht.

**P0-Abnahme:** Jeder Spike hat einen reproduzierbaren Testbericht. Unmögliche Szenarien werden als Grenzen dokumentiert, nicht simuliert oder als erfolgreich ausgegeben.

### 2.2 V1.0 – vorgeschlagener verbindlicher Funktionsumfang

- Uhr digital/analog, Datum, Nachtmodus, Profilwechsel, adaptive Darstellung.
- Mindestens zwei mitgelieferte Skins; sichere, deklarative Skin-Konfiguration.
- Mehrere konfigurierbare Alarme; OS-basierte Benachrichtigung, Berechtigungsprüfung, Testalarm, ehrliche Statusanzeige; kein Versprechen der Gleichwertigkeit zum Systemwecker.
- Direkte unterstützte Audio-URLs (MP3 verbindlich, AAC/HLS nach Spike), Favoriten, Fehlerdiagnose, lokale Audio-Dateien und Offlinebetrieb.
- Lokale unterstützte Videodateien im aktiven App-Zustand, einfache Wiedergabesteuerung.
- Countdown, konfigurierbare Grundansichten und lokale persistente Einstellungen.
- Einschlafhilfe **deaktiviert**, bis ausdrücklich eingeschaltet; V1 darf bei aktiviertem Modul eine einfache Zeitabschaltung enthalten, wenn separat abgenommen.
- Deutsch/Englisch, VoiceOver-Basis, keine Registrierung, keine Pflicht-Cloud, transparente Berechtigungen.

**Scope-Disziplin:** Podcast-Feeds, Wetter, Kalender, Weltzeit, Routinen, AirPlay-Komfort, Skin-Import, TV-Integrationen und Smart Home bleiben zunächst S/F/K, auch wenn Mockups sie zeigen. Ein Mockup ist kein Liefernachweis.

### 2.3 V1.x / V2+

| Version | Kandidaten, nicht zugesagt | Voraussetzung |
|---|---|---|
| V1.x | Podcast-RSS, Downloads, Wiedergabeposition, Playlistimport, erweiterte Weckprofile | Daten-/Sicherheitsprüfung |
| V1.x | Kalender, Weltzeit, Wetter, Kacheleditor | Berechtigungen, Anbieter-/Kostenentscheidung |
| V1.x | Geplante Routinen **im aktiven Vordergrund** | Zustandsmatrix und Konfliktregeln |
| V2+ | Deklarativer Skin-Import/Export, Skin-Editor, Community-Skins | Formatversionierung und Sandboxing |
| V2+ | Apple Music und weitere offizielle Dienste | APIs, Rechte, Testgeräte |
| V2+ | Android/Desktop und evtl. gemeinsamer C++-Core | ADR zur Portierungsstrategie |

## 3. Architektur und Modulgrenzen

### 3.1 Architekturprinzipien

1. **Native-first, ports-aware:** SwiftUI/Swift zuerst; keine frühe C++-Pflicht.
2. **Trennung von UI und Geschäftslogik:** Views halten keine Streaming-, Scheduler- oder Persistenzlogik.
3. **Capability-first:** Funktionen hängen von OS-Version, Berechtigungen, App-Zustand und Medienquelle ab.
4. **Fehler sind sichtbare Zustände:** insbesondere Alarm-/Netzwerkfehler dürfen nicht stillschweigend verschluckt werden.
5. **Offline-first für lokale Funktionen**, nicht für Live-Streams.
6. **Keine versteckte Hintergrundautomatisierung**; nur dokumentierte, erlaubte iOS-Mechanismen.
7. **Datenminimierung:** keine verpflichtende Anmeldung, keine unnötige Telemetrie.
8. **Testbare Ports/Adapter:** Uhrzeit, Medien, Benachrichtigungen, Netzwerk und Dateizugriff sind abstrahiert.

### 3.2 Komponenten

| Komponente | Verantwortung | Abhängigkeiten / Grenzen |
|---|---|---|
| `ClockDomain` | Zeitformate, Zeitzonen, Tick-Planung, Anzeigezustand | injizierbare Clock, Foundation |
| `ProfileManager` | aktive Profile, Defaults, Overrides | Konfigurationsspeicher |
| `SkinRenderer` | Layout, Farben, Assets, Accessibility-Fallback | SwiftUI; kein ausführbarer Skin-Code |
| `MediaCatalog` | Quellen, Favoriten, zuletzt genutzt, Typen | Repository, URL-Validator |
| `MediaResolver` | URL-/Dateityp, Redirect-/Playlistprüfung | URLSession; kein DRM-Bypass |
| `PlayerCoordinator` | Play/Pause/Stop, Medienstatus, Audio-Session | AVFoundation, OS-Interruptions |
| `AlarmRepository` | Alarmdefinitionen und Änderungszustände | lokaler Store |
| `AlarmService` | Planen/Entfernen von OS-Benachrichtigungen | UserNotifications; neuere Adapter separat |
| `RoutineEngine` | zeitgesteuerte Aktionen, Konflikte, Protokoll | **nach P0**, nur erlaubte Zustände |
| `SleepModule` | opt-in Einschlafprogramm | Player, Display; standardmäßig aus |
| `CapabilityRegistry` | iOS-Version, Permissions, Medien-/App-Zustand | Plattformadapter |
| `PermissionCoordinator` | kontextbezogene Einwilligung und Erklärung | iOS APIs |
| `SettingsStore` | atomare, versionierte Einstellungen | UserDefaults/Dateien/Keychain |
| `Diagnostics` | lokale, datensparsame Diagnose | kein Upload ohne ausdrückliche Zustimmung |

### 3.3 Vorgeschlagene Repository-Struktur

```text
SASD-Multimedia-Clock/
├── README.md
├── LICENSE
├── AGENTS.md
├── docs/
│   ├── requirements/        # Lastenheft, Pflichtenheft, Traceability
│   ├── architecture/        # ADRs, Komponenten, Schnittstellen
│   ├── testing/             # Testpläne, Geräte-/Zustandsmatrix
│   ├── security/            # Threat Model, Privacy, Rechte
│   └── releases/            # Gates, Nachweise, Release Notes
├── apps/
│   └── Apple/
│       ├── MultimediaClock.xcodeproj
│       ├── App/
│       ├── Features/
│       │   ├── Clock/
│       │   ├── Alarm/
│       │   ├── Media/
│       │   ├── Profiles/
│       │   ├── Skins/
│       │   ├── Timer/
│       │   └── Sleep/
│       ├── Platform/
│       ├── Resources/
│       └── Tests/
├── packages/
│   └── Core/                # Swift Package: testbare Domain-Logik
├── test-assets/             # nur lizenzierte, kleine Testmedien
└── tools/                   # Build-/Validierungsskripte
```

**Entscheidungsvorbehalt:** Repo-Name, Lizenz, Bundle-ID, Team-ID, Signing und endgültige Ordnerstruktur werden erst im Projekt-Kickoff festgelegt. Ein C++-Verzeichnis wird **nicht** vorsorglich angelegt.

### 3.4 Technologiewahl und Grenzen

| Problem | Vorgeschlagene Lösung | Alternative / Risiko |
|---|---|---|
| UI | SwiftUI mit NavigationStack / iPad-adaptiver Navigation | UIKit-Brücken nur bei Bedarf |
| Medien | AVPlayer, AVAudioSession, AVAsset | Formate/DRM/Audio-Sessions einzeln prüfen |
| Dateien | SwiftUI fileImporter, Security-Scoped-URLs, lokale Kopie nach Nutzerentscheidung | Berechtigungs-/Bookmark-Lebensdauer |
| HTTP | URLSession, ATS-konform, TLS-Prüfung | HTTP-Ausnahmen nur eng begrenzt |
| Persistenz | Codable + versionierte JSON-Datei bzw. UserDefaults für einfache Settings | SwiftData nicht als Voraussetzung für iOS 16 |
| Geheimnisse | Keychain | niemals im Skin-/Export-JSON |
| Benachrichtigungen | UNUserNotificationCenter | kein vollwertiger Systemalarm auf iOS 16 |
| Neuere Wecker-API | separat kompilierter, versionsgeschützter Adapter | SDK-Verfügbarkeit und Verhaltensprüfung |
| Kalender | EventKit (später) | opt-in, kein Pflichtzugriff |
| Tests | XCTest, UI-Tests, reale Geräte | Simulator nicht ausreichend für Weck-/Audiozustände |

## 4. Detaillierte Umsetzung der funktionalen Anforderungen

### 4.1 Uhr (`CLK-*`)

**DES-CLK-01:** `ClockDomain` liefert `Date`/Zeitzoneninformationen; UI formatiert mittels `Date.FormatStyle`/Calendar/Locale. Keine selbst berechnete Sommerzeit und keine hartcodierten UTC-Offsets. 12-/24-Stundenformat ist pro Profil überschreibbar. Sekunden werden nur gerendert, wenn sichtbar.

**DES-CLK-02:** Aktualisierung auf sichtbare Präzision ausrichten (Sekunde oder Minute), bei App-Rückkehr sofort Systemzeit neu lesen. Keine sekündlichen Hintergrund-Timer als Weckmechanismus. Nach Zeit-/Zeitzonenänderung und Mitternacht Anzeige neu berechnen.

**DES-CLK-03:** Analoguhr nutzt denselben Zeitwert wie Digitaluhr; visuelle Animation ist optional und bei `Reduce Motion` reduziert. Zeit-/Datumsanzeige ist semantisch für VoiceOver beschriftet.

**DES-CLK-04:** Nachtmodus: manueller Schalter, optional lokales Zeitfenster, Profilvorgabe. Die App darf im Vordergrund `idleTimerDisabled` verwenden; das garantiert keinen dauerhaften Betrieb nach Sperre, Beenden oder Reboot. Bildschirmhelligkeit nur im Rahmen öffentlicher APIs steuern, vorherigen Zustand nach Möglichkeit wiederherstellen; kein Anspruch auf vollständige Systemhelligkeitskontrolle.

**DES-CLK-05:** OLED-Schonung als optionale Dimmung/Positionsvariation; ausdrücklich **keine** Garantie gegen Einbrennen. Energie-/Wärmehinweise in Einstellungen.

**T-CLK:** Zeit um 23:59:59, Monats-/Jahreswechsel, DST vor/zurück, 12/24h, VoiceOver, Rotation, App-Resume, große Schrift und altes Gerät.

### 4.2 Wecker (`ALM-*`) – sicherheitskritische Produktfunktion

**DES-ALM-01 Datenmodell:** Jeder Alarm besitzt UUID, lokale Uhrzeit, Wochentage/Einmaltermin, Name, Aktivierung, Tonreferenz, Snooze-Option, Zeitzonenpolitik, Erstell-/Änderungszeit, Planungsstatus und letzten Fehler. Eine Alarmdefinition ist **nicht** identisch mit einem erfolgreich registrierten OS-Alarm.

**DES-ALM-02 Zustandsautomat:** `draft → validation → scheduling → scheduled | degraded | failed → triggered/expired → acknowledged`; Änderungen und Löschen synchronisieren den OS-Plan. Nach App-Start werden gewünschte und tatsächlich registrierte Anfragen abgeglichen, soweit die API dies ermöglicht. Bei Planungsfehlern kein grünes „aktiv“-Symbol.

**DES-ALM-03 iOS 16:** Lokale Benachrichtigungen werden nach ausdrücklicher Berechtigung geplant. Die App bezeichnet dies als **Benachrichtigungsalarm**, nicht als garantierten, systemweckergleichen Alarm. Lautlos-/Fokusmodus, Berechtigungen, Töne, Beenden, Neustart und OS-Verhalten können die wahrgenommene Alarmwirkung beeinflussen. Für kritisches Aufwachen wird der integrierte iOS-Wecker empfohlen. Keine Verwendung von Critical Alerts ohne besondere Apple-Berechtigung.

**DES-ALM-04 neuere iOS-Versionen:** Eigener Adapter für verfügbare neuere Alarm-APIs nach SDK-/Entitlement-Prüfung. `#available` schützt Laufzeitpfade, ersetzt aber keine passende Build-Toolchain. UX unterscheidet tatsächliche Garantien und Einschränkungen pro Gerät.

**DES-ALM-05 Interaktion:** Große getrennte Aktionen **Stopp** und **Schlummern** in aktiver App; außerhalb der App nur durch OS erlaubte Interaktion. Ein Snooze wird als eigener neuer Planungsauftrag mit sichtbarem Ergebnis behandelt. Keine Behauptung, die App könne iOS-Sperrbildschirm oder fremde Apps selbständig entsperren.

**DES-ALM-06 Testalarm:** Ein kurzfristiger Test zeigt (a) Berechtigung, (b) registrierte Anfrage, (c) tatsächliche Auslösung und (d) Medienverhalten getrennt. Fehlende Benachrichtigungsberechtigung führt zu Erklärung und Link in Systemeinstellungen, nicht zu einem irreführenden „Erfolg“.

**DES-ALM-07 Medienwunsch:** Radio/Musik/Video sind optionale **Folgeaktionen**, niemals Ersatz für den OS-basierten Alarm. Bei nicht sichtbarer App wird kein automatischer Streamstart versprochen. Kein stiller Fallback, der eine fehlgeschlagene Weckfunktion kaschiert.

**T-ALM:** Siehe Zustandsmatrix in §9; kein Release ohne manuelle Tests auf realen Geräten und dokumentierte Einschränkungen.

### 4.3 Medienquellen und Internetradio (`AUD-*`)

**DES-AUD-01:** Nutzer fügt URL, Anzeigename, optional Kategorie und Favoritenstatus hinzu. Validierung trennt Syntax, Sicherheitsprüfung, Erreichbarkeit und **tatsächliche Dekodierbarkeit**. Eine erfolgreiche HTTP-Antwort beweist keine abspielbare Quelle.

**DES-AUD-02:** Zunächst direkte HTTPS-MP3-Streams. Referenzfall: `https://liveradio.swr.de/sw282p3/swr3/play.mp3`. Stream-Erreichbarkeit kann sich ändern; Testprotokoll enthält Zeitpunkt und Ergebnis. AAC, HLS, M3U und PLS erst nach Format-/Parser-Test als unterstützt kennzeichnen.

**DES-AUD-03:** `MediaResolver` normalisiert URL, begrenzt Redirects/Antwortgrößen/Timeouts, erkennt bekannte Playlisttypen und verhindert rekursive Playlist-Schleifen. Unbekannte Inhalte werden nicht als HTML in der App ausgeführt. Zugangsdaten werden weder in Logs noch in Exporten unmaskiert gespeichert.

**DES-AUD-04:** `PlayerCoordinator` hat Zustände `idle`, `resolving`, `buffering`, `playing`, `paused`, `stopped`, `ended`, `failed`. UI zeigt Ladezustand, Quelle und konkrete Fehlerkategorie; kein unendlicher Spinner. Wiederversuche mit begrenztem exponentiellem Backoff nur nach ausdrücklicher Nutzer-/Featureentscheidung.

**DES-AUD-05:** Audio-Session-Kategorie, Hintergrund-Audio und Fernbedienbarkeit werden im P0-Spike auf realen Geräten ermittelt. Nur nach Apple-konformer Implementierung wird eine Background-Mode-Berechtigung aktiviert. Hintergrund-Audio ist **kein** genereller Mechanismus zum zeitgesteuerten Start neuer Streams.

**DES-AUD-06:** Lautstärke und externe Ausgabegeräte respektieren iOS-Systemsteuerung; keine vorgetäuschte globale Lautstärkekontrolle. Favoriten und letzte Quellen werden lokal persistiert; Titelmetadaten sind optional und dürfen fehlen.

**T-AUD:** SWR3-Referenzstream, absichtlich ungültige URL, DNS-Fehler, TLS-Fehler, HTTP 404/403, leere Antwort, Redirect-Kette, HTML statt Audio, Streamabbruch, Mobilfunk/WLAN-Wechsel, Audio-Interruption, Bluetooth/AirPlay soweit verfügbar.

### 4.4 Lokale Musik und Dateien (`MUS-*`)

**DES-MUS-01:** Auswahl über System-Dateiauswahl. Für dauerhafte Verfügbarkeit wahlweise kontrollierte Kopie in den App-Container; keine Annahme, dass eine externe Provider-URL dauerhaft gültig bleibt. Kopien werden mit Größe und Speicherverbrauch angezeigt und sind löschbar.

**DES-MUS-02:** Unterstützte Formate werden über echte Wiedergabetests festgelegt. Metadaten und Cover sind optional; beschädigte Dateien dürfen keinen Crash verursachen. Lokale Medien sind ohne Internet abspielbar, sofern nicht DRM-/Berechtigungsregeln entgegenstehen.

**DES-MUS-03:** Apple-Music-Integration ist **F**: MusicKit, Autorisierung, Abonnement, DRM, lokale Downloads und erlaubte Playback-Pfade separat prüfen. Keine direkte, allgemeine Zugriffszusage auf „alle iTunes-Dateien“.

### 4.5 Podcasts und Hörbücher (`POD-*`)

**DES-POD-01 [S]:** RSS/Atom-kompatiblen Podcast-Feed nur über sicheren Parser verarbeiten (Größenlimit, Timeouts, keine externen XML-Entities); Episoden-URL und Metadaten extrahieren. Download nur nach Nutzerentscheidung oder konfigurierter Regel; Speicherlimit und Löschfunktion.

**DES-POD-02 [S]:** Wiedergabeposition pro Episode lokal speichern; beim Wiederöffnen fortsetzen. Offline-Verfügbarkeit explizit kennzeichnen. Kapitel, Geschwindigkeit und Hörbuchbibliothek bleiben K.

### 4.6 Video und TV (`VID-*`)

**DES-VID-01:** Lokale unterstützte Video-Dateien und direkte, erlaubte Video-URLs über AVPlayer in **sichtbarer aktiver App**. Fullscreen, Pause, Seek und Seitenverhältnis. Kein Download geschützter Inhalte und keine DRM-Umgehung.

**DES-VID-02:** Geplanter Start einer Lieblings-TV-Sendung wird als **Forschungsfall** mit Quellentyp, Startzeit, App-Zustand und Nutzerinteraktion modelliert. Der P0-Spike prüft: App sichtbar, App im Hintergrund, Gerät gesperrt, App beendet, Gerät neu gestartet, Netzwerk ausgefallen. Ein Deep Link oder eine Benachrichtigung mit Nutzeraktion ist **nicht** gleichbedeutend mit automatischer Wiedergabe.

**DES-VID-03:** YouTube, TV-Mediatheken und kommerzielle Dienste werden **anbieterweise** auf offizielle SDKs, Einbettungsrechte, Autoplay, DRM und Login geprüft. „Beliebige Video-URL“ ist keine pauschale Unterstützung von Webseiten oder Abodiensten.

### 4.7 Einschlafhilfe (`SLP-*`)

**DES-SLP-01:** Feature-Flag `sleep.enabled=false` bei Erstinstallation. Die Einstellungsseite erklärt den Funktionsumfang. Das Einschalten zeigt Optionen, startet aber **keine** Medienwiedergabe.

**DES-SLP-02:** Separater expliziter Start einer Session mit Dauer, Quelle und optionaler Lautstärke-/Display-Abblendung. Countdown verwendet monotone Zeit für Laufzeitberechnung, soweit möglich; beim App-Resume wird der verbleibende Zustand abgeglichen. Hintergrundverhalten ist auf erlaubte iOS-Mechanismen begrenzt.

**DES-SLP-03:** Stop, Abbruch und Ablauf sind idempotent; eine abgebrochene Session darf später nicht unbemerkt neu starten. Das Modul fordert keine zusätzlichen Berechtigungen, solange es deaktiviert ist.

### 4.8 Timer, Weltzeit und Produktivität (`TMR-*`, `WLD-*`, `PRD-*`)

**DES-TMR-01:** Countdown-Zustände `ready/running/paused/completed/cancelled`; Dauer validieren, Null/Negativwerte zurückweisen, nach Unterbrechung aus Zeitstempel rekonstruieren. Ende-Signal nur im erlaubten App-/OS-Kontext. Stoppuhr und mehrere Timer S.

**DES-WLD-01 [S]:** Weltzeit aus IANA-Zeitzonen, keine manuell fixierten UTC-Differenzen. Geschäftszeiten und internationale Meetingplanung K.

**DES-PRD-01 [K]:** Aufgaben und Schnellzugriffe werden als optionale Datenprovider implementiert, nicht als verpflichtende Kernmodule.

### 4.9 Schreibtisch-Dashboard (`DSH-*`)

**DES-DSH-01:** Raster mit Kacheltypen `clock`, `date`, `media`, `nextAlarm` für V1; Erweiterungen `calendar`, `weather`, `worldClock`, `deviceStatus`, `news`, `tasks` später. Jede Kachel hat Größe, Sichtbarkeit, Konfiguration und Fallback für fehlende Daten.

**DES-DSH-02:** iPhone zeigt vereinfachte vertikale Anordnung, iPad mehrere Spalten. Keine horizontal abgeschnittenen kritischen Aktionen. Beispieltermine, Nachrichten und Wetter aus Mockups sind **nur Demo-Daten**; reale Integrationen benötigen Nutzerberechtigungen und Datenanbieter.

**DES-DSH-03 [S]:** Kalender via EventKit nach ausdrücklicher Freigabe, Wetter über separat ausgewählten Anbieter, Standort manuell ohne Pflicht-Geolocation. Daten zeigen Aktualisierungszeitpunkt und Offline-/Fehlerstatus.

### 4.10 Skins und Gestaltung (`SKN-*`)

**DES-SKN-01:** Skin beschreibt **nur Daten**: Farbpalette, Typografie-Token, Assets, Komponentenanordnung, Abstände, optionale Animationen und Anzeigevarianten. Kein JavaScript, Swift-Code, dynamisches Laden ausführbarer Bibliotheken oder unbeschränkter Dateizugriff.

**DES-SKN-02:** V1 liefert mindestens `retro-radio` und `modern-dashboard`. Der Skinwechsel ersetzt nur Präsentationszustand; Alarm-/Medien-/Profilmodelle bleiben erhalten. Ein beschädigter Skin fällt auf einen eingebauten Standardskin zurück.

**DES-SKN-03:** Design-Tokens für Kontrast, Dynamic Type und VoiceOver-Beschriftungen. Ein Skin darf die Alarm-Stopp-/Snooze-Funktion und systemrelevante Hinweise nicht unsichtbar machen. Fremde Winamp-Grafiken/Marken nur bei geklärten Rechten.

**DES-SKN-04 [K/F]:** Eigenes versioniertes JSON-Schema für Import/Export nach Sicherheitsprüfung (Größenlimits, Assettypen, Pfadnormalisierung, Signatur/Prüfsumme optional). Kein verbindlicher Winamp-Skin-Parser.

### 4.11 Zeitprogramme und Automatisierung (`AUT-*`)

**DES-AUT-01 [S]:** Routine = Trigger (lokale Zeit/Wochentag) + optionale Bedingungen + geordnete Aktionen + Aktivierung + Fehlerrichtlinie. Jede Aktion ist capability-geprüft. Ausführung nur, wenn der tatsächliche App-/OS-Zustand sie erlaubt.

**DES-AUT-02:** Prioritätsvorschlag: manueller Nutzerbefehl > aktiver Alarm > explizite Schlafsession > Routine > passive Anzeige. Konflikte führen zu nachvollziehbarer Entscheidung; Routine darf einen aktiven Alarm nicht stillschweigend stoppen.

**DES-AUT-03:** Ausführungsprotokoll mit `planned`, `attempted`, `succeeded`, `skipped`, `failed`, Zeitstempel und Grund. „Geplant“ ist nicht „ausgeführt“. Keine automatische Nachholung eines verpassten Videostarts ohne Nutzerentscheidung.

**DES-AUT-04 [M, falls Routinen aktiviert]:** Jede Routine separat deaktivierbar; beim Anlegen klare Erklärung von Hintergrund-/Sperrgrenzen. Ein Testlauf führt nur explizit bestätigte Medienaktionen aus.

## 5. Datenmodell und Persistenz

### 5.1 Datenobjekte (konzeptionell)

| ID | Objekt | Pflichtfelder | Hinweise |
|---|---|---|---|
| DATA-01 | `AppSettings` | schemaVersion, locale, defaultProfileId, privacyFlags | atomar versioniert |
| DATA-02 | `UsageProfile` | id, name, kind, skinId, widgetLayout, displayOptions | `bedside/desk/media/custom` |
| DATA-03 | `SkinDefinition` | id, schemaVersion, tokens, layout, assetRefs | keine ausführbaren Inhalte |
| DATA-04 | `AlarmDefinition` | id, enabled, localTime, recurrence, sound, status | Planungsstatus separat |
| DATA-05 | `MediaSource` | id, kind, title, locator, favorite, metadata | URL/Dateireferenz; keine Geheimnisse |
| DATA-06 | `PlaybackState` | sourceId, position?, state, lastError? | transient, selektiv persistieren |
| DATA-07 | `TimerDefinition` | id, duration, state, startedAt? | monotone Restzeitlogik |
| DATA-08 | `SleepSettings` | enabled=false, duration?, fade?, mediaId? | Startzustand separat |
| DATA-09 | `RoutineDefinition` | id, enabled, trigger, actions, constraints | zunächst optional |
| DATA-10 | `CapabilitySnapshot` | osVersion, permissionStates, appState | laufzeitbezogen, nicht als Wahrheit dauerhaft speichern |
| DATA-11 | `DiagnosticEvent` | time, component, severity, code | keine Secrets/PII |

### 5.2 Persistenzregeln

- Kleine, nicht geheime Einstellungen in `UserDefaults` oder versionierter lokaler JSON-Konfiguration; größere Medienkataloge ggf. SQLite nach ADR. **Kein Zwang zu SwiftData**, da iOS 16 Ziel ist.
- Schreibvorgänge atomar, Schema-Version vorhanden, Migrationen für jede Breaking-Änderung, Backups/Recovery bei beschädigter Konfiguration.
- Dateizugriffe innerhalb App-Sandbox; Security-Scoped-URLs nur mit korrekt verwaltetem Zugriff; lokale Kopien als eigene Medienbestände kennzeichnen.
- Secrets ausschließlich Keychain; in Exporten standardmäßig ausgeschlossen.
- Favoriten und Einstellungen müssen nach App-Neustart erhalten bleiben; Löschung und Zurücksetzen getrennt anbieten.
- Export/Import erst nach eigener Sicherheits- und Schemaentscheidung; keine ungeprüften externen Dateien ausführen.

### 5.3 Beispiel: deklarative Medienquelle (Illustration, noch kein finales Schema)

```json
{
  "schemaVersion": 1,
  "id": "example-swr3",
  "kind": "stream",
  "title": "SWR3",
  "url": "https://liveradio.swr.de/sw282p3/swr3/play.mp3",
  "favorite": true
}
```

### 5.4 Beispiel: Skin-Metadaten (Illustration)

```json
{
  "schemaVersion": 1,
  "id": "retro-radio",
  "name": "Retro Radio",
  "tokens": {"theme": "dark", "clockStyle": "segmented", "accent": "amber"},
  "layout": ["clock", "date", "media", "nextAlarm"]
}
```

**Hinweis:** Diese Beispiele sind keine verbindliche öffentliche API. JSON-Schema, Migrationen und Limits werden vor externem Import spezifiziert.

## 6. Interne Schnittstellen und Verträge

| ID | Schnittstelle | Operationen / Ergebnis | Fehlerklassen |
|---|---|---|---|
| API-01 | `ClockProviding` | `now()`, `timeZone`, `calendar` | Systemzeitänderung |
| API-02 | `MediaResolving` | `resolve(source) -> ResolvedMedia` | invalidURL, denied, unsupported, timeout, network |
| API-03 | `MediaPlaying` | `play`, `pause`, `stop`, `seek?`, `stateStream` | decoder, interruption, sourceUnavailable |
| API-04 | `AlarmScheduling` | `schedule`, `cancel`, `listScheduled`, `capabilities` | denied, unsupported, schedulingFailed |
| API-05 | `ProfileStoring` | `load`, `save`, `delete`, `reset` | invalid, migration, storage |
| API-06 | `SkinLoading` | `load`, `validate`, `apply`, `fallback` | invalidSchema, missingAsset, unsafe |
| API-07 | `CapabilityChecking` | `evaluate(feature, context)` | unavailableOS, permission, backgroundLimit |
| API-08 | `RoutineExecuting` | `validate`, `simulate`, `execute`, `log` | conflict, unavailable, missedTrigger |

**Vertrag:** Jede Operation meldet einen typisierten Erfolg/Fehler; UI übersetzt technische Fehler in verständliche deutsche/englische Hinweise. Schnittstellen dürfen keinen OS-Hintergrundstart versprechen, den die Plattform nicht garantiert.

## 7. UI/UX und Navigationsmodell

### 7.1 Informationsarchitektur

```text
Start / aktives Nutzungsprofil
├── Uhr
├── Medien (Radio / Musik / Video; später Podcasts)
├── Wecker
├── Timer
└── Einstellungen
    ├── Allgemein / Sprache / Profil
    ├── Anzeige & Skins
    ├── Uhr & Zeit
    ├── Wecker & Berechtigungen
    ├── Medienquellen / Radio / Musik / Video
    ├── Einschlafhilfe [standardmäßig AUS]
    ├── Profile & Szenarien
    ├── Netzwerk / Datenschutz
    └── Info / Diagnose
```

### 7.2 Layoutregeln

- **iPhone:** kompakte Navigation, eine dominante Hauptaktion, große Uhr, Mediensteuerung ohne Überladung; Einstellungen als Liste/Detailnavigation.
- **iPad:** adaptive Seitenleiste und Detailbereich; im Schreibtischprofil konfigurierbare Kacheln; Split View/Stage Manager soweit verfügbar testen.
- **Nachttisch:** Uhr dominant, gedimmter Nachtmodus, wenige große Bedienelemente, klarer nächster Alarm.
- **Schreibtisch:** Uhr, Medien, optionale Informationskacheln; keine Pflichtvernetzung.
- **Multimedia:** Medienquelle, Player, Playlist/Favoriten, Zeitprogramme erst nach Freigabe.
- **Skins:** Skin beeinflusst Optik, **nicht** Funktionsrechte, Sicherheitsmeldungen oder Datenintegrität.

### 7.3 Accessibility und Lokalisierung

- Deutsch und Englisch vollständig, inklusive Fehler-/Berechtigungstexte.
- VoiceOver-Reihenfolge, eindeutige Labels, ausreichend große Touch-Ziele (Zielwert ≥44×44 pt), Dynamic Type, Kontrast, reduzierte Bewegung.
- Uhr als semantische Zeitansage; kein permanentes sekündliches VoiceOver-Spam.
- Keine reine Farbcodierung für Alarmstatus oder Fehler; Text und Symbol ergänzen.
- Test mit vergrößerter Schrift, Bildschirmzoom und VoiceOver auf iPhone/iPad.

## 8. Berechtigungen, Sicherheit, Datenschutz und Rechte

### 8.1 Berechtigungsmatrix

| Zugriff | Wann anfordern? | Ablehnung / Fallback |
|---|---|---|
| Benachrichtigungen | Beim Einrichten eines OS-Alarms | Alarm nicht als wirksam markieren; Hinweis |
| Dateien | Bei expliziter Medienauswahl | übrige App nutzbar |
| Musikmediathek | Erst bei Aktivierung der Integration | lokale Dateien/URL bleiben nutzbar |
| Kalender | Erst beim Hinzufügen einer Kalenderkachel | Kachel deaktiviert/Erklärung |
| Standort | Nicht erforderlich für Grundfunktion; Wetter mit manuellem Ort | keine Pflichtortung |
| Lokales Netzwerk | Nur bei entsprechendem Feature/OS-Bedarf | externe LAN-Quelle nicht verfügbar |

### 8.2 Threat Model (erste Fassung)

| Bedrohung | Gegenmaßnahme | Test |
|---|---|---|
| Bösartige Stream-/Playlist-URL | Scheme-Allowlist, Redirect-/Größen-/Timeout-Limits, Parserhärtung | T-SEC-01 |
| URL mit Credentials | Maskierung, Keychain, kein Export von Secrets | T-SEC-02 |
| Nicht vertrauenswürdige Skin-Datei | nur deklarative Daten, Asset- und Pfadvalidierung | T-SEC-03 |
| Unsichere HTTP-Verbindung | HTTPS Standard; Ausnahmen nur per dokumentiertem ADR | T-SEC-04 |
| Unbemerkte Datenübertragung | keine SDK-Telemetrie per Default, Netzwerk-Inventar | T-SEC-05 |
| Falscher Alarmstatus | Abgleich Planungsstatus, sichtbarer Fehler | T-SEC-06 |
| Veraltete Altgeräte | klare Sicherheitshinweise, keine falschen Sicherheitsversprechen | T-SEC-07 |
| Unzulässige Mediennutzung | Rechte-/DRM-Prüfung, offizielle Integrationen | T-SEC-08 |

### 8.3 Datenschutzregeln

- Keine Pflichtregistrierung, keine Cloudpflicht, keine Werbe-ID und keine versteckte Analytik.
- Logs lokal und sparsam; diagnostischer Export nur nach aktiver Nutzerhandlung und mit Vorschau/Redaktion sensibler Felder.
- Externe Streams geben dem jeweiligen Anbieter technisch Verbindungsdaten preis; dies ist in der Datenschutzerklärung zu erläutern.
- Löschfunktionen für Quellen, Verlauf, heruntergeladene Medien und App-Einstellungen.
- App-Store-Datenschutzangaben und Rechte an Grafiken, Senderlogos, Metadaten und Testmedien vor Veröffentlichung prüfen.
- Keine pauschale Zusage, dass Drittanbieter-Streamingdienste oder Musikabonnements offline funktionieren.

## 9. Qualitätsziele, Teststrategie und Nachweise

### 9.1 Vorläufige Qualitätsziele (nach P0 zu kalibrieren)

| Merkmal | Vorläufiger Zielwert | Messung / Vorbehalt |
|---|---|---|
| Uhranzeige | keine sichtbaren Zeitsprünge außer echter Systemzeitkorrektur | Vergleich gegen Systemzeit |
| App-Start | Ziel ≤3 s bis zur nutzbaren Uhr auf vereinbartem Altgerät | 20 Kaltstarts, P95; nach Baseline verbindlich |
| Skinwechsel | Ziel ≤1 s, kein Verlust von Alarm-/Medienzustand | UI-Test und Persistenzprüfung |
| Medienfehler | kein Crash; Fehlertext und Retry/Abbruch | Fehler-Injektion |
| Offline-Uhr | vollständig nutzbar | Flugmodus-Test |
| lokale Medien | nach Import ohne Netz abspielbar | Flugmodus-Test |
| Speicher | keine unbeschränkten Downloads/Logs | Speicher- und Langzeittest |
| Energie/Wärme | Referenzmessungen bei 1 h/8 h Dauerbetrieb | erst nach Geräte-Baseline Schwellenwerte |
| Barrierefreiheit | Kernaktionen mit VoiceOver erreichbar | manueller Audit |
| Wecker | nur nachweisbare OS-Planung als „geplant“ | Zustandsmatrix, **keine Garantie** |

### 9.2 Testebenen

- **Unit:** ClockDomain, Recurrence-Regeln, Profile, Skin-Validierung, Timer, URL-Validierung, Fehlerzustände.
- **Integration:** AVPlayer/AudioSession, Dateien/Sandbox, UserNotifications, Persistenzmigrationen.
- **UI:** iPhone/iPad, Portrait/Landscape, Dynamic Type, VoiceOver, Profile/Skins, Berechtigungsdialoge.
- **End-to-End:** konkrete Anwendungsfälle `UC-01` bis `UC-12` des Lastenhefts.
- **Realgeräte:** iOS/iPadOS 16, aktuelles iOS/iPadOS, ältere Hardware, lange Laufzeit, Netz-/Audio-/Power-Events.
- **Security/Privacy:** bösartige URLs/Playlisten, Logs, App-Berechtigungen, unzulässige Zugriffe, Datenlöschung.

### 9.3 Kritische Zustandsmatrix für Alarme und Medien

| Zustand | OS-Alarm prüfen | Audiofortsetzung prüfen | automatischer neuer Stream-/Videostart |
|---|---|---|---|
| App sichtbar, Bildschirm aktiv | Ja | Ja | testen, bei kompatibler Quelle möglich |
| App im Hintergrund | Ja | Ja, soweit OS erlaubt | **nicht zusagen** |
| Bildschirm gesperrt | Ja | Audio-Session testen | **nicht zusagen** |
| App vom Nutzer beendet | Ja | normalerweise nicht voraussetzen | **nicht zusagen** |
| Gerät neu gestartet | Ja | kein automatischer App-Start voraussetzen | **nicht zusagen** |
| Benachrichtigung verweigert | Status/Fehler sichtbar | unabhängig | **nicht zusagen** |
| Fokus/Lautlos | hörbare Wirkung real testen | Audio separat | **nicht zusagen** |
| Offline / WLAN weg | lokaler Ton separat prüfen | nur lokale Medien | Netzwerkmedien nicht verfügbar |
| DST/Zeitzonenwechsel | Wiederholungsregeln prüfen | unabhängig | Zeitprogramm gesondert prüfen |

**Ergebnisprotokoll pro Test:** Gerät/Modell, OS-Version, Build/Commit, Berechtigungen, App-Zustand, Uhrzeit, erwartetes Ergebnis, beobachtetes Ergebnis, Log/Screenshot, Status `bestanden/offen/nur dokumentiert`, verantwortliche Person, Datum. Ein Dokumentationshinweis zählt **nicht** als bestandener Test.

### 9.4 Testfälle (Mindestkatalog)

| ID | Vorbedingung / Aktion | Erwartung |
|---|---|---|
| T-001 | Uhr 12/24 h umschalten | Anzeige korrekt, persistiert |
| T-002 | Zeitzone/DST wechseln | lokale Zeit und Wiederholung konsistent |
| T-003 | Retro → Modern → Retro | keine verlorenen Favoriten/Alarme |
| T-004 | iPhone klein, iPad groß, Rotation | Kernaktionen sichtbar |
| T-005 | SWR3-URL eintragen, abspielen | bei Erreichbarkeit Audio, sonst verständlicher Fehler |
| T-006 | ungültige/HTML-URL | kein Crash, kein irreführender Erfolg |
| T-007 | WLAN während Stream ausschalten | definierter Fehler-/Retryzustand |
| T-008 | lokale MP3 importieren, Flugmodus | Wiedergabe funktioniert |
| T-009 | lokale Videodatei in aktiver App | Video mit Pause/Stop |
| T-010 | Benachrichtigungen verweigern | Alarmstatus zeigt Einschränkung |
| T-011 | Testalarm auf realem iOS-16-Gerät | dokumentiertes reales Verhalten |
| T-012 | Alarm bei Sperre/Hintergrund/Beenden | je Zustand protokolliert, keine falsche Garantie |
| T-013 | Einschlafhilfe frisch installiert | deaktiviert, keine Aktivität |
| T-014 | Modul aktivieren, nicht starten | keine Wiedergabe |
| T-015 | Einschlafsession starten/abbrechen | Ende/Abbruch nachvollziehbar |
| T-016 | Daten speichern, App neu starten | Einstellungen und Quellen erhalten |
| T-017 | Skin-Datei manipulieren (falls Import) | sichere Ablehnung/Fallback |
| T-018 | VoiceOver + große Schrift | alle Kernaktionen bedienbar |
| T-019 | Flugmodus ohne Medien | Uhr/Skins/Timer funktionieren |
| T-020 | 8 Stunden Daueranzeige | Ressourcen/Wärme protokolliert |
| T-021 | TV-Routine in allen App-Zuständen (F) | Grenzen mit Belegen dokumentiert |
| T-022 | Konfigurationsschema migrieren | Daten erhalten oder sichere Wiederherstellung |
| T-023 | URL mit Zugangsdaten / Diagnoseexport | keine Geheimnisse im Klartext |
| T-024 | Medienplayer-Interruption (Anruf/Siri) | definierter Pause-/Resumezustand |

### 9.5 CI, Build und Qualitätssicherung

- GitHub Actions oder gleichwertige CI auf geeignetem macOS-Runner; Swift-Lint/Format-Entscheidung per ADR.
- Pull-Request-Checks: Build, Unit-Tests, statische Prüfungen, Lizenz-/Secret-Scan, Dokumentations-/Traceability-Prüfung.
- Release-Gate: signierter Archivbuild, eindeutiger Commit/Tag, Testergebnisse, Geräteprotokolle, Datenschutzhinweise, Screenshots, App-Store-Metadaten und Checksumme der Artefakte.
- UI-/Audio-/Alarm-E2E-Tests auf echten Geräten bleiben zusätzlich notwendig; grüne CI allein reicht nicht.
- Kein Release bei bekannten kritischen falschen Weckversprechen, reproduzierbaren Abstürzen, Datenverlust oder unsicherem Medienimport.

## 10. Abnahme- und Freigabegates

### GATE-P0 – technische Machbarkeit

- [ ] iOS/iPadOS 16 Deployment Target auf realen Referenzgeräten erfolgreich.
- [ ] SWR3-MP3-Stream oder nachweislich erreichbarer gleichwertiger Teststream abspielbar.
- [ ] Lokale MP3 im Flugmodus abspielbar.
- [ ] Alarmzustandsmatrix auf mindestens einem iOS-16-Gerät durchgeführt.
- [ ] Skinwechsel und Profilauswahl demonstriert.
- [ ] Alle Grenzen und offenen Risiken dokumentiert.

### GATE-ARCH – Architekturfreigabe

- [ ] ADRs für Persistenz, Alarmstrategie, Media Resolver, Skinformat und Portabilität.
- [ ] Schnittstellen mit Mock-Implementierungen unit-testbar.
- [ ] Keine versteckte OS-/UI-Kopplung in Domain-Modulen.
- [ ] iOS-16-Abhängigkeiten und aktuelle Build-Toolchain vereinbart.

### GATE-V1 – Produktabnahme

- [ ] Alle für V1 freigegebenen M-Anforderungen auf Anforderungen und Tests rückverfolgbar.
- [ ] `UC-01` bis `UC-12` bewertet; nicht implementierte S/F-Fälle als solche markiert.
- [ ] Kein kritischer offener Fehler, keine verlorenen Alarme/Favoriten nach Neustart.
- [ ] Benachrichtigungs-/Weckgrenzen korrekt in UI, Store-Text und Hilfe.
- [ ] iPhone und iPad, mindestens ein Altgerät, Rotation, VoiceOver, Offline und Dauerbetrieb geprüft.
- [ ] Datenschutz, Rechte, Quellen/Assets und App-Store-Angaben geprüft.
- [ ] Release-Commit, Tag, Build, Signierung, Testbericht, Checksummen und Freigabeentscheidung dokumentiert.

**Entscheidungsregel:** Ein Gate gilt nur bei belegter Prüfung als bestanden. `Nur dokumentiert` ist kein Ersatz für `bestanden`; `offen` blockiert, soweit die Anforderung releasekritisch ist. Verschiebungen benötigen eine genehmigte Scope-Änderung.

## 11. Traceability – Lastenheft → Lösung → Nachweis

| Lastenheft | Technische Umsetzung | Abnahme / Test |
|---|---|---|
| PZ-01, PLAT-01/05 | adaptive SwiftUI-Layouts | T-004, GATE-V1 |
| PZ-02, PROF-01..08 | ProfileManager, Views | T-003/004, UC-01/02 |
| PZ-03, PLAT-02/08/11 | iOS-16-Ziel, Altgeräte-/Energieprüfung | GATE-P0, T-020 |
| PZ-04, PLAT-03/04 | Native SwiftUI, Portabilitäts-ADR | GATE-ARCH |
| PZ-05, AUD-01..10 | Resolver, AVPlayer, MediaCatalog | T-005..007, T-024 |
| PZ-06, SKN-01..10 | SkinRenderer, sichere Tokens | T-003/017 |
| PZ-07, VID-04..07 | TV-Machbarkeits-Spike | T-021, **F – keine Zusage** |
| PZ-08, SLP-01..08 | Opt-in SleepModule | T-013..015 |
| PZ-09, MUS-02, SEC-01..10 | lokale Daten, Privacy, Security | T-008/019/023 |
| CLK-01..11 | ClockDomain, Nachtmodus | T-001/002/004 |
| ALM-01..11 | AlarmRepository, AlarmService | T-010..012, GATE-V1 |
| MUS-01..07 | Dateiimport, lokale Medien | T-008/016 |
| POD-01..06 | PodcastProvider (S/K) | nach V1 gesonderte Tests |
| VID-01..08 | VideoPlayer, CapabilityRegistry | T-009/021 |
| TMR-01..03 | TimerStateMachine | T-019 plus Timer-Unit-Tests |
| WLD-01..02 | Zeitzonenprovider (S/K) | DST-/Zeitzonentests |
| DSH-01..08 | Dashboard-Widgets | T-004, Integrations-Tests später |
| AUT-01..07 | RoutineEngine (S/F/K) | T-021, spätere Routinenmatrix |
| UX-01..06 | Accessibility und adaptive UI | T-018 |
| NFR-01..06 | Performance-/Fehlerstrategie | T-007/016/020 |
| SEC-01..10 | Threat Model, Datenminimierung | T-017/023, T-SEC-01..08 |
| UC-01..12 | E2E-Szenarien | GATE-V1 mit Einzelprotokollen |

**Pflegepflicht:** Vor jedem Release die vollständige ID-Liste des Lastenhefts gegen die Traceability-Tabelle abgleichen. Die Gruppierung oben ist ein Startpunkt, kein Ersatz für die spätere **Einzel-ID-Matrix** mit Owner, Commit, Testfall und Ergebnis.

## 12. Risiken, Gegenmaßnahmen und Entscheidungspunkte

| ID | Risiko | Schwere | Gegenmaßnahme / Entscheidungsbedarf |
|---|---|---|---|
| R-01 | iOS-16-Wecker wird mit Systemwecker verwechselt | kritisch | klare UX, Testmatrix, keine Garantiewerbung |
| R-02 | Automatischer TV-Start bei gesperrtem Gerät nicht möglich | hoch | F-Spike, Vordergrundmodus, Nutzerinteraktion als Alternative |
| R-03 | Big-Sur-Xcode nicht releasefähig | hoch | moderne Mac-/CI-Toolchain vor V1 sichern |
| R-04 | Audio-URLs ändern sich oder sind geoblockiert | mittel | austauschbare Quellen, Fehlerzustände, kein harter Provider-Lock |
| R-05 | DRM/Streamingrechte | hoch | nur offizielle APIs, Anbieterprüfung |
| R-06 | Alte Hardware: Akku, Wärme, OLED | hoch | Dauerbetrieb messen, dimmen, Warnungen |
| R-07 | iOS-16-Unterstützung verteuert neue APIs | mittel | Capability-Adapter, ADR vor Anhebung der Mindestversion |
| R-08 | Skin-Import als Angriffsfläche | hoch | nur deklarative Daten, Import erst nach Härtung |
| R-09 | Unklare Lizenz-/Markenrechte von Skins/Logos | mittel | nur eigene/lizenzierte Assets |
| R-10 | Wetter-/News-APIs verursachen Kosten | mittel | optional, Anbieter- und Budget-ADR |
| R-11 | Hintergrund-Audio verhält sich anders als erwartet | hoch | reale Zustandsmatrix, keine stillen Zusagen |
| R-12 | Zu großer V1-Umfang | hoch | P0/V1/V1.x strikt trennen |
| R-13 | Konfigurationsverlust bei Migration | hoch | Schema-Versionierung, Migrationstests |
| R-14 | Nutzer glaubt, offline sei alles verfügbar | mittel | Verfügbarkeitsindikatoren pro Quelle |

## 13. Architekturentscheidungen (ADR-Backlog)

| ADR | Frage | Vorschlag / Status |
|---|---|---|
| ADR-001 | SwiftUI-App-Struktur und Mindestversion | SwiftUI, iOS 16; **zu bestätigen** |
| ADR-002 | Lokale Persistenz | Codable/JSON + UserDefaults, Keychain; **zu prüfen** |
| ADR-003 | Alarmstrategie | UNUserNotificationCenter für iOS 16, moderne Adapter später; **Spike erforderlich** |
| ADR-004 | Medienresolver und unterstützte Formate | AVPlayer + URLSession, MP3 zuerst; **Spike erforderlich** |
| ADR-005 | Skinformat und Sicherheitsmodell | deklarative Tokens, Import später; **zu bestätigen** |
| ADR-006 | Background-Audio und AudioSession | nur erlaubte Modi; **Spike erforderlich** |
| ADR-007 | C++20-Core vs. Swift Package | Swift Package zuerst, C++ nach Nutzenbeleg; **offen** |
| ADR-008 | TV-/Streamingdienstintegration | pro Anbieter/SDK/DRM; **offen** |
| ADR-009 | Testgeräte und Dauerbetriebsgrenzen | Geräte-/Baseline-Auswahl; **offen** |
| ADR-010 | Xcode/CI/Signing/App-Store | aktuelle Toolchain festlegen; **offen** |
| ADR-011 | Monetarisierung / In-App-Käufe | kein Einfluss auf Kernalarm, **offen** |
| ADR-012 | Lizenz, Branding, Datenanbieter | Rechte-/Kostenprüfung, **offen** |

## 14. Projektvorgehen und Arbeitspakete

### 14.1 Empfohlene Reihenfolge

| Paket | Inhalt | Fertig, wenn ... |
|---|---|---|
| WP-01 | Repo, AGENTS.md, Dokumentations-/CI-Grundgerüst | Build und Tests reproduzierbar |
| WP-02 | iOS-16-Machbarkeit, Geräteliste | echter Gerätebuild, dokumentierte Toolchain |
| WP-03 | ClockDomain + Nachttisch-UI | Zeit-/DST-/Rotationstests grün |
| WP-04 | MediaResolver + URL-Player | SWR3- und Fehlerfälle protokolliert |
| WP-05 | lokale Medien und Offlinebetrieb | MP3 im Flugmodus abspielbar |
| WP-06 | AlarmService + Sicherheits-UX | Zustandsmatrix abgeschlossen |
| WP-07 | Profile + zwei Skins | Wechsel ohne Datenverlust |
| WP-08 | Video in aktiver App | lokales Testvideo abspielbar |
| WP-09 | Timer + optionale Einschlafhilfe | Opt-in- und Abbruchtests grün |
| WP-10 | Settings, Persistenz, i18n, Accessibility | Migration, DE/EN, VoiceOver |
| WP-11 | Langzeittests, Privacy, Release | GATE-V1 vollständig belegt |
| WP-12 | V1.x-Spikes (TV, Podcasts, Wetter etc.) | gesonderte ADRs, keine V1-Blockade |

### 14.2 Definition of Done pro Feature

- Anforderung-ID und priorisierter Scope dokumentiert.
- Implementierung hinter testbarer Schnittstelle; keine unbegründete Plattformkopplung.
- Positiv-/Negativtests einschließlich Berechtigungs-/Offlinefall vorhanden.
- DE/EN-Texte, Accessibility und Datenschutzfolgen geprüft.
- Screenshots/Logs/Build-ID als Evidenz vorhanden; Dokumentation aktualisiert.
- Bei Abweichung von Lastenheft: ADR/Change Request vor Merge.

## 15. Offene Fragen vor Implementierungsfreigabe

1. Welche **konkreten** iPhone-/iPad-Modelle mit iOS/iPadOS 16 stehen für Tests bereit?
2. Welche aktuelle Xcode-/macOS-Umgebung kann Release-Builds erzeugen? Ist ein geeigneter Mac oder macOS-CI-Runner vorhanden?
3. Soll V1 einen **Benachrichtigungsalarm** als Weckfunktion enthalten, oder wird eine als zuverlässig beworbene Weckfunktion bis zu einer neueren Alarm-API ausdrücklich zurückgestellt?
4. Welche Formate gelten verbindlich für V1 neben direktem MP3 (AAC, HLS, lokale MP4/M4A)?
5. Soll die Einschlafhilfe in V1 nur als deaktivierbares Modulgerüst oder bereits als vollständig bedienbarer Einschlaftimer erscheinen?
6. Soll die erste V1-App bereits kostenlos/freemium/Einmalkauf sein? Welche Funktionen bleiben immer frei?
7. Welche Rechte/Lizenzen gelten für SASD-Branding, Senderlogos, Skins und Beispielmedien?
8. Welche Messgeräte und akzeptablen Grenzen für Dauerbetrieb/Akku/Wärme legen wir fest?
9. Wird das Dashboard in V1 auf Uhr/Medien/Alarm begrenzt, während Wetter/Kalender ausdrücklich V1.x bleiben?
10. Wie soll ein Nutzer beim geplanten TV-Start auf die notwendigen App-/Bildschirmzustände hingewiesen werden?

## 16. Abnahmeprotokoll-Vorlage

```markdown
### Abnahme: <Anforderungs-ID> / <Test-ID>
- Build / Git-Commit / Tag:
- Gerät / OS-Version:
- App-Zustand / Berechtigungen / Netzwerk:
- Testdaten / Medienquelle:
- Erwartetes Ergebnis:
- Tatsächliches Ergebnis:
- Evidenz (Log/Screenshot/Video/Checksumme):
- Status: bestanden | offen | nur dokumentiert
- Abweichung / Risiko:
- Prüfer / Datum:
- Freigabeentscheidung:
```

## 17. Glossar

| Begriff | Bedeutung |
|---|---|
| **OS-Alarm** | durch iOS unterstützte Alarm-/Benachrichtigungsfunktion; Fähigkeiten versionsabhängig |
| **Benachrichtigungsalarm** | lokaler Notification-Termin, **nicht** automatisch gleichwertig mit Systemwecker |
| **Medienquelle** | direkte URL, lokale Datei oder künftig ein offizieller Anbieteradapter |
| **Skin** | deklarative optische Beschreibung der Oberfläche |
| **Profil** | Nutzungszusammenstellung aus Layout, Skin und Funktionskonfiguration |
| **Routine** | zeit-/ereignisgesteuerte Folge von Aktionen, nur soweit Plattform erlaubt |
| **Capability** | tatsächlich verfügbare Fähigkeit abhängig von OS, Berechtigung und Zustand |
| **Spike** | zeitlich begrenzter Machbarkeitstest mit reproduzierbarem Ergebnis |
| **Evidenz** | konkreter Nachweis einer erfolgreichen oder fehlgeschlagenen Prüfung |
| **Offline** | Funktion ohne Internet, nur bei lokal vorhandenen Daten/Medien |

## 18. Dokumentstatus und nächste Schritte

**Derzeit nicht freigegeben:** Mindestgeräteliste, aktuelle Toolchain, vollständige Medienformatmatrix, Alarm-Produktversprechen, konkreter V1-Scope und messbare Energie-/Performance-Grenzwerte. Deshalb ist das Pflichtenheft eine **ausführliche technische Arbeitsgrundlage V0.1**, noch kein verbindlicher Implementierungsvertrag.

**Empfohlener nächster Schritt:** Zuerst `ADR-001`, `ADR-003`, `ADR-004` und `ADR-010` mit kurzen P0-Spikes klären. Danach die Muss-Anforderungen einzeln mit Owner, Testfall und Abnahmebeleg in einer Traceability-Matrix erfassen und das Pflichtenheft als V0.2 freigeben.

---

### Quellen- und Validierungshinweis

Primäre Grundlage ist das im Projekt vorliegende Lastenheft V0.1 und die gemeinsame Produktdiskussion. API-Verhalten, App-Store-Vorgaben und Toolchain-Anforderungen ändern sich; **vor Umsetzung/Release** anhand aktueller offizieller Apple-Dokumentation und realer Gerätetests verifizieren. Relevante offizielle Dokumentationsbereiche: SwiftUI, AVFoundation, UserNotifications, AlarmKit, App Transport Security, App Store Review Guidelines, Xcode Release Notes. Dieses Dokument behauptet keine bereits durchgeführten technischen Tests.