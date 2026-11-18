# Abweichungen von den bisherigen Stage-Schritten

Dieses Dokument beschreibt, was das Paket `picompose-core` anders macht als die Schritte `03-picompose`, `04-install-pipewire-audio` und `05-set-hostname` der Stage `01-stage-base` im [PiCompose](https://github.com/florian-asche/PiCompose)-Repository, die das Paket ersetzt. Verglichen wurde mit dem Stand von `main` vor dem Ersetzen (Commit `3e606fe`, 18.11.2026). Gepflegt wird es auf Deutsch, auf Wunsch des Entwicklers.

Pflege: Jede Änderung, die eine Abweichung neu schafft, ändert oder beseitigt, aktualisiert dieses Dokument. Prüfe dabei gegen den Stand von PiCompose, auf dem das Paket installiert wird.

## Nicht im Paket

PiCompose behält diese Schritte, sie sind kein Teil des Pakets:

- **APT-Repository** (`00-apt-repository`): Repository und `picompose-archive-keyring`. Das Paket kommt aus diesem Repository, es kann es nicht selbst einrichten.
- **Docker** (`01-docker`): Docker-apt-Repository, Schlüssel und die Pakete `docker-ce`, `docker-ce-cli`, `containerd.io`, `docker-buildx-plugin` und `docker-compose-plugin`. Ein Paket kann beim Installieren keine anderen Pakete installieren (dpkg ist gesperrt). Das Paket setzt Docker voraus, hat aber keine Abhängigkeit darauf. Der Compose-Manager wartet beim Start bis zu 90 Sekunden auf Docker.
- **Hilfsprogramme** (`02-install-optional-packages`): `git`, `jq`, `curl`, `wget`, `vim`, `htop`, `python3`, `python3-pip` und `dfu-util`. Das Paket hat weder `Depends` noch `Recommends` darauf. Kein Skript des Pakets braucht sie.
- **PipeWire-Pakete** (`pipewire`, `wireplumber`, `pipewire-audio-client-libraries`, `libspa-0.2-bluetooth`, `pipewire-audio`, `pulseaudio-utils`, `pipewire-pulse`): PiCompose installiert sie weiter im Schritt `04-install-pipewire-audio`. Das Paket liefert nur die Konfigurationsdateien.
- **ALSA-Brücke, Benutzer `pi` und Linger:** Der Link `/etc/alsa/conf.d/50-pipewire.conf`, die Gruppe `pipewire` für `pi` und `/var/lib/systemd/linger/pi` setzt weiter der Schritt `04-install-pipewire-audio`. Die Gruppe `pipewire` und die Datei, auf die der Link zeigt, gibt es erst nach der Installation der PipeWire-Pakete, die `postinst` des Pakets läuft aber davor.

## Anders gelöst

- Die Unit-Dateien liegen in `/usr/lib/systemd/system/` statt in `/etc/systemd/system/`. Aktiviert werden sie durch `dh_installsystemd` im Paket (beim Installieren aktiviert, aber nicht gestartet) statt durch `systemctl enable` im Chroot der Stage. Der Inhalt der Units ist unverändert. Das gilt für `picompose.service` und `configure_hostname.service`.
- **Das Verzeichnis `/compose`.** Die Stage legte `/compose`, `README.txt` und `example/` beim Bau des Images direkt an. Das Paket liefert die Vorlagen nach `/usr/share/picompose/compose/` und kopiert sie in der `postinst` nur dorthin, wo sie noch nicht existieren. Änderungen des Benutzers in `/compose` bleiben bei einem Upgrade erhalten, vorhandene Dateien werden nie überschrieben.
- **PipeWire- und WirePlumber-Konfiguration** (`linux-voice-assistant.conf`, `50-alsa-output.conf`) liegen an denselben Orten wie in der Stage, sind aber als Conffiles gekennzeichnet. Änderungen des Benutzers bleiben bei einem Upgrade erhalten und dpkg fragt bei Konflikten. Die Verzeichnisse `/etc/pipewire/pipewire.conf.d` und `/etc/wireplumber/wireplumber.conf.d` legt dpkg an, falls sie fehlen (die Stage legte sie mit `mkdir -p` an).

## Zusätzlich im Paket

- `Depends` auf `cron` (der Compose-Manager ruft `crontab` auf), `iproute2` (das Hostname-Skript ruft `ip` auf) und `systemd`. Die Stage setzt diese Programme als Teil des Basis-Images voraus. Die Begründung steht in der README unter „Required packages".

## Unverändert

- `compose-manager.sh` (Stand nach PiCompose Pull Request 80), `configure_hostname.sh`, beide Units, `README.txt`, das Beispielprojekt (`docker-compose.yml`, `picompose.conf` und `.env`) und die beiden Konfigurationsdateien sind Byte für Byte die Dateien der Stage.
