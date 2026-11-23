# Prompteur & Écran Casté Synchronisés

Système de présentation web léger (HTML / CSS / JavaScript natif) composé de deux interfaces synchronisées en temps réel :
- **`prompteur.html`** : console de pilotage destinée à l'orateur (défilement karaoké caractère par caractère, triple chronométrage, gestion des pauses et import/export de script).
- **`ecran_slides.html`** *(ou `écran_slides.html`)* : fenêtre d'affichage plein écran destinée au vidéoprojecteur, au jury ou au public (diapositives, sous-titres synchronisés et mini-compteur discret).

---

## Arborescence recommandée du projet

```text
projet-presentation/
├── prompteur.html              # Interface orateur (écran principal)
├── ecran_slides.html           # Interface public / écran casté (doit être nommé sans accent pour l'ouverture auto)
├── script.txt                  # (Optionnel) Script texte chargé automatiquement si un serveur local est actif
└── slides/                     # Dossier contenant les diapositives exportées en images
    ├── Slides_01.png
    ├── Slides_02.png
    └── ...
```

> **Note sur le nommage :** Dans `prompteur.html`, le bouton **Écran Casté** appelle `window.open('ecran_slides.html', ...)`. Veillez à nommer le fichier sans accent (`ecran_slides.html`) pour que l'ouverture automatique fonctionne sans erreur.

---

## 1. Fonctionnalités d'usage

### A. `prompteur.html` (Console Orateur)

1. **Triple suivi temporel en temps réel (barre supérieure verrouillée)**
   - **Chronomètre Global** : affiche le temps total écoulé par rapport à la durée totale calculée de la présentation (`MM:SS / MM:SS`), accompagné d'une barre de progression et d'un pourcentage. Passe en rouge en cas de dépassement.
   - **Chronomètre Slide** : mesure le temps passé sur la diapositive en cours (repart à `00:00` à chaque changement de slide). Passe en ambre avec pulsation visuelle sur le bouton *Suivante* lorsque le temps alloué à la slide est dépassé.
   - **Indicateur de Rythme / Timing (Delta)** : compare en continu l'avancement théorique (`début théorique de la slide + temps écoulé sur la slide`) au chronomètre global réel :
     - **Vert (`+MM:SS`)** : orateur en avance.
     - **Bleu (`±00:01`)** : dans le rythme.
     - **Ambre / Rouge (`-MM:SS`)** : en retard (alerte rouge critique à partir de `-15 s`).

2. **Lecture karaoké caractère par caractère & défilement intelligent**
   - **Coloration fluide à 60 FPS** : les mots non lus sont affichés en gris ardoise (`#64748B`), le caractère en cours de lecture brille en jaune or (`#FACC15`) suivi d'une traînée cyan (`#38BDF8`) sur les deux caractères précédents, puis passe en blanc pur (`#F8FAFC`).
   - **Protection anti-coupure** : chaque mot et sa ponctuation attachée sont regroupés (`.word-punct-group`) pour éviter tout retour à la ligne orphelin.
   - **Centrage automatique** : la page défile automatiquement pour maintenir le paragraphe actif au centre de la zone visible sous la barre d'outils.
   - **Navigation au clic** : cliquer sur n'importe quel paragraphe ou carte de pause cale immédiatement la lecture sur cet instant et met à jour l'écran casté.

3. **Gestion automatique des pauses et de la ponctuation**
   - Le moteur analyse chaque phrase et insère des badges visuels de respiration avec jauge de progression à **durée fixe** (non accélérée lorsque la vitesse de lecture augmente) :
     - `--` (pause courte explicite) : `0,50 s`
     - `,` (virgule) : `0,50 s`
     - `;` (point-virgule) et `:` (deux-points) : `0,75 s`
     - `.`, `?`, `!` : `1,00 s`
     - `...` ou `…` : `1,50 s`
     - Fin de paragraphe (`¶`) : `2,00 s`
   - **Blocs de pause dédiés** : la balise `[PAUSE Ns]` génère un bandeau dédié (*Respiration & Regard Audience*) avec compte à rebours.

4. **Import / Export de scripts `.txt` et calcul automatique des durées**
   - Chargement via le bouton **Charger .txt**, par **glisser-déposer (Drag & Drop)** directement sur la fenêtre, ou automatiquement depuis `script.txt` au démarrage.
   - **Syntaxe du fichier `.txt`** :
```text
# Commentaire ignoré
=== SLIDE 1 | PARTIE 1 : INTRODUCTION | TITRE DE LA SLIDE ===
Première phrase de votre discours, avec une pause naturelle.
[PAUSE 3s]
Deuxième paragraphe -- avec une pause courte explicite.

=== SLIDE 2 | 01:30 | PARTIE 2 : DÉVELOPPEMENT | DURÉE FIXE DE 1 MIN 30 ===
Texte dont la durée totale sur la slide sera calée sur 01:30.

=== SLIDE 3 | 02:35 - 04:00 | PARTIE 3 : CONCLUSION | PLAGE HORAIRE EXACTE ===
Texte calé entre 02:35 et 04:00.
```
   - Si aucun horodatage n'est indiqué dans l'en-tête `=== SLIDE N ===`, le script estime automatiquement la durée de chaque slide sur la base de `14,5 caractères/seconde` additionnée aux durées fixes de ponctuation.
   - **Exporter .txt** : télécharge un fichier `script_horodate.txt` contenant tous les en-têtes `=== SLIDE N | MM:SS - MM:SS | PARTIE | TITRE ===` calculés.

5. **Réglages et raccourcis clavier (`prompteur.html`)**
   - **Taille de police** : ajustable de `22 px` à `56 px` (`A-` / `A+` ou touches `-` / `+`).
   - **Vitesse de lecture** : ajustable de `0.80x` à `1.70x` (par pas de `0.05x`). Recalcule uniquement la durée d'élocution des mots tout en conservant intacte la durée des pauses et la position exacte dans la phrase en cours.
   - **Raccourcis clavier** :
     - `Espace`, `P` ou `Entrée` : Démarrer / Mettre en pause.
     - `→` (Flèche droite) : Passer à la slide suivante (démarre automatiquement le chronomètre s'il était arrêté).
     - `←` (Flèche gauche) : Revenir à la slide précédente.
     - `R` : Réinitialiser le prompteur et l'écran casté à la Slide 1 (`00:00`).

---

### B. `ecran_slides.html` (Écran Casté / Public)

1. **Affichage plein écran des diapositives**
   - Recherche automatiquement les images correspondantes dans `slides/` en testant plusieurs conventions courantes (`Slides_01.png`, `Slides_1.png`, `Page_01.png`, `slide-01.png`, `Diapositive1.PNG`, `1.png`, etc. aux formats `.png`, `.jpg`, `.jpeg`, `.webp`).
   - **Préchargement intelligent** : met en cache l'image active et précharge la slide `N + 1` pour des transitions instantanées.
   - **Écran de secours** : si aucune image n'est détectée, affiche proprement le numéro de slide, la partie, le titre et un bouton permettant de sélectionner le dossier local.

2. **Sous-titres synchronisés caractère par caractère**
   - Affiche le discours en direct au rythme exact de l'orateur avec la même animation caractère par caractère (curseur doré, traînée cyan, texte lu).
   - **Deux modes de positionnement** :
     - *Overlay transparent* : superposé à la diapositive, avec réglage de la hauteur (`2%` à `85%`) et de l'opacité du fond flouté (`0%` à `90%`).
     - *Sous la slide* : bandeau dédié sous la diapositive (redimensionne automatiquement l'image pour éviter tout masquage).
   - **Décalage temporel configurable (`0,0 s` à `2,0 s`, défaut `+0,5 s`)** : affiche `…` au tout début de chaque slide pendant le délai choisi afin d'absorber la latence naturelle de prise de parole.
   - **Personnalisation** : couleur (palettes rapides ou sélecteur libre), taille (`18 px` à `56 px`) et aperçu animé en direct lorsque le panneau est ouvert à l'arrêt.

3. **Mini-compteur discret & flash visuel toutes les 10 secondes**
   - Encart discret positionnable aux 4 coins de l'écran affichant un **voyant LED vert/rouge** (état de connexion avec `prompteur.html`) et l'horloge globale (`MM:SS / MM:SS`).
   - **Visibilité adaptative** : quasi transparent par défaut (`opacité 0.24`) pour ne pas distraire le public, il s'illumine brièvement (`opacité 0.96` pendant `2,2 s`) toutes les **10 secondes** pour offrir un repère visuel rapide à l'orateur.
   - **Mode d'apparition** : permanent depuis le début, ou uniquement dans les *X dernières minutes* avant la fin de la présentation.

4. **Barre HUD auto-masquable & raccourcis clavier (`ecran_slides.html`)**
   - La barre de contrôle en haut de l'écran apparaît au mouvement de la souris et se masque automatiquement après `2,5 secondes` d'inactivité.
   - **Raccourcis clavier** :
     - `F` ou **double-clic** : Basculer en mode plein écran.
     - `S` : Activer / Masquer instantanément les sous-titres (synchronisé avec le prompteur).
     - `C` : Ouvrir / Fermer la configuration du mini-compteur.
     - `Espace` ou `→` : Slide suivante.
     - `←` : Slide précédente.
     - `R` : Réinitialiser la présentation.
     - `Échap` : Fermer les panneaux de configuration.

---

## 2. Fonctionnement technique et synchronisation entre les deux écrans

Le système fonctionne sans serveur backend ni base de données grâce à une **architecture hybride 100 % navigateur** combinant communication inter-onglets, persistance locale et moteurs d'animation autonomes.

### A. Double canal de communication (`BroadcastChannel` + `localStorage`)

1. **Canal principal haute vitesse : `BroadcastChannel('prompter_sync_v1')`**
   - Permet aux deux fenêtres (`prompteur.html` et `ecran_slides.html`) d'échanger des objets JSON en mémoire vive avec une latence quasi nulle dès qu'elles partagent la même origine.
2. **Canal secondaire et reprise d'état : `localStorage` (`prompter_sync_state_v1`)**
   - À chaque changement d'état, `prompteur.html` sérialise également l'état complet dans `localStorage`.
   - `ecran_slides.html` écoute l'événement natif `window.addEventListener('storage', ...)` et lit le dernier état sauvegardé au chargement (`window.onload`). Cela garantit qu'une fenêtre castée ouverte tardivement ou actualisée en cours de présentation récupère immédiatement la slide active, les chronomètres et la timeline des sous-titres.

### B. Protocole de messages bidirectionnel

| Direction | Type de message | Rôle technique |
| :--- | :--- | :--- |
| `Prompteur → Écran` | `SLIDE_CHANGE` / `SYNC_STATE` | Transmet l'index de slide, les métadonnées, `isRunning`, `globalElapsed`, `slideElapsed`, `playbackSpeed`, l'état des sous-titres et la timeline complète `subtitleCues`. |
| `Prompteur → Écran` | `TIMER_TICK` | Battement de cœur (*heartbeat*) envoyé toutes les `1000 ms` pour resynchroniser les horloges et maintenir le voyant LED au vert (passage au rouge après `4500 ms` sans signal). |
| `Prompteur → Écran` | `SUBTITLE_UPDATE` | Notifie immédiatement un changement de phrase active. |
| `Prompteur → Écran` | `SUBTITLE_VISIBILITY` | Synchronise l'état `ON/OFF` des sous-titres. |
| `Prompteur → Écran` | `PROMPTER_CLOSED` | Déclenché via `beforeunload` : met en pause l'écran casté et passe le voyant LED au rouge. |
| `Écran → Prompteur` | `JURY_SCREEN_READY` / `REQUEST_SYNC` | Poignée de main (*handshake*) à l'ouverture de l'écran casté ou si la timeline `subtitleCues` est manquante. |
| `Écran → Prompteur` | `COMMAND_NEXT_SLIDE` / `PREV` / `RESET` | Permet de piloter le diaporama depuis l'écran casté tout en conservant `prompteur.html` comme source de vérité unique. |
| `Écran → Prompteur` | `SUBTITLES_STATE_CHANGED` | Met à jour le badge `ON/OFF` du prompteur si l'utilisateur appuie sur `S` dans `ecran_slides.html`. |

### C. Synchronisation des sous-titres et compensation de latence à 60 FPS

- **Pré-découpage temporel (`buildSlideSubtitleCues`)** : Au changement de slide, `prompteur.html` découpe le texte en segments (`subtitleCues`) en calculant les horodatages exacts (`start` et `end` en secondes locales) de chaque caractère et de chaque pause. Ce tableau est envoyé **une seule fois** à `ecran_slides.html` (plutôt que d'inonder le canal à chaque lettre).
- **Compensation du temps de transit** : Chaque message contient `timestamp: Date.now()`. À la réception, si la lecture est en cours (`isRunning === true`), `ecran_slides.html` calcule le délai de transmission :
  $$\text{latencySec} = \max\left(0, \frac{\text{Date.now()} - \text{msg.timestamp}}{1000}\right)$$
  et l'ajoute directement à `globalElapsed` et `slideElapsed`.
- **Autonomie d'affichage à 60 FPS** : Entre deux messages de synchronisation, chaque fenêtre exécute sa propre boucle `requestAnimationFrame` basée sur `performance.now()`, assurant un défilement parfaitement fluide même si l'un des onglets est momentanément ralenti par le système d'exploitation.

---

## 3. Que faire si l'on n'a pas de serveur local ? (Mode `file://`)

Les deux pages sont autonomes et peuvent être ouvertes directement par **double-clic** dans votre explorateur de fichiers (`file:///...`), mais les navigateurs modernes appliquent des règles de sécurité strictes (*CORS* et isolement d'origine sur le protocole `file://`). Voici comment utiliser chaque fonctionnalité sans serveur :

1. **Charger les images des diapositives (`slides/`)**
   - Si votre navigateur bloque la détection automatique des images locales en mode `file://`, cliquez sur le bouton **Dossier slides/** dans la barre supérieure de `ecran_slides.html` (ou sur la carte d'accueil).
   - Ce sélecteur (`<input type="file" webkitdirectory>`) permet de désigner votre dossier `slides/` en une fois : les images sont triées par ordre numérique naturel et chargées en mémoire via `URL.createObjectURL()`, sans aucune restriction de sécurité.

2. **Charger votre script (`script.txt`)**
   - Sur `file://`, le navigateur interdit la requête automatique `fetch('script.txt')`.
   - **Solution** : Cliquez sur **Charger .txt** dans `prompteur.html` ou **glissez-déposez** votre fichier `.txt` directement n'importe où sur la fenêtre du prompteur. Le script est immédiatement interprété et mémorisé dans le `localStorage` du navigateur : il sera ainsi restauré automatiquement lors de vos prochaines ouvertures, même sans serveur.

3. **Assurer la synchronisation entre les deux fenêtres**
   - Ouvrez `prompteur.html` en premier, puis cliquez sur le bouton **Écran Casté** dans sa barre d'outils pour ouvrir `ecran_slides.html` dans une fenêtre liée (`window.open`).
   - **Choix du navigateur** : Privilégiez **Google Chrome**, **Microsoft Edge** ou **Brave**, qui autorisent le partage de `BroadcastChannel` et de `localStorage` entre fenêtres locales issues d'un même dossier. Sur Firefox ou Safari en mode `file://`, chaque fichier local est isolé dans sa propre origine (ce qui bloque le `BroadcastChannel` et `localStorage` entre deux fichiers distincts) : un serveur local est alors fortement recommandé.

---

## 4. Lancer un serveur local avec `miniserve` (`miniserv`)

L'utilisation de **`miniserve`** (serveur HTTP statique ultrarapide en un seul exécutable) permet de servir le dossier via `http://localhost:8080`, levant toutes les restrictions `file://` (chargement automatique de `script.txt`, détection automatique des images dans `slides/` et synchronisation `BroadcastChannel` garantie sur tous les navigateurs).

### Sur Linux

1. **Installation** :
   - **Arch Linux / Manjaro** :
```bash
sudo pacman -S miniserve
```
   - **Via Cargo (Rust)** :
```bash
cargo install miniserve
```
   - **Binaire autonome (Ubuntu, Debian, Fedora, etc.)** :
     Téléchargez `miniserve-*-x86_64-unknown-linux-gnu` depuis les [releases GitHub de miniserve](https://github.com/svenstaro/miniserve/releases), puis rendez-le exécutable :
```bash
chmod +x miniserve-*
sudo mv miniserve-* /usr/local/bin/miniserve
```

2. **Lancement** :
   Ouvrez un terminal dans le dossier contenant `prompteur.html` et `ecran_slides.html`, puis exécutez :
   ```bash
   miniserve -p 8080 .
   ```

---

### Sur macOS

1. **Installation** :
   - **Via Homebrew (recommandé)** :
```bash
brew install miniserve
```
   - **Ou via Cargo (Rust)** :
```bash
cargo install miniserve
```

2. **Lancement** :
   Ouvrez le **Terminal** dans le répertoire du projet et lancez :
   ```bash
   miniserve -p 8080 .
   ```

---

### Sur Windows

1. **Installation** :
   - **Via WinGet (intégré à Windows 10 et Windows 11)** :
```powershell
winget install -e --id svenstaro.miniserve
```
   - **Ou via Scoop** :
```powershell
scoop install miniserve
```
   - **Ou sans installation** : téléchargez `miniserve-*-x86_64-pc-windows-msvc.exe` depuis les [releases GitHub de miniserve](https://github.com/svenstaro/miniserve/releases) et placez-le directement dans le dossier de votre présentation.

2. **Lancement** :
   Ouvrez **PowerShell** ou l'**Invite de commandes (`cmd`)** dans le dossier du projet et exécutez :
   ```powershell
   miniserve -p 8080 .
   ```
   *(Ou `.\miniserve.exe -p 8080 .` si vous avez téléchargé l'exécutable directement dans le dossier.)*

---

### Accès aux interfaces une fois le serveur démarré

1. Ouvrez votre navigateur à l'adresse : **`http://localhost:8080/prompteur.html`**
2. Cliquez sur **Écran Casté** dans la barre supérieure (ou ouvrez **`http://localhost:8080/ecran_slides.html`** dans un second onglet/fenêtre).
3. Déplacez la fenêtre `ecran_slides.html` sur le second moniteur ou le vidéoprojecteur, puis appuyez sur **`F`** pour passer en plein écran.

> **Alternative sans installation (si Python est déjà présent sur la machine) :**
> Vous pouvez obtenir exactement le même résultat avec le module HTTP natif de Python :
> - **Linux / macOS** : `python3 -m http.server 8080`
> - **Windows** : `py -m http.server 8080`
