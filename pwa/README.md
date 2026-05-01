# Archive WhatsApp — Guide de déploiement

PWA (Progressive Web App) qui permet de partager des archives WhatsApp avec
quelques personnes proches via une simple URL. Une fois "installée" sur leur
téléphone, elle s'affiche comme une vraie app native.

## Architecture

- **Toi (Jaffar)** : tu maintiens un dépôt GitHub avec les fichiers PWA. Quand
  tu veux ajouter ou modifier des conversations, tu remplaces le `index.html`
  dans le dépôt et le push.
- **Ton père (et autres proches)** : il visite l'URL **une seule fois**, ajoute
  l'app à son écran d'accueil, et n'a plus jamais à manipuler quoi que ce soit.
  Il tape l'icône, voit les conversations.
- Quand tu pousses une mise à jour, ton père la voit automatiquement la
  prochaine fois qu'il ouvre l'app (le service worker détecte la nouvelle
  version et propose un bouton "Recharger").

## Contenu du dossier

| Fichier | Rôle |
|---|---|
| `index.html` | L'app complète (HTML + CSS + JS + JSZip inline) |
| `manifest.json` | Métadonnées PWA (nom, icônes, couleurs) |
| `sw.js` | Service worker pour fonctionnement offline + détection MAJ |
| `icon-192.png`, `icon-512.png` | Icônes standards |
| `icon-maskable-192.png`, `icon-maskable-512.png` | Icônes adaptatives Android |
| `favicon.png`, `favicon.ico` | Icône onglet navigateur |
| `README.md` | Ce fichier |

## Étape 1 — Créer un compte GitHub (si tu n'en as pas)

1. Va sur [https://github.com/signup](https://github.com/signup)
2. Crée un compte avec ton email — c'est gratuit, 2 minutes
3. Vérifie ton email

## Étape 2 — Créer le dépôt et activer GitHub Pages

1. Une fois connecté, clique sur le bouton vert **"New"** (ou
   [github.com/new](https://github.com/new))
2. **Repository name** : `whatsapp-archive` (ou ce que tu veux, sans espaces)
3. Coche **Public** (obligatoire pour la version gratuite de GitHub Pages)
4. Coche **"Add a README file"**
5. Clique **"Create repository"**

6. Une fois le dépôt créé, clique sur **"Add file" → "Upload files"**
7. **Glisse-dépose tous les fichiers de ce dossier `pwa/`** dans la zone
   d'upload (sauf le README.md du dépôt qui existe déjà — ou écrase-le avec
   le nôtre, comme tu veux)
8. En bas, écris un commit message comme "Initial PWA upload" puis clique
   **"Commit changes"**

9. Va dans **Settings** (onglet en haut) → **Pages** (menu de gauche)
10. Sous "Build and deployment" → **Source** : sélectionne **"Deploy from a branch"**
11. **Branch** : `main`, dossier `/ (root)`, clique **Save**
12. Attends 1-2 minutes. La page Pages t'affichera l'URL :
    `https://TONPSEUDO.github.io/whatsapp-archive/`

C'est cette URL que tu enverras à ton père.

## Étape 3 — Installer l'app sur le téléphone de ton père

Envoie-lui l'URL par SMS ou WhatsApp et explique-lui de :

**Sur Android (Chrome) :**
1. Ouvrir le lien
2. Un bandeau vert "Installer l'app" apparaît en bas → toucher **"Installer"**
3. Confirmer l'installation
4. L'icône Archive WhatsApp apparaît sur l'écran d'accueil
5. Désormais : taper sur l'icône pour ouvrir, comme une vraie app

**Sur iPhone (Safari) :**
1. Ouvrir le lien dans Safari (pas Chrome iOS, ça doit être Safari)
2. Toucher l'icône **Partager** ⬆️ en bas de l'écran
3. Faire défiler et toucher **"Ajouter à l'écran d'accueil"**
4. Toucher **"Ajouter"** en haut à droite

Si le bandeau "Installer" n'apparaît pas sur Android, l'app marche quand même
en mode site web. Ton père peut aussi installer manuellement via le menu Chrome
(⋮) → "Installer l'application" ou "Ajouter à l'écran d'accueil".

## Étape 4 — Workflow pour ajouter des conversations (avec chiffrement)

À chaque fois que tu veux ajouter ou mettre à jour des conversations :

1. **Ouvre ton viewer** (la PWA en ligne ou en local)
2. **Importe les nouveaux .zip** (bouton 📂 Importer)
3. Une fois tout chargé, va dans le menu (⋮) → **"📤 Exporter en fichier
   partageable"**
4. Une boîte de dialogue te propose un **mot de passe fort généré
   automatiquement** (16 caractères aléatoires sécurisés). Tu peux le
   regénérer ou taper le tien.
5. Clique **"Chiffrer et exporter"** — le viewer chiffre tout en AES-256
   et télécharge un fichier `index-XXX-DATE.html`
6. Une fenêtre s'affiche avec :
   - Le nom du fichier téléchargé
   - **L'URL complète à partager** (avec le mot de passe en fragment :
     `https://tonpseudo.github.io/whatsapp-archive/#k=AbC123...`)
   - Un bouton "📋 Copier le lien"
7. **Renomme le fichier téléchargé en `index.html`**
8. Va sur ton dépôt GitHub, clique sur l'`index.html` existant → ✏️ Edit ou
   "Add file → Upload files" pour glisser le nouveau (il écrase l'ancien)
9. Commit en bas, message "Update conversations DD/MM/YYYY"
10. **Attends 1-2 minutes** pour le déploiement GitHub Pages

11. **Envoie l'URL avec mot de passe** à ton père (la première fois). Il la
    met en favori. Désormais il tape l'icône → l'archive se déchiffre
    automatiquement, sans saisir le mot de passe.

### À propos du chiffrement

- **Algorithme** : AES-256-GCM (chiffrement authentifié)
- **Dérivation de clé** : PBKDF2-SHA256, 250 000 itérations
- **Mot de passe par défaut** : 16 caractères aléatoires cryptographiquement
  sécurisés (entropie > 90 bits — incassable en pratique)
- **Comment ça marche** : le mot de passe ne quitte jamais le navigateur. Il
  est dans le fragment d'URL (après le `#`), qui n'est jamais envoyé au
  serveur GitHub. Quand le destinataire ouvre l'URL, le JavaScript local
  extrait le mot de passe du fragment et déchiffre les données.
- **Si quelqu'un trouve ton dépôt GitHub sans avoir le mot de passe** : il
  voit le code, il voit qu'il y a des données chiffrées, mais il ne peut
  rien lire. Tenter de casser le mot de passe par force brute prendrait
  des milliards d'années avec tous les supercalculateurs du monde.
- **Pour révoquer l'accès** : ré-exporte avec un nouveau mot de passe et
  pousse la nouvelle version. L'ancienne URL ne marche plus, même pour ton
  destinataire — il faut lui envoyer la nouvelle.

## Astuces et limitations

### Taille
Chaque conversation avec médias pèse beaucoup à cause de l'encodage base64
(+33% par rapport au .zip d'origine). GitHub Pages a une limite de :
- **100 MB par fichier** (limite stricte)
- **1 GB par dépôt** (recommandé)
- **100 GB de bande passante par mois** (largement suffisant pour 5 personnes)

Si ton archive dépasse 100 MB, il faudra :
- Soit splitter en plusieurs PWA (une par conversation)
- Soit héberger sur Netlify (limite 100 MB/site mais plus flexible)
- Soit passer en architecture Niveau 2 (fichier de données séparé)

### Cache
Le service worker met en cache pour fonctionnement offline. Quand tu pushes
une mise à jour, il faut :
- Que ton père soit en ligne pour récupérer la nouvelle version
- Qu'il rafraîchisse une fois (le bandeau "Recharger" le fait pour lui)
- Sinon il continue de voir l'ancienne version (cache)

### Privé ?
Le dépôt GitHub est **public**, mais l'URL est obscure. Pour rendre l'accès
plus discret :
- Ne partage l'URL qu'avec les bonnes personnes
- N'indexe pas la page (ajouter `<meta name="robots" content="noindex">`
  dans le `<head>` — déjà non-indexable car peu de signaux pour Google)
- Si tu veux vraiment du privé, GitHub propose des dépôts privés avec
  Pages payant ($4/mois) ou tu peux utiliser Netlify avec mot de passe ($19/mois)

### Mise à jour silencieuse
Le service worker vérifie les mises à jour à chaque ouverture. Pour forcer
la propagation :
- Bump le numéro `CACHE_VERSION` dans `sw.js` (passe `v3` à `v4`, etc.)
- Push aussi cette modification de `sw.js` en plus de `index.html`

---

Bonne archivage 🎙️📂
