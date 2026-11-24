# RomCloud

[![Release](https://img.shields.io/github/v/release/biliboobrian/romcloud-android?label=APK&logo=android)](https://github.com/biliboobrian/romcloud-android/releases/latest)
[![Android APK](https://github.com/biliboobrian/romcloud-android/actions/workflows/android.yml/badge.svg)](https://github.com/biliboobrian/romcloud-android/actions/workflows/android.yml)

Frontend Android de rétro-gaming inspiré de [Daijishou](https://github.com/TapiocaFox/Daijishou), dont la bibliothèque de jeux se trouve sur un **serveur distant** : l'application affiche la liste des jeux de chaque système, télécharge une ROM à la demande, puis la lance dans l'émulateur choisi.

> Le code source de l'APK Daijishou n'est pas public : son dépôt GitHub ne contient que les définitions de plateformes et d'émulateurs (JSON, licence MIT). RomCloud est donc une application distincte qui **réutilise ces définitions** : même catalogue de systèmes, mêmes modèles de lancement `am start` (Dolphin, DuckStation, PPSSPP…) ; leurs cœurs libretro sont lancés par les moteurs intégrés de RomCloud.

Le serveur est dans [romcloud-server](https://github.com/biliboobrian/romcloud-server) (image Docker `biliboobrian/romcloud-server`), l’application Windows dans [romcloud-desktop](https://github.com/biliboobrian/romcloud-desktop).

## Dépôts RomCloud

| Dépôt | Contenu |
|---|---|
| [romcloud-server](https://github.com/biliboobrian/romcloud-server) | Serveur (Node.js, image Docker) : bibliothèque de jeux, scraping, API |
| [romcloud-android](https://github.com/biliboobrian/romcloud-android) | Application Android (téléphone, tablette) et Android TV |
| [romcloud-desktop](https://github.com/biliboobrian/romcloud-desktop) | Application Windows (Tauri) et son moteur intégré |
| [romcloud-pc](https://github.com/biliboobrian/romcloud-pc) | RomCloud PC : jeux PC sous Android (Winlator Cmod), application compagnon |
| [romcloud-native](https://github.com/biliboobrian/romcloud-native) | RomCloud Native : jeux PC traduits en ARM64 (prototype de recherche) |

Historique commun jusqu’à la version 1.105 : [romCloud](https://github.com/biliboobrian/romCloud) (archivé).

## Installation

### Application Android — Obtainium (recommandé)

[Obtainium](https://github.com/ImranR98/Obtainium) installe l’APK depuis les [Releases GitHub](https://github.com/biliboobrian/romcloud-android/releases) et vous prévient à chaque nouvelle version.

1. Installez Obtainium depuis ses [Releases](https://github.com/ImranR98/Obtainium/releases/latest) ou [F-Droid](https://f-droid.org/packages/dev.imranr.obtainium.fdroid/).
2. Dans Obtainium : **Ajouter une application**, collez l’URL `https://github.com/biliboobrian/romcloud-android`, et dans **Filtrer les APK par expression régulière** saisissez `RomCloud-\d` (chaque Release contient aussi l’APK TV), puis **Ajouter**.
3. Appuyez sur **Installer** (autorisez Obtainium à installer des applications si Android le demande).

Les mises à jour apparaissent ensuite dans Obtainium dès qu’une nouvelle Release est publiée.

### Application Android — installation manuelle

Téléchargez `RomCloud-<version>.apk` depuis la [dernière Release](https://github.com/biliboobrian/romcloud-android/releases/latest) et ouvrez-le sur le téléphone (autorisez l’installation depuis le navigateur ou le gestionnaire de fichiers).

**Mises à jour** : au démarrage, l’application (téléphone et TV) vérifie la dernière Release GitHub et, si elle est plus récente, propose de la télécharger et de l’installer (Android demande la première fois d’autoriser RomCloud à installer des applications). Les versions de développement ne sont pas vérifiées.

Android 8 minimum. Les APK sont signés avec la clé RomCloud — empreinte SHA-256 du certificat :
`3d7da376fa94f95e0c7d46b26638961c9a818c6f2170c04a6dd8cec9d371d559`
(vérifiable avec `apksigner verify --print-certs RomCloud-<version>.apk`).

### Application Android TV

**RomCloud TV** (`RomCloud-TV-<version>.apk`, paquet `com.romcloud.app.tv`) est une application distincte, pensée pour la télécommande : elle apparaît dans le lanceur Android TV / Google TV avec sa bannière. Même serveur, même version, même clé de signature que l’application téléphone ; les deux peuvent être installées côte à côte.

Installation sur le téléviseur :
- **Obtainium** (installé sur le téléviseur) : même URL que ci-dessus, avec le filtre `RomCloud-TV` ;
- ou l’application **Downloader** (AFTVnews) : saisissez l’adresse de l’APK TV de la [dernière Release](https://github.com/biliboobrian/romcloud-android/releases/latest) ;
- ou depuis un ordinateur : `adb connect <ip-du-téléviseur>` puis `adb install RomCloud-TV-<version>.apk`.

Accès aux fichiers : beaucoup de téléviseurs n’ont pas l’écran « Accès à tous les fichiers ». RomCloud TV l’ouvre s’il existe, sinon il affiche la commande à exécuter une fois depuis un ordinateur (débogage ADB activé dans les Options pour les développeurs) :

```bash
adb shell appops set --uid com.romcloud.app.tv MANAGE_EXTERNAL_STORAGE allow
```

Utilisation à la télécommande :
- **Systèmes** : grille de cartes (image configurée sur le serveur) ; *Actualiser* et *Paramètres* en haut.
- **Jeux** : l’arrière-plan et le haut de l’écran présentent le jeu sélectionné (capture, titre, infos, description, état) ; dessous, les filtres *Tous / Téléchargés / À télécharger*, *Rechercher*, puis les rangées *Téléchargés*, *Ajoutés récemment* et par genre.
- **OK** : joue un jeu téléchargé, sinon propose *Télécharger et jouer* / *Télécharger* ; **appui long sur OK** : fiche du jeu.
- **Fiche du jeu** : Jouer / Télécharger / Annuler, choix de l’émulateur, *Supprimer du téléviseur*.
- Le focus revient sur le dernier élément choisi au retour d’un écran ou d’une partie.

### Premier lancement de l’app

Saisissez l’adresse du serveur (`http://<ip-du-serveur>:8080`) et la clé d’API, **Tester la connexion**, **Enregistrer**, puis autorisez **l’accès à tous les fichiers** (voir [Fonctionnement](#fonctionnement)).

## Développement

Kotlin + Jetpack Compose, minSdk 26 (Android 8), targetSdk 36 (Android 16). Modules Gradle :

| Module | Contenu |
|---|---|
| `core` | Code partagé : API du serveur, cache hors ligne, téléchargements (service de premier plan), lancement des émulateurs (modèles `am start`), réglages, ViewModels, fenêtres communes |
| `app` | Application téléphone / tablette (`com.romcloud.app`) |
| `tv` | Application Android TV (`com.romcloud.app.tv`), interface [tv-material](https://developer.android.com/jetpack/androidx/releases/tv) pour la télécommande |
| `libretrodroid` | Moteur intégré (cœurs libretro) : [LibretroDroid](https://github.com/Swordfish90/LibretroDroid) 0.14.0 repris dans le projet (plus maintenu en amont, GPL 3), avec Oboe et libretro-common aux versions de la 0.14.0 ; ajout du **rendu Vulkan** des cœurs (PPSSPP, Beetle PSX HW… : OpenGL ES par défaut, Vulkan choisi dans les options du cœur ; image Vulkan partagée avec OpenGL ES par deux AHardwareBuffer, sans recopie par le processeur — recopiée si le pilote ne le permet pas —, filtres et diffusion compris) |


## Compiler l'APK

Ouvrir le dépôt dans Android Studio, ou en ligne de commande (JDK 17 à 21) :

```bash
./gradlew :app:assembleRelease   # app/build/outputs/apk/release/app-release.apk (téléphone)
./gradlew :tv:assembleRelease    # tv/build/outputs/apk/release/tv-release.apk (Android TV)
./gradlew :core:testDebugUnitTest
```

L’app cible Android 16 (API 36) et s’installe à partir d’Android 8 (API 26).

**Signature release** : lue depuis les variables d’environnement `ROMCLOUD_KEYSTORE_FILE`, `ROMCLOUD_KEYSTORE_PASSWORD`, `ROMCLOUD_KEY_ALIAS`, `ROMCLOUD_KEY_PASSWORD`, ou depuis `keystore.properties` (non versionné) :

```properties
storeFile=C\:/chemin/vers/romcloud-release.jks
storePassword=...
keyAlias=romcloud
keyPassword=...
```

Sans clé configurée, l’APK release est signé avec la clé de debug.

## Compilation automatique (GitHub Actions)

Le workflow [`.github/workflows/android.yml`](.github/workflows/android.yml) :

- à chaque push sur `main` et pull request : tests unitaires et compilation ; lancement manuel : APK release signés (téléphone et TV), téléchargeables dans l’onglet **Actions** (artifact `RomCloud-0.0.<n°>-<commit>`) ;
- sur un tag `v*` : même chose + **Release GitHub** avec les deux APK (`RomCloud-<version>.apk` et `RomCloud-TV-<version>.apk`) :

  ```bash
  git tag v1.1.0 && git push origin v1.1.0
  ```

Secrets à définir dans *Settings → Secrets and variables → Actions* : `ROMCLOUD_KEYSTORE_BASE64` (keystore encodé en base64), `ROMCLOUD_KEYSTORE_PASSWORD`, `ROMCLOUD_KEY_ALIAS`, `ROMCLOUD_KEY_PASSWORD`. Le `versionName` est celui du tag, le `versionCode` en est tiré (`1.106.0` → `1106000`) : toujours croissant, et au-dessus de celui des versions publiées par l’ancien dépôt romCloud. RomCloud PC est publié par [romcloud-pc](https://github.com/biliboobrian/romcloud-pc), avec la même clé.

Les Releases sont celles qu’Obtainium surveille (voir [Installation](#application-android--obtainium-recommandé)).

## Fonctionnement

- **Premier lancement** : saisir l'adresse du serveur (`http://192.168.x.x:8080`) et la clé d'API, *Tester la connexion*, puis autoriser **l'accès à tous les fichiers** : les ROMs sont écrites dans un dossier partagé (`/storage/emulated/0/RomCloud/<système>/` par défaut) pour que les émulateurs puissent les lire.
- **Systèmes** : cartes avec l’image configurée sur le serveur.
- **Favoris** (Windows, téléphone et TV) : bouton *Ajouter aux favoris* (cœur) dans la fiche d’un jeu, téléchargé ou non. Dès qu’un jeu y est, la rangée **Favoris** apparaît sur l’écran des systèmes, au-dessus de la rangée *Téléchargés* (du plus récemment ajouté au plus ancien ; hors ligne : favoris téléchargés seulement). Avec un profil connecté, les favoris sont ceux du profil, communs à tous ses appareils ; sans profil, ils restent sur l’appareil et sont ajoutés à ceux du profil à la connexion.
- **Recherche globale** (loupe sur l’écran des systèmes, bouton *Rechercher* sur TV) : cherche dans les jeux de **tous les systèmes** ; résultats en cartes avec l’indicateur de téléchargement et le **logo de la console en bas à droite** (image du système, sinon son nom court). Un résultat ouvre la fiche du jeu. Hors ligne, la recherche porte sur les listes déjà ouvertes.
- **Liste des jeux** : deux affichages au choix (bouton en haut à droite, choix mémorisé) :
  - **liste** : jaquette, titre, année, genre, taille ;
  - **carrousel** façon Netflix : bannière du jeu mis en avant (capture en fond, boutons *Jouer* / *Télécharger* et *Infos*), puis rangées défilant horizontalement — *Téléchargés*, *Ajoutés récemment* (au-delà de 20 jeux), puis une rangée par genre. Chaque carte porte le bouton **ⓘ** (fiche du jeu et choix de l’émulateur). Pendant une recherche, les résultats s’affichent en grille.

  Les filtres *Tous / Téléchargés / À télécharger* et la recherche s’appliquent aux deux affichages. Indicateur de chaque jeu :
  - ☁ gris : non téléchargé
  - cercle de progression : téléchargement en cours (pourcentage)
  - ✓ vert : présent sur l'appareil
  - ⚠ rouge : échec du dernier téléchargement
- **Appui sur un jeu** : s'il n'est pas téléchargé, une fenêtre demande de le télécharger d'abord (options « Lancer le jeu une fois téléchargé » et, sous Windows et sur téléphone, « Créer un raccourci sur le bureau » / « sur l’écran d’accueil » : raccourci à la jaquette du jeu qui ouvre RomCloud et lance le jeu, en reprenant la partie s’il y en a une ; aussi créé ou retiré depuis la fiche du jeu, et retiré quand le jeu est supprimé de l’appareil — sur téléphone, Android ne permettant pas à une application de l’enlever de l’écran d’accueil, il y est désactivé) ; s'il l'est, il se lance directement. **Appui long / ⓘ** : fiche détaillée (description, capture, choix de l'émulateur, suppression locale).
- Téléchargements en arrière-plan avec notification, reprise automatique d'un fichier partiel (`.part`). Une fois tout reçu, « Veuillez patienter… » s'affiche jusqu'à ce que le jeu soit jouable. Pendant un téléchargement, la fiche du jeu (et la liste des jeux) affiche la vitesse et, pour un jeu PC ou en plusieurs parties (disques, mises à jour, DLC), le nombre de fichiers restant à recevoir.
- **BIOS** : si le serveur a des BIOS pour le système, la fenêtre de téléchargement propose *Télécharger aussi les BIOS manquants* (cochée par défaut). Pour un jeu déjà présent, RomCloud propose de télécharger les BIOS absents avant de jouer (ou *Jouer quand même*), et la fiche du jeu indique leur état avec un bouton *Télécharger les BIOS*. Ils sont copiés dans le **dossier RomCloud** (*Paramètres*), à leur place selon leur type (voir ci-dessous).
- **Dossier RomCloud** (Windows, téléphone et TV, par défaut `Documents\RomCloud` sous Windows et `RomCloud` du stockage de l’appareil sous Android) : un seul dossier à choisir, avec
  - `roms/<système>/` : les jeux (jeux PC : `roms/windows/<jeu>/`) ;
  - `bios/<système>/` : BIOS et données des cœurs libretro, chemins du serveur gardés (`bios/dc/dc/dc_boot.bin`) : c’est le dossier système donné au cœur ;
  - `system/<console>/<type>/` : clés, firmwares et données des émulateurs sans cœur libretro (`system/switch/keys/prod.keys`, `system/switch/firmware/firmware.zip`, `system/ps3/firmware/PS3UPDAT.PUP`, `system/xbox/data/xbox_hdd.qcow2`) ; sous Windows, RomCloud les installe dans Eden, Ryujinx, RPCS3 et Cemu au lancement ;
  - `drivers/` (Android) : pilotes graphiques (Turnip) des émulateurs externes ;
  - `system/windows/` (Android) : copie de ce que télécharge RomCloud PC — fichiers de Winlator (`winlator/<version>/` : système Linux, Proton…), composants `.wcp` (`components/` : DXVK, VKD3D, Box64, FEXCore) et pilotes Adreno (`drivers/`) ; repris au lieu d’être retéléchargés si RomCloud PC est réinstallé ou ses données effacées ;
  - `saves/<cœur>/` et `states/<cœur>/` : sauvegardes et états du moteur intégré.
  Le serveur indique pour chaque fichier sa zone et son chemin (`area`, `devicePath` de `GET /api/systems/:id/bios`). Les cœurs téléchargés restent dans le dossier de l’application (`%APPDATA%\RomCloud`, stockage privé sous Android : Android n’y charge que du code natif de ce dossier) ; Proton, DXVK et Box64 restent installés dans RomCloud PC (programmes Linux, qui ne s’exécutent pas depuis le stockage partagé) : seule leur copie est dans `system/windows/`. **Passage depuis l’ancienne organisation**, automatique au premier lancement : les dossiers des systèmes de l’ancien dossier des ROMs vont dans `roms/`, les sauvegardes et états du moteur intégré dans `saves/` et `states/`, l’ancien dossier `bios` devient `bios.old` et chacun de ses fichiers est déplacé à sa place dès qu’un système le demande (copié s’il était dans le dossier d’un autre programme) ; les copies de *Téléchargements › RomCloud* (Android) sont supprimées. Choisir ensuite un autre dossier ne déplace pas les fichiers.
- **Émulateurs externes seulement pour les consoles hors moteur intégré** : une console émulée par LibretroDroid (dans sa table des cœurs, avec au moins un cœur téléchargeable : consoles 8 et 16 bits, PlayStation 1 et 2, PSP, N64, DS, 3DS, GameCube, Wii, Dreamcast, Saturn, arcade…) ne propose plus les émulateurs externes de ses modèles Daijishou (DuckStation, melonDS, Azahar, Dolphin, AetherSX2, PPSSPP…) ; ils restent proposés pour les autres (Switch, PS3, PS Vita, Wii U, Naomi, Xbox…). Le serveur les retire aussi des systèmes envoyés aux applications Android et de l’administration (ordre des émulateurs, APK).
- **Réglages des émulateurs externes** (Windows, téléphone et TV) : bouton *Configurer {émulateur}* dans la fiche du jeu (et, sous Windows, *Réglages › Émulateurs*) pour **Eden** (et ses forks Lichii, Orchid, Fritchii sous Windows), **Dolphin** et, sous Windows, **Vita3K** (`config.yml` : rendu, résolution, langue, bouton de validation… ; Vita3K pour Android ne partage pas ses fichiers) : fenêtre de leurs principaux réglages (graphismes, mode TV ou portable, langue, région, performances, son…), écrits clé par clé dans leurs fichiers (Windows : `qt-config.ini`, `Dolphin.ini`, `GFX.ini` ; Android : `config/config.ini` d’Eden, `Config/Dolphin.ini` et `Config/GFX.ini` de Dolphin), l’original gardé une fois en `.romcloud.bak`. Sous Android, ces fichiers sont ouverts par le fournisseur de documents de l’émulateur : RomCloud demande une fois l’accès à son dossier (sélecteur de dossier d’Android, *Utiliser ce dossier*). Liste des réglages commune aux deux : `core/emulator-config/emulator-config.json` (copie de celle de [romcloud-desktop](https://github.com/biliboobrian/romcloud-desktop)). L’émulateur doit être fermé pendant l’enregistrement (il réécrit ses réglages en se fermant). Les autres émulateurs externes de Windows affichent une aide de configuration (firmware, BIOS, manettes).
- **Émulateur non installé** (au lancement, ou bouton *Installer* de la fiche du jeu) : un seul mécanisme, le **miroir des émulateurs** du serveur. Émulateur publié sur GitHub ou Forgejo (liste `GET /api/emulators/android`, gardée sur l’appareil pour le démarrage hors ligne) : RomCloud propose seulement *Télécharger la dernière version*, récupérée par le serveur puis téléchargée depuis lui — Eden (standard, Legacy, Optimized) et Eden Duo pour la Switch, Vita3K, aPS3e et ARMSX3 (PS3, variante adaptée à la version d’Android), Cemu, Flycast (Naomi, Atomiswave), MAME4droid (Game & Watch), X1 Box, hakuX, Xenra et XenDroid (Xbox, Xbox 360), RomCloud PC, Winlator Cmod, Winlator et GameNative (jeux PC). Ces émulateurs passent devant les autres émulateurs externes du système (choix par défaut). Le *Play Store* n’est proposé que pour un émulateur hors du miroir, ou en repli quand le serveur n’a pas pu récupérer la version (application publiée sur le Play Store). Téléchargement avec progression puis installateur d’Android ; l’APK téléchargé doit installer le paquet attendu. La première fois, Android demande d’autoriser RomCloud à installer des applications (*Autoriser cette source*) ; l’installation reprend au retour dans l’application.
- **Caster l’écran sur un Chromecast** (téléphone : icône de diffusion sur les écrans des systèmes et des jeux, entrée *Caster l’écran* du menu de l’émulateur intégré) : Android ne permet pas à une application de recopier elle-même l’écran sur un Chromecast ; RomCloud ouvre l’écran de diffusion d’Android (*Caster* / *Diffuser l’écran*), Smart View sur Samsung ou Google Home (*Caster mon écran*), selon ce que propose l’appareil. Tout l’écran et le son sont recopiés, jeux compris. L’icône indique la diffusion en cours quand Android la signale.
- **Hors ligne** : les listes sont mises en cache ; les jeux déjà téléchargés restent jouables.
- **Shader d’image de l’émulateur intégré** (entrée *Image* de son menu, mémorisé par système) : panneau à droite avec la liste des shaders sous leur nom réel et tous les réglages de celui choisi ; le jeu reste affiché derrière, figé et redessiné avec le shader pour voir l’effet aussitôt. Shaders de LibretroDroid : Nearest, Sharp Bilinear (par défaut : pixels nets sans largeurs inégales ni flou ; largeur de la transition), CUT, CUT2 et CUT3 (tous leurs paramètres : mélange dynamique, contraste et netteté du mélange, netteté fixe, luminance rapide, seuil et contraste des contours, renfort des contours doux, tolérance et distance de recherche des contours nets), CRT (lignes de balayage, éclat), LCD (grille, éclat) ; luminosité, contraste, saturation et gamma pour tous. Anciens profils regroupés : « lissage des contours » et « contours nets » deviennent CUT3, « contours doux » CUT2, « lissage doux » CUT (réglages du profil repris).
- **Run-ahead de l’émulateur intégré** (Windows, téléphone et TV : entrée *Run-ahead* de son menu, mémorisée par système, désactivée par défaut) : 1 à 3 images du jeu calculées en avance à chaque image avec les touches du moment, la dernière affichée puis l’état réel rétabli ; supprime le retard propre au jeu entre la touche et l’image (1 image pour la plupart des jeux, 2 pour certains jeux de combat). Demande plus de puissance (le jeu est calculé 2 à 4 fois par image) et un cœur qui sait enregistrer ses états (sinon coupé avec un message) ; coupé en partie à plusieurs, en liaison entre consoles et pendant la diffusion sur une TV.
- **Langue** : *Paramètres → Langue* : Système, Français ou English (téléphone et TV), appliquée immédiatement. Les messages du serveur suivent la langue choisie.
- **Jeux non identifiés** : *Paramètres → Affichage → Masquer les jeux non identifiés* (téléphone et TV) retire des listes de jeux et de la recherche ceux que le serveur n’a pas pu identifier (introuvables dans les bases de jeux, ou pas encore scrapés).
- **Place occupée** (Windows, téléphone et TV) : sur l’écran des systèmes, chaque carte indique la place prise par les jeux du système sur l’appareil (*Sur ce PC* / *Sur l’appareil*), et une barre à côté du nom de l’application montre l’espace occupé et l’espace libre de la partition du dossier des ROMs (en rouge sous 10 % d’espace libre).
- **Classer les jeux** : *Paramètres → Affichage → Classer les jeux* (Windows, téléphone et TV) : **par nom** ou **par note** (les mieux notés d’abord, les jeux sans note à la fin), dans les listes, la recherche et chaque rangée du carrousel (*Ajoutés récemment* reste par date).
- **Jeux en plusieurs disques, mises à jour et DLC** (Windows, téléphone et TV) : regroupés par le serveur, ils sont téléchargés ensemble dans le dossier du système (taille totale affichée, jeu présent quand tous ses fichiers le sont). Pour un jeu en plusieurs disques, l’application écrit une **liste des disques** (`.m3u`, au nom du premier disque : sauvegardes et états gardent leur nom) et l’ouvre avec le moteur intégré, DuckStation, Mednafen et les émulateurs Android dont le modèle accepte les `.m3u` : **changement de disque en jeu** (menu du moteur intégré). Disques compressés (`.zip`, `.7z`) : premier disque seulement. PlayStation 3 (RPCS3 géré par RomCloud) : mises à jour et DLC en `.pkg` installés automatiquement avant le premier lancement ; ailleurs, ils sont à côté du jeu, à installer depuis l’émulateur.
- **Jouer à plusieurs en réseau local** (Windows, téléphone et TV, entre eux : un PC joue avec un téléphone ou une TV) : quand un autre appareil RomCloud est ouvert sur le même réseau (quel que soit le profil), une icône apparaît sur l’écran principal, avec les parties proposées, et les jeux jouables à plusieurs portent une icône dans les listes (consoles de salon et arcade, jeu à au moins deux joueurs, moteur intégré). *Jouer à plusieurs en réseau* (fiche du jeu) lance le jeu et le propose ; l’autre appareil le rejoint (jeu téléchargé d’abord s’il manque) et l’hôte accepte ou refuse. Netplay synchronisé : chaque appareil émule le jeu, seules les touches circulent, image par image, avec 3 images de délai (environ 50 ms) ; l’état de l’hôte est copié chez l’invité au départ (l’hôte en repart aussi), et une empreinte de la mémoire du jeu (à défaut, de l’état complet) est comparée toutes les 2 secondes (écart : état de l’hôte rechargé chez l’invité). Le cœur sait qu’il joue en réseau : FBNeo y coupe les meilleurs scores enregistrés et fixe l’horloge de la Neo Geo. Cœurs vérifiés entre deux moteurs, sans écart : Mesen, Nestopia, FCEUmm, QuickNES, snes9x, bsnes, Mesen-S, Genesis Plus GX, PicoDrive, Gearsystem, FBNeo, Mupen64Plus-Next, Mednafen PSX (et HW), SwanStation, Mednafen Saturn ; parallel_n64, Yabause, YabaSanshiro, Kronos, Ymir et SMS Plus ne tiennent pas une partie synchronisée et ne la proposent pas. Pendant la partie, charger un état, redémarrer et changer les options du cœur sont désactivés. Découverte par annonces UDP sur le port 47321, connexion TCP directe entre les appareils. Windows : la partie se joue dans le moteur intégré (demande d’un invité acceptée avec A / Entrée, refusée avec B / Échap) ; au premier usage, le pare-feu de Windows demande d’autoriser RomCloud et romcloud-player sur le réseau privé.
- **Relier les consoles portables en réseau local** (Windows, téléphone et TV, entre eux), à part du jeu synchronisé : chaque appareil émule sa console avec **son propre jeu** (Pokémon Rouge relié à Pokémon Bleu), comme avec un câble. Une autre icône (maillon) marque ces jeux dans les listes ; *Relier les consoles en réseau* (fiche du jeu) lance le jeu et propose la liaison, et sur l’autre appareil, la fiche d’un jeu de la même console propose *Se relier à …* (ou *Se relier* depuis l’icône de l’écran principal, avec le même jeu). La liaison passe par le cœur prévu pour elle, quel que soit l’émulateur choisi (sa sauvegarde est reprise si elle est plus récente, et recopiée pour lui en quittant) :
  - **Game Boy et Game Boy Color** : câble Game Link de **Gambatte** (connexion directe entre les deux émulateurs, serveur chez l’hôte, port 56400).
  - **Game Boy Advance** : câble (Pokémon Rubis / Saphir, Advance Wars 1 et 2) et adaptateur sans fil (Pokémon Émeraude, Rouge Feu, Vert Feuille, Mario Golf, Mario Tennis, Mega Man Battle Network 5 et 6, Digimon Racing, DBZ Buu’s Fury, Narnia, Shrek Super Slam, Le Seigneur des Anneaux : Le Tiers Âge, Hamtaro) de **gpSP** ; ses paquets passent par la connexion RomCloud (interface *netpacket* de libretro). Seuls ces jeux sont proposés : gpSP n’émule pas le câble des autres (leur mode à plusieurs attendrait l’autre console devant un écran noir).
  - **PSP** : réseau **ad hoc** de **PPSSPP**, serveur ad hoc intégré chez l’hôte (port 27312), adresse MAC propre à chaque appareil, ports des jeux décalés de 10000 (Android interdit les ports sous 1024) ; d’autres consoles peuvent rejoindre la même partie.
  Les DS (melonDS DS ne relie pas deux consoles) et les autres consoles n’ont pas de liaison. Pendant la liaison, charger un état, redémarrer et changer les options du cœur sont désactivés.
- **Jouer à plusieurs par Internet** (Windows, téléphone et TV, entre eux) : avec un profil connecté, les appareils des autres joueurs connectés au même serveur (tous profils confondus) apparaissent comme ceux du réseau local, « par Internet », avec le nom du profil. Une partie proposée par un joueur connecté l’est aussi par Internet ; elle se rejoint de la même façon (icône de l’écran principal, ou fiche du jeu proposé). Tout passe par le **relais du serveur RomCloud** : les deux appareils ouvrent une connexion vers le serveur, qui fait passer les données de l’un à l’autre, sans rien ouvrir sur les box. Le délai des touches est adapté aux allers-retours mesurés jusqu’au serveur (3 à 15 images). Jeu synchronisé, câble GBA (gpSP) et câble Game Boy (Gambatte, sa connexion passe elle aussi par le relais) ; l’ad hoc de la PSP reste au réseau local (nombreux ports). Le serveur doit être joignable par les deux appareils ; derrière un proxy inverse, les requêtes `Upgrade` du chemin `/api/play/relay` doivent être transmises (comme des WebSockets : avec nginx, `proxy_http_version 1.1`, `proxy_set_header Upgrade $http_upgrade`, `proxy_set_header Connection "upgrade"` et un `proxy_read_timeout` long).
- **Lancement** : les modèles Daijishou sont interprétés (`-n`, `-a`, `-d`, `-t`, `-c`, `-e/--es`, `--ez`, `--ei`, `--esa`, `-f`, `--activity-*`…) avec les placeholders `{file.path}`, `{file.uri}` (via FileProvider), `{file.mime}`. Pour un système sans modèle, le sélecteur « Ouvrir avec » d'Android est proposé.

## Limites connues

- Un jeu = un fichier. Pour les jeux multi-fichiers (`.cue` + `.bin`), privilégiez `.chd` ou `.m3u` + fichiers, ou une archive `.zip` si le cœur la supporte.
- Les placeholders `{tags.*}` de Daijishou (Steam, Vita, PS3…) ne sont pas gérés : ces plateformes ne sont pas pertinentes pour des ROMs téléchargées.
- Les modèles utilisant `{file.uri}` reçoivent une URI `content://` de RomCloud au lieu d'une URI SAF comme dans Daijishou ; la plupart des émulateurs l'acceptent, certains peuvent exiger `{file.path}`.
- La base Libretro ne contient presque aucune image pour l’Amstrad GX4000 (pas de jaquettes) : configurez ScreenScraper pour ce système. À défaut, l’écran-titre ou la capture Libretro sert de couverture.
- Les identifiants ScreenScraper de quelques systèmes rares ne sont pas pré-remplis : on peut les saisir dans *Réglages* du système.
