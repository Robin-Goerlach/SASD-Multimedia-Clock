# SASD Multimedia Clock – Lastenheft

**Version:** 0.1 · **Datum:** 08.11.2026 · **Status:** Entwurf zur Abstimmung  
**Dokument-ID:** SASD-MC-LH-001

> Dieses Lastenheft beschreibt Anforderungen aus Nutzersicht (das **Was**), nicht bereits die technische Lösung (das **Wie**). Die Prioritäten sind Vorschläge. Grundlage sind unsere Produktentscheidungen und die vorhandene `SASD_Multimedia_Uhr_AppStore_Funktionsrecherche.md`.

## 1. Vision und Ausgangslage

**SASD Multimedia Clock** gibt älteren iPhones und iPads eine neue Aufgabe: als Radiowecker, Nachttischuhr, Schreibtischuhr und Multimedia-Dashboard. Die sinnvolle **Weiterverwendung älterer Apple-Geräte ist ein ausdrückliches zentrales Produktziel** und kein Nebeneffekt. Die App soll eine moderne, flexible Alternative zur einfachen Systemuhr bieten: optisch konfigurierbar, mit lokalen und netzbasierten Medien, optionalen Zeitprogrammen und ohne verpflichtenden Cloud-Account.

**Leitgedanke:** „Gib deinem alten iPhone oder iPad eine neue Aufgabe.“

### 1.1 Verbindlich beschlossene Leitlinien

| ID | Entscheidung |
|---|---|
| PZ-01 | iPhone **und** iPad gleichberechtigte Zielgeräte |
| PZ-02 | Multi-Use: Nachttisch, Schreibtisch, Multimedia; keine getrennten Apps |
| PZ-03 | Ältere Geräte weiterverwenden; **iOS/iPadOS 16** als angestrebte Mindestversion, sofern vertretbar |
| PZ-04 | Native Apple-App zuerst mit Swift/SwiftUI; Android und Desktop später |
| PZ-05 | Audio aus direkten unterstützten URLs, insbesondere Internetradio-MP3-Streams |
| PZ-06 | Konfigurierbare Skins ähnlich dem Winamp-Prinzip; nicht nur Retro |
| PZ-07 | Lieblings-TV-Show als automatisch geplantes Wunschszenario – Machbarkeit offen |
| PZ-08 | Einschlafhilfe nur optional und **ausdrücklich vom Nutzer aktivierbar** |
| PZ-09 | Offline nur für tatsächlich lokal verfügbare Inhalte; Datenschutz wichtig |

### 1.2 Zielgruppen und Nutzen

| Nutzergruppe | Nutzen |
|---|---|
| Besitzer älterer iPhones/iPads | Funktionierendes Altgerät sinnvoll weiterverwenden |
| Nachttischnutzer | Große Uhr, Nachtmodus, Wecker, Radio, optional Einschlafhilfe |
| Schreibtischnutzer | Uhr, Kalender, Wetter, Weltzeit, Medien und Informationen |
| Multimedia-Nutzer | Musik, Streams, Video, Podcasts, Favoriten, Routinen |
| Nutzer mit Seh-/Bedienbedarf | Große Schrift, Kontrast, einfache Bedienung, VoiceOver |
| Technikinteressierte | URLs, Skins, Profile, Integrationen und individuelle Layouts |

## 2. Priorisierung und Geltungsbereich

**B** = beschlossene Produktleitlinie; **M** = vorgeschlagene Muss-Anforderung V1; **S** = Soll-Anforderung; **K** = spätere Option; **F** = Machbarkeitsprüfung, **keine Zusage**; **X** = bewusst außerhalb V1. Diese Kennungen gelten für die nachfolgenden Anforderungen.

**V1.0** ist eine benutzbare erste Veröffentlichung. Ein früher Prototyp darf einen kleineren Umfang haben. Externe Anbieter, Streamingrechte und iOS-Beschränkungen dürfen nie stillschweigend als gelöst angenommen werden.

## 3. Nutzungskonzept und Profile

**PROF-01 [M] Nachttisch:** Große Uhr, nächster Alarm, Nachtmodus, Radio-Bedienelemente, besonders große Stopp-/Snooze-Tasten.  
**PROF-02 [M] Schreibtisch:** Uhr, Datum, Mediensteuerung und konfigurierbare Informationskacheln.  
**PROF-03 [M] Multimedia:** Uhr plus Audio-/Videoplayer, Medienquellen, Playlists und Zeitprogramme.  
**PROF-04 [M]** Profile sind jederzeit umschaltbar und ihre Einstellungen werden gespeichert.  
**PROF-05 [S]** Nutzerdefinierte Profile, Duplizieren, Umbenennen, Zurücksetzen, Export.  
**PROF-06 [S]** Automatischer Profilwechsel nach Zeitfenster, soweit zuverlässig möglich.  
**PROF-07 [M]** Profilwechsel darf weder ungewollt einen Alarm aktivieren noch Wiedergabe starten.  
**PROF-08 [M]** Bedienung auf iPhone und iPad, Hoch- und Querformat, adaptiv ohne abgeschnittene Bedienelemente.

## 4. Funktionale Anforderungen

### 4.1 Uhr und Zeit

| ID | Prio | Anforderung |
|---|---|---|
| CLK-01 | M | Digitale und analoge Uhr; korrekte lokale Zeit |
| CLK-02 | M | 12/24-Stundenformat; Sekunden ein-/ausblendbar |
| CLK-03 | M | Datum, Wochentag, optional Kalenderwoche |
| CLK-04 | M | Vollbild, Hoch-/Querformat, flexible Skalierung |
| CLK-05 | M | Nachtmodus mit manueller und zeitgesteuerter Aktivierung |
| CLK-06 | M | Bildschirm im aktiven Vordergrund wach halten, soweit iOS erlaubt |
| CLK-07 | S | Flip-, LED-, LCD-, Nixie-, Retro- und Minimaldesigns |
| CLK-08 | S | OLED-Schonung durch Dimmung und leichte Positionsverschiebung |
| CLK-09 | S | Weltzeiten mit Ort, Zeitzone, Sommerzeit und UTC-Offset |
| CLK-10 | K | Stundensignal, Zeitansage, animierte Zeiger, Jahresfortschritt |
| CLK-11 | M | Nächster Alarm und aktuelle Medienquelle optional sichtbar |

**Abnahme:** Zeit/Datum korrekt auch bei Zeitzonenwechsel und Sommerzeitumstellung; Darstellungen bleiben auf kleinen und großen Geräten lesbar. Einbrennschutz ist keine Garantie gegen Displayschäden.

### 4.2 Wecker und Alarmzuverlässigkeit

| ID | Prio | Anforderung |
|---|---|---|
| ALM-01 | M | Mehrere Alarme erstellen, benennen, ändern, aktivieren und löschen |
| ALM-02 | M | Einmalig, täglich oder bestimmte Wochentage |
| ALM-03 | M | Lokaler Alarmton, nächster Alarm sichtbar |
| ALM-04 | M | Deutlich getrennte Stopp- und Snooze-Bedienung |
| ALM-05 | S | Snooze-Dauer und Alarmprofile konfigurierbar |
| ALM-06 | M | Berechtigungs- und Weckstatus mit ehrlicher Einschränkungsanzeige |
| ALM-07 | M | Testalarm/Selbsttest ohne Warten auf den Morgen |
| ALM-08 | F | Wecken mit Internetradio, Playlist oder Video |
| ALM-09 | F | Ansteigende Lautstärke, simulierte Sonnenaufhellung |
| ALM-10 | F | Neuere AlarmKit-Funktionen auf unterstützten OS-Versionen |
| ALM-11 | K | Feiertags-/Urlaubsausnahmen, Schichtpläne, Weckaufgaben |

**Kritische Einschränkung:** Unter iOS 16 sind lokale Benachrichtigungen nicht gleichbedeutend mit einem vollwertigen Systemwecker. Eine App kann nicht beliebig aus gesperrtem/beendetem Zustand aufwachen und Streams starten. Die App darf keine unbelegte „garantierte Weckfunktion“ versprechen. Alarm und optionales Medienprogramm sind getrennte Funktionen; der tatsächliche Zustand (App sichtbar, Hintergrund, gesperrt, beendet, nach Neustart) wird getestet und dokumentiert. Bei kritischen Weckzeiten auf Systemwecker hinweisen.

### 4.3 Internetradio und URL-Audio

| ID | Prio | Anforderung |
|---|---|---|
| AUD-01 | M | Direkte HTTP(S)-Audio-URL eingeben, prüfen, speichern und abspielen |
| AUD-02 | M | Direkte MP3-Streams; AAC und HLS nach Kompatibilitätstest |
| AUD-03 | M | Play/Pause/Stop, Lautstärke, Stummschaltung |
| AUD-04 | M | Favoriten, letzte Quellen, benutzerdefinierte Sendernamen |
| AUD-05 | M | Verständliche Fehler bei Netz, DNS, TLS, Timeout, HTTP, Format |
| AUD-06 | S | Wiederverbinden nach Streamunterbrechung |
| AUD-07 | S | M3U-/PLS-Import und sichere Weiterleitungsauswertung |
| AUD-08 | S | Senderlogos, Titel-/Interpretmetadaten, Senderverzeichnis |
| AUD-09 | S | AirPlay/Bluetooth-Ausgabe über verfügbare Systemfunktionen |
| AUD-10 | F | Hintergrund-Audio mit zulässigen iOS-Audio-Sessions |
| AUD-11 | K | Senderverzeichnis mit Genres/Ländern, Timer und Senderplätzen |

**Verbindlicher Testfall:** `https://liveradio.swr.de/sw282p3/swr3/play.mp3` als eigene Quelle eintragen und bei erreichbarem Stream abspielen. **„Per URL erreichbar“ heißt nicht „jede beliebige Webseite/DRM-Quelle abspielbar“.** Unterstützte Formate und Protokolle werden verbindlich in einer Medienkompatibilitätsmatrix festgelegt. Anbieter können URLs und Rechte ändern.

### 4.4 Musik und lokale Dateien

- **MUS-01 [M]** Vom Nutzer gewählte lokale Audiodateien abspielen.
- **MUS-02 [M]** Offline-Wiedergabe lokal verfügbarer, unterstützter Medien.
- **MUS-03 [S]** Playlists, Zufallswiedergabe, Wiederholung, Fortschritt, Cover und Metadaten.
- **MUS-04 [S]** Import aus Dateien-App, soweit Sandbox und Zugriffsrechte erlauben.
- **MUS-05 [F]** Apple-Music-/Musikmediathek-Integration unter offiziellen APIs, Berechtigungen und DRM-Regeln.
- **MUS-06 [K]** Weitere Streamingdienste nur über offizielle Integrationen; keine DRM-Umgehung.
- **MUS-07 [S]** Wiedergabe bei Profil-/Skinwechsel möglichst unterbrechungsfrei.

### 4.5 Podcasts und Hörbücher

- **POD-01 [S]** Podcast-RSS-Feed per URL abonnieren und Episoden anzeigen.
- **POD-02 [S]** Abspielen, Position merken, vor-/zurückspringen.
- **POD-03 [S]** Episoden herunterladen und gezielt löschen; Speicherbedarf anzeigen.
- **POD-04 [K]** Kapitelmarken, Geschwindigkeit, automatische Episodenfolge.
- **POD-05 [M]** Klar anzeigen, wenn eine Folge ohne Internet nicht verfügbar ist.
- **POD-06 [K]** Hörbuch-/lokale Audiobuch-Unterstützung mit Lesezeichen.

### 4.6 Video und TV

- **VID-01 [M]** Lokale unterstützte Videos in aktiver App wiedergeben.
- **VID-02 [S/F]** Direkte unterstützte Video-URLs/HLS im aktiven Betrieb.
- **VID-03 [M]** Vollbild, Seitenverhältnis, Lautstärke und Wiedergabesteuerung.
- **VID-04 [F]** Geplante Lieblings-TV-Show als zeitgesteuertes Szenario.
- **VID-05 [F]** Automatischer Videostart **bei sichtbarer aktiver App** testen.
- **VID-06 [B]** Kein Versprechen, dass eine gesperrte/beendete App TV automatisch startet.
- **VID-07 [F]** YouTube, Mediatheken und kommerzielle Videodienste einzeln auf offizielle APIs, Einbettungsregeln, DRM und Deep Links prüfen.
- **VID-08 [K]** Video-Playlists, wiederkehrende Sendungen, visuelle Morgenroutinen.

**Akzeptanzfall:** „Werktags um 07:00 Lieblingssendung starten“ wird für die Zustände *Vordergrund / Hintergrund / gesperrt / beendet / offline* separat bewertet. Ein Hinweis mit Nutzerinteraktion ist ein möglicher Fallback, aber **kein** automatischer Start.

### 4.7 Einschlafhilfe – ausdrücklich optional

- **SLP-01 [B]** Standardmäßig deaktiviert.
- **SLP-02 [M]** Nutzer muss das Modul bewusst aktivieren.
- **SLP-03 [M]** Aktivieren des Moduls startet **keine** Wiedergabe.
- **SLP-04 [S]** Einschlaftimer mit frei wählbarer Dauer.
- **SLP-05 [S]** Audio/Video wählen, Lautstärke ausblenden, Bildschirm dimmen.
- **SLP-06 [S]** Nach Ende Nachtuhr anzeigen, Ablauf jederzeit abbrechen/verlängern.
- **SLP-07 [K]** Naturklänge, Geräuschmischungen, gespeicherte Schlafprofile.
- **SLP-08 [X]** Medizinische Schlafdiagnosen in V1.

### 4.8 Timer, Produktivität, Weltzeit

- **TMR-01 [M]** Countdown mit Start, Pause, Fortsetzen, Abbrechen, Ende-Signal.
- **TMR-02 [S]** Stoppuhr und mehrere parallele Timer.
- **TMR-03 [K]** Pomodoro, Fokus-/Pausenzyklen, Statistiken.
- **WLD-01 [S]** Mehrere Zeitzonen; lokale Uhr und Weltzeit nebeneinander.
- **WLD-02 [K]** Internationale Meeting-Zeitplanung, Feiertage, Geschäftszeiten.
- **PRD-01 [K]** Aufgabenlisten, Erinnerungen und Schnellzugriffe.

### 4.9 Schreibtisch-Dashboard

| ID | Prio | Funktion | Grenze |
|---|---|---|---|
| DSH-01 | M | Uhr und konfigurierbare Kachelanordnung | Auch auf kleinem iPhone bedienbar |
| DSH-02 | S | Nächste Termine und Tageskalender | EventKit-Berechtigung |
| DSH-03 | S | Wetter, Temperatur, Vorhersage | Datenanbieter und Kosten prüfen |
| DSH-04 | S | Standort manuell statt Pflichtortung | Datenschutz |
| DSH-05 | S | Akkustand, Ladezustand, Netzwerkstatus | Nur OS-verfügbare Daten |
| DSH-06 | K | Nachrichten/RSS-Feeds | Nutzungsrechte, Aktualität |
| DSH-07 | K | To-dos, Notizen, App-Schnellzugriffe | Integrationen prüfen |
| DSH-08 | K/F | Smart-Home-Status/Steuerung | Berechtigungen, Netzwerk, Sicherheit |

### 4.10 Skins und Darstellung

- **SKN-01 [B]** Austauschbare Skins, Retro und moderne Gestaltung gleichberechtigt.
- **SKN-02 [M]** Mindestens zwei unterschiedliche mitgelieferte Skins (Retro-Radiowecker und modernes Dashboard).
- **SKN-03 [M]** Skinwechsel ohne Verlust von Alarmen, Quellen und Profilen.
- **SKN-04 [M]** Farben, Schriftgrößen, Kontrast und sichtbare Elemente anpassen.
- **SKN-05 [S]** Vorschau und jederzeitige Rückkehr zum Standardskin.
- **SKN-06 [S]** Frei konfigurierbare Kacheln, Größen und Positionen.
- **SKN-07 [K/F]** Eigenes deklaratives Skinformat, Import/Export, Editor und Community-Skins.
- **SKN-08 [M]** Fremde Skins dürfen keine ausführbaren Skripte einschleusen.
- **SKN-09 [M]** Barrierefreie Standardbedienung und Alarm-Stopp bleiben immer erreichbar.
- **SKN-10 [M]** Keine ungeprüfte Übernahme fremder Winamp-Skins, Marken oder Grafiken.

### 4.11 Zeitprogramme und Automatisierung

- **AUT-01 [S]** Wiederkehrende Zeitprogramme nach Uhrzeit und Wochentag anlegen.
- **AUT-02 [S]** Aktionen: Profil ändern, Anzeige ändern, Timer, Medienstart im erlaubten App-Zustand.
- **AUT-03 [M]** Vor Aktivierung technische Grenzen und erforderliche Nutzerinteraktionen zeigen.
- **AUT-04 [S]** Konflikte zwischen Routinen erkennen; Priorität festlegen.
- **AUT-05 [S]** Testlauf und Ereignisprotokoll (geplant, ausgelöst, ausgeführt, fehlgeschlagen).
- **AUT-06 [M]** Alle Programme einzeln deaktivierbar; kein unbeabsichtigter Medienstart.
- **AUT-07 [K]** Kurzbefehle, Home Assistant und weitere Auslöser.

## 5. Plattform- und Hardwareanforderungen

- **PLAT-01 [B]** iPhone und iPad, Ziel-Mindestversion iOS/iPadOS 16.0.
- **PLAT-02 [B]** Mindestversion nur nach dokumentierter Aufwand-/Nutzenabwägung anheben.
- **PLAT-03 [B]** Swift/SwiftUI nativ zuerst; spätere Android-, Windows-, Linux-, macOS-Versionen nicht Teil V1.
- **PLAT-04 [F]** Gemeinsamer C++20-Core erst nach iOS-Prototyp entscheiden; nicht vorschnell erzwingen.
- **PLAT-05 [M]** Unterschiedliche Bildschirmgrößen, Safe Areas, Rotation und Multitasking berücksichtigen.
- **PLAT-06 [M]** Bildschirm bei aktiver App wach halten, soweit iOS erlaubt.
- **PLAT-07 [M]** Nachtmodus, Dimmung, Energiesparverhalten, Anzeige bei Stromversorgung.
- **PLAT-08 [S]** OLED-Schonung und Hinweise zu Wärme/Akkualterung im Dauerbetrieb.
- **PLAT-09 [M]** Gespeicherte Einstellungen nach App-Neustart wiederherstellen.
- **PLAT-10 [M]** Kein automatischer App-Neustart nach Geräte-Neustart versprechen.
- **PLAT-11 [M]** Nutzer auf Sicherheitsrisiken ungepatchter Altgeräte hinweisen.

## 6. Nichtfunktionale Anforderungen

### 6.1 Usability und Barrierefreiheit

- **UX-01 [M]** Große, eindeutig beschriftete Bedienflächen, insbesondere Alarm Stop/Snooze.
- **UX-02 [M]** Lesbarkeit aus typischer Nachttisch-/Schreibtischentfernung.
- **UX-03 [M]** VoiceOver, Dynamic Type soweit sinnvoll, hohe Kontraste, reduzierte Animationen.
- **UX-04 [M]** Deutsche und englische Oberfläche, weitere Sprachen später.
- **UX-05 [M]** Keine Kernfunktion ausschließlich über versteckte Gesten.
- **UX-06 [S]** Einrichtungsassistent ohne Accountpflicht; Profile als Startvorlagen.

### 6.2 Leistung, Stabilität, Ressourcen

- **NFR-01 [M]** Flüssige Zeitanzeige auch auf vereinbarten Altgeräten.
- **NFR-02 [M]** Keine unnötigen Netzwerkzugriffe deaktivierter Module.
- **NFR-03 [M]** Stream-/Video-Fehler dürfen die App nicht abstürzen lassen.
- **NFR-04 [M]** Einstellungen robust und verlustarm speichern.
- **NFR-05 [S]** CPU, RAM, Energiebedarf, Temperatur und Startzeit auf Referenzgeräten messen; Grenzwerte nach Baseline definieren.
- **NFR-06 [M]** Nach Netzwerkverlust Uhr und lokale Medien weiter nutzbar.

### 6.3 Datenschutz und Sicherheit

- **SEC-01 [B]** Kein verpflichtendes Konto, keine verpflichtende Cloud.
- **SEC-02 [M]** Berechtigungen nur bei konkretem Bedarf anfordern (Benachrichtigungen, Kalender, Standort, Mediathek).
- **SEC-03 [M]** Datenminimierung; keine versteckte Telemetrie oder Werbe-ID.
- **SEC-04 [M]** Favoriten, Verlauf und Konfiguration lokal löschbar.
- **SEC-05 [M]** URL- und Playlist-Eingaben validieren; Redirects, TLS und Authentifizierung sicher behandeln.
- **SEC-06 [M]** HTTPS bevorzugen; HTTP-Ausnahmen einzeln bewerten.
- **SEC-07 [M]** Keine Tokens oder Passwörter in Klartextlogs; Keychain für erforderliche Geheimnisse.
- **SEC-08 [S]** Konfiguration exportieren/importieren, ohne Geheimnisse versehentlich offenzulegen.
- **SEC-09 [M]** Datenanbieter, Rechte, App-Store-Datenschutzangaben transparent dokumentieren.
- **SEC-10 [M]** DRM, Markenrechte, Urheberrechte und Streamingbedingungen beachten.

### 6.4 Offline-Fähigkeit

| Bereich | Erwartung ohne Internet |
|---|---|
| Uhr, Skins, lokale Einstellungen | Vollständig |
| Lokale Audio-/Videodateien | Soweit zuvor lokal gespeichert und zugänglich |
| Lokal gespeicherte Podcastfolgen | Soweit heruntergeladen und nutzbar |
| Live-Internetradio und externe Streams | Nicht verfügbar |
| Wetter/Nachrichten | Allenfalls zuletzt gespeicherte Daten mit Zeitstempel |
| Apple Music / externe Mediatheken | Abhängig von Rechten, Downloadstatus und APIs |

## 7. Konfigurations- und Bedienoberflächen

**Einstellungen (iPad: Seitenleiste, iPhone: kompakte Navigation):** Allgemein; Anzeige & Skins; Uhr & Zeit; Wecker; Medienquellen; Radio & Streams; Musik; Video; Einschlafhilfe; Timer & Werkzeuge; Profile & Szenarien; Netzwerk; Datenschutz; Info.

**Allgemein:** Sprache, Einheiten, Profil, Startansicht, Ausrichtung, Nachtzeiten, Daueranzeige.  
**Medien:** Quelle/URL, Typ, Verbindungstest, Name, Favorit, lokale Dateien, Berechtigungen.  
**Wecker:** Zeit, Wiederholung, Ton, Snooze, Test, Berechtigungsstatus, optionale Medienaktion.  
**Skins:** Vorschau, Farben, Schrift, Layout, Standard wiederherstellen.  
**Datenschutz:** Berechtigungen, externe Dienste, Datenlöschung, Export.

Die bisherigen Visualisierungen für Retro-Nachttischuhr, iPad-Konfiguration und Schreibtisch-Dashboard dienen als **Designinspiration**, nicht als verbindliche Pixelvorlage. Alle dargestellten Daten, Termine und Nachrichten sind Demo-Inhalte.

## 8. Hauptanwendungsfälle und Abnahme

| ID | Szenario | Akzeptanzkriterium |
|---|---|---|
| UC-01 | Altes iPad als Schreibtischuhr | Uhr/Medien sichtbar; Einstellungen bleiben erhalten |
| UC-02 | iPhone als Radiowecker | Alarmstatus ehrlich dargestellt; Medienstart nicht fälschlich garantiert |
| UC-03 | SWR3-URL hinzufügen | Quelle prüfen, speichern und bei Erreichbarkeit abspielen |
| UC-04 | Skin wechseln | Darstellung ändert sich; Alarme und Favoriten bleiben |
| UC-05 | Einschlafhilfe nutzen | Vor Aktivierung inaktiv; nach Aktivierung gesonderter Start nötig |
| UC-06 | Lieblingsshow um 07:00 | Vordergrundbetrieb getestet; gesperrter Zustand als Grenze dokumentiert |
| UC-07 | WLAN fällt aus | Uhr läuft weiter; Streamfehler verständlich; lokale Medien abspielbar |
| UC-08 | Rotation und Gerätewechsel | iPhone/iPad bedienbar, keine abgeschnittenen Kernfunktionen |
| UC-09 | Nachtbetrieb | Nachtmodus/Dimmung funktionieren; Wärme-/Stromverhalten gemessen |
| UC-10 | Berechtigung verweigert | App bleibt als Uhr nutzbar; eingeschränkte Funktion zeigt Hinweis |
| UC-11 | Zeitumstellung | Datum, Zeitzone und wiederkehrende Alarmzeiten konsistent |
| UC-12 | Beschädigte URL/Playlist | Kein Absturz, keine unsichere Ausführung, verständliche Meldung |

## 9. Teststrategie und Release-Gates

**Gerätematrix:** Mindestens ein älteres iPhone und iPad mit iOS/iPadOS 16 sowie ein neueres Referenzgerät; konkrete Modelle vor Implementierung bestimmen.  
**Zustandsmatrix Wecker:** App Vordergrund/Hintergrund/beendet, Bildschirm gesperrt, Fokus/Lautlos, Benachrichtigungen verweigert, Netzverlust, Reboot, Sommerzeitwechsel.  
**Medienmatrix:** MP3/AAC/HLS, lokale Datei, URL-Redirect, TLS-Fehler, 404, Timeout, Unterbrechung, Bluetooth/AirPlay.  
**UI-Matrix:** Hoch-/Querformat, kleine/große Displays, Kontrast, VoiceOver, reduzierte Bewegung, Skins.  
**Dauerbetrieb:** Ladezustand, Wärme, Speicher, Energieverbrauch, nächtliche Helligkeit und Wiederaufnahme.

**Release-Gate:** Keine Freigabe bei kritischen Datenschutzproblemen, Datenverlust, Abstürzen in Kernabläufen, unbeabsichtigtem Medienstart oder irreführenden Weckversprechen. Jede Muss-Anforderung erhält im Testplan eine Testfall-ID und Evidenz.

## 10. Produkt- und Geschäftsmodell

**Positionierung:** Weiterverwendung älterer Geräte + personalisierbare Uhr + Radio/Multimedia. **Preismodell noch offen.** Zu prüfen: kostenlose Basis, einmaliges Pro-Upgrade, optionale Skinpakete; Abos nur bei echten laufenden Zusatzleistungen. Aufdringliche Werbung im Nachttischbetrieb ist besonders kritisch. Preise und Umsatzannahmen benötigen gesonderte Marktvalidierung.

**Erfolgsmessung:** Aktivierung auf Altgeräten, wiederkehrende Nutzung, zuverlässige Streamwiedergabe, geringe Absturzrate, Bedienbarkeit, Nutzerfeedback, Zahlungsbereitschaft. Konkrete Zielwerte nach Pilotphase definieren.

## 11. Risiken und Abhängigkeiten

| Risiko | Folge | Behandlung |
|---|---|---|
| iOS-Sperr-/Hintergrundgrenzen | Medienroutinen funktionieren nicht wie gewünscht | Prototyp, klare Capability-Anzeige, Fallback |
| iOS 16 und ältere Geräte | Fehlende APIs, Performance, Sicherheitsupdates | Geräte-/Versionsmatrix, Feature-Gates, Neubewertung nur bei Evidenz |
| Alter Intel-Mac/Big Sur | Aktuelle Xcode-/App-Store-Toolchain fehlt | Entwicklungsumgebung früh prüfen; aktuelle Build-Umgebung einplanen |
| Streaming-URLs ändern sich | Senderausfälle | Editierbare Quellen, Tests, Fehleranzeigen |
| DRM und Anbieterregeln | Kein beliebiger TV-/Musikzugriff | Offizielle APIs und Rechteprüfung |
| Daueranzeige | Akkuverschleiß, Wärme, OLED-Einbrennen | Dimmung, Schonmodus, Hinweise, Messungen |
| Externe Wetter-/Nachrichtenquellen | Laufende Kosten, Datenschutz | Optional, Anbieterprüfung |
| Funktionsfülle | Komplexe Bedienung | Profile, Module, klare Navigation |
| Fremde Skins | Sicherheit/Urheberrecht | Deklaratives Format, Validierung, keine Skripte |

## 12. Releasevorschlag

### P0 – Machbarkeitsprototyp

SwiftUI-App auf iPhone/iPad; iOS-16-Build; digitale/analoge Uhr; zwei Skins; Nachttisch-/Schreibtischansicht; URL-MP3-Stream; lokale Datei; Timer. Technische Spikes zu Wecker, Hintergrundbetrieb und Video-Autostart.

### V1.0 – Erste Veröffentlichung (noch freizugeben)

Drei Profile; stabile Uhr; Nachtmodus; mindestens zwei Skins; URL-Audio, Favoriten, lokale Audiodateien; einfacher Videoplayer bei aktiver App; Timer; nachvollziehbarer Alarmstatus und nur nachweislich funktionierende Alarmwege; Datenschutz, Accessibility und robuste Einstellungen. Optional aktivierbare Einschlafhilfe, sofern priorisiert.

### V1.x – Komfort

Podcasts, Downloads, Playlists, erweiterte Weltzeit, Kalender/Wetter, zusätzliche Skins, Audio-Fades und nachgewiesene Zeitprogramme.

### V2+ – Erweiterung

Skinimport/-editor, Smart Home, weitere Mediendienste, Community-Skins, plattformübergreifender Core und Android-/Desktop-Portierungen.

## 13. Offene Produktentscheidungen und Prüfaufträge

| ID | Offene Frage | Nachweis |
|---|---|---|
| OD-01 | Welche Altgeräte stehen als Testhardware bereit? | Modell-/iOS-Inventar |
| OD-02 | Welche Xcode-Version ist verfügbar? | `sw_vers`, `xcodebuild -version` |
| OD-03 | Welche Audioformate/Playlisttypen verbindlich? | MP3/AAC/HLS/M3U/PLS-Prototyp |
| OD-04 | Was leistet der Wecker unter iOS 16 tatsächlich? | Realgerät-Zustandsmatrix |
| OD-05 | Welche neueren AlarmKit-Funktionen sind ergänzend möglich? | API-/Versionsmatrix |
| OD-06 | Welche TV-/Videodienste sind legal integrierbar? | Anbieter-/API-/DRM-Prüfung |
| OD-07 | Was kann die Zeitautomatisierung im Vordergrund? | Funktionierender Spike |
| OD-08 | Einschlafhilfe V1 oder V1.x? | Aufwand-/Nutzenentscheidung |
| OD-09 | Skinformat in V1 oder später? | Zwei interne Skins und Formatentwurf |
| OD-10 | Welche Dashboard-Kacheln sind V1-Pflicht? | Nutzer- und Scope-Abstimmung |
| OD-11 | Kauf, Freemium, Abo? | Marktvalidierung |
| OD-12 | C++20-Core oder zunächst modularer Swift-Code? | Architekturentscheidung nach Prototyp |
| OD-13 | Podcast-Offline-Downloads und Speichergrenzen? | Rechte-/UX-Konzept |
| OD-14 | App-Store-Anforderungen zum Releasezeitpunkt? | Aktuelle Richtlinienprüfung |

## 14. Dokumentation, Traceability und nächste Schritte

Empfohlene Repository-Struktur:

```text
docs/
  product/LASTENHEFT.md
  product/PRODUCT-VISION.md
  product/DECISIONS.md
  product/APPSTORE-INSPIRATION.md
  requirements/REQUIREMENTS-MATRIX.md
  requirements/USE-CASES.md
  architecture/PFLICHTENHEFT.md
  architecture/ADR/
  testing/DEVICE-MATRIX.md
  testing/ALARM-RELIABILITY.md
  testing/MEDIA-COMPATIBILITY.md
  testing/RELEASE-GATES.md
```

Jede geänderte Produktentscheidung wird mit Datum, Begründung und betroffenen Anforderungen dokumentiert. Muss-Anforderungen werden später mit Testfällen und Abnahmebelegen verknüpft.

### Freigabecheckliste

- [ ] V1-Muss-/Soll-/Kann-Scope gemeinsam freigegeben.
- [ ] Altgeräte und Build-Umgebung bekannt.
- [ ] Wecker- und Medienstartgrenzen auf echten Geräten getestet.
- [ ] Medienformate und Quellen verbindlich definiert.
- [ ] Datenschutz-/Streamingrechte bewertet.
- [ ] iPhone-/iPad-UX-Prototyp geprüft.
- [ ] Pflichtenheft, Roadmap und Testplan aus diesem Lastenheft abgeleitet.

## Anhang A – Weitere Inspiration, nicht V1-zugesagt

Flip-Clock, Nixie, LED, Retro-Holzoptik, Pixel-Shift, animierte Hintergründe, eigene Fotos, Widgets, StandBy, Live Activities, Siri-Kurzbefehle, CarPlay, Apple Watch, AirPlay, Chromecast, Sonos, HomePod, Weckaufgaben (Rechnen/QR), Sonnenaufgang, Schlaftracking, Apple Health, Naturgeräusche, Hörbücher, DLNA/UPnP, SMB/WebDAV, RSS-News, Pomodoro, Aufgaben, internationale Meetingplanung, externe Displays, Bluetooth-Tasten, Fernbedienung, Skin-Marktplatz, Cloud-Synchronisierung. Vor Aufnahme in einen Release jeweils Nutzerwert, Machbarkeit, Datenschutz, Kosten und Rechte prüfen.

## Anhang B – Quellen- und Validierungshinweis

Die bereits erstellte **App-Store-Funktionsrecherche** enthält Beispiele und Anbieterbeschreibungen. Diese sind Inspiration und kein unabhängiger Praxistest. Für die Umsetzung müssen die dann aktuellen Apple-Dokumentationen zu SwiftUI, AVFoundation, UserNotifications, AlarmKit, MusicKit, EventKit und App-Store-Regeln geprüft werden. Die exemplarische SWR3-URL ist ein Testfall und keine Verfügbarkeitsgarantie.

---

**Ende – Lastenheft V0.1 (Entwurf).**