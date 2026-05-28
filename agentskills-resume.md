# Résumé de la norme Agent Skills (agentskills.io)

## Vue d'ensemble

**Agent Skills** est un format ouvert, publié par Anthropic le **18 décembre 2025**, qui standardise la façon de donner des « compétences » supplémentaires à un agent IA.

L'idée est simple : plutôt que de re-prompter un agent ou de coder un plugin propriétaire pour chaque fournisseur, on package une expertise dans un dossier que n'importe quel agent compatible peut découvrir et charger à la demande.

Microsoft, OpenAI, Atlassian, Figma, Cursor, GitHub et une vingtaine d'autres plateformes l'ont déjà adopté, ce qui en fait *de facto* le pendant « compétences » de ce que MCP est aux outils.

## Structure d'un skill

Un *skill* est un simple répertoire contenant au minimum un fichier `SKILL.md`.

```
mon-skill/
├── SKILL.md          ← obligatoire (frontmatter YAML + Markdown)
├── scripts/          ← optionnel — code exécutable (Python, Bash, JS…)
├── references/       ← optionnel — documentation technique / domaine
└── assets/           ← optionnel — gabarits, images, données
```

## Le fichier SKILL.md

Il commence par un *frontmatter* YAML suivi du contenu Markdown qui constitue les instructions détaillées.

### Champs obligatoires du frontmatter

| Champ | Contrainte | Rôle |
|---|---|---|
| `name` | ≤ 64 caractères | Identifiant du skill |
| `description` | 1 à 1024 caractères | Explique **ce que** fait le skill **et quand** l'utiliser — c'est ce texte que l'agent lit pour décider de l'activer |

### Sous-dossiers facultatifs

- **`scripts/`** — code exécutable que l'agent peut lancer ; les langages supportés dépendent de l'implémentation (Python, Bash, JavaScript les plus courants).
- **`references/`** — documentation technique ou spécifique au domaine, consultée à la demande.
- **`assets/`** — gabarits, images, fichiers de données.

## Principe clé : le *progressive disclosure*

L'agent ne charge initialement que la **description courte**. Il ne lit le `SKILL.md` complet que s'il juge le skill pertinent, et n'ouvre les références ou n'exécute les scripts qu'au besoin.

Cela évite de saturer la fenêtre de contexte et permet d'avoir des bibliothèques de skills très larges sans coût à l'inférence.

## Gouvernance

- **Maintenance** : Anthropic, sur GitHub (`agentskills/agentskills`).
- **Licences** : Apache 2.0 pour le code, CC-BY-4.0 pour la documentation.
- **Écosystème** : SDK de référence et exemples disponibles, contributions communautaires ouvertes.

## Lien avec HERMES

Votre base SQLite d'agentskills stocke vraisemblablement des skills conformes à ce format — métadonnées du frontmatter indexées en table, contenu du `SKILL.md` et chemins vers `scripts/` / `references/`. Cela rend votre workspace potentiellement **portable** vers n'importe quel autre runtime compatible Agent Skills.

## Sources

- [Specification — Agent Skills](https://agentskills.io/specification)
- [GitHub — agentskills/agentskills](https://github.com/agentskills/agentskills)
- [anthropics/skills — agent-skills-spec.md](https://github.com/anthropics/skills/blob/main/spec/agent-skills-spec.md)
- [Anthropic Opens Agent Skills Standard — Unite.AI](https://www.unite.ai/anthropic-opens-agent-skills-standard-continuing-its-pattern-of-building-industry-infrastructure/)
- [Use Agent Skills in VS Code](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
- [Agent Skills — Strapi blog](https://strapi.io/blog/what-are-agent-skills-and-how-to-use-them)
