#!/bin/bash

# =========================================================
# IGOR OS v7.0 - THE GENESIS (Master Build Script)
# Archi põhi | Qortal Auth | Eesti Edition
# =========================================================

# 1. EHITUSKESKKONNA ETTEVALMISTUS
echo "[-] Paigaldame vajalikud tööriistad..."
sudo pacman -S --needed archiso git wget curl --noconfirm

BUILD_DIR="igor_iso_factory"
rm -rf $BUILD_DIR
mkdir -p $BUILD_DIR
cp -r /usr/share/archiso/configs/releng/* $BUILD_DIR/
cd $BUILD_DIR

# 2. SÜSTEEMI PAKETID
echo "[-] Defineerime tarkvara (Cinnamon + Qortal valmidus)..."
cat <<EOT >> packages.x86_64
cinnamon
lightdm
lightdm-webkit2-greeter
pipewire
pipewire-pulse
networkmanager
jre-openjdk
python-requests
wget
curl
git
EOT

# 3. QORTAL GENESIS LOGIN SCREEN (Sinu pildi stiilis)
echo "[-] Loome Qortal Authentication liidese..."
mkdir -p airootfs/usr/share/lightdm-webkit/themes/igor-qortal/
cat <<'EOF' > airootfs/usr/share/lightdm-webkit/themes/igor-qortal/index.html
<html>
<head>
    <style>
        body { background-color: #0d1117; color: white; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; font-family: sans-serif; }
        .auth-card { background: #161b22; border: 1px solid #30363d; padding: 30px; border-radius: 12px; width: 400px; text-align: center; box-shadow: 0 10px 30px rgba(0,0,0,0.5); }
        h1 { font-size: 24px; margin-bottom: 5px; }
        .nonce { color: #58a6ff; font-family: monospace; font-size: 10px; margin-bottom: 20px; word-break: break-all; opacity: 0.7; }
        input { width: 100%; padding: 12px; margin: 10px 0; background: #0d1117; border: 1px solid #30363d; color: white; border-radius: 6px; box-sizing: border-box; }
        .btn-login { width: 100%; padding: 12px; background: #238636; border: none; color: white; border-radius: 6px; cursor: pointer; font-weight: bold; font-size: 16px; }
        .btn-login:hover { background: #2ea043; }
        .btn-create { background: none; border: none; color: #58a6ff; cursor: pointer; margin-top: 15px; font-size: 14px; text-decoration: underline; }
    </style>
</head>
<body>
    <div class="auth-card">
        <h1>Igor OS</h1>
        <div class="nonce">Challenge: A5-XAeuNc0HD1BLEz3pvgGA3z8qsqBAF</div>
        <div style="text-align: left; font-size: 14px; color: #8b949e; margin-bottom: 5px;">Backup Password</div>
        <input type="password" id="pass" placeholder="Enter password">
        <button class="btn-login" onclick="window.lightdm.authenticate(null)">Continue With Saved Account</button>
        <button class="btn-create" onclick="alert('Uue identiteedi loomine käivitub...')">Create New Identity</button>
    </div>
</body>
</html>
EOF

# 4. INSTALLERI SKRIPT (See, mis seadistab süsteemi pärast installi)
echo "[-] Loome süsteemi seadistaja (Eesti lipp + Paneel paremal)..."
mkdir -p airootfs/root/
cat <<'EOF' > airootfs/root/install_igoros.sh
#!/bin/bash
echo "--- IGOR OS v7.0 FINALIZING INSTALL ---"

# 1. QORTAL NATIVE CORE
mkdir -p /opt/qortal
cd /opt/qortal
bash <(curl -fsSL https://link.qortal.dev/linux-script || wget -qO- https://link.qortal.dev/linux-script)

# 2. EESTI LOKAAT JA PANEEL PAREMALE
localectl set-x11-keymap ee
# Rakendame paneeli asukoha ja musta teema
sudo -u $(logname) dbus-launch gsettings set org.cinnamon panels-enabled "['1:0:right']"
sudo -u $(logname) dbus-launch gsettings set org.cinnamon.desktop.interface gtk-theme 'Adwaita-dark'

# 3. EESTI LIPP MENÜÜSSE
mkdir -p /usr/share/igoros/
curl -L -o /usr/share/igoros/estonia.png https://upload.wikimedia.org/wikipedia/commons/thumb/8/8f/Flag_of_Estonia.svg/32px-Flag_of_Estonia.svg.png
sudo -u $(logname) dbus-launch gsettings set org.cinnamon.applets.menu@cinnamon.org menu-icon "/usr/share/igoros/estonia.png"

# 4. LOGIN SCREENI AKTIVEERIMINE
sed -i 's/greeter-session=.*/greeter-session=lightdm-webkit2-greeter/g' /etc/lightdm/lightdm.conf
echo -e "[greeter]\nwebkit_theme = igor-qortal" > /etc/lightdm/lightdm-webkit2-greeter.conf
systemctl enable lightdm
systemctl enable NetworkManager

echo "INSTALL TEHTUD. REBOOT!"
EOF
chmod +x airootfs/root/install_igoros.sh

# 5. ISO KÜPSETAMINE
echo "[-] Ehitame ISO-faili (see võib võtta aega)..."
sudo mkarchiso -v -w work/ -o out/ .
