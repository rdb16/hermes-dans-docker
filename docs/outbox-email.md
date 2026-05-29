# Envoi d'email sécurisé via le pattern Outbox

Ce document décrit comment l'agent Hermes (qui tourne dans un conteneur
Docker et exécute du contenu web non fiable) fait parvenir par email le
résumé des blogs scrapés, **sans jamais détenir les identifiants Gmail**.

## Principe : séparation des privilèges

L'action privilégiée (envoyer un email avec de vrais identifiants) est
isolée de l'action non fiable (scraper le web).

- L'**agent**, dans le conteneur, se contente de **déposer un fichier**
  dans `/workspace/outbox/` (= `~/HERMES-WORKSPACE/outbox/` côté hôte).
- Un **service de confiance côté hôte** détient seul les credentials
  Gmail, ramasse le fichier et l'envoie via l'API Gmail.

Conséquence : même si un blog piégé injecte un prompt à l'agent, le pire
qu'il puisse faire est d'écrire un fichier. Il ne peut ni envoyer un mail
arbitraire, ni voler le token, ni changer le destinataire.

## Composants

| Élément | Emplacement | Rôle |
|---|---|---|
| `gmail_outbox_sender.py` | `~/.hermes/bin/` (hors volume) | Service d'envoi |
| `gmail_oauth_bootstrap.py` | `~/.hermes/bin/` (hors volume) | Obtention unique du refresh token |
| `outbox-sender.conf` | `~/.hermes/` (hors volume) | Config (destinataire fixe, chemins, quotas) |
| `com.hermes.outbox-sender.plist` | `~/Library/LaunchAgents/` | Déclencheur launchd (WatchPaths) |
| `outbox/` | dans le volume `/workspace` | Point de transfert agent → hôte |

Les scripts et la config vivent **hors** du volume monté : le conteneur
ne peut donc pas les altérer pour détourner les credentials.

## Mesures de sécurité

- **Moindre privilège** : scope OAuth `gmail.send` uniquement (aucun
  accès en lecture à la boîte mail).
- **Aucun secret sur disque** : `client_id`, `client_secret` et
  `refresh_token` sont dans le Trousseau macOS (`account=hermes`).
- **Destinataire fixe** côté hôte : l'agent ne choisit pas le `To:`.
- **Contenu = pièce jointe** : jamais interprété comme HTML.
- **Plafonds** : taille de fichier (1 MiB) et quota d'envois/jour (20).
- **`docker_forward_env: []`** doit rester vide : aucun secret injecté
  dans le conteneur.

## Installation (une seule fois)

1. Dans Google Cloud, créer un client OAuth de type « Desktop app » et
   activer l'API Gmail. Récupérer `client_id` et `client_secret`.

2. Obtenir le refresh token et stocker les secrets dans le Trousseau :

   ```bash
   python3 ~/.hermes/bin/gmail_oauth_bootstrap.py
   ```

3. Vérifier / ajuster `~/.hermes/outbox-sender.conf` (destinataire,
   expéditeur, quotas).

4. Charger le service launchd :

   ```bash
   launchctl unload ~/Library/LaunchAgents/com.hermes.outbox-sender.plist 2>/dev/null
   launchctl load   ~/Library/LaunchAgents/com.hermes.outbox-sender.plist
   ```

5. Test de bout en bout :

   ```bash
   echo "test" > ~/HERMES-WORKSPACE/outbox/test.txt
   sleep 12 && python3 ~/.hermes/bin/gmail_outbox_sender.py
   tail ~/.hermes/logs/outbox-sender.log
   ```

## Instruction à donner à l'agent Hermes

Dans le prompt de l'agent, remplacer « écris le résultat dans `/tmp` »
par :

> Écris le résumé dans `/workspace/outbox/`. Écris d'abord sous un nom
> temporaire terminé par `.part`, puis renomme-le en `.txt` une fois
> l'écriture terminée (écriture atomique). N'envoie aucun email
> toi-même : le dépôt du fichier suffit.

L'écriture `.part` → rename garantit que le service ne ramasse jamais un
fichier à moitié écrit.
