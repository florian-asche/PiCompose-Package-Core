# To do for PiCompose-Package-Core

Open points and work that was postponed. Remove an entry in the change that settles it.

1. **Test on Raspberry Pi OS Trixie:** Install the package on a Raspberry Pi OS Trixie system with the Docker repository, start `picompose.service`, check the compose projects, the hostname service and the PipeWire configuration. Nothing was tested on hardware.

2. **Review the Recommends:** The convenience tools of the image (git, jq, curl, wget, vim, htop, python3, python3-pip, dfu-util) are recommended, not required. Check whether the image should install them another way.
