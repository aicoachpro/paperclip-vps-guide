# Paperclip auf einem VPS installieren — Anleitung fuer Anfaenger

Diese Anleitung erklaert dir **Schritt fuer Schritt**, wie du Paperclip auf einem eigenen Server im Internet installierst. Du brauchst keine Vorkenntnisse — nur einen Computer und etwas Geduld.

---

## Was bauen wir hier eigentlich?

**Paperclip** ist eine Plattform, die KI-Agenten organisiert wie ein Unternehmen. Stell dir vor, du gruendest eine Firma, aber statt Menschen stellst du KI-Mitarbeiter ein:

- Ein **CEO-Agent** leitet die Firma, schreibt Plaene und stellt Mitarbeiter ein
- Ein **Engineer-Agent** programmiert Software
- Jeder Agent hat ein Budget, eine Rolle und Aufgaben

Das Ganze laeuft auf einem Server im Internet (einem "VPS"), damit die Agenten rund um die Uhr arbeiten koennen — auch wenn dein Computer aus ist.

**Links:**
- Website: https://paperclip.ing
- Dokumentation: https://docs.paperclip.ing
- Quellcode: https://github.com/paperclipai/paperclip (kostenlos, Open Source)

---

## Was du brauchst (Einkaufsliste)

Bevor es losgeht, brauchst du drei Dinge:

### 1. Einen VPS (virtueller Server)

Ein VPS ist ein Computer im Internet, den du mietest. Anbieter sind z.B.:
- **Hostinger** (ab ca. 5 EUR/Monat) — https://www.hostinger.de
- **Hetzner** (ab ca. 4 EUR/Monat) — https://www.hetzner.com
- **DigitalOcean** (ab ca. 6 USD/Monat) — https://www.digitalocean.com

**Beim Bestellen waehle:**
- Betriebssystem: **Ubuntu 24.04**
- RAM: mindestens **2 GB** (4 GB empfohlen)
- Speicher: mindestens **20 GB**

Nach der Bestellung bekommst du:
- Eine **IP-Adresse** (z.B. `123.45.67.89`)
- Einen **Benutzernamen** (meistens `root`)
- Ein **Passwort**

Schreib dir diese drei Dinge auf — du brauchst sie gleich.

### 2. Ein Claude-Abo (Max oder API)

Die KI-Agenten in Paperclip brauchen ein "Gehirn". Dafuer nutzen wir Claude von Anthropic.

**Option A — Claude Max-Abo (empfohlen):**
- Kostet ca. 100 USD/Monat (Flatrate, keine Extra-Kosten)
- Abschliessen unter: https://claude.ai
- Du brauchst dort auch **Claude Code** installiert auf deinem Computer

**Option B — Anthropic API-Key:**
- Bezahlung nach Verbrauch
- API-Key erstellen unter: https://console.anthropic.com

### 3. Ein Terminal-Programm

Ein Terminal ist ein Fenster, in das du Befehle tippst. Klingt altmodisch, ist aber der einfachste Weg einen Server zu steuern.

- **Mac:** Das Programm "Terminal" ist vorinstalliert. Oeffne es ueber Spotlight (Cmd + Leertaste, dann "Terminal" tippen).
- **Windows:** Lade "Windows Terminal" aus dem Microsoft Store oder nutze "PowerShell" (vorinstalliert).
- **Linux:** Terminal ist vorinstalliert.

---

## Inhalt

| Teil | Thema | Dauer |
|------|-------|-------|
| [Teil 1](#teil-1-mit-dem-server-verbinden) | Mit dem Server verbinden | 5 Min |
| [Teil 2](#teil-2-den-server-vorbereiten) | Den Server vorbereiten | 10 Min |
| [Teil 3](#teil-3-paperclip-installieren) | Paperclip installieren | 10 Min |
| [Teil 4](#teil-4-paperclip-dauerhaft-laufen-lassen) | Paperclip dauerhaft laufen lassen | 5 Min |
| [Teil 5](#teil-5-firewall-einrichten) | Firewall einrichten | 5 Min |
| [Teil 6](#teil-6-claude-code-mit-paperclip-verbinden) | Claude Code verbinden | 10 Min |
| [Teil 7](#teil-7-im-browser-anmelden) | Im Browser anmelden | 5 Min |
| [Teil 8](#teil-8-push-benachrichtigungen-optional) | Push-Benachrichtigungen (optional) | 10 Min |
| [Teil 9](#teil-9-wartung-und-problemloesung) | Wartung und Problemloesung | Nachschlagewerk |

---

## Teil 1: Mit dem Server verbinden

### Was passiert hier?

Du verbindest dich von deinem Computer aus mit dem gemieteten Server. Das nennt man eine "SSH-Verbindung" — eine verschluesselte Leitung zwischen deinem Computer und dem Server.

### So geht's

1. Oeffne das **Terminal** auf deinem Computer
2. Tippe folgenden Befehl ein (ersetze die Platzhalter durch deine echten Daten):

```bash
ssh root@DEINE-IP-ADRESSE
```

Beispiel: Wenn deine IP `123.45.67.89` ist, tippst du:
```bash
ssh root@123.45.67.89
```

3. Beim ersten Mal fragt das Terminal: `Are you sure you want to continue connecting?` — Tippe `yes` und druecke Enter.

4. Gib dein **Passwort** ein. Wichtig: Beim Tippen werden **keine Zeichen angezeigt** — das ist normal und ein Sicherheitsfeature. Einfach blind tippen und Enter druecken.

### Checkpoint

Du siehst jetzt etwas wie:
```
root@server:~#
```
Das bedeutet: Du bist auf dem Server eingeloggt. Alles was du jetzt tippst, passiert auf dem Server, nicht auf deinem Computer.

> **Tipp:** Um die Verbindung zu beenden, tippe `exit` und druecke Enter. Du kannst dich jederzeit erneut verbinden.

---

## Teil 2: Den Server vorbereiten

### Was passiert hier?

Wir installieren die Software, die Paperclip zum Laufen braucht. Das ist wie das Einrichten eines neuen Computers — Betriebssystem aktualisieren, Programme installieren.

### Schritt 1: System aktualisieren

Dieser Befehl aktualisiert alle Programme auf dem Server auf die neueste Version:

```bash
apt update && apt upgrade -y
```

Das kann ein paar Minuten dauern. Warte bis wieder `root@server:~#` erscheint.

### Schritt 2: Node.js installieren

Node.js ist die Programmierumgebung, in der Paperclip laeuft. Vergleichbar mit Java fuer andere Programme.

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs
```

### Schritt 3: PostgreSQL-Client installieren

Paperclip speichert Daten in einer Datenbank. Wir installieren ein Werkzeug, um spaeter in die Datenbank schauen zu koennen:

```bash
apt install -y postgresql-client-16
```

### Checkpoint

Pruefe ob alles installiert ist:

```bash
node -v
npm -v
```

Du solltest Versionsnummern sehen, z.B. `v20.20.0` und `10.8.2`. Die genauen Zahlen koennen abweichen — wichtig ist, dass keine Fehlermeldung kommt.

---

## Teil 3: Paperclip installieren

### Was passiert hier?

Wir erstellen einen eigenen Benutzer fuer Paperclip (aus Sicherheitsgruenden) und installieren dann die Software.

### Schritt 1: Paperclip-Benutzer anlegen

Aus Sicherheitsgruenden sollte Paperclip nicht als "root" (Administrator) laufen. Wir erstellen einen eigenen Benutzer:

```bash
adduser --system --group --home /home/paperclip --shell /bin/bash paperclip
```

### Schritt 2: Paperclip installieren

Jetzt installieren wir Paperclip. Der folgende Befehl laedt alles herunter und richtet es ein:

```bash
sudo -u paperclip bash -c 'cd /home/paperclip && npx paperclipai onboard --yes'
```

> **Was bedeutet `sudo -u paperclip`?** Damit sagen wir dem Server: "Fuehre den folgenden Befehl als Benutzer 'paperclip' aus, nicht als root."

Das dauert ein paar Minuten. Paperclip erstellt dabei automatisch:
- Eine Datenbank fuer alle Daten
- Eine Konfigurationsdatei
- Einen CEO-Agent (deinen ersten KI-Mitarbeiter)

### Schritt 3: Konfiguration anpassen

Jetzt muessen wir Paperclip sagen, dass es von ueberall erreichbar sein soll (nicht nur vom Server selbst). Oeffne die Konfigurationsdatei:

```bash
nano /home/paperclip/.paperclip/instances/default/config.json
```

> **Was ist `nano`?** Ein einfacher Texteditor im Terminal. Du kannst darin mit den Pfeiltasten navigieren.

Suche den Bereich `"server"` und stelle sicher, dass dort steht:

```json
"server": {
    "deploymentMode": "authenticated",
    "exposure": "public",
    "host": "0.0.0.0",
    "port": 3100,
    "serveUi": true
}
```

Suche dann den Bereich `"auth"` und aendere ihn zu:

```json
"auth": {
    "baseUrlMode": "explicit",
    "publicBaseUrl": "http://DEINE-IP-ADRESSE:3100",
    "disableSignUp": false
}
```

> **Wichtig:** Ersetze `DEINE-IP-ADRESSE` durch die echte IP deines Servers (z.B. `http://123.45.67.89:3100`).

**Speichern und schliessen:**
1. Druecke `Strg + O` (das ist der Buchstabe O, nicht Null), dann Enter zum Speichern
2. Druecke `Strg + X` zum Schliessen

### Checkpoint

Pruefe ob die Konfiguration gueltig ist:

```bash
sudo -u paperclip npx paperclipai doctor
```

Du solltest `1 passed, 0 failed` sehen. Falls Fehler angezeigt werden, pruefe nochmal die Konfigurationsdatei — meistens fehlt ein Komma oder Anfuehrungszeichen.

---

## Teil 4: Paperclip dauerhaft laufen lassen

### Was passiert hier?

Wenn du Paperclip einfach nur startest und dann das Terminal schliesst, stoppt es wieder. Wir richten es so ein, dass es:
- Automatisch startet wenn der Server hochfaehrt
- Sich selbst neu startet wenn es abstuerzt

Das nennt man einen "systemd-Service".

### Schritt 1: Service-Datei erstellen

Kopiere den folgenden Block komplett und fuege ihn ins Terminal ein:

```bash
cat > /etc/systemd/system/paperclip.service << 'EOF'
[Unit]
Description=Paperclip AI Agent Orchestration
After=network.target

[Service]
Type=simple
User=paperclip
Group=paperclip
WorkingDirectory=/home/paperclip
ExecStart=/usr/bin/npx paperclipai run
Restart=on-failure
RestartSec=10
Environment=HOME=/home/paperclip
Environment=PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

[Install]
WantedBy=multi-user.target
EOF
```

### Schritt 2: Service aktivieren und starten

```bash
systemctl daemon-reload
systemctl enable paperclip
systemctl start paperclip
```

> - `daemon-reload` = dem System sagen, dass es eine neue Service-Datei gibt
> - `enable` = beim Serverstart automatisch mitstarten
> - `start` = jetzt sofort starten

### Schritt 3: Warten und pruefen

Warte ca. 10 Sekunden, dann pruefe:

```bash
systemctl status paperclip
```

Du solltest `active (running)` in gruener Schrift sehen.

Pruefe auch, ob der Port offen ist:

```bash
ss -tlnp | grep 3100
```

Du solltest eine Zeile sehen die `0.0.0.0:3100` enthaelt. Das bedeutet: Paperclip laeuft und ist bereit fuer Verbindungen von ueberall.

### Checkpoint

| Was du siehst | Bedeutung |
|---------------|-----------|
| `active (running)` | Alles in Ordnung |
| `failed` | Etwas stimmt nicht — pruefe die Logs mit `journalctl -u paperclip -f` |
| Keine Ausgabe bei `ss` | Paperclip laeuft nicht oder hoert auf dem falschen Port |

### Nuetzliche Befehle fuer spaeter

| Was du willst | Befehl |
|---------------|--------|
| Status anzeigen | `systemctl status paperclip` |
| Stoppen | `systemctl stop paperclip` |
| Neustarten | `systemctl restart paperclip` |
| Live-Logs ansehen | `journalctl -u paperclip -f` |

> **Tipp:** Bei den Live-Logs druecke `Strg + C` um wieder rauszukommen.

---

## Teil 5: Firewall einrichten

### Was passiert hier?

Eine Firewall schuetzt deinen Server, indem sie nur bestimmte Verbindungen durchlaesst. Wir oeffnen zwei "Tueren":
- Port 22 fuer SSH (damit du dich weiterhin verbinden kannst)
- Port 3100 fuer Paperclip (damit du es im Browser oeffnen kannst)

### Schritt 1: Ports freigeben

```bash
ufw allow 22/tcp
ufw allow 3100/tcp
ufw enable
```

Bei der Frage `Command may disrupt existing SSH connections. Proceed with operation?` tippe `y` und druecke Enter.

### Schritt 2: Hostinger/Provider-Firewall (wichtig!)

Viele VPS-Anbieter haben **zusaetzlich** eine eigene Firewall in ihrem Web-Panel. Diese ist unabhaengig von der Firewall auf dem Server selbst.

**Bei Hostinger:**
1. Logge dich im Hostinger-Panel ein
2. Gehe zu VPS > dein Server > Firewall
3. Fuege eine Regel hinzu: Port `3100`, Protokoll `TCP`, Erlauben

**Bei Hetzner:**
1. Gehe in die Hetzner Cloud Console
2. Firewalls > Regel hinzufuegen: Port `3100`, TCP, Eingehend

**Bei DigitalOcean:**
1. Networking > Firewalls
2. Inbound Rule: Port `3100`, TCP

### Checkpoint

Oeffne auf deinem Computer ein **neues Terminal-Fenster** (nicht das auf dem Server!) und tippe:

```bash
curl -s http://DEINE-IP-ADRESSE:3100/api/health
```

Du solltest eine Antwort sehen die mit `{"status":"ok"` beginnt. Damit ist Paperclip von aussen erreichbar!

Falls du eine Fehlermeldung bekommst oder nichts passiert:
- `Connection refused` = Paperclip laeuft nicht (zurueck zu Teil 4)
- `Connection timed out` = Firewall blockiert (pruefe beide Firewalls nochmal)

---

## Teil 6: Claude Code mit Paperclip verbinden

### Was passiert hier?

Die Agenten in Paperclip brauchen ein KI-Modell als "Gehirn". Wir verbinden Paperclip mit Claude, damit die Agenten denken und arbeiten koennen. Wenn du ein Claude Max-Abo hast, kostet das nichts extra.

### Variante A: Mit Claude Max-Abo (empfohlen)

#### Schritt 1: Claude Code auf deinem Computer installieren

Falls noch nicht geschehen — auf deinem **lokalen Computer** (nicht auf dem Server!):

**Mac:**
```bash
npm install -g @anthropic-ai/claude-code
```

**Windows (in PowerShell als Administrator):**
```bash
npm install -g @anthropic-ai/claude-code
```

#### Schritt 2: Claude Code auf deinem Computer einloggen

Auf deinem **lokalen Computer**:

```bash
claude auth login
```

Dein Browser oeffnet sich. Melde dich mit deinem Claude-Account an und genehmige den Zugang.

#### Schritt 3: Claude Code auf dem Server installieren

Jetzt auf dem **Server** (ueber SSH):

```bash
npm install -g @anthropic-ai/claude-code
```

#### Schritt 4: Zugangsdaten vom Computer auf den Server kopieren

Auf einem Server ohne Browser funktioniert der normale Login nicht. Deshalb kopieren wir die Zugangsdaten von deinem Computer auf den Server.

**Auf deinem lokalen Computer** (neues Terminal-Fenster oeffnen):

```bash
security find-generic-password -s "Claude Code-credentials" -a "$(whoami)" -w > /tmp/claude-credentials.json
```

> Dieser Befehl liest deine Claude-Zugangsdaten aus dem Mac-Schluesselbund und speichert sie in einer temporaeren Datei.

> **Windows-Nutzer:** Auf Windows liegen die Credentials unter `%USERPROFILE%\.claude\.credentials.json`. Kopiere diese Datei direkt.

Jetzt die Datei auf den Server kopieren:

```bash
scp /tmp/claude-credentials.json root@DEINE-IP-ADRESSE:/home/paperclip/.claude/.credentials.json
```

> `scp` ist wie "Datei kopieren", aber ueber das Internet auf einen anderen Computer.

Danach die **temporaere Datei loeschen** (Sicherheit!):

```bash
rm /tmp/claude-credentials.json
```

#### Schritt 5: Berechtigungen auf dem Server setzen

Zurueck auf dem **Server** (ueber SSH):

```bash
chown paperclip:paperclip /home/paperclip/.claude/.credentials.json
chmod 600 /home/paperclip/.claude/.credentials.json
```

> - `chown` = die Datei gehoert dem paperclip-Benutzer
> - `chmod 600` = nur der Besitzer darf die Datei lesen (Sicherheit)

#### Schritt 6: Pruefen ob es funktioniert

```bash
sudo -u paperclip claude auth status
```

**Erwartete Ausgabe:**
```json
{
  "loggedIn": true,
  "authMethod": "claude.ai",
  "subscriptionType": "max"
}
```

Wenn du `"loggedIn": true` siehst — perfekt! Weiter geht's.

#### Schritt 7: Paperclip neu starten

Damit Paperclip die neue Verbindung nutzt:

```bash
systemctl restart paperclip
```

### Variante B: Mit Anthropic API-Key (alternative)

Falls du kein Max-Abo hast, kannst du einen API-Key nutzen. Dieser wird nach Verbrauch abgerechnet.

1. Erstelle einen API-Key unter https://console.anthropic.com
2. Auf dem Server:

```bash
echo 'ANTHROPIC_API_KEY=DEIN-API-KEY-HIER-EINFUEGEN' >> /home/paperclip/.paperclip/instances/default/.env
systemctl restart paperclip
```

> Ersetze `DEIN-API-KEY-HIER-EINFUEGEN` durch deinen echten Key (beginnt mit `sk-ant-`).

### Checkpoint

Warte ca. 30 Sekunden nach dem Neustart, dann:

```bash
systemctl status paperclip
```

Wenn `active (running)` steht und keine Fehler in den Logs sind (`journalctl -u paperclip --no-pager | tail -5`), ist Claude erfolgreich verbunden.

---

## Teil 7: Im Browser anmelden

### Was passiert hier?

Paperclip hat eine Weboberflaeche (wie eine Website). Dort siehst du deine KI-Firma, die Agenten und ihre Arbeit.

### Schritt 1: Paperclip im Browser oeffnen

Oeffne deinen Browser (Chrome, Firefox, Safari, ...) und gehe zu:

```
http://DEINE-IP-ADRESSE:3100
```

Beispiel: `http://123.45.67.89:3100`

### Schritt 2: Account erstellen

Du siehst eine Login-Seite. Klicke auf "Sign Up" und erstelle deinen Account mit E-Mail und Passwort.

### Schritt 3: Firma und Ziel anlegen

Nach dem Login wirst du durch ein Onboarding gefuehrt:
1. Gib deiner Firma einen Namen
2. Setze ein Ziel (z.B. "Eine Buchhaltungs-App bauen")
3. Ein CEO-Agent wird automatisch erstellt

### Schritt 4: Zusaetzliche Benutzer einladen

Wenn Kollegen auch Zugang brauchen, schicke ihnen einfach den Link `http://DEINE-IP-ADRESSE:3100`. Sie koennen sich selbst registrieren.

### Schritt 5: Registrierung schliessen

**Wichtig:** Wenn alle Benutzer registriert sind, schliesse die Registrierung. Sonst kann sich jeder mit dem Link anmelden.

Auf dem Server (ueber SSH):

```bash
sed -i 's/"disableSignUp": false/"disableSignUp": true/' /home/paperclip/.paperclip/instances/default/config.json
systemctl restart paperclip
```

### Checkpoint

Du siehst im Browser dein Paperclip-Dashboard mit:
- Deiner Firma
- Dem CEO-Agent
- Einem ersten Task ("Hire your first engineer")

Der CEO wird beim naechsten "Heartbeat" (automatischer Aktivierungszyklus, alle 30-60 Minuten) anfangen zu arbeiten — eigenstaendig Plaene schreiben, Mitarbeiter einstellen und Tasks delegieren.

---

## Teil 8: Push-Benachrichtigungen (optional)

### Was passiert hier?

Du richtest es so ein, dass du eine Nachricht auf dein Handy bekommst, wenn die Agenten etwas Wichtiges tun (z.B. einen neuen Mitarbeiter einstellen oder eine Aufgabe erledigen).

Dafuer nutzen wir den Dienst **Pushover**.

### Voraussetzungen

1. Lade die **Pushover-App** auf dein Handy (iOS oder Android, einmalig ca. 5 EUR)
2. Erstelle einen Account auf https://pushover.net
3. Erstelle eine neue App unter https://pushover.net/apps/build (Name z.B. "Paperclip")
4. Notiere dir:
   - Deinen **User Key** (auf der Pushover-Startseite)
   - Den **API Token** (auf der Seite der App die du erstellt hast)

### Schritt 1: Notification-Script erstellen

Auf dem Server (ueber SSH), erstelle eine neue Datei:

```bash
nano /home/paperclip/paperclip-notify.sh
```

Fuege folgenden Inhalt ein (ersetze die zwei Platzhalter mit deinen echten Keys):

```bash
#!/bin/bash
PUSHOVER_TOKEN="DEIN-APP-TOKEN-HIER"
PUSHOVER_USER="DEIN-USER-KEY-HIER"
STATE_FILE="/home/paperclip/.paperclip-notify-last"
DB_CMD="PGPASSWORD=paperclip psql -h 127.0.0.1 -p 54329 -U paperclip -d paperclip -t -A"

if [ -f "$STATE_FILE" ]; then
    LAST_CHECK=$(cat "$STATE_FILE")
else
    LAST_CHECK=$(date -u +"%Y-%m-%d %H:%M:%S")
fi

EVENTS=$($DB_CMD -c "SELECT action || ': ' || COALESCE(details::json->>'title', details::json->>'name', details::json->>'type', '') FROM activity_log WHERE created_at > '$LAST_CHECK' AND action IN ('agent.created', 'agent.hire_created', 'issue.updated', 'issue.created', 'approval.created', 'approval.approved') ORDER BY created_at" 2>/dev/null)

if [ -n "$EVENTS" ]; then
    MSG=$(echo "$EVENTS" | head -5)
    COUNT=$(echo "$EVENTS" | wc -l | tr -d " ")

    curl -s -X POST https://api.pushover.net/1/messages.json \
        -d "token=$PUSHOVER_TOKEN" \
        -d "user=$PUSHOVER_USER" \
        --data-urlencode "title=Paperclip Update" \
        --data-urlencode "message=$COUNT neue Events:
$MSG" \
        -d "priority=0" \
        -d "sound=cosmic" > /dev/null 2>&1
fi

date -u +"%Y-%m-%d %H:%M:%S" > "$STATE_FILE"
```

Speichern mit `Strg + O`, Enter, dann `Strg + X`.

### Schritt 2: Script ausfuehrbar machen

```bash
chmod +x /home/paperclip/paperclip-notify.sh
chown paperclip:paperclip /home/paperclip/paperclip-notify.sh
```

### Schritt 3: Automatisch alle 5 Minuten ausfuehren

Wir richten einen sogenannten "Cronjob" ein — das ist wie ein Wecker, der alle 5 Minuten das Script ausfuehrt:

```bash
(crontab -u paperclip -l 2>/dev/null; echo "*/5 * * * * /home/paperclip/paperclip-notify.sh") | crontab -u paperclip -
```

### Checkpoint

Teste ob es funktioniert:

```bash
sudo -u paperclip /home/paperclip/paperclip-notify.sh
```

Du solltest innerhalb weniger Sekunden eine Push-Nachricht auf deinem Handy bekommen. Falls nicht:
- Pruefe deine Pushover-Keys (Tippfehler?)
- Pruefe ob die Pushover-App auf dem Handy installiert und eingeloggt ist

---

## Teil 9: Wartung und Problemloesung

### Wichtige Pfade auf dem Server

Falls du mal etwas suchen oder pruefen musst — hier liegt alles:

| Was | Wo auf dem Server |
|-----|-------------------|
| Paperclip-Konfiguration | `/home/paperclip/.paperclip/instances/default/config.json` |
| Datenbank | `/home/paperclip/.paperclip/instances/default/db/` |
| Log-Dateien | `/home/paperclip/.paperclip/instances/default/logs/` |
| Automatische Backups | `/home/paperclip/.paperclip/instances/default/data/backups/` |
| Claude-Zugangsdaten | `/home/paperclip/.claude/.credentials.json` |
| Service-Konfiguration | `/etc/systemd/system/paperclip.service` |

### Nuetzliche Befehle

| Was du wissen willst | Befehl |
|----------------------|--------|
| Laeuft Paperclip? | `systemctl status paperclip` |
| Was machen die Agenten? | Oeffne `http://DEINE-IP:3100` im Browser |
| Ist Claude verbunden? | `sudo -u paperclip claude auth status` |
| Automatischer Gesundheitscheck | `sudo -u paperclip npx paperclipai doctor` |
| Manuelles Backup erstellen | `sudo -u paperclip npx paperclipai db:backup` |
| Live-Logs anschauen | `journalctl -u paperclip -f` (Beenden mit Strg+C) |

### Problem: "Ich komme nicht auf die Seite im Browser"

1. **Laeuft Paperclip?**
   ```bash
   systemctl status paperclip
   ```
   Falls `failed` oder `inactive`: `systemctl start paperclip`

2. **Hoert es auf dem richtigen Port?**
   ```bash
   ss -tlnp | grep 3100
   ```
   Du solltest `0.0.0.0:3100` sehen. Falls `127.0.0.1:3100` steht, aendere `host` in der Config auf `"0.0.0.0"` (siehe Teil 3).

3. **Firewall?**
   ```bash
   ufw status
   ```
   Port 3100 muss `ALLOW` sein. Pruefe auch die Firewall deines VPS-Anbieters (Teil 5).

### Problem: "Die Agenten machen nichts"

Die Agenten werden durch "Heartbeats" geweckt — das passiert automatisch alle 30-60 Minuten. Habe etwas Geduld nach dem ersten Start.

Falls nach einer Stunde nichts passiert:
1. Pruefe ob Claude verbunden ist:
   ```bash
   sudo -u paperclip claude auth status
   ```
   Wenn `"loggedIn": false` — wiederhole Teil 6.

2. Pruefe die Logs auf Fehler:
   ```bash
   journalctl -u paperclip --no-pager | tail -20
   ```

### Problem: "Claude-Zugangsdaten abgelaufen"

Die Zugangsdaten laufen nach einiger Zeit ab. Wenn die Agenten ploetzlich nicht mehr arbeiten:

1. Auf deinem **lokalen Computer** neu einloggen:
   ```bash
   claude auth login
   ```

2. Credentials erneut exportieren und hochladen:
   ```bash
   security find-generic-password -s "Claude Code-credentials" -a "$(whoami)" -w > /tmp/claude-credentials.json
   scp /tmp/claude-credentials.json root@DEINE-IP-ADRESSE:/home/paperclip/.claude/.credentials.json
   rm /tmp/claude-credentials.json
   ```

3. Auf dem Server Berechtigungen setzen und neustarten:
   ```bash
   chown paperclip:paperclip /home/paperclip/.claude/.credentials.json
   chmod 600 /home/paperclip/.claude/.credentials.json
   systemctl restart paperclip
   ```

### Problem: "Paperclip startet nicht (Doctor-Fehler)"

```bash
sudo -u paperclip npx paperclipai doctor
```

Dieser Befehl prueft automatisch alle Einstellungen und sagt dir genau, was falsch ist. Die haeufigsten Ursachen:
- `publicBaseUrl` fehlt in der Config → Siehe Teil 3, Schritt 3
- `baseUrlMode` steht auf `auto` statt `explicit` → Siehe Teil 3, Schritt 3

---

## Geschafft!

Wenn du bis hierhin gekommen bist, hast du:

- Einen eigenen Server im Internet eingerichtet
- Paperclip installiert und als dauerhaften Dienst konfiguriert
- Die Firewall sicher konfiguriert
- Claude Code als KI-Gehirn fuer deine Agenten verbunden
- Deinen ersten Account erstellt und die Registrierung gesichert
- (Optional) Push-Benachrichtigungen auf dein Handy eingerichtet

Deine KI-Firma laeuft jetzt rund um die Uhr. Der CEO-Agent wird eigenstaendig Plaene schreiben, Mitarbeiter einstellen und Aufgaben verteilen. Du kannst jederzeit im Browser vorbeischauen und sehen, was passiert — oder dich per Push-Nachricht informieren lassen.

---

## Lizenz

Diese Anleitung ist frei nutzbar und teilbar. Paperclip selbst ist MIT-lizensiert (Open Source).
