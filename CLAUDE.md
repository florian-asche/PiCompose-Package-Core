# AI working guide for PiCompose-Package-Core

Package `picompose-core` for the [PiCompose APT repository](https://github.com/florian-asche/PiCompose-Repository). It replaces the steps `03-picompose`, `04-install-pipewire-audio` and `05-set-hostname` of the stage `01-stage-base` of the [PiCompose](https://github.com/florian-asche/PiCompose) image as a Debian package. Read `README.md` for what the package contains. When this file and the code disagree, the code wins, fix this file in the same change.

## Rules

- **linux-voice-assistant is read only.** Never change anything in the linux-voice-assistant repository, not locally and not on GitHub. It is a reference for what the voice assistant needs. The PiCompose repository is the consumer of this package and may be changed when the developer asks for it. Follow its `AGENTS.md` there.
- Nothing is posted to GitHub (pull requests, discussions, comments) and nothing is pushed without the explicit consent of the developer. A human reviews every change, see `.github/AI_POLICY.md`.
- Documentation, code comments, commit and PR text are in English, whatever language the conversation is in.
- **Open points live in `todo.md`.** Read it before you start. When the developer postpones something ("we do that later") or you find something you cannot settle in this change, add an entry there, and remove the entry in the change that settles it.
- Do not edit `debian/changelog`. The workflows write it: every pull request needs exactly one of the labels `major`, `minor` and `patch` (tick one box under "Version bump" in the pull request description), and the version follows from it when the pull request is merged.
- The packages never download anything while they are installed. Where the PiCompose stage downloaded sources while it built the image, the package build downloads the same (see `debian/rules`), and no third-party sources are stored in this repository.
- Maintainer scripts must be idempotent, run without prompts, and not fail when the hardware is missing. Units are installed to `/usr/lib/systemd/system` and enabled with `dh_installsystemd`, without `systemctl` calls. Defaults for user-owned locations are shipped below `/usr/share` and copied by `postinst` only if missing. The one exception is the example in `/compose/example`, which is a reference and is replaced on every install and upgrade.
- Hardware behavior (audio, boot configuration, kernel modules, I2C) cannot be tested here. Say that it was not tested on hardware, never claim it was.

## Commands

| Task | Command |
| --- | --- |
| Build and lint like the CI | in a `debian:trixie` container: `apt-get build-dep ./`, `dpkg-buildpackage -us -uc -b`, `lintian --fail-on error ../*.deb` |
| Build with the scripts of the APT repository | `PiCompose-Repository/scripts/build-package.sh <this directory> amd64` |
| Test a maintainer script | run it with `sh` in a container against a copy of the files it changes |

## Compatibility with Raspberry Pi OS (required for every change)

The package has to work on the Raspberry Pi OS that the PiCompose images are built on. Judge that for **every** change, before you finish, and write the result into the pull request description under "Compatibility with Raspberry Pi OS".

1. **Find out what the images use today.** Do not assume it. Read these files of the PiCompose repository (`/home/florian/repos/PiCompose` if it is checked out, otherwise `https://github.com/florian-asche/PiCompose`):
   - `.github/workflows/build-image-template.yml`: the Debian release (`releaseversion`, default `trixie`), the pi-gen repository and branch (`pi-gen-repository`, `pi-gen-version`), the user (`username`) and the runner architecture.
   - `.github/workflows/build-all.yml`: which images exist and which stages each of them builds (`stage-list`), so which images contain the stage of this package.
   - The stage directory `01-stage-base` and its `README`/`docs/` for what the image does today.
2. **Check this package against it.** What to look at:
   - The Debian release: do all packages in `Depends`, `Recommends` and `Build-Depends` exist in that release and in the Raspberry Pi archive (`archive.raspberrypi.com/debian`)? Check with `apt-cache policy` in a container of that release.
   - The kernel and firmware layout of Raspberry Pi OS for that release (kernel version, `/boot/firmware`, `config.txt`, `usercfg.txt`).
   - The architectures: the images are built for arm64, check whether armhf is still needed.
   - Defaults of the base system that the package relies on: the user `pi` with the UID 1000, systemd, PipeWire and WirePlumber, `cron`.
   Specific to this package:
   - Docker: it is not part of this package, the PiCompose image installs it. The compose manager has to work with the Docker version the image gets from the Docker apt repository for the release (`docker compose` plugin).
   - PipeWire and WirePlumber: the configuration syntax and the file locations of the versions in the release (WirePlumber 0.5 reads `wireplumber.conf.d/*.conf`). The package ships only the configuration files, the image installs the PipeWire packages, so check that the file locations match the versions the image installs.
   - The user `pi` with the UID 1000, which `postinst`, the units and the compose projects assume.
3. **Write down the verdict**: which release, kernel and image definition you looked at, what you checked and how (commands), what you could not check (hardware), and what has to change if the base moves, for example to a new Debian release. If a change makes the package fit only a newer or only an older base than the images use, say so and ask the developer before going on.
