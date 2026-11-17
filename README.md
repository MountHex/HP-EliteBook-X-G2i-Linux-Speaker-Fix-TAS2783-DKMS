# HP EliteBook X G2i - Linux Speaker Fix - TAS2783-DKMS

A DKMS fix for the built-in speakers on the HP EliteBook X G2i running Fedora Linux.

## Why this exists

The HP EliteBook X G2i's speakers didn't work out of the box on Fedora.

I originally got them working using the audio-matching driver from [hp-elitebook-x-g2i-dkms](https://github.com/meirdick/hp-elitebook-x-g2i-dkms), along with a custom ALSA UCM configuration.

Everything worked until the Linux 7.3 kernel updates, when the speakers went silent again.

After comparing the working and broken kernel drivers, I traced the regression to a change in how the TAS2783 amplifiers handle stereo channels.

This project disables that change while keeping the rest of the newer driver intact.

## Before installing

**This is an additional fix, not a standalone solution for the laptop's audio.**

You'll also need:

- The original HP audio-matching driver from [hp-elitebook-x-g2i-dkms](https://github.com/meirdick/hp-elitebook-x-g2i-dkms).
- The TAS2783 ALSA UCM configuration, included in this repository as `tas2783.conf`.

The original project provides driver source code rather than a complete Fedora installer.

See [INSTALL.md](INSTALL.md) for the setup instructions, including the prerequisites.

## Tested and working

Tested configuration:

- **Laptop:** HP EliteBook X G2i
- **Operating system:** Fedora 45 KDE
- **Kernel:** `7.3.4-300.fc45.x86_64`
- **Audio hardware:** Realtek RT712 and four Texas Instruments TAS2783 SoundWire amplifiers

Verified results:

- Speakers working after reboot.
- Left and right stereo playback working.
- DKMS installation verified.
- Secure Boot module signing verified.

The DKMS configuration deliberately restricts this fix to the tested Fedora kernel `7.3.4-300.fc45.x86_64` and disables automatic installation on future kernels. This prevents the modified driver from overriding newer kernel versions that may have different implementations or already include an upstream fix.

Support for additional kernels can be added after compatibility has been verified.

## Credits and licence

Based on the Linux TAS2783 SoundWire driver developed by Texas Instruments.

Thanks to the contributors of [hp-elitebook-x-g2i-dkms](https://github.com/meirdick/hp-elitebook-x-g2i-dkms) for the original audio workaround.

The modified driver retains the original copyright notices and is licensed under **GPL-2.0**. See [LICENSE](LICENSE).
