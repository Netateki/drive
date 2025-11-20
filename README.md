# 🦀 RepoEval-Etoile

**RepoEval** (Repository Evaluation) est un outil d'évaluation automatique pour des projets étudiants gérés avec Git, écrit en Rust.

## 🎯 But du Projet

L'objectif est d'automatiser des tâches répétitives pour un enseignant ou un correcteur. Au lieu d'aller manuellement dans chaque projet étudiant pour cloner le dépôt, compter les commits, lancer des tests, etc., cet outil fait tout en une seule commande.

### Le processus global

1.  **Entrée** : Prend une liste d'étudiants et de leurs dépôts (via un fichier CSV).
2.  **Analyse Git** : Analyse l'historique Git de chaque dépôt pour extraire des métriques quantitatives (activité, fréquence des commits...).
3.  **Exécution** : Lance une série de commandes personnalisables sur chaque projet (tests, linter, `cloc`...).
4.  **Rapport** : Centralise toutes ces informations dans un fichier CSV final.

---

## 📂 Architecture du Code

Chaque fichier a une responsabilité unique (principe de séparation des responsabilités).

### `main.rs`  Orchestra
* **Rôle** : Le point d'entrée.
* Il ne fait aucun travail de détail, mais appelle les autres modules séquentiellement :
  1. Parsing des arguments (`cli`)
  2. Chargement de la config (`config`)
  3. Analyse Git (`git_stats`)
  4. Exécution des commandes (`command_exec`)
  5. Génération du rapport (`report`)

### `cli.rs` Interface Ligne de Commande
* **Rôle** : Gère les arguments terminaux (`clap`).
* Définit les entrées obligatoires (dossiers, fichiers) et optionnelles.
* Génère automatiquement l'aide (`--help`).

### `config.rs` Configuration
* **Rôle** : Lit le fichier `config.toml`.
* Rend l'outil flexible en permettant de définir quelles commandes lancer sans recompiler le code.

### `git_stats.rs` Analyseur Git
* **Rôle** : Extrait les métriques des dépôts.
* Utilise la librairie **`git2`** pour interagir directement avec les objets Git (commits, trees) sans dépendre de l'exécutable git système.

### `command_exec.rs` Exécuteur
* **Rôle** : Lance les processus externes.
* Exécute les commandes définies dans la config sur chaque dépôt étudiant et capture `stdout`, `stderr` et le code de sortie.

### `report.rs` Reporter
* **Rôle** : Génère le CSV final.
* Fusionne les données `Student`, les `GitStats` et les `CommandResults`.

---

## 🚀 Utilisation

### Commande de base

Le programme utilise **`clap`** pour gérer les arguments.

```bash
cargo run -- <root_dir> <input_csv> <output_csv> [--config-path <chemin>]
