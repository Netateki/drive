# ⭐ RepoEval-Etoile

**RepoEval** est un outil CLI (ligne de commande) écrit en Rust pour automatiser l'évaluation de dépôts Git d'étudiants.

## 🎯 Objectif du Projet

L'objectif est d'automatiser les tâches répétitives de correction. Au lieu d'aller manuellement dans chaque projet étudiant pour cloner le dépôt, compter les commits et lancer des tests, cet outil effectue tout le processus en une seule commande.

### Le Processus Global

1.  **Entrée** : Prend une liste d'étudiants et de leurs dépôts (via un fichier CSV).
2.  **Analyse Git** : Analyse l'historique Git de chaque dépôt pour extraire des métriques quantitatives (activité, fréquence des commits, etc.).
3.  **Exécution** : Exécute une série de commandes personnalisables sur chaque projet (tests unitaires, linter, `cloc`, etc.).
4.  **Rapport** : Centralise toutes ces informations dans un fichier CSV unique.

---

## 📂 Architecture Modulaire

Chaque fichier a une responsabilité unique (Principe de séparation des responsabilités).

### `main.rs` : Le Chef d'Orchestre
Point d'entrée de l'application. Il ne fait aucun travail de détail mais appelle les modules séquentiellement :
1.  Parsing des arguments via `cli`.
2.  Chargement de la configuration via `config`.
3.  Analyse Git via `git_stats`.
4.  Exécution des commandes via `command_exec`.
5.  Génération du rapport via `report`.

### `cli.rs` : Interface Ligne de Commande
Gère les arguments passés au terminal. Utilise la librairie **`clap`** pour :
* Définir les arguments de manière déclarative.
* Gérer le parsing automatique.
* Générer l'aide (`--help`) et les messages d'erreurs.

### `config.rs` : Configuration
Lit et interprète le fichier `config.toml`. C'est ce module qui rend l'outil flexible en permettant de définir quelles commandes externes exécuter sur les dépôts.

### `git_stats.rs` : Analyseur Git
Le cœur de l'analyse.
* **Bibliothèque utilisée** : `git2` (bindings Rust pour libgit2).
* **Fonctionnement** : Il ne cherche pas manuellement le dossier `.git`. Il utilise `Repository::open(path)` qui détecte proprement la racine du dépôt.
* **Métriques** : Nombre de commits, dates, intervalles de temps.

### `command_exec.rs` : Exécuteur de Tâches
Prend les commandes définies dans la config et les exécute dans le contexte de chaque dépôt étudiant. Il capture `stdout`, `stderr` et le code de retour.

### `report.rs` : Générateur de Rapport
Rassemble les données `Student`, `GitStats` et `CommandResults` pour produire le CSV final.

---

## 🚀 Utilisation

### Commande de base

```bash
cargo run -- <root_dir> <input_csv> <output_csv> [--config-path <chemin>]
````

  * `--` : Sépare les arguments de Cargo de ceux du programme.

### Arguments

| Argument | Type | Description | Exemple |
| :--- | :--- | :--- | :--- |
| **`root_dir`** | Requis | Dossier contenant les sous-dossiers des dépôts étudiants. | `depots/` |
| **`input_csv`** | Requis | Fichier CSV listant les étudiants. | `etudiants.csv` |
| **`output_csv`** | Requis | Chemin du fichier de rapport à générer. | `rapport.csv` |
| **`--config-path`** | Optionnel | Chemin vers la config (défaut: `config.toml`). | `-c my_config.toml` |

-----

## 📄 Formats de Données

### 1\. Fichier d'entrée (`input.csv`)

Doit contenir une ligne d'en-tête stricte pour être désérialisé dans la structure `Student`. Le `repo_name` doit correspondre au nom du dossier dans `root_dir`.

```csv
nom,login,repo_name
Jean Dupont,jdupont,projet-dupont
Marie Curie,mcurie,projet-curie
```

### 2\. Fichier de sortie (`output.csv`)

Le rapport généré contient :

  * Infos étudiant (`nom`, `login`).
  * Stats Git (`nb_commits`, `first_commit`, `last_commit`, `avg_time_between_commits`...).
  * **`command_results`** : Une colonne spéciale contenant du **JSON** avec le résultat détaillé des tests/commandes.

Exemple de JSON dans le CSV de sortie :

```json
[
  {
    "name": "run_tests",
    "status": 0,
    "stdout": "Tests passed...",
    "parsed_output": null
  }
]
```

-----

## ⚙️ Détails Techniques

  * **Langage** : Rust 🦀
  * **Gestion des erreurs** : Utilisation idiomatique de `Result<()>` dans le `main` avec l'opérateur `?` pour propager les erreurs proprement.
  * **Bibliothèques clés** :
      * `clap` (CLI)
      * `git2` (Git)
      * `serde` / `csv` (Données)
      * `toml` (Config)

<!-- end list -->

```
```
