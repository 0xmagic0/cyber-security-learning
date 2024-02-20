# Android Application Testing Setup

## Overview
- Prerequisites
- Install Genymotion and Create VM
- Setup Burp Suite.
- Setup Frida
- SSL Pinning Bypass

## Prerequisites
- Virtual Box - Read the prerequisites for your OS [Genymotion - Requirements](https://docs.genymotion.com/desktop/Get_started/Requirements/)

## Install Genymotion and Create VM
- Go to [Genymotion Download](https://www.genymotion.com/product-desktop/download/)
- Select the release appropriate to your OS
- Once the file has been downloaded, run the executable
- Search for the Genymotion application and open it
- Create an account and login
- Create an VM - Android device e.g.: Google Pixel with Android 10

## Burp Suite Setup
- Export the certificate in DER format with the name `cacert.der`
- Open the Terminal
- Change the certificate format from DER to PEM
```bash
openssl x509 -inform DER -in cacert.der -out cacert.pem
```
- Get the certificate hash value
```bash
openssl x509 -inform PEM -subject_hash_old -in cacert.pem | head -1
```
-  The output would be similar to `9a5ba575.0`
- Change the file name
```bash
mv cacert.pem 9a5ba575.0
```
- Ensure that the shell is not read only
```bash
adb remount
```
- If previous command fails then remount with
```bash
adb shell
#mount root directory / as rw
mount -o rw,remount /
```
- Push the certificate to the device, connect to the device via ADB, and place the certificate to the /system/etc/security/cacerts/ directory
```bash
adb push 9a5ba575.0 /sdcard/
adb shell
mv /sdcard/9a5ba575.0 /system/etc/security/cacerts/
chmod 664 /system/etc/security/cacerts/9a5ba575.0
```
- Set Android global proxy
```bash
adb shell settings put global http_proxy 192.168.1.xx:8080
```
- Before turning off the decide, disable global proxy
```bash
adb shell settings put global http_proxy :0
```

## Setup Frida

## SSL pinning bypass
- Create a python virtual environment
```bash
python3 -m venv frida_env
# activate environment
source frida_env/bin/activate
# install tools
pip install frida frida-tools objection
# to deactivate the environment simply run
deactivate
```

