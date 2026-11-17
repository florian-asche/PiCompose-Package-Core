# PiCompose-Package-Core

Package `picompose-core` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). It is the content of the first stage of the [PiCompose](https://github.com/florian-asche/PiCompose) image as a package: Docker, the compose manager, the PipeWire audio setup and the hostname setup.

## Contents

| Part | Files |
| --- | --- |
| Docker apt repository | `/etc/apt/sources.list.d/docker.sources`, `/usr/share/keyrings/docker.asc` |
| Compose manager | `/usr/bin/compose-manager.sh`, `picompose.service` (enabled, not started at install) |
| Defaults for `/compose` | `/usr/share/picompose/compose/` (README and example), copied to `/compose` by the package if missing |
| PipeWire audio | `/etc/pipewire/pipewire.conf.d/linux-voice-assistant.conf`, `/etc/wireplumber/wireplumber.conf.d/50-alsa-output.conf` |
| Hostname | `/usr/bin/configure_hostname.sh`, `configure_hostname.service` (enabled, not started at install) |

The package depends on the Docker packages (`docker-ce`, `docker-ce-cli`, `containerd.io`, `docker-buildx-plugin`, `docker-compose-plugin`) and on the PipeWire packages of the image. The tools the image carries for convenience (git, jq, curl, wget, vim, htop, python3, python3-pip, dfu-util) are recommended.

The `postinst` of the package:

- creates `/compose` and copies the README and the example if they are missing
- links the PipeWire-ALSA bridge into `/etc/alsa/conf.d`
- adds the user `pi` to the group `pipewire` and enables lingering for `pi`, if that user exists

## Install

The Docker packages come from the Docker apt repository, which this package configures. Apt has to know that repository before it can install the package, so on a system without it, add it first:

```sh
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /usr/share/keyrings/docker.asc
sudo tee /etc/apt/sources.list.d/docker.sources >/dev/null <<'EOT'
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: trixie
Components: stable
Signed-By: /usr/share/keyrings/docker.asc
EOT
sudo apt update
sudo apt install picompose-core
```

The package then takes over both files.

## Notes

- `compose-manager.sh` is the script of the PiCompose image with one fix: the cron jobs call `/usr/bin/compose-manager.sh`, where the script is installed. The image calls `/usr/local/bin/compose-manager.sh`, which does not exist, see [PiCompose pull request 80](https://github.com/florian-asche/PiCompose/pull/80). When that is merged, the script is the same again.
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
