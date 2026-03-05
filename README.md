#!/bin/bash

# =========================================================
# IGOR OS v6.5 - THE SOVEREIGN (Official Master Script)
# Archi põhi | Black Cinnamon | Qortal Auth | Native Drivers
# =========================================================

# Kontrollime õigusi
if [[ $EUID -ne 0 ]]; then
   echo "Seda skripti peab käivitama sudo-ga!"
   exit 1
fi

# Värvid
BLACK='\033[1;30m'
CYAN='\033[0;36m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

clear
echo -e "${BLACK}"
echo "   _____  _____  ____  _____     ____   ____ "
echo "  |_   _|/ ____|/ __ \|  __ \   / __ \ / ____|"
echo "    | | | |  __| |  | | |__) | | |  | | (___  "
echo "    | | | | |_ | |  | |  _  /  | |  | |\___ \ "
echo "   _| |_| |__| | |__| | | \ \  | |__| |____) |"
echo "  |_____|\_____|\____/|_|  \_\  \____/|_____/ "
echo -e "${NC}"
echo -e "         -- THE SOVEREIGN EDITION | v6.5 --"

# 1. ETTEVALMISTUS
echo -e "${CYAN}[1/5] Paigaldame ISO ehitusvahendid...${NC}"
pacman -S --needed archiso git wget curl --noconfirm

BUILD_DIR="igor_iso_factory"
rm -rf $BUILD_DIR
mkdir -p $BUILD_DIR
cp -r /usr/share/archiso/configs/releng/* $BUILD_DIR/
cd $BUILD_DIR

# 2. TARKVARA JA DRAIVERID
echo -e "${CYAN}[2/5] Defineerime tarkvara ja draiverid...${NC}"
cat <<EOT >> packages.x86_64
cinnamon
lightdm
lightdm-webkit2-greeter
pipewire
pipewire-pulse
pipewire-alsa
pipewire-jack
wireplumber
networkmanager
network-manager-applet
wget
curl
git
mesa
xf86-video-intel
nvidia-utils
jre-openjdk
python-requests
EOT

# 3. QORTAL AUTHENTICATION UI (Sinu pildi stiilis)
echo -e "${CYAN}[3/5] Loome Qortal-põhise sisselogimisakna...${NC}"
mkdir -p airootfs/usr/share/lightdm-webkit/themes/igor-qortal/
cat <<'EOF' > airootfs/usr/share/lightdm-webkit/themes/igor-qortal/index.html
<html>
<head>
    <style>
        body { background-color: #0d1117; color: white; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; font-family: sans-serif; }
        .auth-card { background: #161b22; border: 1px solid #30363d; padding: 40px; border-radius: 8px; width: 400px; box-shadow: 0 10px 30px rgba(0,0,0,0.5); }
        h1 { font-size: 22px; text-align: center; margin-bottom: 5px; }
        .nonce { color: #58a6ff; font-family: monospace; font-size: 10px; margin-bottom: 20px; text-align: center; word-break: break-all; }
        input { width: 100%; padding: 12px; margin: 10px 0; background: #0d1117; border: 1px solid #30363d; color: white; border-radius: 6px; }
        button { width: 100%; padding: 12px; background: #1f6feb; border: none; color: white; border-radius: 6px; cursor: pointer; font-weight: bold; margin-top: 10px; }
        button:hover { background: #388bfd; }
    </style>
</head>
<body>
    <div class="auth-card">
        <h1>Qortal Authentication</h1>
        <div class="nonce">Challenge nonce: A5-XAeuNc0HD1BLEz3pvgGA3z8qsqBAF</div>
        <div style="font-size: 14px; color: #8b949e;">Backup Password</div>
        <input type="password" id="pass" placeholder="Enter backup password">
        <button onclick="window.lightdm.authenticate(null)">Continue With Saved Account</button>
    </div>
</body>
</html>
EOF

# 4. IGOR OS NATIVE INSTALLER (See, mis jookseb mälupulgal)
echo -e "${CYAN}[4/5] Kirjutame süsteemi installeri...${NC}"
mkdir -p airootfs/root/
cat <<'EOF' > airootfs/root/install_igoros.sh
#!/bin/bash
echo "--- IGOR OS v6.5 INSTALLER STARTING ---"

# 1. QORTAL NATIVE INSTALL
mkdir -p /opt/qortal
cd /opt/qortal
bash <(curl -fsSL https://link.qortal.dev/linux-script || wget -qO- https://link.qortal.dev/linux-script)

# 2. TEENUSED
systemctl enable NetworkManager
systemctl enable lightdm

# 3. BLACKOUT SETTINGS
sed -i 's/greeter-session=.*/greeter-session=lightdm-webkit2-greeter/g' /etc/lightdm/lightdm.conf
mkdir -p /etc/lightdm/lightdm-webkit2-greeter.conf.d/
echo -e "[greeter]\nwebkit_theme = igor-qortal" > /etc/lightdm/lightdm-webkit2-greeter.conf

# 4. CINNAMON DARK MODE
gsettings set org.cinnamon.desktop.interface gtk-theme 'Adwaita-dark'
gsettings set org.cinnamon.theme name 'Cinnamon-Dark'

echo "IGOR OS INSTALLED. REBOOT AND LOGIN WITH QORTAL."
EOF
chmod +x airootfs/root/install_igoros.sh

# 5. ISO KÜPSETAMINE
echo -e "${GREEN}[5/5] Ehitame buuditavat ISO-faili...${NC}"
mkarchiso -v -w work/ -o out/ .

echo -e "\n${GREEN}TEHTUD! ISO asub kaustas: $BUILD_DIR/out/${NC}"
