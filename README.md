# PiCompose-Package-Core

[![Version](https://github.com/florian-asche/PiCompose-Package-Core/actions/workflows/version.yml/badge.svg?branch=main)](https://github.com/florian-asche/PiCompose-Package-Core/actions/workflows/version.yml) [![GitHub Release Version](https://img.shields.io/github/v/release/florian-asche/PiCompose-Package-Core?label=version)](https://github.com/florian-asche/PiCompose-Package-Core/releases) [![GitHub License](https://img.shields.io/github/license/florian-asche/PiCompose-Package-Core)](https://github.com/florian-asche/PiCompose-Package-Core/blob/main/LICENSE)

Package `picompose-core` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). It is the content of the first stage of the [PiCompose](https://github.com/florian-asche/PiCompose) image as a package: the compose manager, the PipeWire audio setup and the hostname setup. Docker is not part of it.

What differs from how the stage does it is documented in [diff.md](diff.md) (in German).

## Contents

| Part | Files |
| --- | --- |
| Compose manager | `/usr/bin/compose-manager.sh`, `picompose.service` (enabled, not started at install) |
| Defaults for `/compose` | `/usr/share/picompose/compose/` (README and example), copied to `/compose` by the package if missing |
| PipeWire audio | `/etc/pipewire/pipewire.conf.d/linux-voice-assistant.conf`, `/etc/wireplumber/wireplumber.conf.d/50-alsa-output.conf` |
| Hostname | `/usr/bin/configure_hostname.sh`, `configure_hostname.service` (enabled, not started at install) |

The package depends on the PipeWire packages of the image. It does **not** depend on Docker: the Docker apt repository and the Docker packages (`docker-ce`, `docker-ce-cli`, `containerd.io`, `docker-buildx-plugin`, `docker-compose-plugin`) are installed by the PiCompose image, because a package cannot install other packages while it is installed. The compose manager needs Docker and waits up to 90 seconds for it at boot. The tools the image carries for convenience (git, jq, curl, wget, vim, htop, dfu-util) are recommended. PiCompose itself needs no Python on the device.

The `postinst` of the package:

- creates `/compose` and copies the README and the example if they are missing
- links the PipeWire-ALSA bridge into `/etc/alsa/conf.d`
- adds the user `pi` to the group `pipewire` and enables lingering for `pi`, if that user exists

## Install

Install Docker first (the PiCompose image does that), then the package:

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
