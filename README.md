# PiCompose-Package-Core

[![Version](https://github.com/florian-asche/PiCompose-Package-Core/actions/workflows/version.yml/badge.svg?branch=main)](https://github.com/florian-asche/PiCompose-Package-Core/actions/workflows/version.yml) [![GitHub Release Version](https://img.shields.io/github/v/release/florian-asche/PiCompose-Package-Core?label=version)](https://github.com/florian-asche/PiCompose-Package-Core/releases) [![GitHub License](https://img.shields.io/github/license/florian-asche/PiCompose-Package-Core)](https://github.com/florian-asche/PiCompose-Package-Core/blob/main/LICENSE)

Package `picompose-core` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). The [PiCompose](https://github.com/florian-asche/PiCompose) image installs it to get the compose manager, the hostname setup and the PipeWire configuration files. Docker and the PipeWire packages are not part of it.

## Contents

| Part | Files |
| --- | --- |
| Compose manager | `/usr/bin/compose-manager.sh`, `picompose.service` (enabled, not started at install) |
| Defaults for `/compose` | `/usr/share/picompose/compose/` (README and example, including the `.env`), copied to `/compose` by the package if missing |
| PipeWire and WirePlumber configuration | `/etc/pipewire/pipewire.conf.d/linux-voice-assistant.conf`, `/etc/wireplumber/wireplumber.conf.d/50-alsa-output.conf` (conffiles, the directories are created by the package if they do not exist) |
| Hostname | `/usr/bin/configure_hostname.sh`, `configure_hostname.service` (enabled, not started at install) |

The `postinst` of the package creates `/compose` and copies the README and the example if they are missing. It does nothing else.

## Required packages

Only what the scripts of the package need to run is a dependency of the package (`Depends`). Everything else is installed by the PiCompose image.

| Package | Needed for |
| --- | --- |
| `cron` | `compose-manager.sh` sets up the schedule with `crontab` |
| `iproute2` | `configure_hostname.sh` reads the network interface with `ip` |
| `systemd` | the two units and `hostnamectl` in `configure_hostname.sh` |

Required at runtime, but **not** a dependency, because the PiCompose image installs it from the Docker apt repository:

- Docker Engine with the `docker compose` plugin (`docker-ce`, `docker-ce-cli`, `containerd.io`, `docker-buildx-plugin`, `docker-compose-plugin`). The compose manager waits up to 90 seconds for Docker at boot.

Not required and not installed by this package, the PiCompose image installs them in its own steps:

- the PipeWire packages (`pipewire`, `wireplumber`, `pipewire-audio-client-libraries`, `libspa-0.2-bluetooth`, `pipewire-audio`, `pulseaudio-utils`, `pipewire-pulse`). The configuration files of the package only take effect once they are installed.
- the PipeWire-ALSA bridge, the group membership of `pi` in `pipewire` and the lingering of `pi`
- the convenience tools (`git`, `jq`, `curl`, `wget`, `vim`, `htop`, `python3`, `python3-pip`, `dfu-util`)

Keep the table up to date: a new `Depends` has to be listed here with the reason.

## Install

The PiCompose image installs the package after the Docker step:

```sh
sudo apt install picompose-core
```

## Notes

- `compose-manager.sh` is the script of the PiCompose image without changes (state of the commit that fixed its cron path, PiCompose pull request 80).
- The hostname service only renames a host that is called `picompose`, once.
- This is hardware independent. Audio drivers are separate packages.

## Versioning and changelog

The version of the package is the first entry of `debian/changelog`, which is part of the package. You do not edit it by hand, the workflows write it.

1. **Pull request:** Every pull request needs exactly one of the labels `major`, `minor` and `patch`. Tick one box under "Version bump" in the description or set the label by hand. The check "Version label" fails until there is exactly one.
2. **Merge to `main`:** The workflow "Version" raises the version, adds the pull request titles to `debian/changelog`, commits it to `main`, creates the tag `v<version>` as pre-release and starts the build, which publishes to the suite `trixie-nightly`.
3. **Release:** Turn the pre-release into a release in the GitHub UI. That publishes the same version to the suite `trixie`.
4. **Any other branch:** `<version>+dev<timestamp>.<sha>`, kept as workflow artifact for 14 days, nothing is published.

The repository needs the secret `HUB_DISPATCH_TOKEN` and a branch protection of `main` that requires the check "Version label" and lets `github-actions[bot]` push, see the [DEVELOPMENT.md](https://github.com/florian-asche/PiCompose-Repository/blob/main/DEVELOPMENT.md) of the APT repository. The repository also has to be listed in `sources.yml` of the APT repository.

## Local build

```sh
sudo apt install build-essential debhelper devscripts fakeroot lintian
dpkg-buildpackage -us -uc -b
lintian ../picompose-core_*.deb
```

## License

This project is released under the [MIT License](LICENSE).
