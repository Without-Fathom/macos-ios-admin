# uniFLOW SmartClient for macOS: download and build wrapper PKG

The purpose of this guide is to explain how to turn the ISO provided by the uniFLOW Online admin portal into something that can actually be used on macOS. The ISO contains the SmartClient installer package and a tenant configuration file, which need to be deployed together so that the client is configured for your organisation's uniFLOW tenant, and can authenticate the user using Single Sign-On.  
  
**Prerequisite:** uniFLOW Online admin access to manage SmartClient installers. Local macOS admin credentials are required to test/install the resulting PKG.

### 1. Locate the current Mac SmartClient installer

In **uniFLOW Online**:

a. Go to **Extensions**.\
b. Select **uniFLOW SmartClient**.\
c. Select **Installer configuration and creation**.\
d. Select **Manage installers**.

<img width="1542" height="513" alt="Manage installers" src="https://github.com/user-attachments/assets/4cb5343b-d7b5-4117-b677-a890e50a7935" />

*Double-click SmartClientMac.iso*

e. Select **SmartClientMac.iso** in the left pane.\
f. Open the **Installer versions** tab.

> [!IMPORTANT]
> Do not assume the version marked Published is the newest version. In the example captured on 11 Nov 2026, 2024.3.0.2 was Published, but 2026.1.0.3 was available above it and was the working current package.

g. Select the newest appropriate macOS release, currently **2026.1.0.3** in our example.\
h. Click the **⋯ three-dot menu** at the far right of that version and select **Download**.

<img width="1553" height="407" alt="Download installer version" src="https://github.com/user-attachments/assets/4a5cf56b-4471-494f-bf14-e65fd6c085d3" />

### 2. Mount the ISO

Open the downloaded ISO in macOS.

Confirm the mounted volume is:

```
/Volumes/SmartClientMac
```


The important contents are:

```
SmartClientForMac.pkg

.tenantcfg.plist
```


### 3. Create a clean wrapper workspace


```
rm -rf "$HOME/uniflow-package"

mkdir -p "$HOME/uniflow-package/root/private/tmp/uniflowclient" "$HOME/uniflow-package/root/Library/Preferences/NT-ware/uniFLOW" "$HOME/uniflow-package/scripts"
```


Copy in the SmartClient installer:

```
cp "/Volumes/SmartClientMac/SmartClientForMac.pkg" "$HOME/uniflow-package/root/private/tmp/uniflowclient/"
```


Copy in the tenant configuration:

```
cp "/Volumes/SmartClientMac/.tenantcfg.plist" "$HOME/uniflow-package/root/Library/Preferences/NT-ware/uniFLOW/.tenantcfg.plist"
```


### 4. Create the installation script

Paste the complete block:

```bash
cat > "$HOME/uniflow-package/scripts/postinstall" <<'EOF'

#!/bin/sh

/usr/sbin/installer -pkg "/private/tmp/uniflowclient/SmartClientForMac.pkg" -target /

RESULT=$?

if [ "$RESULT" -ne 0 ]; then

    exit "$RESULT"

fi

rm -rf "/private/tmp/uniflowclient"

exit 0

EOF
```


Make it executable:

```
chmod 755 "$HOME/uniflow-package/scripts/postinstall"
```


### 5. Build the final PKG

For **2026.1.0.3**:

```
pkgbuild --root "$HOME/uniflow-package/root" --scripts "$HOME/uniflow-package/scripts" --identifier "<your_tenant_domain>.uniflow-smartclient" --version "2026.1.0.3" --install-location / "$HOME/Desktop/uniFLOW-SmartClient-2026.1.0.3.pkg"
```


The finished package is:

```
~/Desktop/uniFLOW-SmartClient-2026.1.0.3.pkg
```


### 6. Test

```
sudo /usr/sbin/installer -pkg "$HOME/Desktop/uniFLOW-SmartClient-2026.1.0.3.pkg" -target / -verboseR
```


A successful install ends with:

```
installer: The install was successful.
```


### What the wrapper does

The resulting package:

1. Places `.tenantcfg.plist` under:

   /Library/Preferences/NT-ware/uniFLOW/
2. Temporarily stages the official SmartClient installer under:

   /private/tmp/uniflowclient/
3. Runs the official `SmartClientForMac.pkg`.
4. Stops with the vendor installer's exit code if installation fails.
5. Removes the temporary installer after success.

### ⚠️ Lesson from 2024.3.0.2

Do **not** simply use the version labelled **Published** without comparing the available Installer versions.

During this procedure, the published **2024.3.0.2** `SmartClientForMac.pkg` failed on the current M-series Mac with:

```
This package is incompatible with this version of macOS.

The package is attempting to install the content to the system volume.
```


The subsequently downloaded **2026.1.0.3** vendor installer installed successfully, and the wrapper built from it was then also successfully installed.

So the practical rule is:

> **For macOS, check Installer versions and download the newest supported SmartClientMac.iso rather than blindly using the Published version.**


