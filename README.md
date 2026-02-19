# 📸 OctoPi Bambu A1 Pipeline (Pi 3B+ / Zero 2 W)

Ce projet fournit une infrastructure CI/CD complète avec GitHub Actions pour générer automatiquement une image **OctoPi Custom**. 
Elle est optimisée pour Raspberry Pi 3 B+ et Zero 2 W avec une caméra **Logitech C270**, **OctoEverywhere**, et l'intégration **Bambu Lab A1**.

## 🛠 Fonctionnalités
- Image générée automatiquement à chaque push via **CustomPiOS** (Fork guysoft).
- Webcam Logitech C270 configurée en **720p à 30fps**.
- **OctoEverywhere** intégré pour l'accès externe.
- **Bambu Connect** prêt à être utilisé pour la Bambu Lab A1.
- Optimisation système avec gouverneur processeur réglé sur `performance`.
- **MotionEye** en option via Docker pour la surveillance ou l'enregistrement avancé.

## 🚀 Installation & Flash

1. **Téléchargez la dernière Release**
   Une image au format `.zip` est automatiquement construite dans l'onglet **Releases** de ce repository GitHub par notre GitHub Actions CI/CD en forkant l'architecture guysoft/OctoPi.
   Vous pouvez aussi inclure le fichier `rpi-imager.json` directement dans **Raspberry Pi Imager** (Option "Utiliser une image personnalisée OS").

2. **Édition du WiFi**
   Avant de retirer la carte SD de l'ordinateur, ouvrez la partition `boot` et modifiez le fichier `octopi-wpa-supplicant.txt` avec les identifiants de votre réseau en vous basant sur le modèle `src/config/wpa_supplicant.conf.template`.

3. **Clé API Bambu & OctoEverywhere**
   À la racine de la carte SD (ou via SSH dans `/home/pi/bambu/`), renseignez le fichier `bambu.json` (voir `src/config/bambu.json`) avec le code d'accès de l'imprimante A1 (situé dans l'écran de l'imprimante). 
   Pour OctoEverywhere, complétez le setup via l'interface web pour le lier à votre compte (L'API key sera automatiquement configuré si injectée via secret Github).

## 📱 Utilisation avec OctoApp (iPhone)
Une fois OctoPrint démarré :
1. Téléchargez **OctoApp** sur l'App Store de votre iPhone.
2. Ajoutez votre imprimante grâce au QR Code OctoEverywhere.
3. Le flux 720p 30 FPS de la C270 sera disponible avec une fluidité impressionnante sans configurations supplémentaires grâce au `motion.conf` optimisé intégré automatiquement lors du process CI/CD !

## 🖨 Modèles 3D Recommandés pour Mount Bambu A1
Pour fixer correctement votre Logitech C270 à la Bambu A1 sans vibrations :
- [Bambu Lab A1 mini / A1 Logitech C270 Mount (MakerWorld)](https://makerworld.com/en/models/112690)
- [C270 Simple Clip-on Mount (Printables)](https://www.printables.com/model/685957-bambu-a1-mini-c270-camera-mount)

## 🐳 MotionEye Backup (Optionnel)
Si vous souhaitez bénéficier de MotionEye pour un système de rétention plus complexe :
```bash
docker-compose up -d
```

## ⚙️ Structure du Repository
- `.github/workflows/build.yml` : Workflow CustomPiOS / Action Docker logic
- `src/config/` : Payload configurations poussées dans l'image
- `docker-compose.yml` : Options standalone
- `rpi-imager.json` : Snippet format Imager

Amusez-vous bien ! 👾
