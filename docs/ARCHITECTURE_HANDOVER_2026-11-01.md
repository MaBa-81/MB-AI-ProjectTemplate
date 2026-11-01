# MB-AI-ProjectTemplate – Architekturgedanken und Übergabe an den nächsten Chat

**Datum:** 01.11.2026  
**Status:** Diskussionsstand / Anforderungssammlung, **keine** abgeschlossene Spezifikation  
**Zweck:** Verbindliche Gesprächsübergabe, damit die Entwicklung in einem eigenen Chat fortgesetzt werden kann, ohne die Entscheidungen und Hintergründe zu verlieren.

> **Wichtig:** Dieses Repository ist öffentlich. Dieses Dokument enthält daher keine Zugangsdaten, privaten Netzwerkadressen, personenbezogenen Details oder internen Rechnerpfade.

## 1. Ausgangspunkt und Motivation

Bei der Entwicklung des privaten Firmware-/Hardwareprojekts MBPPL wurden GitHub, OpenHands, ein lokal laufender Codex-Agent, Testskripte, ein RP2040 und ein Oszilloskop eingesetzt. Die KI konnte zwar Aufgaben übernehmen, die Entwicklung erforderte aber weiterhin viel persönliche Anwesenheit am Laborrechner. Agenten verbrachten teils viele Minuten mit Analyse, Freigaben, Archivierung oder Wartezeiten, obwohl einzelne Messungen nur kurz dauerten. Ein kurzzeitiger Netzwerkausfall erschwerte die Einschätzung, ob ein Auftrag noch lief. Messberichte und Rohdaten mussten gesondert für die gemeinsame Nutzung archiviert werden.

Das zentrale Ziel lautet daher:

**Ein Entwickler erteilt von einem beliebigen Ort einen Auftrag an ein GitHub-Projekt, bestimmt bei Bedarf den ausführenden Rechner, lässt Agenten und lokale Werkzeuge kontrolliert arbeiten und erhält später überprüfbare Ergebnisse – ohne vor dem Laborrechner warten zu müssen.**

Entscheidend ist die **tatsächlich eingesparte menschliche Warte- und Bedienzeit**, nicht die Anzahl eingesetzter KI-Komponenten.

## 2. Korrigierte Reihenfolge der Projekte

Es wurden zwei eigenständige GitHub-Repositories angelegt:

- **MB-AI-ProjectTemplate** – die universelle, möglichst schlanke Grundlage für neue private Entwicklungsprojekte.
- **MB-AgentBridge** – die separat entwickelte Laufzeitkomponente für die Anbindung entfernter Rechner, lokaler Agenten und Werkzeuge. Dort liegt bereits `docs/MB_AGENTBRIDGE_CONCEPT.md` mit einer früheren Anforderungssammlung.

**Wichtige Erkenntnis:** Eigentlich hätte zuerst das Template entworfen werden sollen. Erst danach sollten neue Projekte daraus entstehen, darunter auch MB-AgentBridge. Das ist noch korrigierbar, da die Bridge bisher nur ein frühes Konzept besitzt.

**Nächster Schwerpunkt ist ausschließlich MB-AI-ProjectTemplate.** Die eigentliche Bridge-Implementierung folgt später. Der bestehende MBPPL-Chat bleibt für MBPPL-Firmware, Tests und Messungen reserviert.

## 3. Geltungsbereich des Templates

Das Template soll als gemeinsame Grundlage für **private** Entwicklungsprojekte aller Art dienen: Elektronik, Embedded-Firmware, Software, Prüfgeräte, CAD, Dokumentation und Mischprojekte.

Es soll dabei **extrem schlank, modular und stark abgegrenzt** bleiben:
- nur gemeinsamer Kern, keine umfangreichen projektspezifischen Frameworks;
- optionale Erweiterungen/Profiles je nach Projekttyp;
- einheitliche Dokumentations-, Agenten-, Test- und Freigaberegeln;
- keine unnötigen Abhängigkeiten;
- kein Nachbau bereits etablierter Plattformen.

Berufliche Projekte und Unternehmens-GitLab sind **nicht** Bestandteil dieses privaten Systems.

GitHub-Template-Repositories dienen als **Startvorlage**, nicht als automatische Vererbung: spätere Template-Änderungen werden nicht von selbst in bereits erzeugte Projekte übernommen. Für gemeinsam aktualisierbare Funktionalität sind wiederverwendbare Workflows, versionierte Aktionen oder externe Pakete zu prüfen.

## 4. Grundarchitektur – derzeitiges gemeinsames Verständnis

```text
       Entwickler + ChatGPT
      (Auftrag, Ziel-PC, Freigabe)
                 |
                 v
       GitHub-PROJEKTREPOSITORY
       - Issues / Actions / PRs
       - Cloud-Agent (z. B. OpenHands)
       - Auftrag, Status, Freigaben
       - Bridge-/Ressourcenverzeichnis
       - Quellcode, Tests, Ergebnisse
                 |
        Auftrag an Zielrechner
         /                 \
        v                   v
  Bridge Keller-PC    Bridge Wohnungs-PC
  (Hintergrunddienst) (Hintergrunddienst)
        |                   |
  lokaler Agent       lokaler Agent
  Codex/anderer       Codex/anderer
        |                   |
  lokale Werkzeuge    lokale Werkzeuge
  Compiler, Pico,     Compiler, Tests,
  Oszilloskop, ...    Dokumentation, ...
        \                   /
         Ergebnisse zurück
             zu GitHub
```

**Architekturgrundsätze:**

1. **Das jeweilige GitHub-Projektrepository ist der logische zentrale Knotenpunkt** für Aufträge, Entwicklung, Nachweise und Ergebnisse. Es ist nicht zwingend ein Echtzeit-Kommunikationsserver.
2. **Bridges kommunizieren nicht direkt miteinander.** Es ist zunächst kein eigener zentraler Bridge-Koordinator vorgesehen.
3. **Der Entwickler wählt den Zielrechner bewusst aus dem Chat heraus** (z. B. `keller-pc` oder `wohnungs-pc`). Automatisches Scheduling oder Lastverteilung ist ausdrücklich **kein** MVP-Ziel.
4. **Die Bridge ist die Brücke zum Werkzeugkasten des Zielrechners**, nicht selbst der entwickelnde KI-Agent. Sie übermittelt Aufträge, startet/überwacht einen lokalen Ausführer und meldet Ergebnisse zurück.
5. **Der lokale Ausführer ist austauschbar:** zunächst möglicherweise Codex, später andere Agenten oder rein deterministische Test-Runner. Die Architektur darf nicht auf einen Hersteller oder ein Modell festgelegt sein.
6. **VS Code ist optional**, nicht erforderlich. Lokale Agenten können, soweit deren reale Schnittstellen es erlauben, über CLI/SDK als Hintergrundprozess laufen.
7. Die Bridge läuft **unabhängig von einem Monitorfenster** als Hintergrunddienst. Ein separates Monitorprogramm visualisiert Aufträge, Status, Laufzeiten, Logs, Fehler und Ressourcen.
8. Ein neuer Template-basierter Projektstart bringt die **Bridge-Anbindung und Auftragskonventionen** mit, **nicht eine Kopie des laufenden Bridge-Dienstes**. Der Dienst wird pro freigegebenem Rechner installiert und kann mehreren Projekten dienen.
9. Die Werkzeuge selbst sollen **lokal** gesteuert werden. Keine GitHub-Rundreise pro SCPI-Kommando, Debug-Operation oder Messschleife.

## 5. Die drei klaren Zuständigkeiten

### A. Template / GitHub-Projekt
- Projektgrundstruktur und Agentenregeln;
- Cloud-Agent-Einbindung (OpenHands-ähnlicher Ablauf über GitHub Issues/Actions, wie bei MBPPL);
- Standard für Auftrag, Zielrechner, erlaubte Aktionen, Freigaben und Ergebnisreferenzen;
- Sichtbarkeit vorhandener Bridges und ihrer Fähigkeiten;
- Pull Requests, Versionsverwaltung, Dokumentation und Nachweise;
- wiederverwendbare Workflows, soweit sinnvoll.

### B. MB-AgentBridge auf einem Rechner
- als Hintergrunddienst installiert, dauerhaft oder bedarfsorientiert erreichbar;
- registriert stabile Rechner-/Bridge-ID und aktuelle Erreichbarkeit;
- erkennt lokale Ressourcen, Werkzeuge, SDKs, Agenten und angeschlossene Geräte, soweit technisch möglich;
- unterscheidet **erkannt**, **konfiguriert**, **freigegeben** und **aktuell verfügbar**;
- erhält nur für diesen Rechner adressierte Aufträge;
- startet geeigneten lokalen Agenten/Test-Runner und überwacht dessen Zustand;
- führt Zeitlimits, Heartbeats, Abbruch, Wiederaufnahme und Fehlerbehandlung aus;
- überträgt Status, Protokolle und Artefakte an das jeweilige Projekt;
- bietet eine lokale Monitor-Oberfläche. Diese darf geschlossen werden, ohne den Dienst zu stoppen.

### C. Lokaler Agent / Werkzeugkasten
- Codex, anderer KI-Agent oder ein deterministischer Runner;
- kompiliert, programmiert, testet, misst oder erzeugt Dokumente;
- verwendet lokale Werkzeuge und Geräte nach expliziter Freigabe;
- schreibt reproduzierbare Logs, Rohdaten und Ergebnisberichte;
- kann bei erlaubtem Umfang Code ändern und PRs vorbereiten, jedoch nicht automatisch ohne Freigabe mergen.

## 6. Bridge-Verzeichnis und Ressourcenprofil

Damit der Entwickler aus ChatGPT einen Zielrechner auswählen kann, muss es ein **für das jeweilige GitHub-Projekt einsehbares Verzeichnis** geben:

- stabile Bridge-ID und Anzeigename;
- Online-/Offline-Status und Zeitpunkt der letzten Meldung;
- Betriebssystem und grundlegende Rechnerfähigkeiten;
- installierte und freigegebene Agenten/Runner;
- verfügbare Toolchains, SDKs, Compiler und Debugger;
- erkannte Messgeräte und Schnittstellen, soweit freigegeben;
- freigegebene Projektarbeitsverzeichnisse bzw. Repository-Zuordnungen;
- belegte/freie Ressourcen und laufende Aufträge.

**Wichtig:** Automatisch gefundene Hardware ist nicht automatisch zur Benutzung freigegeben. Der Benutzer entscheidet, welche Ressourcen angeboten werden. Private Pfade, Seriennummern, IP-Adressen oder Sicherheitsdetails dürfen nicht versehentlich in öffentliche Repositories geschrieben werden.

**Offen:** In welchem GitHub-Mechanismus die Bridge-Registrierung und der dynamische Status sinnvoll gespeichert werden (z. B. Actions/Runner-Labels, API, Issues, geeigneter Statusdienst). Nicht voreilig eine eigene zentrale Datenbank oder einen zusätzlichen Server erfinden.

## 7. Monitorprogramm

Der Benutzer hat ein **sichtbares, separates Monitorprogramm ausdrücklich gefordert**. Das ist eine Kernanforderung, kein spätes optionales Nice-to-have.

Mindestansicht:
- Bridge-ID, Verbindung zu GitHub, letzte Kommunikation;
- laufender Auftrag, Projekt/Issue/Job-ID, lokaler Agent;
- Auftragsphase, Fortschritt soweit bestimmbar, Laufzeit, letzter Aktivitätszeitpunkt;
- Ereignisprotokoll, stdout/stderr, Fehlermeldungen;
- vorhandene/freigegebene Werkzeuge, Ressourcenbelegung;
- kontrollierter Abbruch und ggf. Wiederaufnahme, sofern technisch unterstützt.

Das Monitorprogramm ist eine **Oberfläche für den Hintergrunddienst**, nicht selbst der Dienst. Für die Fernüberwachung muss der Status zusätzlich über GitHub abrufbar sein; das lokale Fenster darf dafür nicht offen bleiben.

## 8. GitHub Actions / Self-hosted Runner – bewusst noch nicht entschieden

Die Diskussion geriet zwischenzeitlich ins Stocken, weil zwei Konzepte vermischt wurden:

- **GitHub Self-hosted Runner:** Bereits existierender GitHub-Dienst auf einem eigenen Rechner, der GitHub-Actions-Jobs lokal ausführt. Könnte als vorhandener Mechanismus für die Jobzustellung dienen.
- **Unsere Bridge:** Zusätzliche Steuerung lokaler Agenten, Ressourcen, längerer Aufträge, Monitoring und Ergebnisrückgabe.

Die beiden sind **nicht zwangsläufig Alternativen**; sie könnten kombiniert werden. Ob das technisch, sicherheitlich und hinsichtlich längerer Agentensitzungen sinnvoll ist, muss **durch Recherche und einen kleinen Machbarkeitstest** geprüft werden.

**Keine Architekturentscheidung für oder gegen Self-hosted Runner ist bisher getroffen.** Auch das genaue GitHub-Auftragsprotokoll ist offen.

## 9. Cloud-Agent und lokale Agenten

Das Template soll **bereits eine funktionsfähige cloudseitige Agentenintegration** bereitstellen, vergleichbar mit dem OpenHands-Workflow, der bei MBPPL über GitHub orchestriert wurde.

Der Cloud-Agent kann:
- Issues bearbeiten und Code analysieren;
- Änderungen entwickeln und PRs erstellen;
- definierte lokale Tests über die Bridge anfordern;
- Ergebnisse auswerten und Folgearbeiten vorschlagen.

Der lokale Agent wird vom jeweiligen Rechner angeboten und über die Bridge gestartet. Codex ist der erste denkbare Kandidat; die Schnittstelle muss vorab geprüft werden. Ein VS-Code-Fenster ist nicht vorausgesetzt. Andere Agenten können später über Adapter integriert werden.

**Grenze:** Es ist nicht vorauszusetzen, dass eine bereits interaktiv laufende VS-Code-Codex-Sitzung aus der Ferne übernommen oder nahtlos fortgesetzt werden kann. Dies ist gesondert zu verifizieren. Modell- und Agentenwechsel dürfen nicht als bereits implementiert dargestellt werden.

## 10. Beispiel eines vollständigen Auftrags

**Auftrag:** „Im Projekt MBPPL den Test T01-B2 auf dem Keller-PC durchführen und die Messdaten dokumentieren.“

1. Benutzer formuliert den Auftrag in ChatGPT und wählt ausdrücklich `keller-pc`.
2. Ein verfügbarer GitHub-Connector/Workflow legt den Auftrag im Projekt ab.
3. GitHub/Cloud-Agent prüft Testbeschreibung, Berechtigungen und Voraussetzungen.
4. Der an `keller-pc` adressierte Auftrag erreicht die dortige Bridge.
5. Die Bridge validiert Projekt, Rechte, lokale Ressourcen und Idempotenzkennung.
6. Die Bridge startet den lokalen Agenten oder direkt einen deterministischen Test-Runner.
7. Der lokale Runner kompiliert/flasht, steuert die Geräte, erfasst Messdaten und bewertet sie.
8. Die Bridge überwacht Zeitlimits, Heartbeats, Fehler und die Ressourcensperren.
9. JSON-Bericht, Logs, Waveforms, Konfigurationen, Prüfsummen und relevante Firmware-/Commit-IDs werden archiviert.
10. GitHub erhält nachvollziehbaren Status und Artefaktreferenzen; der Cloud-Agent kann die Ergebnisse analysieren.
11. Benutzer kann die Ergebnisse von einem anderen Gerät prüfen und weitere Aufträge freigeben.

Nicht jede Einzeloperation darf einen KI-Agentenaufruf oder eine GitHub-Rundreise erfordern. Ein lokales Skript sollte einen kompletten, klar definierten Test möglichst autonom ausführen.

## 11. Zuverlässigkeit, Sicherheit und Nachweise

Aus den MBPPL-Erfahrungen ausdrücklich abzuleiten:

- **Keine endlosen Agentenläufe:** harte Zeitlimits pro Job und Schritt, Heartbeats, Status „wartet“, „arbeitet“, „blockiert“ und „Fehler“.
- **Wiederaufnahme statt blindem Neustart:** Netzwerkunterbrechung, Prozess- oder Rechnerneustart dürfen weder zu Datenverlust noch zu doppelten Flash-/Mess-/Commit-Aktionen führen.
- **Idempotenz und Persistenz:** eindeutige Job-ID, Zustandsjournal, bestätigte Schritte, Abgleich vor Wiederholung.
- **Ressourcen-Sperren:** ein Gerät oder Repository nicht unkoordiniert parallel bearbeiten.
- **Kontrollierte Rechte:** Lesen, Entwickeln, Testen, Flashen, Commit/Push, Merge und gefährliche Hardwareaktionen getrennt autorisieren.
- **Keine beliebigen Shell-Befehle aus untrusted GitHub-Inhalten**; Eingaben validieren und Berechtigungen begrenzen.
- **GitHub als dauerhafter Nachweis:** Quellcode, Berichte, Rohdaten bzw. Referenzen und Metadaten müssen für andere Agenten verfügbar sein.
- **Artefaktstrategie:** geeignete kleine Dateien im Repository; große Binärdateien ggf. LFS/Artefaktspeicher nach Prüfung; kein stilles Weglassen von Rohdaten.
- **Lokale Sicherheit:** Testgrenzen, sichere Ausgangszustände, Not-Stopp und Hardwarefreigaben.
- **Keine automatischen Merges oder Releases** ohne festgelegte Freigabe.
- **Private und berufliche Entwicklungsumgebungen getrennt halten.**

## 12. Was wir bewusst NICHT bauen wollen

- keine eigene GitHub-/CI-Plattform;
- keinen neuen KI-Agenten als Bridge-Kern;
- keine direkte Bridge-zu-Bridge-Kommunikation;
- keine automatische Rechnerwahl in der ersten Version;
- keine riesige Ressourcen-Discovery vor dem ersten funktionsfähigen Test;
- keine fest verdrahtete Abhängigkeit von VS Code, Codex, Rigol oder RP2040;
- keine Vollautomatisierung ohne Rechte- und Freigabekonzept;
- keine Vermischung mit MBPPL-Firmwareentwicklung in diesem Architektur-Chat.

## 13. Arbeitsweise und nächste Schritte

**Die Reihenfolge ist bewusst geändert worden:** Erst prüfen, welche etablierten Lösungen bereits existieren; dann die eigenen Komponenten so klein wie möglich halten.

### Schritt 1 – Markt-/Technikvergleich (als Nächstes)
Kurz und konkret untersuchen:
- GitHub Actions und Self-hosted Runner;
- GitHub Issues/PRs als Auftrags- und Rückmeldekanal;
- OpenHands bzw. vorhandene Cloud-Agenten-Workflows;
- Codex CLI/SDK und vergleichbare lokale Agenten;
- etablierte HIL-/Laborautomatisierung;
- vorhandene Lösungen für Job-Monitoring, Artefakte und Wiederaufnahme.

Für jeden Baustein: **Was kann er bereits? Was fehlt uns? Müssen wir überhaupt etwas Eigenes entwickeln?** Keine pauschale Annahme, dass alle Funktionen ohne Einschränkungen kombinierbar sind.

### Schritt 2 – Architekturzeichnung und Zuständigkeitstabelle
Eine **übersichtliche, gemeinsame Systemzeichnung** erstellen. Kennzeichnen:
- bestehende Systeme, die wir nur konfigurieren;
- eigene Komponenten, die wir tatsächlich entwickeln müssen;
- offene Entscheidungen und Risiken.

### Schritt 3 – Ein einziger End-to-End-Machbarkeitstest
Minimaler Ablauf:
**ChatGPT → GitHub-Projekt → explizit gewählter Rechner → lokaler Runner/Agent → Status/Ergebnis zurück nach GitHub.**

Für den ersten Durchlauf genügt ein harmloses, deterministisches Skript ohne angeschlossene Laborhardware. Danach erst Codex und einen echten MBPPL-Test integrieren.

### Schritt 4 – Template standardisieren
Erst auf Basis der erfolgreichen Machbarkeit:
- minimale Projektstruktur;
- Agentenregeln und GitHub-Workflows;
- standardisierte Job-/Resultatsformate;
- Bridge-Konfiguration/Anbindung;
- kurze Einrichtung und Akzeptanztests.

### Schritt 5 – MB-AgentBridge daraus entwickeln
Eigenständiger Dienst plus Monitorprogramm, zunächst auf einem Rechner; danach zweiter Rechner, Ressourcenprofil und mehrere Agenten. Bestehendes Konzept im Bridge-Repository dabei überprüfen und an den neuen Architekturstand anpassen.

## 14. Entscheidungen versus offene Fragen

### Bereits gemeinsam festgelegt
- GitHub-Projektrepository ist der logische zentrale Knotenpunkt.
- Keine direkte Kommunikation zwischen Bridges.
- Zielrechner wird vom Benutzer/ChatGPT explizit gewählt.
- Bridge als schlanker Hintergrunddienst, mit eigenständigem Monitorprogramm.
- Lokale Agenten austauschbar, VS Code nicht verpflichtend.
- Cloud-Agenten-Workflow gehört zur Template-Grundausstattung.
- Template universell, aber modular und sehr schlank.
- Bestehende Lösungen bevorzugen, keine Infrastruktur doppelt erfinden.
- Architektur und Machbarkeit **vor** großer Implementierung.
- MBPPL-Projektarbeit und Infrastrukturentwicklung in getrennten Chats.

### Noch offen und nicht stillschweigend entscheiden
- GitHub Self-hosted Runner als Transportweg: ja/nein/Hybrid?
- Auftrags-/Statusformat, Registrierung und Sichtbarkeit der Bridges.
- Authentifizierung und Rechteverwaltung über mehrere Projekte.
- Technische Agentenadapter und Grenzen von Sitzungspersistenz.
- Monitor-UI-Technologie und lokaler Zugriff.
- Artefaktablage und Größenlimits.
- Wie Template-Änderungen in bereits existierende Projekte gelangen.
- Welche lokalen Werkzeuge der erste Pilot wirklich benötigt.

## 15. Auftrag für den nächsten Chat

**Thema:** `MB-AI-ProjectTemplate – Gesamtarchitektur und vorhandene Lösungen`.

Den neuen Chat mit dieser Datei beginnen. **Nicht sofort programmieren.** Zuerst die vorhandenen Lösungen systematisch mit den Anforderungen vergleichen, anschließend eine einfache Architekturzeichnung und Zuständigkeitstabelle erstellen. **Kleine Schritte, ein Thema nach dem anderen.** Keine zusätzlichen Projekte oder unnötigen Features vorschlagen. Entscheidungen mit dem Entwickler abstimmen, statt die Architektur ungefragt festzulegen.

**Leitsatz:** Nicht mehr KI und Infrastruktur um ihrer selbst willen, sondern weniger Warten, weniger manuelle Betreuung und nachweisbar schnellere Entwicklung.
