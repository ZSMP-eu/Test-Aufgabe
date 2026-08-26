# Developer-Testaufgabe – ZSMP.eu

## Advanced Daily Rewards & Streak System

Entwickle ein Daily-Rewards-System für unser SMP-Netzwerk. Spieler sollen einmal pro Tag eine konfigurierbare Belohnung abholen können. Für aufeinanderfolgende Login-Tage erhalten sie eine steigende Streak. Verpasste Tage setzen die Streak zurück.

Wir bewerten nicht nur den Funktionsumfang, sondern besonders Architektur, Persistenz, Zeitberechnung, Performance und Schutz vor mehrfachen Claims.

## Technische Vorgaben

- Java 21
- Paper API 1.21.11
- Maven oder Gradle
- MySQL oder MariaDB
- LuckPerms für Berechtigungen
- Adventure API/MiniMessage
- Git-Repository mit README und nachvollziehbarer Commit-Historie
- kein NMS
- keine blockierenden Datenbankzugriffe auf dem Main Thread
- Bukkit-/Paper-API-Aufrufe nur auf dem richtigen Thread
- sauberes Schließen von Datenbankverbindungen und Tasks

## Pflichtumfang

### 1. Spielerfunktion

Implementiere:

```text
/daily
```

Der Befehl öffnet eine GUI mit mindestens sieben konfigurierbaren Belohnungstagen. Die GUI muss anzeigen:

- aktuellen Streak
- heutigen Belohnungsstatus
- bereits abgeholte Tage
- nächste Belohnung
- verbleibende Zeit bis zum nächsten Claim

Der Spieler darf die Tagesbelohnung nur einmal innerhalb des konfigurierten Tagesintervalls abholen.

### 2. Streak-Logik

- Erster Claim startet die Streak bei Tag 1.
- Ein Claim am nächsten gültigen Tag erhöht die Streak.
- Wird ein kompletter Tag verpasst, beginnt die Streak wieder bei Tag 1.
- Nach dem letzten konfigurierten Tag beginnt der Zyklus erneut.
- Ein Serverneustart darf den Status nicht zurücksetzen.
- Zeitzone und Tageswechsel müssen sauber dokumentiert und konfigurierbar sein.
- Zeit nicht anhand der Client-Uhr berechnen.

### 3. Belohnungen

Mindestens folgende Reward-Typen sollen unterstützt werden:

- Items
- Commands
- Kombination mehrerer Rewards

Beispiel:

```yaml
rewards:
  1:
    items:
      - material: IRON_INGOT
        amount: 8
  2:
    commands:
      - "eco give %player% 250"
  3:
    items:
      - material: DIAMOND
        amount: 1
```

Kann eine Belohnung wegen eines vollen Inventars nicht vollständig ausgegeben werden, darf sie nicht verloren gehen. Das Verhalten muss sinnvoll gelöst und dokumentiert werden.

### 4. Persistenz

Speichere mindestens:

```text
playerUuid
currentStreak
lastClaimAt
lastClaimDay
totalClaims
```

Daten müssen nach Neustarts erhalten bleiben. Speichere Änderungen zuverlässig beim Claim oder bei einer Streak-Änderung. Ein periodischer Speicherintervall allein reicht nicht aus.

### 5. Admin-Funktionen

Mit `zsmp.daily.admin`:

```text
/dailyadmin reset <spieler>
/dailyadmin info <spieler>
```

Mit `zsmp.daily.reload`:

```text
/dailyadmin reload
```

Alle Befehle benötigen vollständige Permission- und Eingabeprüfungen.

## Konfiguration

Mindestens konfigurierbar:

- Belohnungstage und Reward-Typen
- Tageswechsel/Zeitzone
- Nachrichten und GUI-Texte
- Materialien, Mengen und Commands
- Inventar-voll-Verhalten
- Permissions
- Datenbankverbindung
- Anzeige eines Countdowns

Alle sichtbaren Texte müssen MiniMessage unterstützen.

## Architektur und Sicherheit

Trenne mindestens Command, GUI, Geschäftslogik, Repository/Storage und Konfiguration. Datenbankzugriffe gehören nicht in GUI- oder Event-Klassen.

Die Lösung muss verhindern, dass ein Spieler durch doppelte Klicks, parallele Requests, Reconnects oder Serverneustarts mehrfach dieselbe Belohnung erhält. Kritische Claims müssen atomar oder anderweitig race-condition-sicher verarbeitet werden.

## Tests

Teste mindestens:

1. erster Claim startet Tag 1;
2. ein zweiter Claim am selben Tag wird abgelehnt;
3. der nächste gültige Tag erhöht die Streak;
4. ein verpasster Tag setzt die Streak zurück;
5. der Status bleibt nach Neustart erhalten;
6. doppelte Claims durch schnelle Klicks sind nicht möglich;
7. volle Inventare führen nicht zum Verlust der Belohnung;
8. Admin-Befehle sind korrekt geschützt.

## Bonusaufgaben

- PlaceholderAPI-Unterstützung
- `/daily top` mit Claim-Statistiken
- mehrere Reward-Zyklen für verschiedene Gruppen
- tägliche Benachrichtigung beim Join
- serverübergreifende Synchronisierung über Redis
- Tests mit Testcontainers

## Abgabe

Das Repository muss enthalten:

- vollständigen Quellcode
- funktionierenden Build
- README mit Setup, Installation, Konfiguration und Architektur
- SQL-Migration oder automatische Tabellenerstellung
- Tests oder nachvollziehbare manuelle Testdokumentation
- bekannte Einschränkungen und bewusste Vereinfachungen

Keine echten Zugangsdaten committen.

## Bewertung – 100 Punkte

| Bereich | Punkte |
|---|---:|
| Funktionalität und Streak-Logik | 25 |
| Persistenz und Datenbanksicherheit | 20 |
| Architektur und Wartbarkeit | 20 |
| Async-Verarbeitung, Performance und Thread-Sicherheit | 15 |
| GUI, Commands und Konfiguration | 10 |
| Tests und Dokumentation | 10 |
| **Gesamt** | **100** |

Eine kleinere, stabile und gut dokumentierte Lösung ist besser als ein überladenes System mit Datenverlust oder Exploits.

---

# Developer Assignment – ZSMP.eu

## Advanced Daily Rewards & Streak System

Build a Daily Rewards system for our SMP network. Players must be able to claim one configurable reward per day. Consecutive claim days increase the player’s streak; missing a complete day resets the streak.

We evaluate more than feature count. Architecture, persistence, date handling, performance, and protection against duplicate claims are especially important.

## Technical requirements

- Java 21
- Paper API 1.21.11
- Maven or Gradle
- MySQL or MariaDB
- LuckPerms for permissions
- Adventure API/MiniMessage
- Git repository with README and understandable commit history
- no NMS
- no blocking database or I/O operations on the main thread
- Bukkit/Paper API calls on the correct thread
- clean shutdown of database connections and tasks

## Required features

Implement:

```text
/daily
```

The command opens a GUI with at least seven configurable reward days. The GUI must show the current streak, today’s reward status, claimed days, the next reward, and the remaining time until the next claim.

The player may claim a daily reward only once during the configured daily interval.

The first claim starts streak day 1. A valid claim on the next day increases the streak. Missing a complete day resets the streak to day 1. After the final configured day, the cycle starts again. A restart must not reset player data. Timezone and day-boundary behavior must be documented and configurable. Never rely on the client clock.

Support at least items, commands, and combinations of multiple rewards. If a reward cannot fit into a full inventory, it must not be lost; document your chosen solution.

Persist at least:

```text
playerUuid, currentStreak, lastClaimAt, lastClaimDay, totalClaims
```

Save changes reliably when a claim or streak change occurs. A periodic save interval alone is not sufficient.

## Admin commands

With `zsmp.daily.admin`:

```text
/dailyadmin reset <player>
/dailyadmin info <player>
```

With `zsmp.daily.reload`:

```text
/dailyadmin reload
```

All commands require complete permission and input validation.

## Configuration

Rewards, reward types, timezone/day boundary, messages, GUI texts, item amounts, commands, full-inventory behavior, permissions, database settings, and countdown display must be configurable. All visible texts must support MiniMessage.

## Architecture and security

Separate commands, GUI, business logic, repository/storage, and configuration. Database access must not be placed inside GUI or event classes.

Prevent duplicate rewards caused by double clicks, parallel requests, reconnects, or restarts. Critical claims must be atomic or otherwise race-condition safe.

## Tests

Test the first claim, duplicate same-day claims, streak increases, missed-day resets, restart persistence, rapid double claims, full inventories, and admin permission checks.

## Optional bonus features

PlaceholderAPI, `/daily top`, multiple reward cycles, join notifications, Redis synchronization, or Testcontainers.

## Submission

Submit the complete source code, a working build, README with setup/configuration/architecture, SQL migration or automatic table creation, tests or manual test documentation, and known limitations. Never commit real credentials.

## Evaluation – 100 points

| Area | Points |
|---|---:|
| Functionality and streak logic | 25 |
| Persistence and database safety | 20 |
| Architecture and maintainability | 20 |
| Async processing, performance and thread safety | 15 |
| GUI, commands and configuration | 10 |
| Tests and documentation | 10 |
| **Total** | **100** |

A smaller, stable, and well-documented solution is better than an overloaded system with data loss or exploitable behavior.
