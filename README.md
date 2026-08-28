# 🎮 Steam — Analyse globale du marché du jeu vidéo

Projet réalisé dans le cadre de la certification **Jedha — Data Science & Data Engineering** (bloc Big Data).

> **Contexte (fictif)** : Ubisoft souhaite lancer un nouveau jeu et demande une analyse globale
> du marché Steam pour comprendre **quels facteurs influencent la popularité et les ventes d'un jeu vidéo**.

---

## 📎 Notebook publié

> 🔗 **Lien Databricks** : _à compléter après publication_
>
> `https://…databricks.com/…`

*(Le lien est généré depuis Databricks via le bouton **Publish**. Si la publication est refusée
pour cause de taille, découper le notebook aux frontières de parties indiquées ci-dessous.)*

---

## 📊 Données

| | |
|---|---|
| **Source** | `s3://full-stack-bigdata-datasets/Big_Data/Project_Steam/steam_game_output.json` |
| **Origine** | Extraction de l'API **SteamSpy** |
| **Volume** | 55 691 entrées, structure JSON imbriquée (`data` → 22 champs, dont une struct `tags` de plusieurs centaines de colonnes) |
| **Nature** | Semi-structuré : struct imbriquée, tableaux (`categories`), champs texte multi-valués (`genre`, `languages`) |

### Pièges du dataset — et comment ils sont traités

| Piège | Traitement dans le notebook |
|---|---|
| `price` et `initialprice` sont en **centimes de dollar US**, stockés en **texte** | Cast + division par 100 → `price_usd` (partie 2.3). Sans ce cast, `min()`/`max()` comparent lexicographiquement et renvoient des résultats faux. |
| Les valeurs manquantes sont des **chaînes vides**, pas des `NULL` | Normalisation `"" → NULL` sur toutes les colonnes texte (partie 2.2), avant tout comptage. |
| Le catalogue contient des **logiciels** vendus sur Steam, pas seulement des jeux | Le champ `type` est **inopérant** ici : il vaut `game` pour 55 690 entrées sur 55 691, logiciels compris. Le tri se fait donc sur le **genre déclaré** (liste `GENRES_LOGICIELS`, partie 2.6), qui isole les catégories Steam non ludiques. |
| Un jeu déclare **plusieurs genres** | `explode` assumé et documenté ; `countDistinct("appid")` partout où il faut compter des jeux. |
| `release_date` mélange **plusieurs formats** | Année par regex (robuste) + date complète par `coalesce` sur 4 formats (partie 2.3). |
| `owners` est une **fourchette** (`"1,000,000 .. 2,000,000"`) | Parsing des bornes et milieu de fourchette `owners_mid`, utilisé comme proxy de ventes avec réserves explicites. |

---

## 🗂 Structure du notebook

`Steam_project_1.ipynb` — 70 cellules, à importer dans Databricks.

| Partie | Contenu |
|---|---|
| **1. Chargement** | Lecture S3, exploration du schéma imbriqué |
| **2. Préparation** | Aplatissement, normalisation, typage, variables dérivées, contrôles qualité, périmètre |
| **3. Analyse macro** | Éditeurs · qualité (ratio brut vs pondéré vs Wilson) · temporalité et effet Covid · prix · promotions · langues · classification par âge · fonctionnalités Steam |
| **4. Analyse par genre** | Représentation · qualité pondérée · prix · revenu estimé · spécialités des éditeurs |
| **5. Analyse par plateforme** | Répartition · combinaisons exactes · prix et qualité · genres préférentiels (indice de sur-représentation) |
| **6. Facteurs de succès** | Matrice de corrélation · localisation · portage · prix · fonctionnalités · qualité → **la réponse à la question centrale** |
| **7. Synthèse** | Synthèse chiffrée générée depuis les dataframes · lecture analytique · recommandations métier (clairement séparées des constats data) |

Les frontières de découpage, si la publication Databricks refuse le fichier :
**parties 1-2** · **partie 3** · **parties 4-7**.

---

## 🧪 Méthodologie — choix à défendre

- **Score de Wilson** plutôt que ratio brut pour les classements de qualité : la borne basse de
  l'intervalle de confiance à 95 % intègre le volume d'avis, ce qui évite qu'un jeu à 104 avis
  et 100 % de positifs domine le classement.
- **Ratio pondéré** (`Σ positifs / Σ avis`) affiché à côté du **ratio moyen** (moyenne des ratios
  par jeu) : les deux chiffres sont justes, ils ne répondent pas à la même question.
- **Indice de sur-représentation** plutôt que pourcentage brut pour le portage par genre : dire
  que 100 % des genres sont sur Windows n'apprend rien.
- **`owners` plutôt que le nombre d'avis** comme proxy de ventes, avec les réserves énoncées
  (fourchette large, prix courant ≠ prix de vente historique, commission Steam non déduite).
- **Moyenne affichée à côté de la médiane** en partie 6 : `owners` est une variable **en paliers**
  (10 000 / 35 000 / 75 000 / 350 000…), donc les médianes retombent sur le même palier et
  paraissent plates même quand la distribution se déplace nettement.
- **Synthèse générée par le code** : les chiffres de la partie 7 sont extraits des dataframes,
  ils ne peuvent donc pas diverger des tableaux du notebook.

---

## ▶️ Reproduire l'analyse

1. Importer `Steam_project_1.ipynb` dans un workspace Databricks (*Workspace → Import → File*).
2. L'attacher à un cluster (runtime Spark 3.x, Python 3).
3. Exécuter les cellules dans l'ordre — aucune configuration préalable n'est requise, le bucket
   S3 est public.
4. Configurer les visualisations : chaque `display()` est précédé d'un commentaire indiquant
   le type de graphique conseillé.
5. Publier via le bouton **Publish**, puis reporter l'URL en haut de ce README.

---

## 🛠 Stack

Databricks · Apache Spark (PySpark) · `pyspark.sql.functions` · `pyspark.ml.stat` (corrélations) · visualisations natives Databricks
