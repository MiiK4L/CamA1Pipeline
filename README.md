# 🖨️ OctoPi Custom — Bambu A1 + C270 + Tailscale

Pipeline CI/CD GitHub Actions générant automatiquement une image **OctoPi personnalisée** pour **Raspberry Pi 3 B+** et **Zero 2 W**, prête à monitorer une **Bambu Lab A1** avec une **Logitech C270** en 720p 30fps.

---

## 📦 Ce que contient l'image

| Composant | Détail |
|-----------|--------|
| **OctoPrint** | Dernière version stable |
| **Plugin OctoEverywhere** | Accès à distance + app iPhone OctoApp |
| **Plugin Bambu Printer** | Monitoring Bambu Lab A1 via MQTT |
| **Webcam Logitech C270** | Configurée en 1280×720 @ 30fps |
| **Tailscale** | VPN mesh (connexion automatique au premier boot) *(optionnel)* |
| **MotionEye** | Surveillance / enregistrement via Docker *(optionnel)* |
| **Gouverneur CPU** | Réglé en `performance` pour fluidité max |

---

## 🚀 Étape 1 — Forker ce repo et configurer les secrets GitHub

### 1.1 Forker le repo

Clique sur **Fork** en haut à droite de la page GitHub pour copier ce repo sur ton compte.

### 1.2 Configurer les secrets GitHub Actions

Va dans **Settings → Secrets and variables → Actions → New repository secret** de ton fork.

#### 🔑 `GITHUB_TOKEN` (automatique)
> Déjà disponible, rien à faire.

---

#### 🌐 `TAILSCALE_AUTH_KEY` *(optionnel — pour accès VPN au Pi)*

> **Si tu veux te connecter à l'OctoPi depuis partout sans ouvrir de port :**

1. Va sur [https://login.tailscale.com/admin/settings/keys](https://login.tailscale.com/admin/settings/keys)
2. Clique **Generate auth key**
3. Options recommandées :
   - ✅ **Reusable** *(si tu veux re-flasher sans recréer de clé)*
   - ✅ **Ephemeral** *(le Pi disparaît de ta liste s'il est hors ligne longtemps)*
4. Copie la clé (`tskey-auth-xxxx...`)
5. Ajoute le secret GitHub : `TAILSCALE_AUTH_KEY` = `tskey-auth-xxxx...`

> **Si tu ne veux PAS Tailscale :** laisse ce secret vide ou ne le crée pas. Le Pi sera uniquement accessible en local sur ton réseau Wi-Fi.

---

#### 🔵 `OCTOEVERYWHERE_API_KEY` *(optionnel — pour liaison auto au compte OctoEverywhere)*

> **Uniquement nécessaire si tu veux que chaque nouvelle image soit automatiquement liée à ton compte OctoEverywhere.**

- Si tu ne le mets pas, la liaison se fait manuellement via l'interface OctoPrint au premier boot (une popup apparaît automatiquement).
- Récupère ta clé sur [https://octoeverywhere.com/appportal/v1/](https://octoeverywhere.com)

> **Recommandation :** laisse ce secret vide pour la première installation. La liaison manuelle prend 30 secondes.

---

## ⚙️ Étape 2 — Lancer le build

Le build se lance **automatiquement à chaque push** sur la branche `main`.

Pour le déclencher manuellement :
- Va dans **Actions → Build Custom OctoPi Image → Run workflow**

> ⏱️ **Temps de build estimé : 1h30 – 2h** (compilation ARM native via QEMU sur serveurs GitHub).

Quand le build est terminé, l'image apparaît dans l'onglet **Releases** de ton repo.

---

## 💾 Étape 3 — Flasher la carte SD

### Option A — Raspberry Pi Imager (recommandé)

1. Télécharge [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Clique **Choisir l'OS → Utiliser une image personnalisée**
3. Télécharge le `.zip` depuis l'onglet **Releases** de ton repo GitHub

### Option B — Via `rpi-imager.json` (futur)

Une fois que le build a tournée au moins une fois, le fichier `rpi-imager.json` contiendra l'URL et le hash SHA256 de la dernière image. Tu pourras le charger directement dans **Raspberry Pi Imager → Choisir l'OS → Charger une liste personnalisée**.

---

## 📡 Étape 4 — Configurer le Wi-Fi (obligatoire)

> ⚠️ La personnalisation de Raspberry Pi Imager est **grisée** pour les images custom — configure le Wi-Fi manuellement après le flash.

**Après le flash**, la partition `bootfs` (FAT32) apparaît automatiquement sur ton bureau Mac.

**Méthode 1 (Trixie/Bookworm) — modifier `wifi.nmconnection`** :

Ouvre le fichier `bootfs/wifi.nmconnection` et modifie uniquement les deux lignes `ssid` et `psk` :

```ini
[connection]
id=preconfigured
type=wifi

[wifi]
mode=infrastructure
ssid=TON_RÉSEAU_WIFI

[wifi-security]
key-mgmt=wpa-psk
psk=TON_MOT_DE_PASSE

[ipv4]
method=auto

[ipv6]
addr-gen-mode=default
method=auto
```

**Méthode 2 (fallback) — si `wifi.nmconnection` absent** :

Crée un fichier `wpa_supplicant.conf` à la racine de `bootfs` :

```ini
country=FR
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1

network={
    ssid="TON_RÉSEAU_WIFI"
    psk="TON_MOT_DE_PASSE"
    key_mgmt=WPA-PSK
}
```

**Activer SSH** dans tous les cas : crée un fichier vide `ssh` (sans extension) à la racine de `bootfs` :

```bash
# Terminal Mac :
touch /Volumes/bootfs/ssh
```

**Éjecte la carte SD** puis insère-la dans le Pi.

---

## 🖨️ Étape 5 — Configurer la Bambu Lab A1

### Trouver les infos de connexion de l'A1

Sur l'imprimante elle-même :
1. **Écran → Paramètres → Réseau**
2. Note l'**adresse IP** de l'imprimante (ex: `192.168.1.42`)
3. **Écran → Paramètres → Compte → Code d'accès**
4. Note le **code d'accès** (8 chiffres)
5. **Écran → Paramètres → À propos → Numéro de série**
6. Note le **numéro de série** (format `X1Cxxxxxxxx`)

### Compléter `bambu.json`

Deux options :

**Option A — Avant de flasher :** modifie `src/config/bambu.json` dans ton repo :
```json
{
  "printer_ip": "192.168.1.42",
  "access_code": "12345678",
  "serial_number": "X1Cxxxxxxxxx",
  "mqtt_port": 8883
}
```
Puis commit + push → relance un build → reflash.

**Option B — Après le boot (recommandé)** : connecte-toi via SSH ou via l'interface OctoPrint :
- **OctoPrint → Paramètres → Bambu Printer → Configurer**
- Renseigne IP, code d'accès et numéro de série directement.

---

## 🔌 Étape 6 — Premier démarrage

1. Insère la carte SD dans le Raspberry Pi
2. Branche la **Logitech C270** sur un port USB
3. Alimente le Pi

Attends ~2 minutes, puis accède à OctoPrint :

| Méthode | URL |
|---------|-----|
| **Réseau local** | `http://octopi.local` |
| **Tailscale** *(si configuré)* | `http://octopi-a1` depuis n'importe où |
| **IP directe** | `http://192.168.1.XXX` (voir ton routeur) |

**Accès SSH :**

```bash
ssh pi@octopi.local
# Mot de passe par défaut : raspberry
```

> ⚠️ Change le mot de passe dès ta première connexion : `passwd`
> Si `octopi.local` ne répond pas, utilise l'IP directe : `ssh pi@192.168.1.XXX`

---

## 📱 Étape 7 — Configurer OctoApp (iPhone)

1. Télécharge **OctoApp** sur l'App Store
2. Ouvre OctoPrint dans Safari → tu verras une popup OctoEverywhere
3. Clique **Lier mon compte** et connecte-toi à OctoEverywhere
4. OctoApp détectera automatiquement ton imprimante
5. Le flux 720p 30fps de la C270 sera disponible immédiatement 📷

---

## 🔒 Étape 8 — Vérifier Tailscale *(si configuré)*

Au premier boot, le service `tailscale-autoconnect` :
1. Attend que le Wi-Fi soit actif
2. Exécute `tailscale up --authkey=... --hostname=octopi-a1`
3. **Supprime la clé d'auth du disque** (sécurité)
4. Le Pi apparaît dans ton compte Tailscale sous le nom `octopi-a1`

Vérifie sur [https://login.tailscale.com/admin/machines](https://login.tailscale.com/admin/machines) que `octopi-a1` est bien connecté (point vert).

> Si le Pi n'apparaît pas après 5 minutes : SSH en local (`ssh pi@octopi.local`) et lance `sudo tailscale status`.

---

## 🐳 MotionEye (optionnel — enregistrement vidéo)

MotionEye permet de conserver les vidéos de tes impressions sur le Pi.

```bash
# Sur le Pi via SSH
docker-compose up -d
```

Interface accessible sur : `http://octopi.local:8765`

---

## 🖨️ Supports 3D pour la C270 sur Bambu A1

| Modèle | Lien |
|--------|------|
| Bambu A1 / A1 mini — C270 Mount | [MakerWorld](https://makerworld.com/en/models/112690) |
| C270 Clip-on Mount universel | [Printables](https://www.printables.com/model/685957-bambu-a1-mini-c270-camera-mount) |

---

## 🗂️ Structure du repo

```
.
├── .github/workflows/build.yml   # Pipeline CI/CD principal
├── src/config/
│   ├── bambu.json                # Template connexion Bambu A1
│   ├── motion.conf               # Config webcam C270 (720p 30fps)
│   ├── octoprint.cfg             # Plugins OctoPrint
│   └── wpa_supplicant.conf.template  # Template Wi-Fi
├── docker-compose.yml            # MotionEye (optionnel)
├── rpi-imager.json               # Snippet Raspberry Pi Imager
└── README.md
```

---

## ❓ Récapitulatif des features optionnelles

| Feature | Requis ? | Configuration |
|---------|----------|---------------|
| Tailscale | ❌ Optionnel | Secret GitHub `TAILSCALE_AUTH_KEY` |
| OctoEverywhere auto-link | ❌ Optionnel | Secret GitHub `OCTOEVERYWHERE_API_KEY` |
| Bambu A1 IP/Code d'accès | ✅ Pour monitoring | `src/config/bambu.json` ou via UI OctoPrint |
| Wi-Fi | ✅ Obligatoire | `octopi-wpa-supplicant.txt` sur la carte SD |
| Logitech C270 | ✅ Pour webcam | Brancher sur USB avant le boot |
| MotionEye | ❌ Optionnel | `docker-compose up -d` sur le Pi |
