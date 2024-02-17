# iOS Application Testing Setup

## Overview
- Prerequisites.
- Jailbreak The Device.
- Setup Burp Suite.
- Install Tools.

## Prerequisites
- Macbook (Optional).
- iPhone Model and iOS version compatible with a jailbreak method.
- AppleID Account.

## Jailbreak The Device

### Select a Jailbreak Method

For this guide I would be using an iPhone 8 running iOS v16.5.
Different hardware and software versions might require a different jailbreak method.
A guide to find the appropriate method for your device is shown below.

1. Go to https://ios.cfw.guide/get-started/
2. Select the device of choice (iPhone). Scroll down to the device version (iPhone8), and click "Show more".
3. From the table displaying the several jailbreak methods available, select the one matching your OS version.
4. For this guide I will be using `palera1n`.

### Running The Jailbreak on The iOS Device

1. Go to https://ios.cfw.guide/installing-palera1n/
2.  Follow the installation instructions for your OS.
3. Make sure to read the instructions. If using a Macbook with an Apple Silicon Chip, a USB-A to Lighting cable and USB-C to USB-A adapter might be needed to run the jailbreak on the iPhone.
4. Once the palera1n is installed, run the command shown below to jailbreak the device
```bash
palera1n -e thid_should_crash=0
```
5. Follow the instructions on the `ios.cfw.guide` and on the CLI to ensure a successful jailbreak.
6. Once the device boots, open the palera1n application.
7. Install Sileo. Set the password for sudo commands. 
Note: The default password to `SSH` into the device is alpine. The users are `mobile` and `root`.

## Setup Burp Suite

Follow the Portswigger Guide: https://portswigger.net/burp/documentation/desktop/mobile/config-ios-device

1. Ensure that the iPhone and the Macbook are on the same Wi-Fi network.
2. Set Burp Suite to listen on all interfaces.
3. Go to `Setting > Wi-Fi` and configure the iOS device to use a manual proxy.
4. Enter the host machine IP address and Burp Suite's listening port (8080).
5. On the iOS device browse to `http://burpsuite` and download the CA Certificate.
6. Go to `Settings > General > VPN & Device Management` and install the CA Certificate.
7. Go to `Settings > General > About > Certificate Trust Settings` and toggle the PortSwigger CA certificate on.

## Install Tools

### Overview
- Add sources
- Frida
- Objection
- SLL Kill Switch
- HideJB
- appinst
- Appsync Unified
- ipatool
- ldid

### Add Sources
Palera1n should have added repositories to Sileo already; however, there are some that need to be added manually to download specific tools.
1. Open Sileo
2. Select Sources
3. Click on the "+" sign and add the URLs provided below:
    - https://cydia.akemi.app/
    - https://build.frida.re/
    - http://apt.thebigboss.org/repofiles/cydia/
### Frida and Objection
#### Install Frida on The iPhone
1. Open Sileo.
2. Search for "Frida".
3. Follow the UI instructions to install the package.
#### Install Frida and Objection on The Computer
```bash
# This first step can be omitted if desired, but it is recommended.
# Recommended: Create a virtual environment to install the tools.
python3 -m venv mobile-tools
# activate the environment
# source /path/to/venv/bin/activate
source mobile-tools/bin/activate
# Install frida-tools and objection
pip install frida-tools objection
# Verify installation
frida --version
objection --version
# Once done performing testing, the environment can be deactivated
deactivate
```

### Simple Frida and Objection Commands
- Ensure that the frida-server is running on the device.
```bash
# SSH into the device
ssh mobile@ip
# Run the frida-server
sudo frida-server
```
- Enumerate applications on the iOS device. Note: The device needs to be connected via USB.
```bash
frida-ps -Uia
```
- Explore the application with Objection.
```bash
objection -g "applicationName" explore
```
- Bypass SSL Pinning with Objection
```bash
ios sslpinning disable
```
### SSL Kill Switch
Note: This tool is another alternative to bypass SSL pinning.
1. SSH into the device as `root`.
2. Download the SSL Kill Switch .deb package and install it.
```bash
wget https://github.com/nabla-c0d3/ssl-kill-switch2/releases/download/0.14/com.nablac0d3.sslkillswitch2_0.14.deb
dpkg -i com.nablac0d3.sslkillswitch2_0.14.deb
# Restart the Springboard
killall -HUP Springboard
```
3. Go into `Settings` and scroll down. The SSL Kill Switch App should be installed.
4. Open the app and activate it when performing security testing. otherwise keep it deactivated.
### HideJB
Note: Bypass Jailbreak detection on certain applications.
1. Open Sileo, search for "HideJB", click on the package, and follow the steps to install it.
2. Once the Springboard restarts, go to `Settings` and scroll down to the the "HideJB" tool. Open it and activate it.
### appinst and Appsync Unified
Note: To install .ipa files into the iOS device
1. Open Sileo and search for "appinst" and "Appsync Unified". Follow the steps to install both applications.
2. To install `.ipa` files into the iOS device:
```bash
# Send .ipa file to the iOS device
scp /local/file/path.ipa mobile@ip:/path/on/device
# To install the app, first ssh into the device and use appinst
appinst application.ipa
```
### ipatool
Note: CLI tool to search and download .ipa files from the AppStore.
Repo: https://github.com/majd/ipatool
Installation using Brew
```bash
brew tap majd/repo
brew install ipatool
```
### ipatool Usage
```bash
# Authenticate
ipatool auth login -e yourAppleIdEmail@email.com
# Search for App limiting the results
ipatoo search "appName" --limit 4
# Download
ipatool download --bundle-identifier com.name.here
# Some applications might require to obtain a license from the App Store to download them.
# Run the "purchase" command and then re-run the "download" command.
ipatool purchase --bundle-identifier com.name.here
```
### ldid
Note: This tool would be used to pseudo-signs a binaries so we don't have to the the manual process.
Below is an example downloading and installing Damn Vulnerable iOS App (DVIA) using ldid.
1. SSH into the device and run the commands shown below.
```bash
sudo apt-get install wget ldid
wget https://github.com/prateek147/DVIA-v2/raw/master/DVIA-v2-swift.ipa
appinst DVIA-v2-swift.ipa
cd /var/containers/Bundle/application

#Get the app BundleId
find . -type f -name "DVIA-v2.app"

#The output should give a 32 UUID value separated by hyphens
cd 6F47179F-C8C8-47F3-B51C-79F4A1F328CC
# Sign the application
sudo ldid -S DVIA-v2.app
```

