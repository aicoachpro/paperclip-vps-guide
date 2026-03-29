# Paperclip VPS Setup Guide

Komplette Anleitung: Paperclip AI-Agent-Plattform auf einem VPS einrichten, mit Claude Code (Max-Abo) verbinden und produktionsreif konfigurieren.

## Was ist Paperclip?

Paperclip ist eine Open-Source-Plattform zur Orchestrierung von KI-Agenten. Statt einzelne Chatbots zu bedienen, verwaltest du ein virtuelles Unternehmen mit Organigramm, Hierarchien, Budgets und Aufgaben. Ein CEO-Agent kann eigenstaendig Mitarbeiter einstellen, Tasks delegieren und Ergebnisse liefern.

- Website: https://paperclip.ing
- Dokumentation: https://docs.paperclip.ing
- GitHub: https://github.com/paperclipai/paperclip
- Lizenz: MIT (Open Source)

## Inhalt

- [Voraussetzungen](#voraussetzungen)
- [Teil 1: Paperclip installieren](#teil-1-paperclip-installieren)
- [Teil 2: systemd-Service einrichten](#teil-2-paperclip-als-systemd-service-einrichten)
- [Teil 3: Firewall konfigurieren](#teil-3-firewall-konfigurieren)
- [Teil 4: Claude Code Auth (Max-Abo)](#teil-4-claude-code-auth-einrichten-max-abo)
- [Teil 5: Benutzerverwaltung](#teil-5-erster-login-und-benutzerverwaltung)
- [Teil 6: Push-Benachrichtigungen](#teil-6-push-benachrichtigungen-einrichten-optional)
- [Teil 7: Wartung & Troubleshooting](#teil-7-wartung-und-troubleshooting)

---

## Voraussetzungen

| Was | Minimum |
|-----|---------|
| **VPS** | Ubuntu 22.04+ oder Debian 12+, mindestens 2 GB RAM, 20 GB Disk |
| **Node.js** | Version 20 oder hoeher |
| **Claude Max-Abo** | Fuer Claude Code als Agent-Runtime (alternativ: Anthropic API-Key) |
| **SSH-Zugang** | Root-Zugang auf den VPS |

---

## Teil 1: Paperclip installieren

### 1.1 Mit dem VPS verbinden

```bash
ssh root@DEINE-IP
```

### 1.2 System aktualisieren und Node.js installieren

```bash
apt update && apt upgrade -y

# Node.js 20 installieren (falls noch nicht vorhanden)
curl -fsSL https://deb.nodesource.com/setup_20.x | bash -
apt install -y nodejs

# Version pruefen
node -v   # sollte v20.x.x zeigen
npm -v    # sollte 10.x.x zeigen
```

### 1.3 Paperclip-Benutzer anlegen

Paperclip sollte nicht als root laufen. Wir erstellen einen eigenen System-User:

```bash
adduser --system --group --home /home/paperclip --shell /bin/bash paperclip
```

### 1.4 Paperclip installieren und einrichten

```bash
# Als paperclip-User das Setup starten
sudo -u paperclip bash -c 'cd /home/paperclip && npx paperclipai onboard --yes'
```

Das Setup erstellt automatisch:
- Eine eingebettete PostgreSQL-Datenbank
- Die Konfiguration unter `/home/paperclip/.paperclip/instances/default/`
- Einen CEO-Agent als ersten KI-Mitarbeiter

### 1.5 Konfiguration anpassen

Die Konfigurationsdatei liegt unter:
```
/home/paperclip/.paperclip/instances/default/config.json
```

Oeffne sie und stelle sicher, dass folgende Werte gesetzt sind:

```bash
nano /home/paperclip/.paperclip/instances/default/config.json
```

**Wichtige Einstellungen:**

```json
{
  "server": {
    "deploymentMode": "authenticated",
    "exposure": "public",
    "host": "0.0.0.0",
    "port": 3100,
    "serveUi": true
  },
  "auth": {
    "baseUrlMode": "explicit",
    "publicBaseUrl": "http://DEINE-IP:3100",
    "disableSignUp": false
  }
}
```

> **Wichtig:** `host` muss `"0.0.0.0"` sein, damit die App von aussen erreichbar ist. Bei `"127.0.0.1"` waere sie nur lokal zugaenglich.

> **Wichtig:** `baseUrlMode` muss `"explicit"` sein und `publicBaseUrl` muss gesetzt werden, wenn `deploymentMode` auf `"authenticated"` und `exposure` auf `"public"` steht. Sonst startet Paperclip nicht.

---

## Teil 2: Paperclip als systemd-Service einrichten

Damit Paperclip automatisch nach einem Neustart des Servers wieder laeuft:

### 2.1 Service-Datei erstellen

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

### 2.2 Service aktivieren und starten

```bash
systemctl daemon-reload
systemctl enable paperclip
systemctl start paperclip
```

### 2.3 Pruefen ob alles laeuft

```bash
# Status anzeigen
systemctl status paperclip

# Port pruefen (sollte 0.0.0.0:3100 zeigen)
ss -tlnp | grep 3100

# Logs ansehen
journalctl -u paperclip -f
```

### 2.4 Wichtige Service-Befehle

| Befehl | Was es tut |
|--------|-----------|
| `systemctl start paperclip` | Starten |
| `systemctl stop paperclip` | Stoppen |
| `systemctl restart paperclip` | Neustarten |
| `systemctl status paperclip` | Status anzeigen |
| `journalctl -u paperclip -f` | Logs live verfolgen |

---

## Teil 3: Firewall konfigurieren

### 3.1 UFW (Ubuntu Firewall)

```bash
ufw allow 22/tcp      # SSH nicht aussperren!
ufw allow 3100/tcp    # Paperclip
ufw enable
```

### 3.2 Hostinger-Firewall

Falls du einen Hostinger-VPS nutzt: Im **Hostinger Panel** unter **VPS > Firewall** muss Port 3100 ebenfalls freigegeben sein. Hostinger hat eine eigene Firewall-Schicht die unabhaengig von UFW laeuft.

### 3.3 Erreichbarkeit testen

Von deinem lokalen Rechner:
```bash
curl -s http://DEINE-IP:3100/api/health
```

Erwartete Antwort:
```json
{"status":"ok","version":"...","deploymentMode":"authenticated",...}
```

---

## Teil 4: Claude Code Auth einrichten (Max-Abo)

Mit einem Claude Max-Abo kannst du Claude Code als Agent-Runtime nutzen — ohne zusaetzliche API-Kosten.

### 4.1 Claude Code CLI installieren

```bash
npm install -g @anthropic-ai/claude-code
```

### 4.2 Authentifizierung einrichten

Auf einem VPS ohne Desktop/Browser funktioniert `claude auth login` nicht direkt (das Terminal friert ein). Stattdessen nutzen wir den **Credential-Transfer** vom lokalen Rechner.

#### Schritt 1: Auf dem lokalen Mac/PC einloggen

Falls noch nicht geschehen, auf deinem lokalen Rechner:

```bash
claude auth login
```

Browser oeffnet sich, mit deinem Claude-Account anmelden.

#### Schritt 2: Credentials vom Mac exportieren

Auf dem Mac werden die Credentials im Keychain gespeichert. So holst du sie raus:

```bash
security find-generic-password -s "Claude Code-credentials" -a "$(whoami)" -w > /tmp/claude-credentials.json
```

#### Schritt 3: Credentials auf den VPS kopieren

```bash
scp /tmp/claude-credentials.json root@DEINE-IP:/home/paperclip/.claude/.credentials.json
```

#### Schritt 4: Berechtigungen setzen

Auf dem VPS:

```bash
chown paperclip:paperclip /home/paperclip/.claude/.credentials.json
chmod 600 /home/paperclip/.claude/.credentials.json
```

#### Schritt 5: Auth pruefen

```bash
sudo -u paperclip claude auth status
```

Erwartete Ausgabe:

```json
{
  "loggedIn": true,
  "authMethod": "claude.ai",
  "apiProvider": "firstParty",
  "subscriptionType": "max"
}
```

#### Schritt 6: Lokale Credentials-Datei loeschen

```bash
rm /tmp/claude-credentials.json
```

> **Sicherheitshinweis:** Die Credentials-Datei enthaelt OAuth-Tokens. Behandle sie wie ein Passwort. Loesche temporaere Kopien sofort.

#### Schritt 7: Paperclip neu starten

```bash
systemctl restart paperclip
```

### 4.3 Alternative: Anthropic API-Key

Falls du kein Max-Abo hast, kannst du stattdessen einen API-Key von https://console.anthropic.com verwenden:

```bash
# In die .env-Datei eintragen
echo 'ANTHROPIC_API_KEY=sk-ant-DEIN-KEY' >> /home/paperclip/.paperclip/instances/default/.env

# Paperclip neustarten
systemctl restart paperclip
```

> **Hinweis:** API-Key-Nutzung wird nach Verbrauch abgerechnet. Mit dem Max-Abo (Credential-Transfer) faellt keine zusaetzliche Gebuehr an.

---

## Teil 5: Erster Login und Benutzerverwaltung

### 5.1 Im Browser anmelden

Oeffne im Browser:
```
http://DEINE-IP:3100
```

Erstelle deinen Admin-Account. Beim Onboarding wird automatisch eine Company mit CEO-Agent angelegt.

### 5.2 Zweiten Benutzer anlegen

Solange Sign-Up aktiviert ist (`disableSignUp: false`), kann sich jeder mit dem Link registrieren. Teile die URL mit Kollegen die Zugang brauchen.

### 5.3 Sign-Up deaktivieren

Wenn alle Benutzer registriert sind, Sign-Up schliessen:

```bash
# In der config.json
sed -i 's/"disableSignUp": false/"disableSignUp": true/' \
  /home/paperclip/.paperclip/instances/default/config.json

# Paperclip neustarten
systemctl restart paperclip
```

---

## Teil 6: Push-Benachrichtigungen einrichten (optional)

Mit Pushover kannst du dich benachrichtigen lassen wenn die Agenten etwas Wichtiges tun.

### 6.1 Voraussetzungen

- Pushover-Account (https://pushover.net)
- Pushover-App erstellen unter https://pushover.net/apps/build
- Du brauchst: **API Token** und **User Key**

### 6.2 Notification-Script erstellen

Erstelle die Datei `/home/paperclip/paperclip-notify.sh`:

```bash
#!/bin/bash
PUSHOVER_TOKEN="DEIN-APP-TOKEN"
PUSHOVER_USER="DEIN-USER-KEY"
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

### 6.3 Script einrichten

```bash
chmod +x /home/paperclip/paperclip-notify.sh
chown paperclip:paperclip /home/paperclip/paperclip-notify.sh
```

### 6.4 Cronjob einrichten (alle 5 Minuten)

```bash
(crontab -u paperclip -l 2>/dev/null; echo "*/5 * * * * /home/paperclip/paperclip-notify.sh") | crontab -u paperclip -
```

### 6.5 Testen

```bash
sudo -u paperclip /home/paperclip/paperclip-notify.sh
```

Du solltest eine Push-Nachricht auf deinem Handy bekommen.

---

## Teil 7: Wartung und Troubleshooting

### 7.1 Wichtige Pfade

| Was | Pfad |
|-----|------|
| Konfiguration | `/home/paperclip/.paperclip/instances/default/config.json` |
| Datenbank | `/home/paperclip/.paperclip/instances/default/db/` |
| Logs | `/home/paperclip/.paperclip/instances/default/logs/` |
| Backups | `/home/paperclip/.paperclip/instances/default/data/backups/` |
| Secrets | `/home/paperclip/.paperclip/instances/default/secrets/master.key` |
| Claude Credentials | `/home/paperclip/.claude/.credentials.json` |
| systemd Service | `/etc/systemd/system/paperclip.service` |
| Notification Script | `/home/paperclip/paperclip-notify.sh` |

### 7.2 Nuetzliche CLI-Befehle

```bash
# Health-Check
sudo -u paperclip npx paperclipai doctor

# Konfiguration aendern
sudo -u paperclip npx paperclipai configure

# Claude Auth Status pruefen
sudo -u paperclip claude auth status

# Datenbank-Backup manuell erstellen
sudo -u paperclip npx paperclipai db:backup
```

### 7.3 Haeufige Probleme

#### App ist von aussen nicht erreichbar

1. Pruefen ob der richtige Host gesetzt ist: `grep host config.json` → muss `0.0.0.0` sein
2. Pruefen ob `publicBaseUrl` gesetzt ist (bei authenticated + public mode)
3. Firewall pruefen: `ufw status` und Hostinger-Panel
4. Port pruefen: `ss -tlnp | grep 3100`

#### Prozesse frieren ein (Status T+)

Passiert wenn die Terminal-Session verloren geht. Loesung:
```bash
pkill -9 -f "paperclipai run"
systemctl start paperclip
```

Deshalb ist der systemd-Service so wichtig — er startet die App automatisch neu.

#### Claude Auth Token abgelaufen

Die OAuth-Tokens haben ein Ablaufdatum. Wenn die Agenten nicht mehr funktionieren:

1. Auf dem lokalen Rechner neu einloggen: `claude auth login`
2. Credentials erneut exportieren und auf den VPS kopieren (siehe Teil 4)
3. `systemctl restart paperclip`

#### Doctor-Check schlaegt fehl

```bash
sudo -u paperclip npx paperclipai doctor
```

Zeigt dir genau was falsch konfiguriert ist und gibt Hinweise zur Behebung.

---

## Lizenz

Dieses Setup-Guide ist frei nutzbar. Paperclip selbst ist MIT-lizensiert.
