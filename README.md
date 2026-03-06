#!/bin/bash
# =========================================================
# IGOR OS v7.0 - THE GENESIS (Account Creation Edition)
# =========================================================

# ... (baasseaded jäävad samaks, mis v6.5-s) ...

# 1. TÄIENDATUD SISSELOGIMISAKEN (Create Account funktsiooniga)
mkdir -p airootfs/usr/share/lightdm-webkit/themes/igor-qortal/
cat <<'EOF' > airootfs/usr/share/lightdm-webkit/themes/igor-qortal/index.html
<html>
<head>
    <style>
        body { background-color: #0d1117; color: white; display: flex; justify-content: center; align-items: center; height: 100vh; margin: 0; font-family: sans-serif; }
        .auth-card { background: #161b22; border: 1px solid #30363d; padding: 30px; border-radius: 12px; width: 450px; text-align: center; }
        input { width: 100%; padding: 12px; margin: 10px 0; background: #0d1117; border: 1px solid #30363d; color: white; border-radius: 6px; }
        .btn-login { background: #238636; color: white; border: none; padding: 12px; width: 100%; border-radius: 6px; cursor: pointer; font-weight: bold; }
        .btn-create { background: transparent; color: #58a6ff; border: 1px solid #58a6ff; padding: 10px; width: 100%; border-radius: 6px; cursor: pointer; margin-top: 15px; }
        .seed-box { display: none; background: #000; border: 1px dashed #f1e05a; padding: 15px; margin-top: 10px; font-family: monospace; }
    </style>
</head>
<body>
    <div class="auth-card" id="main-card">
        <h1>Igor OS</h1>
        <p style="color: #8b949e;">Qortal Identity Access</p>
        
        <div id="login-section">
            <input type="password" id="pass" placeholder="Qortal Backup Password">
            <button class="btn-login" onclick="login()">Login</button>
            <button class="btn-create" onclick="showCreate()">Create New Identity</button>
        </div>

        <div id="create-section" style="display:none;">
            <h3>New Wallet Seed</h3>
            <div class="seed-box" id="seed-phrase" style="display:block;">Generating...</div>
            <p style="font-size: 11px; color: #f85149;">WRITE THIS DOWN! This is your only access to Igor OS.</p>
            <button class="btn-login" onclick="finalizeCreate()">I Saved It, Let's Go!</button>
        </div>
    </div>

    <script>
        function showCreate() {
            document.getElementById('login-section').style.display = 'none';
            document.getElementById('create-section').style.display = 'block';
            // Siin kutsub API-t, et genereerida uus 12-sõnaline seed
            document.getElementById('seed-phrase').innerText = "apple banana cherry dog elephant fox goat house ice jump kite lion"; 
        }

        function login() { window.lightdm.authenticate(null); }
        function finalizeCreate() { location.reload(); }
    </script>
</body>
</html>
EOF

# 2. PAIGALDAJA TÄIENDUS (Genereerib vajadusel uue Linuxi kasutaja Qortali põhiselt)
cat <<'EOF' >> airootfs/root/install_igoros.sh
# Igor OS User Provisioning
echo "Setting up auto-user-creation based on Qortal auth..."
# Skript, mis lennult teeb kasutaja kui Qortal auth õnnestub
cat <<'USER_EOF' > /usr/local/bin/igor-user-gen
#!/bin/bash
# Kui kasutajat pole, loo see Qortal ID põhjal
if ! id "$1" &>/dev/null; then
    useradd -m -G wheel,video,audio -s /bin/bash "$1"
    echo "Welcome to Igor OS, $1"
fi
USER_EOF
chmod +x /usr/local/bin/igor-user-gen
EOF
