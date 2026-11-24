# 🐧 Booting, Bootloaders & Installing Linux

> **A study guide covering:** how a computer boots · bootloaders · BIOS/MBR vs UEFI/GPT · how GNU and Linux form one OS · installing on Windows and Mac · fixing common installation problems.
>
> Bracketed numbers like **[1]** point to the [References](#-references) section at the end.

---

## 📑 Table of Contents

1. [Introduction to Booting and Bootloaders](#1-introduction-to-booting-and-bootloaders)
2. [BIOS/MBR vs UEFI/GPT](#2-biosmbr-vs-uefigpt)
3. [How GNU and Linux Make a Complete OS](#3-how-gnu-and-linux-make-a-complete-os)
4. [Linux Distributions (Flavours) and Desktop Environments](#4-linux-distributions-flavours-and-desktop-environments)
5. [Installation Steps: Windows and Mac](#5-installation-steps-windows-and-mac)
6. [Common Installation Challenges (and Fixes)](#6-common-installation-challenges-and-fixes)
7. [Quick Command Cheat Sheet](#7-quick-command-cheat-sheet)
8. [References](#-references)

---

## 1. Introduction to Booting and Bootloaders

### 1.1 What is booting?

**Booting** is the process of starting a computer, from the moment you press the power button until the operating system is ready to use.

The word comes from **"bootstrapping"**, pulling yourself up by your own bootstraps. The machine starts with nothing loaded and has to load itself, step by step.

### 1.2 The boot flow

```
Power on → Firmware (BIOS/UEFI) → Bootloader → Kernel → Init system → Login screen
```

| Stage | What happens |
|---|---|
| **Power on** | The CPU starts and runs the first instructions stored in firmware. |
| **Firmware (BIOS/UEFI)** | Runs **POST** (Power-On Self Test) to check RAM, CPU, keyboard, etc. Then finds a bootable device (SSD, USB, network). |
| **Bootloader** | A small program on the disk. It finds the OS kernel and loads it into RAM. If several OSes are installed, it shows a menu. |
| **Kernel** | Loads into memory, detects and initializes hardware, mounts the root filesystem, and starts the first process. |
| **Init system** (systemd on most distros) | Starts services such as networking, display manager and sound. |
| **Login screen** | The user logs in and the desktop appears. |

### 1.3 What is a bootloader?

A **bootloader** is a small program that loads the operating system kernel into memory and hands control over to it. Without it, the firmware would not know how to start the OS.

**Why do we need one?**

- Firmware is too simple to understand filesystems and kernels.
- It enables **dual boot** (for example Windows and Linux on one machine).
- It can pass **options to the kernel** (recovery mode, safe mode, and so on). The kernel documentation describes these as *kernel command-line parameters*, some of which are interpreted by the boot loader itself **[4]**.

### 1.4 Common bootloaders

| Bootloader | Used by |
|---|---|
| **GRUB2** | Most Linux distributions |
| **systemd-boot** | Fedora (some setups), Arch users, lightweight UEFI systems |
| **Windows Boot Manager** | Windows |
| **boot.efi** | macOS |
| **U-Boot** | Embedded devices, Raspberry Pi-type boards |

---

## 2. BIOS/MBR vs UEFI/GPT

### 2.1 Four terms, two pairs

- **BIOS** and **UEFI** are **firmware**: the first software that runs at power-on.
- **MBR** and **GPT** are **partition schemes**: how a disk's partitions are recorded.
- They are usually paired as **BIOS + MBR** (old) and **UEFI + GPT** (modern).

### 2.2 Brief history

- **BIOS** came from the 1980s IBM PC era. It runs in 16-bit mode with a tiny text-only interface.
- **MBR** was introduced in 1983. It is a 512-byte sector at the start of the disk holding about 446 bytes of boot code, a 64-byte partition table and a 2-byte signature.
- The limits became obvious: disks grew past 2 TB, and users wanted more than 4 partitions, faster boot and better security.
- **UEFI** started as Intel's EFI in the late 1990s. An industry group formalized it as UEFI in 2005, and GPT is part of that standard.
- **Adoption:** Windows 8 (2012) pushed UEFI and Secure Boot onto mainstream PCs, and Windows 11 requires them.

### 2.3 BIOS vs UEFI

| | **BIOS** | **UEFI** |
|---|---|---|
| Age | 1980s | 2000s onward |
| Mode | 16-bit | 32/64-bit |
| Interface | Text-only, keyboard | Graphical, mouse support |
| Boot speed | Slower | Faster |
| Bootloader location | First sector of the disk | A file on the **EFI System Partition** (FAT32) |
| Secure Boot | ❌ No | ✅ Yes |
| Extras | None | Network boot, drivers, a built-in boot manager |

> 💡 **EFI System Partition (ESP):** on UEFI/GPT systems the device boots from this partition; Microsoft's partition documentation calls it the *system partition* on GPT drives **[13]**.

### 2.4 MBR vs GPT

| | **MBR** | **GPT** |
|---|---|---|
| Max disk/partition size | 2 TB | Practically unlimited (theoretical 18 exabytes) |
| Partitions | 4 primary (or 3 + an extended with logical ones) | 128 by default |
| Partition table copies | One | Primary plus a backup at the end of the disk |
| Corruption protection | None | CRC checksums |
| Works with | BIOS (and UEFI in legacy mode) | UEFI (BIOS possible with extra setup) |

These figures match Microsoft's documentation: MBR is limited to four partitions and 2 TB, while GPT allows up to 128 partitions in the Windows implementation, supports far larger sizes, and replicates and CRC-protects its partition table **[13]**. GPT also keeps a *protective MBR* in sector 0 so older MBR-only tools do not mistake the disk for an empty one **[13]**.

---

## 3. How GNU and Linux Make a Complete OS

### 3.1 Two halves of one system

- **GNU (1983/84):** Stallman's project built the tools first. By the early 1990s it had assembled almost the whole system except a working kernel. Its own kernel, **Hurd**, was slow to mature **[14]**.
- **Linux (1991):** only a **kernel**, with no compiler, shell or utilities of its own.
- **Together:** the Linux kernel plus the GNU tools became a complete, usable free OS **[14]**.

> 📝 **Naming note:** the GNU Project asks that this combination be called **"GNU/Linux"**, since the system as a whole is largely GNU with Linux as its kernel **[14]**. In everyday use it is simply called "Linux".

### 3.2 What GNU contributes

| Component | Role |
|---|---|
| `gcc` | Compiler |
| `glibc` | The C library, which programs use to talk to the kernel |
| `bash` | Shell (the command line) |
| `coreutils` | `ls`, `cp`, `mv`, `cat` and other basic commands |
| `binutils` | Linker, assembler and related tools |
| `make`, `gdb` | Build tool, debugger |

### 3.3 What Linux contributes

- Process and memory management
- Device drivers
- Filesystems
- Networking stack

### 3.4 What a distro adds on top

- Init system (**systemd**)
- Package manager (**apt**, **dnf**, **pacman**)
- Desktop environment (**GNOME**, **KDE**)
- Installer, default apps and security updates

### 3.5 Layer diagram

```
┌──────────────────────────────────┐
│               User               │
├──────────────────────────────────┤
│        Desktop and apps          │
├──────────────────────────────────┤
│     GNU libraries and tools      │
├──────────────────────────────────┤
│           Linux kernel           │
├──────────────────────────────────┤
│             Hardware             │
└──────────────────────────────────┘
```

---

## 4. Linux Distributions (Flavours) and Desktop Environments

### 4.1 What is a distro?

A **Linux distribution (distro)** is the Linux kernel together with GNU tools, a package manager, a desktop environment and an installer, bundled and maintained by a team or company. Different distributions make different choices about stability, software updates and ease of use.

### 4.2 Main distribution families

| Family | Distros | Package manager | Character |
|---|---|---|---|
| **Debian** | Debian, Ubuntu, Linux Mint, Pop!_OS, Kali | `apt` | Large software library and generally beginner-friendly |
| **Red Hat** | Fedora, RHEL, Rocky Linux, AlmaLinux | `dnf` | Enterprise-focused, with Fedora as the community-driven upstream |
| **Arch** | Arch Linux, Manjaro, EndeavourOS | `pacman` | Rolling release, latest software and a more hands-on approach |
| **SUSE** | openSUSE, SUSE Linux Enterprise (SLE) | `zypper` | Strong administration tools; widely used in enterprise environments |
| **Independent** | Alpine Linux, NixOS, Gentoo | Varies | Specialist distributions: lightweight, reproducible or highly customizable |

### 4.3 Release models

- **Fixed release:** A new version is released on a planned schedule, often every 6 months to 2 years (for example, Ubuntu, Debian and Fedora). This makes upgrades more predictable.
- **LTS (Long-Term Support):** A fixed release that receives security and maintenance updates for an extended period, often 5 years or more (for example, Ubuntu LTS).
- **Rolling release:** Updates are delivered continuously rather than through major version releases (for example, Arch Linux and openSUSE Tumbleweed). You get newer software sooner, but updates may require more attention.

### 4.4 Desktop environments

A **desktop environment (DE)** controls the graphical look and feel of the system—panels, menus, windows, settings and the file manager. It does not change the underlying Linux distribution or kernel.

- **GNOME:** Modern, clean and minimal; the default desktop on standard Ubuntu and Fedora Workstation.
- **KDE Plasma:** Highly customizable, with a familiar Windows-like layout.
- **XFCE:** Lightweight and suitable for older or lower-powered hardware.
- **Cinnamon:** Traditional desktop layout; the default on Linux Mint Cinnamon Edition.
- **MATE / LXQt:** Other lightweight desktop environment options.

> **Remember:** A distribution and a desktop environment are different things. For example, Fedora is a distribution, while GNOME and KDE Plasma are desktop environments. A distribution may offer multiple desktop editions.

---

## 5. Installation Steps: Windows and Mac

### 5.A 🪟 On a Windows laptop (dual boot)

#### Before you start

- [ ] **Back up** your important data.
- [ ] **Save the BitLocker recovery key** if BitLocker is on (Settings → Device encryption, or your Microsoft account). Resizing partitions or changing boot settings can trigger a recovery-key prompt.
- [ ] **Turn off Fast Startup:** Control Panel → Power Options → *Choose what the power buttons do* → *Change settings that are currently unavailable* → untick **Turn on fast startup** → Save. Then shut down fully. Ubuntu's community documentation explains that Fast Startup puts Windows into a hybrid shutdown state that GRUB cannot reliably start Windows from **[7]**.
- [ ] **Check you're on UEFI/GPT** with `msinfo32` (look at *BIOS Mode*).

> ⚠️ **BitLocker and "install alongside":** Ubuntu's documentation states the installer cannot read a BitLocker-encrypted Windows partition, so the guided "alongside Windows" option cannot be used safely until BitLocker is turned off. See the official Ubuntu Desktop installation tutorial **[15]**.

#### Steps

1. **Download the ISO** from the distro's official site (for example Ubuntu or Linux Mint).
2. **Verify the checksum** (optional but good practice):
   ```powershell
   certutil -hashfile file.iso SHA256
   ```
3. **Make a bootable USB** (8 GB or more) using **Rufus**, **Balena Etcher** or **Ventoy**. In Rufus choose **GPT** with **UEFI**.
4. **Free up disk space:** open *Disk Management*, right-click the **C:** drive, choose **Shrink Volume**, and free about 40–100 GB. **Leave the space unallocated and don't format it**; the Ubuntu installer handles it **[8][9]**.
5. **Boot from the USB:** restart and press the boot-menu key (F12, F2, Esc or Del, depending on the brand). Pick the USB entry that says **UEFI**.
6. **Try the live session first** and test Wi-Fi, Bluetooth, sound and display.
7. **Start the installer**, then choose language, keyboard layout and network.
8. **Pick the install type:**
   - **"Install alongside Windows"** is easiest and keeps Windows.
   - **"Something else" / manual** lets you set `/` (root), an optional `/home` and swap. It reuses the existing EFI partition and does **not** format it.
   - ❌ Avoid **"Erase disk and install"** on a dual-boot machine; it wipes everything.
9. **Create your user** and set the timezone.
10. **Finish and reboot**, removing the USB when asked.
11. **Choose the OS from the GRUB menu.** After first boot, update the system:
    ```bash
    sudo apt update && sudo apt upgrade
    ```
    (or use the Software Updater).

#### 🛡️ Safer alternative: a VM on Windows

1. Install **VirtualBox** or **VMware Workstation Player**.
2. Create a new VM with **2+ CPU cores, 4 GB RAM and 25+ GB disk**.
3. Attach the ISO, start the VM and run the installer as above.

---

### 5.B 🍎 On a Mac

First check which kind you have: **Apple menu → About This Mac**. **Apple Silicon** (M-series) and **Intel** Macs work very differently.

#### Apple Silicon Macs (M-series)

These cannot boot a normal x86 Linux USB. There are two ways.

**Option 1: Virtual machine (recommended for beginners)**

1. Install a VM app: **UTM** (free), **Parallels** or **VMware Fusion**.
2. Download the **ARM64 (aarch64)** ISO of your distro. The regular x86 ISO will not work.
3. Create a new Linux VM with **4 GB+ RAM** and **30+ GB disk**.
4. Boot from the ISO, run the installer, then reboot without the ISO.

**Option 2: Bare-metal install with Asahi Linux**

1. Use **Fedora Asahi Remix** (a collaboration between the Fedora Asahi SIG and the Asahi Linux project).
2. Run the **Asahi installer from macOS Terminal**. It prepares the system, downloads the image and sets up the boot process **[10]**.
3. **Check hardware support first.** Support varies by chip and model; Fedora's documentation points to the *Feature Support* page on the Asahi Linux site **[10]**. Fedora's announcement lists M1 and M2 machines as supported **[10]**, so do not assume newer chips work without checking.
4. Know the limits: you must keep a **working macOS installation**, because the installer and bootloader updates rely on macOS admin credentials; avoid touching the macOS container partition; and the installer does not currently support full-disk encryption **[10]**.
5. **Back up first.**

#### Intel Macs (dual boot)

1. **Back up** with Time Machine.
2. Download the **x86_64 ISO** and make a USB with Balena Etcher.
3. **Free up space:** Disk Utility → Partition, and resize macOS to leave room.
4. **If it has a T2 security chip** (roughly 2018–2020 models): boot to Recovery (**Cmd + R**) → **Startup Security Utility** → set Secure Boot to **No Security** and **Allow booting from external or removable media** **[11][12]**.
5. Restart while holding **Option (⌥)** and pick the USB entry (shown as **"EFI Boot"**).
6. Run the live session, then the installer. Install into the free space and be careful **not to wipe macOS**.
7. Hold **Option** at startup to choose between macOS and Linux.

> 🔧 **Expect extra work:** Wi-Fi, Bluetooth, keyboard backlight and similar items often need extra drivers. T2 Macs benefit from the **t2linux** project, whose wiki provides T2-ready ISOs and distro-specific guides, and recommends following it over the distro's generic instructions **[11]**.
>
> 🔒 **Security tip from t2linux:** with Secure Boot off you can enable a *firmware password*, and after installing you can re-select "Disallow booting from external or removable media" **[11]**.

---

## 6. Common Installation Challenges (and Fixes)

### 6.1 USB doesn't boot or doesn't appear in the boot menu

| | |
|---|---|
| **Cause** | Corrupted ISO, wrong USB write mode (GPT/UEFI vs MBR/BIOS), or the wrong boot key. |
| **Fix** | Verify the ISO checksum and recreate the USB with Rufus, Etcher or Ventoy. Check the laptop's boot-menu key (F2, F12, Esc or Del). Make sure the USB entry you pick matches your firmware mode (UEFI vs legacy). |

> 💡 On T2 Macs, if you see an Apple message that a software update is required, Secure Boot may still be enforced; re-check Startup Security Utility and try the *other* orange "EFI Boot" entry **[11]**.

### 6.2 Secure Boot blocks the installer

| | |
|---|---|
| **Cause** | Some distros or custom kernels are not signed with keys the firmware trusts. |
| **Background** | Most x86 hardware ships with Microsoft certificates in firmware. Ubuntu uses a Microsoft-signed **shim** and a Canonical-signed **GRUB**; since Ubuntu 18.04 LTS, Secure Boot verifies the bootloader, kernel **and** kernel modules **[1]**. |
| **Fix** | Ubuntu, Fedora and Debian usually work with Secure Boot on. If you hit trouble: **(a)** enroll a key when prompted (**MOK**, Machine Owner Key), or **(b)** disable Secure Boot in the UEFI settings. |

**MOK enrollment, in short** **[1]**:

1. During installation or when installing a third-party (DKMS) driver, a MOK is generated.
2. On the next reboot, the blue **MokManager** screen appears.
3. Choose **Enroll MOK**, check the fingerprint, confirm, and enter the password you set.
4. Reboot. Later DKMS modules are then signed automatically.

> ⚠️ **Don't skip the MokManager screen.** If you choose "Continue boot" instead of enrolling, third-party modules (for example NVIDIA drivers) will not load while Secure Boot is on.

For unofficial or self-built kernels, all kernels must be signed to be loaded by GRUB under Secure Boot. Alternatively, Ubuntu documents disabling validation in shim with `sudo mokutil --disable-validation` (then reboot) **[1]**. Canonical also documents how to sign your own modules and EFI binaries **[3]**.

### 6.3 Windows missing from the GRUB menu, or Windows won't start after installing Linux

| | |
|---|---|
| **Cause** | **Fast Startup** left Windows in a hybrid-shutdown state that GRUB cannot boot from **[7]**; or the firmware boot order was changed; or BitLocker prompts for a recovery key because boot settings changed. |
| **Fix** | In Windows, disable Fast Startup and do a full shutdown **[7]**. Enter the UEFI settings and check the boot order (Windows Boot Manager vs the Linux entry). In Linux, run `sudo update-grub` so GRUB re-scans for Windows. Keep your BitLocker recovery key handy. |

> This entry is not in your original notes (item 3 was missing); it is one of the most common dual-boot problems, so it is included here.

### 6.4 Black screen or freeze at boot or on first login

| | |
|---|---|
| **Cause** | Usually an NVIDIA or hybrid-graphics driver problem. |
| **Fix** | Boot with **`nomodeset`**, then install the proper GPU driver (for example Ubuntu's *Additional Drivers*). |

**How to add `nomodeset` temporarily:**

1. At the GRUB menu, highlight your entry and press **`e`**.
2. Find the line starting with `linux` and add `nomodeset` at the end.
3. Press **Ctrl + X** (or F10) to boot.

**What it does:** per the Linux kernel documentation, `nomodeset` disables kernel modesetting, so graphics (DRM) drivers do not change display modes or do accelerated rendering **[4]**. It is a **troubleshooting** option, not a permanent setting.

### 6.5 Wi-Fi not working

| | |
|---|---|
| **Cause** | The card (often **Broadcom**, some **Realtek** or **MediaTek**) needs proprietary firmware or a newer kernel. |
| **Fix** | Connect by **Ethernet** or **phone USB tethering**, then install the driver or firmware package. Choosing **"Install third-party software"** during installation helps. |

**Steps:**

1. Open **Software & Updates → Additional Drivers** and see whether a proprietary driver is offered (common for Broadcom chips).
2. Identify your chipset first, because Ubuntu offers several different Broadcom drivers (the proprietary *STA* driver and the open *b43* firmware path) and the right one depends on the exact card **[6]**.
3. Check the kernel log for missing firmware:
   ```bash
   sudo dmesg | grep -i firmware
   ```
4. Very new Wi-Fi chips may need a **newer kernel**.

> Ubuntu's own documentation states its goal is for cards to work automatically, and points to *Additional Drivers* as the first place to look **[6]**.

### 6.6 Bluetooth not working or missing

| | |
|---|---|
| **Cause** | Missing firmware, a kernel too old for the hardware, or a **combo Wi-Fi/Bluetooth chip** whose Wi-Fi part has the same issue. |
| **Fix** | Update the system and kernel, install `linux-firmware`, and check `rfkill list` in case the radio is blocked. |

**Steps:**

```bash
sudo apt update && sudo apt upgrade     # update system and kernel
sudo apt install linux-firmware          # firmware blobs
rfkill list                              # look for "blocked"
sudo rfkill unblock bluetooth            # clear a soft block
```

**Understanding `rfkill`:** the kernel documentation describes two kinds of radio block **[5]**:

| Type | Meaning |
|---|---|
| **Soft block** | Set by software; you can clear it with `rfkill unblock`. |
| **Hard block** | A read-only block (often a physical switch or key); software **cannot** override it. |

If `rfkill list` shows **Hard blocked: yes**, check for a physical wireless switch or an Fn-key combination.

---

## 7. Quick Command Cheat Sheet

| Task | Command / Action |
|---|---|
| Check UEFI vs BIOS (Windows) | `msinfo32` → *BIOS Mode* |
| Check UEFI vs BIOS (Linux) | `ls /sys/firmware/efi` (exists only on UEFI boots) |
| Verify ISO checksum (Windows) | `certutil -hashfile file.iso SHA256` |
| Verify ISO checksum (Linux/Mac) | `sha256sum file.iso` / `shasum -a 256 file.iso` |
| Check Secure Boot state | `mokutil --sb-state` |
| Re-scan for other OSes in GRUB | `sudo update-grub` |
| Edit boot options once | Press **`e`** at the GRUB menu |
| List radio blocks | `rfkill list` |
| Unblock Bluetooth | `sudo rfkill unblock bluetooth` |
| Look for firmware errors | `sudo dmesg \| grep -i firmware` |
| Update Ubuntu/Debian | `sudo apt update && sudo apt upgrade` |

---

## 📚 References

**Secure Boot, MOK and signing**

1. Ubuntu Security Documentation. *UEFI Secure Boot.* <https://documentation.ubuntu.com/security/docs/security-features/platform-protections/secure-boot/>
2. Ubuntu Wiki. *UEFI/SecureBoot.* <https://wiki.ubuntu.com/UEFI/SecureBoot>
3. Canonical. *How to sign things for Secure Boot.* <https://canonical.com/blog/how-to-sign-things-for-secure-boot>

**Linux kernel documentation (kernel.org)**

4. The Linux Kernel documentation. *The kernel's command-line parameters* (`kernel-parameters.txt`, includes `nomodeset`). <https://www.kernel.org/doc/Documentation/admin-guide/kernel-parameters.txt> · Patch documenting `nomodeset`: <https://lore.kernel.org/lkml/20211112133230.1595307-6-javierm@redhat.com/>
5. The Linux Kernel documentation. *rfkill: RF kill switch support.* <https://kernel.org/doc/html/latest/driver-api/rfkill.html>

**Ubuntu community and vendor documentation**

6. Ubuntu Community Help Wiki. *Broadcom STA (Wireless).* <https://help.ubuntu.com/community/BroadcomSTA(Wireless)>
7. Ubuntu Community Help Wiki. *Installing Xubuntu* (Fast Startup section). <https://help.ubuntu.com/community/InstallingXubuntu>
8. Ubuntu Community Help Wiki. *DualBoot/Windows.* <https://help.ubuntu.com/community/DualBoot/Windows>
9. Dell Support. *How to Install Ubuntu and Windows 8 Through Windows 11 as a Dual Boot on your Dell Computer.* <https://www.dell.com/support/article/SLN301754/en>
15. Canonical. *Install Ubuntu Desktop* (official tutorial). <https://canonical-ubuntu-desktop-documentation.readthedocs-hosted.com/en/latest/tutorial/install-ubuntu-desktop/>

**Mac installation**

10. Fedora Docs. *Fedora Asahi Remix: FAQ* and *Deviations.* <https://docs.fedoraproject.org/ca/fedora-asahi-remix/faq/> · <https://docs.fedoraproject.org/nl/fedora-asahi-remix/deviations/> · Announcement: <https://fedoramagazine.org/?p=39462&preview=true>
11. t2linux Wiki. *Pre install steps.* <https://wiki.t2linux.org/guides/preinstall/>
12. Apple Platform Security. *Startup Security Utility on a Mac with an Apple T2 Security Chip.* <https://support.apple.com/en-gw/guide/security-pdf/secc7b34e5b5/web>

**Partitioning (MBR/GPT/UEFI)**

13. Microsoft Learn. *Windows and GPT FAQ.* <https://learn.microsoft.com/windows-hardware/manufacture/desktop/windows-and-gpt-faq> · *UEFI/GPT-based hard drive partitions.* <https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/configure-uefigpt-based-hard-drive-partitions>

**GNU and Linux**

14. GNU Project. *Linux and the GNU System.* <https://www.gnu.org/gnu-and-linux> · *GNU/Linux FAQ.* <https://www.gnu.org/gnu/gnu-linux-faq>
