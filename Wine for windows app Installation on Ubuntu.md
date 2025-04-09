
# Wine and MobaXterm Installation on Ubuntu

---

## 1. Wine: Download and Install

### Enable 32-bit Architecture (For 64-bit systems)

```bash
sudo dpkg --add-architecture i386
```

### Download and Add the Repository Key

```bash
sudo wget -nc -O /usr/share/keyrings/winehq-archive.key https://dl.winehq.org/wine-builds/winehq.key
```

### Add the Repository Based on Ubuntu Codename

```bash
sudo wget -nc -P /etc/apt/sources.list.d/ https://dl.winehq.org/wine-builds/ubuntu/dists/$(lsb_release -sc)/winehq-$(lsb_release -sc).sources
```

### Update and Install Wine (Choose One: `winehq-stable` or `winehq-devel`)

```bash
sudo apt update
sudo apt install --install-recommends winehq-stable
```

### Configure Wine

```bash
winecfg
```

---

## 2. MobaXterm: Download and Install

- Download the **MobaXterm Personal Preview Version** from the official site.

- Unzip the downloaded package:

```bash
unzip MobaXterm_Personal_22.2_Preview2.zip
```

- Navigate to the unzipped directory and run MobaXterm using Wine:

```bash
cd MobaXterm_Personal_22.2_Preview2/
wine MobaXterm_Personal_22.2_Preview2.exe
```

---

## 3. Add Application Shortcut for MobaXterm

### Download an Icon

- Download the `icon.jpg` and save it in the MobaXterm folder.

### Create `.desktop` File

- Navigate to one of the following directories to create a desktop shortcut:

```bash
cd /usr/share/applications     # For global shortcut (requires sudo)
```

or

```bash
cd ~/.local/share/applications # For local user shortcut
```

- Create and edit the `.desktop` file:

```bash
vi mobaXtrem.desktop
```

### Paste the Following Content into the File

```ini
[Desktop Entry]
Name=MobaXterm
Exec=wine /path/to/MobaXterm_Personal_22.2_Preview2.exe
Type=Application
StartupNotify=true
Icon=/path/to/MobaXterm/icon.jpg
Comment=Run MobaXterm via Wine
Categories=Utility;Application;
```

