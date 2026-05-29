# HERMES de 'Nous Reseach'

C'est un projet open-source sérieux, bien maintenu, pas un truc obscur.

## Ce qu'est Hermes Agent

Ce n'est pas un modèle, c'est un agent autonome auto-améliorant. Concrètement, c'est un assistant IA local qui :

Tourne en boucle avec un LLM (celui que tu choisis — Gemma 4 est recommandé parce qu'il marche bien avec MLX sur Apple Silicon)
Apprend de l'expérience : il crée ses propres « skills » (comme les skills que j'utilise) au fil des tâches et les améliore
Garde la mémoire des conversations passées avec recherche full-text
Peut lancer des sous-agents en parallèle
S'intègre à Telegram, Discord, Slack, WhatsApp, Signal, email
Peut exécuter des tâches planifiées via cron

Pourquoi tout ce setup (venv, Node, Playwright, unicode)
Le venv Python 3.11 contient le cœur de l'agent (boucle de raisonnement, gestion mémoire, outils). Les dépendances Node + Playwright activent l'outil navigateur : Hermes peut ouvrir Chromium, lire des pages, remplir des formulaires, naviguer — c'est ce qui lui permet de faire des recherches web ou d'interagir avec des sites. Unicode sert au traitement propre des textes multilingues récupérés. L'installateur fait tout d'un bloc pour que tout soit opérationnel dès le premier lancement.
Les risques, honnêtement
Le code lui-même est safe (Nous Research est une équipe connue, le repo est public sur GitHub avec beaucoup de contributeurs et d'étoiles). Le vrai enjeu est ce que tu autorises l'agent à faire, parce que Hermes est très capable :
D'abord il a accès terminal —

**il peut exécuter des commandes shell sur ton Mac.**

C'est à ça qu'il doit sa puissance, mais ça veut dire qu'il peut en théorie installer des trucs, supprimer des fichiers, envoyer des requêtes réseau. **Par défaut il y a un workflow d'approbation pour les commandes sensibles**, mais c'est à toi de rester vigilant.
Ensuite il a accès au système de fichiers (lecture/écriture). Utile pour travailler sur tes documents, mais ça veut dire qu'il voit ce qu'il y a dans les dossiers qu'il parcourt.
Le navigateur via Playwright ouvre une session isolée par défaut (pas ton Chrome perso), donc pas d'accès automatique à tes comptes bancaires ou autres. Mais si tu configures des logins persistants, attention.
Le cron/scheduler signifie qu'il peut tourner en arrière-plan à heures fixes — assure-toi de savoir ce qu'il exécute.
L'intégration messageries (Telegram, WhatsApp, etc.) ne s'active que si tu la configures explicitement.
Comment réduire la surface
Hermes supporte plusieurs niveaux d'isolation. Tu peux :

Lancer l'exécution des commandes dans Docker plutôt qu'en local (option terminal backend: docker) — c'est la plus sûre
Désactiver des toolsets avec hermes tools (par exemple couper browser ou terminal si tu n'en as pas besoin au début)
Activer le mode « command approval » pour que chaque commande shell te soit soumise avant exécution

Gemma 4 recommandé — pas de risque particulier
Gemma 4 est un modèle ouvert de Google, qui tourne 100% local via Ollama/MLX. Aucune donnée ne sort de ton Mac côté modèle. Le « risque » associé au choix du modèle est juste : il faut assez de RAM unifiée (16 Go minimum pour les petites variantes, 32 Go pour les grosses sur MLX). Si ton Mac a 8 Go, choisis plutôt un modèle plus petit ou accepte qu'Ollama bascule sur Metal au lieu de MLX.
TL;DR
Outil légitime et puissant. Aucun risque de malware. Les risques viennent du fait que tu donnes à un agent autonome l'accès terminal et fichiers de ton Mac — donc commence avec le backend Docker ou le **mode approbation activé**, teste sur des tâches inoffensives, et n'active les intégrations messageries que quand tu en as vraiment besoin.

**Niveau 1** :
 où vivent les fichiers de l'outil
Ça, c'est l'aspect cosmétique/hygiène. Par convention, un outil comme Hermes installe ses choses à des endroits standards sous ~/ :
le code et le venv souvent dans ~/.hermes/ ou ~/hermes-agent/ (dossier caché ou pas selon l'installeur)
la config dans ~/.config/hermes/
les données (mémoire, logs, skills appris) dans ~/.local/share/hermes/ ou ~/Library/Application Support/hermes/ sur macOS

Ces emplacements sont sains et n'interfèrent pas avec tes autres fichiers cachés (.ssh, .aws, .gnupg, .zshrc, etc.). Les installeurs sérieux ne touchent pas à ces trucs-là.
Donc non, le fait que l'installation soit **« dans home » n'est pas un problème en soi** — ton ~/ contient déjà des dizaines de dossiers cachés qui coexistent très bien.
****
**Niveau 2 :**
où l'agent travaille (beaucoup plus important)
Ce qui compte vraiment pour la sécurité, ce n'est pas où est installé Hermes, mais depuis quel dossier tu le lances et à quels dossiers il a accès quand il tourne. Un agent avec terminal + file system access utilise typiquement le répertoire courant (cwd) comme point de départ pour tout ce qu'il fait.
Si tu fais cd ~ puis hermes, il voit tout ton home : .ssh/, Documents, Desktop, tout. Si tu fais cd ~/projets/mon-truc puis hermes, il est mentalement « dans ce dossier » et ne pense pas à aller fouiller ailleurs — même s'il en a techniquement le droit au niveau OS.
Ce que je te recommande
Crée un dossier dédié pour travailler avec Hermes, par exemple ~/hermes-workspace/ ou ~/Agents/hermes/, et lance toujours l'agent depuis là. C'est ton « sandbox mental » : tout ce que l'agent produit, tous les projets sur lesquels tu lui demandes de bosser, atterrissent dedans. Tu gardes tes fichiers sensibles hors de son chemin naturel.
Pour une isolation vraiment stricte, deux options plus musclées :
Option Docker (la plus propre) : Hermes supporte un backend Docker pour l'exécution terminal. Les commandes que l'agent lance tournent dans un conteneur isolé, avec seulement le dossier que tu as monté comme accessible. Ton ~/.ssh devient invisible pour lui, même s'il voulait y aller. Ça se configure dans hermes config ou via le wizard au lancement.
Option utilisateur dédié (niveau paranoïaque) : créer un compte macOS séparé juste pour faire tourner Hermes. Plus lourd mais l'isolation est totale.
Pour ton cas concret
Si Hermes est déjà installé dans ~/ avec ses fichiers cachés, laisse-le là — déplacer les fichiers d'un outil après installation casse souvent les chemins codés en dur.

 **Par contre, crée dès maintenant un dossier de travail séparé, genre** :
mkdir -p ~/hermes-workspace
cd ~/hermes-workspace
hermes
Et prends l'habitude de toujours lancer l'agent depuis là. Au premier démarrage, quand il te propose un backend d'exécution, choisis Docker si tu l'as installé — c'est le meilleur compromis sécurité/confort.
Bonus : ajoute aussi ~/.ssh, ~/.aws, ~/.config/gh à une liste d'exclusion si Hermes propose un système de « forbidden paths » dans sa config — certaines versions le permettent.

## ⚠️ Isolation Docker — fix critique du 2026-05-28

Un test canary a révélé une **fuite vérifiée** : malgré `terminal.backend: docker` présent dans `~/.hermes/config.yaml`, le tool terminal continuait à exécuter les commandes **sur l'hôte**, exposant `~/.ssh`, `~/.aws`, et tout `/Users/rdb/` à l'agent.

### La cause exacte

`terminal.backend` du YAML n'est PAS lu au runtime. Le tool terminal lit uniquement la variable d'env `TERMINAL_ENV` depuis `~/.hermes/.env`. Éditer `config.yaml` à la main ne synchronise jamais cette variable, et `hermes config show` affiche `Backend: docker` (qui lit le YAML) en masquant la divergence — piège silencieux total.

### Le fix appliqué (déjà permanent)

**`TERMINAL_ENV=docker` a été écrit dans `~/.hermes/.env`.** Cette valeur **persiste entre tous les lancements de Hermes** — strictement aucune action à refaire au démarrage, ni aujourd'hui ni demain. Le `.env` n'est pas écrasé par Hermes et est rechargé à chaque démarrage du process.

### La règle absolue à respecter pour la suite

**Ne JAMAIS éditer `~/.hermes/config.yaml` à la main pour les options `terminal.*`.** Utiliser systématiquement la commande suivante, qui est la SEULE à déclencher la sync `config.yaml ↔ .env` :

```bash
hermes config set terminal.<clé> <valeur>
```

Exemple pour le backend :

```bash
hermes config set terminal.backend docker
```

Toute édition directe du YAML laisse silencieusement le backend sur sa valeur précédente (par défaut `local` = aucune isolation, accès complet à `~/`).

### Vérification en une commande (à faire après tout doute)

```bash
grep ^TERMINAL_ENV ~/.hermes/.env
```

Doit retourner exactement :

```text
TERMINAL_ENV=docker
```

Si la ligne est absente, commentée, ou vaut `local` → la fuite est revenue. Relancer :

```bash
hermes config set terminal.backend docker
```

### Test d'isolation reproductible

Côté hôte (Mac) :

```bash
echo "canary-$(uuidgen)" > /tmp/hermes-canary.txt
cat /tmp/hermes-canary.txt
```

Puis dans la TUI Hermes, taper :

```text
Affiche le résultat BRUT de : cat /tmp/hermes-canary.txt
```

Sortie attendue si l'isolation fonctionne :

```text
cat: /tmp/hermes-canary.txt: No such file or directory   (exit 1)
```

Si l'UUID s'affiche → fuite confirmée, refaire le fix ci-dessus.

### Effet secondaire à connaître

`hermes config set` réécrit `~/.hermes/config.yaml` en forme canonique et **supprime tous les commentaires** (defaults Hermes + tes annotations personnelles). Toujours sauvegarder le fichier avant si des commentaires sont précieux :

```bash
cp ~/.hermes/config.yaml ~/.hermes/config.yaml.bak.$(date +%Y%m%d)
```

## 🖼️ Coller une capture d'écran dans le prompt — fix du 2026-05-28

Hermes accepte les images collées directement dans le prompt (drag-and-drop dans la TUI, ou raccourci de collage). Deux bugs empêchaient ce flux de fonctionner et ont été corrigés sur `~/.hermes/hermes-agent` (commits `fb9064b5e` et `06fc360f8`, mergés sur `main`).

### ⚠️ Prérequis non négociable

**Cette fonctionnalité ne marche QU'AVEC un modèle multimodal (vision).** Avec un modèle texte-seul, l'image est soit ignorée, soit convertie en description textuelle par un appel auxiliaire — résultat dégradé.

Modèles multimodaux recommandés selon le provider :

- **Anthropic** : `claude-opus-4-7`, `claude-sonnet-4-6`, `claude-haiku-4-5`
- **OpenAI** : `gpt-4o`, `gpt-4o-mini`, `gpt-5` series
- **Google** : `gemini-2.5-pro`, `gemini-3-flash`
- **Local (Ollama/MLX)** : `gemma-3` (les variantes texte-only ne fonctionneront pas)

Vérifier le modèle courant avec `hermes model` avant de coller une image.

### Les deux bugs corrigés

1. **`cli.py:11764` — `TypeError: can only concatenate str (not "list") to str`** : quand une image était attachée, les préfixes système (voice prefix, model-switch note, skills-reload note) tentaient `str + message` alors que `message` était une liste multimodale. Crash systématique du thread `run_agent` à chaque image attachée juste après un `/model`.
2. **`title_generator.py:48` — `HTTP 400: prompt is too long: 1245301 tokens > 1000000 maximum`** : la génération automatique de titre de session prenait `user_message[:500]` sur une liste, stringifiait le résultat et embarquait la data URL base64 complète de l'image (1MB+) dans le prompt envoyé au modèle auxiliaire. Une seule capture suffisait à dépasser la fenêtre de contexte de 1M tokens.

### Validation rapide après mise à jour

Dans la TUI, avec un modèle multimodal sélectionné :

```text
[coller une capture d'écran]
Décris-moi cette image.
```

Sortie attendue : description correcte de l'image, **et** aucune trace de `TypeError` ou `prompt is too long` dans `~/.hermes/logs/agent.log`.

Si l'agent répond mais qu'un warning `⚠ Auxiliary title generation failed` apparaît, c'est que le fix `title_generator.py` n'est pas appliqué — re-sync `~/.hermes/hermes-agent` sur `main`.

## 📨 Veille blogdumoderateur → email sécurisé (pattern outbox)

Feature ajoutée le 2026-05-29 : l'agent Hermes scrape les nouveaux articles
de [blogdumoderateur.com](https://www.blogdumoderateur.com/) et le résumé
part par email via Gmail, **sans jamais exposer les identifiants Gmail au
conteneur** qui exécute du contenu web non fiable.

### Principe — séparation des privilèges

- L'**agent** (conteneur Docker) se contente de déposer un fichier `.txt`
  dans `/workspace/outbox/` (écriture atomique `.part` → rename).
- Un **service de confiance côté hôte** détient seul les credentials
  Gmail, ramasse le fichier et l'envoie.

Conséquence : même en cas d'injection de prompt via un blog piégé, le pire
que l'agent puisse faire est d'écrire un fichier — il ne peut ni envoyer un
mail arbitraire, ni voler le token, ni changer le destinataire.

### Composants

Hors du volume monté (donc inaltérables par le conteneur), dans `~/.hermes/`
et **non versionnés** dans ce repo par sécurité :

- `bin/gmail_outbox_sender.py` — envoi via l'API Gmail (scope `gmail.send`
  uniquement), stdlib pure. Déclenché par launchd `com.hermes.outbox-sender`
  (WatchPaths sur l'outbox + filet périodique).
- `bin/gmail_oauth_bootstrap.py` — obtention unique du refresh token (à
  lancer dans un vrai terminal). Client OAuth de type « Application de
  bureau » obligatoire (sinon `redirect_uri_mismatch`).
- `bin/run-veille.sh` — wrapper de lancement de l'agent.
- `outbox-sender.conf` — destinataire/expéditeur fixes, quotas.

Versionnés dans ce repo : `prompt-agent-hermes.txt`, `docs/outbox-email.md`.

### Sécurité

- Scope OAuth **`gmail.send`** seulement (aucune lecture de la boîte mail).
- Secrets dans le **Trousseau macOS** (`account=hermes`), jamais sur disque
  ni dans git.
- **Destinataire figé côté hôte** (anti-exfiltration).
- Contenu envoyé en **pièce jointe** (jamais interprété comme HTML), avec
  plafond de taille et quota d'envois par jour.
- `docker_forward_env` reste **vide** : aucun secret injecté dans le
  conteneur.

### Déduplication « depuis le dernier envoi »

Le filtrage se fait **par URL d'article**, pas par date (l'agent, surtout en
Haiku, ne filtre pas les dates de façon fiable) :

- l'agent exclut les URLs déjà présentes dans
  `/workspace/.veille/seen-urls.txt` ; s'il ne reste rien de neuf, il ne
  dépose aucun fichier (donc aucun email) ;
- après un envoi réussi, le sender ajoute les URLs du digest à
  `~/.hermes/veille/sent-urls.txt`.

### Lancement et nettoyage des conteneurs

L'agent se lance via `~/.hermes/bin/run-veille.sh`, qui publie la liste de
référence dans le volume, injecte la date du jour, puis lance l'agent.

Hermes nomme chaque conteneur `hermes-<uuid>` (non configurable) et, en mode
`container_persistent: true`, ne fait que l'**arrêter** → ils s'accumulent
en `Exited`. Le wrapper supprime donc, en fin de run (via un `trap`),
**uniquement** le conteneur créé par ce run — sans toucher à une éventuelle
session `hermes chat` lancée en parallèle.

### Installation (une seule fois)

```bash
python3 ~/.hermes/bin/gmail_oauth_bootstrap.py
launchctl load ~/Library/LaunchAgents/com.hermes.outbox-sender.plist
```

Voir `docs/outbox-email.md` pour le détail complet.
