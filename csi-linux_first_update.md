# CSI Linux 2023.2 Repo & GPG First Update "Laziness" Fix

Since this build is from 2023, the repo cache is stale, GPG keys have expired, and SSL handshakes are failing.
Instead of manually fixing 15 different GPG errors this block fixes the stale configs.

## What’s actually happening here?
- We force-update ca-certificates first. Without this it won't trust the very servers it's trying to download keys from.
- Directly addresses broken keys for Google Chrome, Element, Oxen, BellSoft, and Vulns.xyz.
- Stops the annoying i386 warnings on repos that only care about 64-bit packages.
- Switches the Wine repo from focal (the wrong version) to jammy (the version CSI 2023.2 actually uses).
- Uses force-confold so apt doesn't wake you up to ask about dnsmasq.conf. It will keep existing configuration files as they are.

💻 The "I just want it to work" Copy-Paste

```
# 1. FIX SSL: Update CA Certificates so the VM trusts 2026-era HTTPS connections
sudo apt update -o Dir::Etc::sourcelist="sources.list" -o Dir::Etc::sourceparts="-"
sudo apt install --only-upgrade ca-certificates -y
sudo update-ca-certificates --fresh

# 2. HOUSEKEEPING: Kill legacy files that cause duplicate entry errors
sudo rm -f /etc/apt/sources.list.d/apt-vulns-sexy.list
sudo rm -f /etc/apt/trusted.gpg.d/apt-vulns-xyz.gpg

# 3. GPG REFRESH: Fetch keys and shove them into the proper /usr/share/keyrings path
# Chrome
wget -q -O - https://dl.google.com/linux/linux_signing_key.pub | gpg --dearmor | sudo tee /usr/share/keyrings/google-chrome.gpg > /dev/null
# Element
sudo wget -O /usr/share/keyrings/element-io-archive-keyring.gpg https://packages.element.io/debian/element-io-archive-keyring.gpg
# Oxen
curl -s https://deb.oxen.io/pub.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/oxen.gpg > /dev/null
# Vulns.xyz
curl -sSf https://apt.vulns.xyz/kpcyrd.pgp | gpg --dearmor | sudo tee /usr/share/keyrings/apt-vulns-xyz.gpg > /dev/null
# BellSoft
wget -q -O - https://download.bell-sw.com/pki/GPG-KEY-bellsoft | gpg --dearmor | sudo tee /usr/share/keyrings/bellsoft.gpg > /dev/null

# 4. REPO REWRITE: Use 'signed-by' tags to make APT stop complaining about trust
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome.gpg] https://dl.google.com/linux/chrome/deb/ stable main" | sudo tee /etc/apt/sources.list.d/google-chrome.list
echo "deb [signed-by=/usr/share/keyrings/element-io-archive-keyring.gpg] https://packages.element.io/debian/ default main" | sudo tee /etc/apt/sources.list.d/element-io.list
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/oxen.gpg] https://deb.oxen.io jammy main" | sudo tee /etc/apt/sources.list.d/oxen.list
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/apt-vulns-xyz.gpg] https://apt.vulns.xyz stable main" | sudo tee /etc/apt/sources.list.d/apt-vulns-xyz.list
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/bellsoft.gpg] https://apt.bell-sw.com/ stable main" | sudo tee /etc/apt/sources.list.d/bellsoft.list
# Force Wine to Jammy (CSI 2023.2 is based on Ubuntu 22.04)
echo "deb [arch=amd64,i386] https://dl.winehq.org/wine-builds/ubuntu/ jammy main" | sudo tee /etc/apt/sources.list.d/wine.list

# 5. THE FINISHER: Full system upgrade
# force-confold keeps your dnsmasq.conf and other modified configs intact.
sudo apt update
sudo apt upgrade -y -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold"
```

Note: This recovery documentation was generated with AI assistance to ensure the syntax is tight and the pro-lazy vibes are authentic.
