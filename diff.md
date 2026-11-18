# Abweichungen von der Stage `01-stage-picompose`

Dieses Dokument beschreibt, was das Paket `picompose-core` anders macht als die gleichnamige Stage im [PiCompose](https://github.com/florian-asche/PiCompose)-Repository. Verglichen wurde mit dem Stand von `main` (Commit `784abfb`, 17.11.2026). Gepflegt wird es auf Deutsch, auf Wunsch des Entwicklers.

Pflege: Jede Änderung, die eine Abweichung neu schafft, ändert oder beseitigt, aktualisiert dieses Dokument. Prüfe dabei gegen den aktuellen `main` von PiCompose.

## Nicht im Paket

- **Docker-Repository und Docker-Pakete.** Der Schritt `01-packages` (`01-run.sh`: Docker-apt-Repository einrichten, Schlüssel laden) und die Pakete `docker-ce`, `docker-ce-cli`, `containerd.io`, `docker-buildx-plugin` und `docker-compose-plugin` bleiben in PiCompose. Ein Paket kann beim Installieren keine anderen Pakete installieren (dpkg ist gesperrt). Das Paket setzt Docker voraus, hat aber keine Abhängigkeit darauf. Der Compose-Manager wartet beim Start bis zu 90 Sekunden auf Docker.

## Anders gelöst

- **Hilfsprogramme.** `git`, `jq`, `curl`, `wget`, `vim`, `htop` und `dfu-util` (Paketliste `01-packages/02-packages`) werden als `Recommends` eingetragen. Die Stage installiert sie immer. `python3` und `python3-pip` stehen ebenfalls in der Paketliste, werden aber nicht eingetragen: Kein Skript von PiCompose (Compose-Manager, Audio-, Hostname-Skripte) nutzt Python, nur `scripts/generate-imager-json.py` im CI. Das Image installiert sie weiter über seine eigene Paketliste.
- **PipeWire-Pakete** (`03-install-pipewire-audio/01-packages`) sind `Depends` des Pakets, mit denselben Paketnamen wie in der Stage.
- Die Unit-Datei liegt in `/usr/lib/systemd/system/` statt in `/etc/systemd/system/`. Aktiviert wird sie durch `dh_installsystemd` im Paket (beim Installieren aktiviert, aber nicht gestartet) statt durch `systemctl enable` im Chroot der Stage. Der Inhalt der Unit ist unverändert. Das gilt für `picompose.service` und `configure_hostname.service`.
- **Das Verzeichnis `/compose`.** Die Stage legt `/compose`, `README.txt` und `example/` beim Bau des Images direkt an. Das Paket liefert die Vorlagen nach `/usr/share/picompose/compose/` und kopiert sie in der `postinst` nur dorthin, wo sie noch nicht existieren. Änderungen des Benutzers in `/compose` bleiben bei einem Upgrade erhalten, vorhandene Dateien werden nie überschrieben.
- **PipeWire- und WirePlumber-Konfiguration** (`linux-voice-assistant.conf`, `50-alsa-output.conf`) liegen an denselben Orten wie in der Stage, sind aber als Conffiles gekennzeichnet. Änderungen des Benutzers bleiben bei einem Upgrade erhalten und dpkg fragt bei Konflikten.
- **ALSA-Brücke.** Den Link `/etc/alsa/conf.d/50-pipewire.conf` setzt die `postinst` mit demselben `ln -sf` wie die Stage.
- **Benutzer `pi`.** Die Stage trägt `pi` immer in die Gruppe `pipewire` ein und legt `/var/lib/systemd/linger/pi` an. Das Paket macht beides nur, wenn der Benutzer `pi` und die Gruppe `pipewire` existieren.

## Zusätzlich im Paket

- `Depends` auf `cron` (der Compose-Manager ruft `crontab` auf), `iproute2` (das Hostname-Skript ruft `ip` auf) und `systemd`. Die Stage setzt diese Programme als Teil des Basis-Images voraus.

## Unverändert

- `compose-manager.sh` (Stand nach PiCompose Pull Request 80), `configure_hostname.sh`, beide Units, `README.txt`, das Beispielprojekt und die beiden Konfigurationsdateien sind Byte für Byte die Dateien der Stage.
