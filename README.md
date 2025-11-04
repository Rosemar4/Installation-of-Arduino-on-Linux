 Arduino IDE Installation on WSL Ubuntu 22.04

A  guide to installing and configuring Arduino IDE on Windows Subsystem for Linux (WSL) Ubuntu 22.04, including USB port access configuration.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Step 1: Install Arduino IDE on WSL Ubuntu](#step-1-install-arduino-ide-on-wsl-ubuntu)
- [Step 2: Configure USB Port Access (USBIPD)](#step-2-configure-usb-port-access-usbipd)
- [Step 3: Verify Arduino Connection](#step-3-verify-arduino-connection)
- [Step 4: Create Desktop Shortcut](#step-4-create-desktop-shortcut)
- [Step 5: Set Permissions](#step-5-set-permissions)
- [Step 6: Upload Your First Sketch](#step-6-upload-your-first-sketch)
- [Troubleshooting](#troubleshooting)

 

## Prerequisites

- Windows 10/11 with WSL2 enabled
- Ubuntu 22.04 installed on WSL
- Arduino board (Arduino Uno, Mega, Nano, etc.)
- USB cable to connect Arduino to your computer

 

## Step 1: Install Arduino IDE on WSL Ubuntu

### 1.1 Update System Packages
```bash
sudo apt update
sudo apt upgrade -y
```

### 1.2 Download Arduino IDE
Visit the [Arduino official website](https://www.arduino.cc/en/software) or use wget:

```bash
cd ~/Downloads
wget https://downloads.arduino.cc/arduino-ide/arduino-ide_2.3.6_Linux_64bit.AppImage
```

*Note: Replace `2.3.6` with the latest version number if different.*

### 1.3 Make the AppImage Executable
```bash
chmod +x arduino-ide_2.3.6_Linux_64bit.AppImage
```

### 1.4 Install FUSE (Required for AppImage)
```bash
sudo apt install libfuse2
```

### 1.5 Run Arduino IDE
```bash
./arduino-ide_2.3.6_Linux_64bit.AppImage
```

---

## Step 2: Configure USB Port Access (USBIPD)

This is the **critical step** for WSL users. Without this, WSL cannot access USB devices connected to Windows.

### 2.1 Install USBIPD on Windows

1. Open **PowerShell as Administrator** on Windows
2. Install USBIPD-WIN using winget:

```powershell
winget install --interactive --exact dorssel.usbipd-win
```

Alternatively, download from [USBIPD-WIN GitHub Releases](https://github.com/dorssel/usbipd-win/releases)

### 2.2 Install USBIP Tools on WSL Ubuntu

Open your WSL Ubuntu terminal:

```bash
sudo apt install linux-tools-generic hwdata
sudo update-alternatives --install /usr/local/bin/usbip usbip /usr/lib/linux-tools/*-generic/usbip 20
```

### 2.3 Connect Arduino to WSL

1. **Plug in your Arduino** to your computer's USB port

2. **On Windows PowerShell (Administrator)**, list all USB devices:
```powershell
usbipd list
```

You should see output like:
```
BUSID  VID:PID    DEVICE                                           STATE
1-4    2341:0043  Arduino Uno                                      Not attached
```

3. **Bind the Arduino device** (one-time setup):
```powershell
usbipd bind --busid 1-4
```
*Replace `1-4` with your actual BUSID*

4. **Attach the device to WSL**:
```powershell
usbipd attach --wsl --busid 1-4
```

### 2.4 Verify in WSL Ubuntu

```bash
ls /dev/tty*
```

You should see `/dev/ttyACM0` (or `/dev/ttyUSB0` for some boards) in the output.

---

## Step 3: Verify Arduino Connection

### 3.1 Check USB Device in WSL
```bash
lsusb
```

You should see your Arduino listed:
```
Bus 001 Device 002: ID 2341:0043 Arduino SA Uno R3 (CDC ACM)
```

### 3.2 Check Port Permissions
```bash
ls -l /dev/ttyACM0
```

---

## Step 4: Create Desktop Shortcut

### 4.1 Move Arduino IDE to Applications Folder
```bash
sudo mkdir -p /opt/arduino-ide
sudo mv ~/Downloads/arduino-ide_2.3.6_Linux_64bit.AppImage /opt/arduino-ide/
```

### 4.2 Create Desktop Entry
```bash
nano ~/.local/share/applications/arduino-ide.desktop
```

Add the following content:
```ini
[Desktop Entry]
Name=Arduino IDE
Comment=Arduino Integrated Development Environment
Exec=/opt/arduino-ide/arduino-ide_2.3.6_Linux_64bit.AppImage
Icon=/opt/arduino-ide/arduino-icon.png
Terminal=false
Type=Application
Categories=Development;Electronics;
```

### 4.3 Download Arduino Icon (Optional)
```bash
sudo wget https://www.arduino.cc/favicon.ico -O /opt/arduino-ide/arduino-icon.png
```

### 4.4 Make Desktop Entry Executable
```bash
chmod +x ~/.local/share/applications/arduino-ide.desktop
```

### 4.5 Update Desktop Database
```bash
update-desktop-database ~/.local/share/applications
```

---

## Step 5: Set Permissions

### 5.1 Add User to dialout Group
```bash
sudo usermod -a -G dialout $USER
sudo usermod -a -G plugdev $USER
```

### 5.2 Create udev Rules
```bash
sudo nano /etc/udev/rules.d/99-arduino.rules
```

Add the following lines:
```
SUBSYSTEMS=="usb", ATTRS{idVendor}=="2341", ATTRS{idProduct}=="0043", GROUP="dialout", MODE="0666"
SUBSYSTEMS=="usb", ATTRS{idVendor}=="2341", ATTRS{idProduct}=="0001", GROUP="dialout", MODE="0666"
SUBSYSTEMS=="usb", ATTRS{idVendor}=="2a03", GROUP="dialout", MODE="0666"
SUBSYSTEMS=="usb", ATTRS{idVendor}=="1a86", ATTRS{idProduct}=="7523", GROUP="dialout", MODE="0666"
```

### 5.3 Reload udev Rules
```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

### 5.4 Log Out and Back In
For group changes to take effect:
```bash
exit
```
Then restart your WSL session.

---

## Step 6: Upload Your First Sketch

### 6.1 Launch Arduino IDE
```bash
/opt/arduino-ide/arduino-ide_2.3.6_Linux_64bit.AppImage
```

Or search for "Arduino IDE" in your application menu.

### 6.2 Configure Board and Port

1. Go to **Tools → Board** → Select your Arduino board (e.g., Arduino Uno)
2. Go to **Tools → Port** → Select `/dev/ttyACM0`

### 6.3 Write a Test Sketch

```cpp
void setup() {
  Serial.begin(9600);
}

void loop() {
  Serial.println("Hello from Arduino on WSL!");
  delay(1000);
}
```

### 6.4 Upload the Sketch

1. Click the **Upload** button (right arrow icon)
2. Wait for "Upload Complete" message
3. Open **Tools → Serial Monitor**
4. Set baud rate to **9600**
5. You should see "Hello from Arduino on WSL!" printed every second

---

## Troubleshooting

### Issue: `/dev/ttyACM0` not found

**Solution:**
1. Ensure Arduino is properly attached to WSL:
   ```powershell
   # On Windows PowerShell (Administrator)
   usbipd list
   usbipd attach --wsl --busid <YOUR-BUSID>
   ```

2. Verify in WSL:
   ```bash
   ls /dev/tty*
   dmesg | grep tty
   ```

### Issue: Permission Denied when uploading

**Solution:**
```bash
sudo chmod 666 /dev/ttyACM0
```

Or ensure you're in the dialout group:
```bash
groups
# Should show: dialout plugdev
```

### Issue: Arduino not detected in usbipd list

**Solution:**
1. Try a different USB cable
2. Try a different USB port
3. Restart Arduino IDE
4. Reinstall Arduino drivers on Windows

### Issue: Upload fails with "avrdude: ser_open(): can't open device"

**Solution:**
1. Close any serial monitor or program using the port
2. Detach and reattach USB:
   ```powershell
   # Windows PowerShell (Administrator)
   usbipd detach --busid <BUSID>
   usbipd attach --wsl --busid <BUSID>
   ```

### Issue: USBIPD attach fails

**Solution:**
1. Ensure WSL2 is installed (not WSL1):
   ```powershell
   wsl --list --verbose
   ```
   
2. Update WSL:
   ```powershell
   wsl --update
   ```

3. Restart WSL:
   ```powershell
   wsl --shutdown
   ```

### Issue: Arduino IDE won't launch

**Solution:**
```bash
sudo apt install libfuse2 libgtk-3-0 libwebkit2gtk-4.0-37
```

---

## Quick Reference: Daily Workflow

### Starting Your Arduino Session

1. **Connect Arduino to PC**
2. **In Windows PowerShell (Administrator):**
   ```powershell
   usbipd attach --wsl --busid <YOUR-BUSID>
   ```

3. **In WSL Ubuntu:**
   ```bash
   ls /dev/ttyACM0  # Verify connection
   /opt/arduino-ide/arduino-ide_2.3.6_Linux_64bit.AppImage
   ```

### Ending Your Session

**In Windows PowerShell (Administrator):**
```powershell
usbipd detach --busid <YOUR-BUSID>
```

---

## Additional Resources

- [Arduino Official Documentation](https://docs.arduino.cc/)
- [USBIPD-WIN GitHub](https://github.com/dorssel/usbipd-win)
- [WSL Documentation](https://docs.microsoft.com/en-us/windows/wsl/)
- [Arduino IDE Download](https://www.arduino.cc/en/software)

---

## Contributing

Found an issue or have improvements? Feel free to:
- Open an issue
- Submit a pull request
- Share your experience

---

## License

This guide is provided as-is for educational purposes. Arduino and related trademarks belong to their respective owners.

---

## Author

Created by: [Rosey]  
Last Updated: November 2025  
Arduino IDE Version: 2.3.6

<img width="1366" height="768" alt="Screenshot (212)" src="https://github.com/user-attachments/assets/24d56bca-0417-4bc9-84e3-f21e84322bd2" />

 
