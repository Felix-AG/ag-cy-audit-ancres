# Ancrages du journal d'audit AG&CY

Ce dépôt conserve, hors de la machine qui les produit, les empreintes de fin du journal d'audit de la passerelle
`ag` (dépôt privé `ag-cy-workspace`, dossier `ag-mcp/logs/`). Il ne contient aucune donnée métier : seulement des
noms de fichiers (date de démarrage et PID d'un processus de la passerelle), des nombres de lignes et des empreintes
SHA-256.

## Ce que ça prouve
Le journal est chaîné ligne à ligne (SHA-256) : la chaîne seule détecte une ligne modifiée, insérée ou retirée au
milieu d'un fichier, mais pas un fichier supprimé, une fin tronquée ni une réécriture complète avec chaîne
recalculée. Une fois ancrée ici, l'empreinte de la dernière ligne fige tout ce qui la précède.

- `ancres/ancre-<AAAAMMJJTHHMMSS>Z.json` : un manifeste par jour ouvré, cumulatif (tout le journal), produit par le
  flow Kestra `agcy.ancrage_audit` après vérification du journal contre le manifeste précédent ; un journal en défaut
  n'est jamais ancré (le flow échoue).
- Version 1 (jusqu'au 2026-10-15) : `fichiers` liste chaque fichier (nom, lignes, empreinte de fin) ; `empreinte` est
  le SHA-256 de `{"fichiers": …}`.
- Version 2 (depuis le 2026-10-15) : `fichiers` ne liste plus que les fichiers du jour ; chaque jour révolu tient en une
  entrée de `jours` : nombre de fichiers clos et SHA-256 de leurs entrées (même forme qu'en version 1), plus, un par
  un, ses fichiers sans `stop` (`ouverts`). `empreinte` couvre `{"fichiers": …, "jours": …}`. Un fichier d'un jour
  révolu ne peut plus apparaître (son nom est l'heure de démarrage du processus), et un fichier clos ne change plus :
  un écart sur un jour est une altération ; les manifestes version 1 de ce dépôt permettent d'en localiser le fichier.
- Toute empreinte est contrôlée à la relecture (manifeste altéré → signalé).

## Vérifier
Sur la machine qui porte le journal, avec un clone de ce dépôt :

    uv run --project ag-mcp python -m ag_mcp.audit --ancres <clone>/ancres

Code 1 si un fichier ancré a disparu, si sa fin a été tronquée ou réécrite, ou si une ligne a été altérée.

## Limites
- La branche `main` interdit le force-push et la suppression (règle du dépôt) : un ancrage poussé ne peut pas être
  effacé par la clé de déploiement. Un compte administrateur du dépôt peut lever la règle : l'historique public
  (commits, forks, caches) reste alors le dernier témoin.
- Un ancrage par jour ouvré : ce qui est écrit puis effacé entre deux ancrages n'est pas couvert.
