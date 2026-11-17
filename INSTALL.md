# Installation — HP EliteBook X G2i Speaker Fix

These instructions are for Fedora Linux on the HP EliteBook X G2i.

This fix addresses speakers going silent after upgrading to Linux kernel 7.3. It builds on the original HP audio workaround, so you'll need that working first.

**Currently tested:** Fedora 45, kernel `7.3.4-300.fc45.x86_64`.

## 1. Install the original HP audio driver

The laptop needs the `snd-soc-acpi-intel-match-hp-g2i` driver to recognise its RT712 codec and four TAS2783 speaker amplifiers correctly.

The source is available from [meirdick/hp-elitebook-x-g2i-dkms](https://github.com/meirdick/hp-elitebook-x-g2i-dkms).

First, install the build requirements.

**In Konsole:**

```bash
sudo dnf install dkms gcc make git kernel-devel-$(uname -r) alsa-ucm alsa-sof-firmware
```

Clone the original repository into a working directory of your choice:

```bash
git clone https://github.com/meirdick/hp-elitebook-x-g2i-dkms.git
```

From the directory containing the cloned repository, install its audio-driver source:

```bash
sudo install -d /usr/src/snd-soc-acpi-intel-match-hp-g2i-1.0

sudo cp -a hp-elitebook-x-g2i-dkms/snd-soc-acpi-intel-match-hp-g2i/dkms.conf \
  hp-elitebook-x-g2i-dkms/snd-soc-acpi-intel-match-hp-g2i/src \
  /usr/src/snd-soc-acpi-intel-match-hp-g2i-1.0/
```

Register, build and install the driver for your running kernel:

```bash
sudo dkms add -m snd-soc-acpi-intel-match-hp-g2i -v 1.0
sudo dkms build -m snd-soc-acpi-intel-match-hp-g2i -v 1.0 -k "$(uname -r)"
sudo dkms install -m snd-soc-acpi-intel-match-hp-g2i -v 1.0 -k "$(uname -r)"
```

If the driver is already installed, there's no need to install it again.

## 2. Add the speaker configuration

The laptop also needs a TAS2783 ALSA UCM configuration.

This project will include a file named `tas2783.conf`, matching the configuration tested on the HP EliteBook X G2i.

Install it here:

`/usr/share/alsa/ucm2/sof-soundwire/tas2783.conf`

From the directory containing that file:

```bash
sudo install -m 0644 tas2783.conf \
  /usr/share/alsa/ucm2/sof-soundwire/tas2783.conf
```

**Check for an existing file first.** Don't overwrite a working audio configuration without making a backup.

Reboot and confirm Fedora recognises the laptop's Speaker output.

On kernel 7.3.4, the speakers may still be silent at this stage. That's the regression this project fixes in the next section.

If Fedora doesn't recognise the Speaker output at all, check the original audio-matching driver and ALSA UCM configuration before continuing.

## 3. Install the kernel 7.3 speaker fix

This is the additional driver that restores audio on kernel 7.3.4.

**Important:** The current release supports only `7.3.4-300.fc45.x86_64`. Confirm your running kernel with `uname -r` before continuing.

Clone this project, then open Konsole in the directory containing its source files.

Install the DKMS source:

```bash
sudo install -d /usr/src/snd-soc-tas2783-sdw-hp-g2i-1.0

sudo install -m 0644 tas2783-sdw.c tas2783.h Makefile dkms.conf \
  /usr/src/snd-soc-tas2783-sdw-hp-g2i-1.0/
```

Register, build and install:

```bash
sudo dkms add -m snd-soc-tas2783-sdw-hp-g2i -v 1.0
sudo dkms build -m snd-soc-tas2783-sdw-hp-g2i -v 1.0 -k 7.3.4-300.fc45.x86_64
sudo dkms install -m snd-soc-tas2783-sdw-hp-g2i -v 1.0 -k 7.3.4-300.fc45.x86_64
```

### Secure Boot

If Secure Boot is enabled, the DKMS driver must be signed with a key trusted by your system.

On my Fedora installation, DKMS automatically signed the module using an existing enrolled Machine Owner Key (MOK).

Check that your signing key is enrolled before rebooting. If you haven't configured DKMS module signing before, you'll need to complete that setup first.

Don't disable Secure Boot just to install this fix.

Reboot, then test the speakers.

## 4. Check that it's working

Run:

```bash
dkms status -m snd-soc-tas2783-sdw-hp-g2i -v 1.0
modinfo -n snd_soc_tas2783_sdw
```

DKMS should report the driver as installed, and the module path should point to the DKMS-managed driver under `/extra/`.

Test stereo playback:

```bash
speaker-test -D default -c 2 -t wav -l 1
```

Both left and right channels should play correctly.

## Removing the fix

If the new driver causes problems, boot a known-working kernel and uninstall the custom speaker driver:

```bash
sudo dkms uninstall -m snd-soc-tas2783-sdw-hp-g2i -v 1.0 -k 7.3.4-300.fc45.x86_64
```

DKMS should restore the original module it archived during installation. Verify the module selection before rebooting into 7.3.4 again.

This removes the additional speaker workaround, **not** the original HP audio-matching driver or ALSA UCM configuration.

## Future kernel updates

The current package is deliberately restricted to kernel 7.3.4, with automatic installation disabled.

New kernel versions need to be tested before extending support. A future upstream kernel may fix the regression without this workaround.
