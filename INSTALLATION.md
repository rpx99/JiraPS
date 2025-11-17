# JiraPS Installation mit API v3 Unterstützung

## Voraussetzungen

- Windows PowerShell 5.1 oder höher ODER PowerShell Core 6.0+
- Windows, macOS oder Linux

## Installationsmethoden

### Methode 1: Lokale Installation aus diesem Repository (Empfohlen für API v3)

Diese Methode installiert die aktualisierte Version mit API v3 Unterstützung direkt aus diesem Repository.

#### Schritt 1: Repository klonen oder herunterladen

```powershell
# Option A: Mit Git klonen
git clone https://github.com/rpx99/JiraPS.git
cd JiraPS

# Option B: Als ZIP herunterladen und entpacken
# Lade das Repository von GitHub herunter und entpacke es
```

#### Schritt 2: Zum API v3 Branch wechseln

```powershell
git checkout claude/upgrade-jira-api-v3-019fSPSQ2x9F7ivyKTSQrFNq
```

#### Schritt 3: Modul installieren

```powershell
# Finde das PowerShell Module Verzeichnis
$modulePath = $env:PSModulePath -split ';' | Select-Object -First 1

# Erstelle JiraPS Verzeichnis falls nicht vorhanden
$jiraPSPath = Join-Path $modulePath "JiraPS"
New-Item -ItemType Directory -Path $jiraPSPath -Force

# Kopiere die Modul-Dateien
Copy-Item -Path ".\JiraPS\*" -Destination $jiraPSPath -Recurse -Force

Write-Host "JiraPS wurde erfolgreich nach $jiraPSPath installiert" -ForegroundColor Green
```

#### Schritt 4: Modul importieren und verwenden

```powershell
# Modul importieren
Import-Module JiraPS -Force

# Version überprüfen
Get-Module JiraPS

# Jira-Session erstellen
New-JiraSession -Server 'https://your-domain.atlassian.net' -Credential (Get-Credential)

# Issues suchen (verwendet jetzt API v3)
Get-JiraIssue -Query "project = TEST AND status = Open"
```

### Methode 2: Installation via PowerShell Gallery (Nur für stabile Version)

**HINWEIS:** Die PowerShell Gallery Version enthält noch NICHT die API v3 Updates. Verwende Methode 1 für API v3 Unterstützung.

```powershell
Install-Module JiraPS -Scope CurrentUser
```

## Wichtige Änderungen in der API v3 Version

### Was wurde geändert?

1. **Search Endpoint**: `/rest/api/2/search` → `/rest/api/3/search/jql`
   - Verwendet jetzt POST statt GET
   - JSON Body statt Query Parameter
   - Token-basierte Paginierung (`nextPageToken`) statt Offset (`startAt`)

2. **Alle anderen Endpoints**: `/rest/api/2/*` → `/rest/api/3/*`

3. **Paginierung**: Automatische Unterstützung für beide Paginierungsmethoden
   - Neue API v3 `/search/jql`: Token-basiert
   - Legacy Endpoints: Offset-basiert (falls noch unterstützt)

### Kompatibilität

- ✅ Alle bestehenden Befehle funktionieren weiterhin
- ✅ Keine Änderungen an der PowerShell Syntax erforderlich
- ✅ Automatische Erkennung des Paginierungstyps
- ✅ Abwärtskompatibel mit Jira Cloud (ab August 2025)

## Beispiele

### Basis-Verwendung

```powershell
# Jira-Session erstellen
$cred = Get-Credential
New-JiraSession -Server 'https://your-domain.atlassian.net' -Credential $cred

# Einzelnes Issue abrufen
Get-JiraIssue -Key "PROJECT-123"

# Issues mit JQL suchen
Get-JiraIssue -Query "project = MYPROJECT AND assignee = currentUser()"

# Mit Paginierung (alle Issues abrufen)
Get-JiraIssue -Query "project = MYPROJECT" -PageSize 50

# Nur bestimmte Felder abrufen
Get-JiraIssue -Query "project = MYPROJECT" -Fields "summary,status,assignee"
```

### Erweiterte Beispiele

```powershell
# Issues mit Filter
$filter = Get-JiraFilter -Id 12345
Get-JiraIssue -Filter $filter

# Issues erstellen
$issue = New-JiraIssue -Project "TEST" -IssueType "Task" -Summary "Neue Aufgabe"

# Benutzer suchen
Get-JiraUser -UserName "john.doe@example.com"

# Projekt-Informationen
Get-JiraProject -Key "MYPROJECT"
```

## Fehlerbehebung

### Problem: "The requested API has been removed"

**Lösung:** Stelle sicher, dass du die aktualisierte Version aus diesem Branch verwendest, nicht die alte Version von PowerShell Gallery.

### Problem: Modul kann nicht importiert werden

```powershell
# Prüfe die installierten Module
Get-Module -ListAvailable JiraPS

# Entferne alte Versionen
Get-Module JiraPS | Remove-Module -Force

# Deinstalliere alte Versionen von PowerShell Gallery
Uninstall-Module JiraPS -AllVersions

# Installiere die aktualisierte Version neu (siehe Methode 1)
```

### Problem: Paginierung funktioniert nicht richtig

Die neue API v3 hat einige bekannte Probleme mit der Paginierung. Falls Probleme auftreten:

```powershell
# Verwende eine größere PageSize (max 100)
Get-JiraIssue -Query "project = MYPROJECT" -PageSize 100

# Oder begrenze die Anzahl der Ergebnisse
Get-JiraIssue -Query "project = MYPROJECT" -First 1000
```

## Deinstallation

```powershell
# Modul entfernen
Remove-Module JiraPS -Force

# Von PowerShell Gallery installierte Version deinstallieren
Uninstall-Module JiraPS -AllVersions

# Manuell installierte Version löschen
$modulePath = (Get-Module JiraPS -ListAvailable).ModuleBase
Remove-Item $modulePath -Recurse -Force
```

## Weitere Ressourcen

- [JiraPS Dokumentation](https://atlassianps.org/docs/JiraPS/)
- [GitHub Issues](https://github.com/AtlassianPS/JiraPS/issues)
- [Atlassian API Dokumentation](https://developer.atlassian.com/cloud/jira/platform/rest/v3/)
- [API v2 zu v3 Migration](https://community.atlassian.com/forums/Jira-articles/Your-Jira-Scripts-and-Automations-May-Break-if-they-use-JQL/ba-p/3001235)
