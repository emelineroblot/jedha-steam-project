# 🎮 Steam — Analyse globale du marché du jeu vidéo

Projet réalisé dans le cadre de la certification **Jedha — Data Science & Data Engineering** (bloc Big Data).

> **Contexte (fictif)** : Ubisoft souhaite lancer un nouveau jeu et demande une analyse globale
> du marché Steam pour comprendre **quels facteurs influencent la popularité et les ventes d'un jeu vidéo**.

EDA distribuée sur **55 691 entrées** de JSON imbriqué, avec Databricks et PySpark.

---

## 📎 Sorties d'exécution Databricks

La fonction *Publish* de Databricks ne génère plus d'URL publique partageable. Les sorties
d'exécution sont donc fournies sous forme de **captures d'écran** :

> 📁 **[`screens_databricks/`](screens_databricks/)** — 61 images numérotées dans l'ordre de
> lecture du notebook (`00001.jpg` → `00061.jpg`), couvrant l'intégralité des cellules, des
> tableaux de résultats et des visualisations. Chaque capture porte le **numéro de cellule
> Databricks** en haut à droite.

Le notebook `Steam_project_1.ipynb` contient lui aussi toutes ses sorties : il se lit
directement dans GitHub, dans Jupyter, ou après réimport dans Databricks.

---

## 🔑 Résultats clés

### Le marché

| | |
|---|---|
| Catalogue analysé | **54 725 jeux**, sur 55 691 entrées Steam |
| Éditeurs distincts | 29 497 — le plus prolifique (Big Fish Games, 422 jeux) pèse **0,77 %** |
| Prix moyen / médian | **7,58 $ / 4,99 $** — 42,6 % du catalogue entre 0 et 5 $, 13,5 % de gratuits |
| Jeux au-dessus de 60 $ | **63** (0,12 %) — le point de prix AAA n'existe quasiment pas sur Steam |
| En promotion (instantané) | 2 495 jeux (4,6 %), remise moyenne 57,6 % |
| Ratio positif moyen / pondéré | 73,74 % / **85,88 %** |
| Windows / Mac / Linux | **99,98 % / 23,05 % / 15,33 %** — 73,96 % du catalogue est Windows seul |
| Effet Covid | **+20,4 % de sorties en 2020**, après −9,3 % en 2019 · record en 2021 (8 722 jeux) |
| Genre le plus représenté | Indie — 39 681 jeux, soit 72,7 % des jeux… et aussi le mieux noté (88,47 % pondéré) |
| Genre au revenu estimé le plus élevé | Action — 58 756 M$ cumulés · MMO en tête par jeu (8,6 M$) |
| Interdits aux moins de 18 ans | 230 jeux (0,42 %) — **borne basse**, la déclaration d'âge est facultative |

### La réponse à la question centrale

> *« Understand what factors affect the popularity or sales of a video game. »*

Les deux seules corrélations fortes avec le nombre de possesseurs — joueurs simultanés (0,774) et
volume d'avis (0,578) — sont **d'autres mesures de la même chose**. Toutes les variables sur
lesquelles un studio peut réellement agir restent **sous 0,10**.

Mais par segment, les écarts sont considérables :

| Levier | Moyenne des possesseurs | Médiane | Corrélation |
|---|---|---|---|
| Localisation (1 → 13+ langues) | 48 614 → **770 057** — **× 15,8** | inchangée | 0,089 |
| Qualité perçue (< 50 % → 90 %+) | 117 022 → **450 551** — **× 3,9** | inchangée | 0,024 |
| Portage (1 → 3 plateformes) | 103 335 → **321 637** — **× 3,1** | inchangée | 0,036 |

**Trois leviers, trois fois le même schéma** : un effet fort sur la moyenne, **nul sur la médiane**,
et une corrélation linéaire négligeable. C'est la signature de relations **non linéaires concentrées
dans la queue de distribution**.

> **Sur Steam, rien ne fait sortir un jeu de l'anonymat de façon fiable, mais plusieurs facteurs
> amplifient nettement un succès déjà amorcé.**

Le contre-intuitif à retenir : un jeu excellent n'a pas plus de possesseurs *médians* qu'un jeu
médiocre. En revanche sa moyenne est quatre fois plus élevée, il se vend 70 % plus cher et reçoit
trois fois plus d'avis. **La qualité amplifie, elle ne déclenche pas.**

### Ce que ça implique pour Ubisoft

- **Prix** — viser la tranche **30-60 $**, pas le seuil symbolique des 60 $. Elle ne pèse que 1,8 %
  du catalogue mais domine tous les indicateurs : meilleure moyenne de possesseurs, 937 avis médians
  (contre 17 pour les jeux à moins de 10 $), meilleur revenu moyen estimé.
- **Genre** — arbitrer sur le revenu, pas sur la satisfaction : six points de ratio pondéré séparent
  les neuf genres principaux, alors que les écarts de revenu vont de un à sept en cumulé.
- **Plateformes** — Windows en priorité absolue ; Mac et Linux après validation commerciale, avec
  une réserve de causalité explicite (voir *Limites*).
- **Qualité** — un amplificateur, pas un déclencheur. Pour un éditeur disposant d'une force de
  frappe marketing, c'est précisément la situation où l'investissement paie.

---

## 📊 Données

| | |
|---|---|
| **Source** | `s3://full-stack-bigdata-datasets/Big_Data/Project_Steam/steam_game_output.json` |
| **Origine** | Extraction de l'API **SteamSpy** |
| **Volume** | 55 691 entrées, structure JSON imbriquée (`data` → 22 champs, dont une struct `tags` de plusieurs centaines de colonnes) |
| **Nature** | Semi-structuré : struct imbriquée, tableaux (`categories`), champs texte multi-valués (`genre`, `languages`) |

### Pièges du dataset — et comment ils sont traités

Aucun de ces pièges ne provoque d'erreur : le notebook tourne, les graphiques s'affichent, et les
chiffres sont faux. C'est ce qui rend la phase de préparation plus déterminante que l'analyse
elle-même sur ce dataset.

| Piège | Traitement dans le notebook |
|---|---|
| `price` et `initialprice` sont en **centimes de dollar US**, stockés en **texte** | Cast + division par 100 → `price_usd` (§2.3). Sans ce cast, la moyenne s'affiche à 758 au lieu de 7,58 $, et `min()`/`max()` comparent **lexicographiquement** — `"12499"` passe avant `"9999"`. |
| Les valeurs manquantes sont des **chaînes vides**, pas des `NULL` | Normalisation `"" → NULL` sur les 15 colonnes texte (§2.2), avant tout comptage. Un `isNull()` naïf conclut à tort que le dataset est complet — il manque en réalité 134 éditeurs, 127 développeurs, 161 genres. |
| Le catalogue contient des **logiciels** vendus sur Steam, pas seulement des jeux | Le champ `type` est **inopérant** : il vaut `game` pour 55 690 entrées sur 55 691, Blender et Aseprite compris. Le tri se fait donc sur le **genre déclaré** (liste `GENRES_LOGICIELS`, §2.6) → **965 entrées écartées**. |
| Un jeu déclare **plusieurs genres** (2,83 en moyenne) | `explode` assumé et documenté ; `countDistinct("appid")` partout où il faut compter des jeux. |
| `release_date` mélange **plusieurs formats** | Année par regex (robuste à tous les formats) + date complète par `coalesce` sur **6 motifs** via `try_to_date` (§2.3) → 99,8 % / 99,6 % de couverture. |
| `owners` est une **fourchette** (`"1,000,000 .. 2,000,000"`) | Parsing des bornes et milieu de fourchette `owners_mid`, proxy de ventes assumé avec réserves explicites. |
| `owners_mid` est une variable **en paliers** | 10 000 / 35 000 / 75 000 / 350 000… Toute médiane retombe sur le même palier : la **moyenne est donc affichée à côté** dans toute la partie 6. |

---

## 🗂 Structure du notebook

`Steam_project_1.ipynb` — **70 cellules** (52 de code, 18 de markdown), toutes sorties conservées.

| Partie | Contenu |
|---|---|
| **1. Chargement** | Lecture S3, exploration du schéma imbriqué |
| **2. Préparation** | Aplatissement, normalisation, typage, 17 variables dérivées, contrôles qualité, périmètre |
| **3. Analyse macro** | Éditeurs · qualité (ratio brut vs pondéré vs Wilson) · temporalité et effet Covid · prix · promotions · langues · classification par âge · fonctionnalités Steam |
| **4. Analyse par genre** | Représentation · qualité pondérée · prix · revenu estimé · spécialités des éditeurs |
| **5. Analyse par plateforme** | Répartition · combinaisons exactes · prix et qualité · genres préférentiels (indice de sur-représentation) |
| **6. Facteurs de succès** | Matrice de corrélation · localisation · portage · prix · fonctionnalités · qualité → **la réponse à la question centrale** |
| **7. Synthèse** | Synthèse chiffrée générée depuis les dataframes · lecture analytique · recommandations métier (clairement séparées des constats data) |

---

## ✅ Conformité au brief

### Les questions posées

| Question de l'énoncé | Section | Réponse |
|---|---|---|
| Quel éditeur a publié le plus de jeux ? | §3.1 | Big Fish Games — 422 jeux, soit 0,77 % du catalogue |
| Quels sont les jeux les mieux notés ? | §3.2 | classement par score de Wilson : Flowers, A Short Hike, Portal 2, Hades… |
| Y a-t-il des années avec plus de sorties ? Effet Covid ? | §3.3 | record 2021 (8 722) · **+20,4 % en 2020** |
| Comment les prix sont-ils distribués ? | §3.4 | moyenne 7,58 $, médiane 4,99 $, 7 tranches |
| Y a-t-il beaucoup de promotions ? | §3.5 | 2 495 jeux (4,6 %), remise moyenne 57,6 % |
| Quelles langues sont les plus représentées ? | §3.6 | anglais 98,97 %, allemand 25,2 %, français 24,1 % · 53,2 % de monolingues |
| Beaucoup de jeux interdits aux -16/-18 ? | §3.7 | 230 jeux 18+ (0,42 %), 76 en 16-17 — borne basse assumée |
| Quels sont les genres les plus représentés ? | §4.2 | Indie 39 681 · Action 23 759 · Casual 22 086 |
| Des genres au meilleur ratio positif/négatif ? | §4.3 | Indie 88,47 % — 6 points d'écart sur les 9 genres principaux |
| Des éditeurs ont-ils des genres de prédilection ? | §4.6 | Big Fish → 97 % Casual+Adventure · Square Enix → Action+RPG · SEGA, le plus diversifié |
| Quels sont les genres les plus lucratifs ? | §4.5 | Action 58 756 M$ cumulés · MMO 8,6 M$ par jeu |
| La plupart des jeux sont-ils sur Windows/Mac/Linux ? | §5.1–5.2 | 99,98 / 23,05 / 15,33 % · combinaisons exactes détaillées |
| Des genres préférentiellement sur certaines plateformes ? | §5.4 | Strategy sur-représenté sur Mac (indice 1,20), Early Access délaissé (0,63) |
| **Objectif principal : les facteurs de popularité / ventes** | **§6** | **partie entière, 8 cellules** |

### Les exigences techniques

| Exigence | Où |
|---|---|
| Databricks + PySpark | tout le notebook (Spark Connect, `pyspark.sql.functions`) |
| **Visualisations natives Databricks** | 36 `display()`, dont une vingtaine configurés en bar / pie / line charts |
| Structure imbriquée (`getField`, notation pointée) | §2.1 — `F.col("data.platforms.windows")` |
| `explode()` | 5 usages : langues, genres sensibles, catégories, genres, fonctionnalités |
| Manipulation de champs texte | `regexp_replace`, `regexp_extract`, `split`, `trim` (§2.3, §3.6) |
| Manipulation de champs date | `try_to_date` × 6 formats + `coalesce`, `date_format` (§2.3) |
| Agrégations et `groupBy` | `countDistinct`, `percentile_approx`, `sum`, `avg`, `pivot` |
| Plusieurs dataframes par niveau d'analyse | `df_clean` · `df_games` · `df_genres` · `df_langues` · `df_categories` |

---

## 🧪 Méthodologie — choix à défendre

- **Filtrer les logiciels par le genre, pas par `type`.** Le champ prévu pour ça ne distingue rien
  (`game` pour 55 690 entrées sur 55 691). Le contrôle qui valide le filtre : les comptages par
  genre de jeu sont **strictement inchangés** avant et après — aucun jeu n'a été perdu.
- **Score de Wilson** plutôt que ratio brut pour les classements de qualité : la borne basse de
  l'intervalle de confiance à 95 % intègre le volume d'avis, ce qui évite qu'un jeu à 104 avis
  et 100 % de positifs domine le classement.
- **Ratio pondéré** (`Σ positifs / Σ avis`) affiché à côté du **ratio moyen** (moyenne des ratios
  par jeu) : les deux chiffres sont justes, ils ne répondent pas à la même question.
- **`owners` plutôt que le nombre d'avis** comme proxy de ventes, avec les réserves énoncées
  (fourchette large, prix courant ≠ prix de vente historique, commission Steam non déduite,
  double comptage inter-genres).
- **Moyenne affichée à côté de la médiane** en partie 6 : `owners` étant une variable en paliers,
  les médianes retombent sur le même palier et paraissent plates même quand la distribution se
  déplace nettement.
- **Indice de sur-représentation** plutôt que pourcentage brut pour le portage par genre : dire
  que 100 % des genres sont sur Windows n'apprend rien.
- **Synthèse générée par le code** : les chiffres de la partie 7 sont extraits des dataframes,
  ils ne peuvent donc pas diverger des tableaux du notebook.

---

## ⚠️ Limites de l'analyse

- **`owners` est une variable en paliers**, pas une mesure continue. C'est la limite la plus
  structurante : elle écrase les médianes et interdit toute analyse fine de la diffusion. Les
  revenus estimés sont des **ordres de grandeur relatifs**, jamais des montants.
- **Photographie instantanée** : ni historique de prix, ni historique d'avis, ni suivi des jeux
  retirés du catalogue. Toute lecture temporelle porte sur les *survivants*, et la dernière année
  du dataset est **tronquée à novembre 2022**.
- **La séparation jeux / logiciels est une heuristique**, construite sur une liste de genres — pas
  une vérité de terrain.
- **`required_age` est déclaratif** et vaut 0 pour 98,8 % des jeux : Steam n'impose une
  classification que pour certains contenus. Les chiffres par âge sont des bornes basses.
- **Biais de sélection sur le portage** : les jeux disponibles sur Mac et Linux affichent de
  meilleurs indicateurs, mais un studio ne finance un portage que pour un jeu qui marche déjà. La
  causalité va du succès vers le portage autant que l'inverse.
- **Corrélations linéaires uniquement**, sur des distributions à queue très lourde où Pearson est
  peu adapté. Un coefficient de Spearman, ou une analyse sur les rangs, serait plus robuste.

**Pistes d'approfondissement** : exploiter le champ `tags` (plusieurs centaines de tags
communautaires, bien plus fins que la vingtaine de genres officiels) ; analyser `short_description`
en NLP ; modéliser `owners_mid` par une régression pour hiérarchiser les facteurs de la partie 6
plutôt que de les observer un à un.

---

## ▶️ Reproduire l'analyse

1. Importer `Steam_project_1.ipynb` dans un workspace Databricks (*Workspace → Import → File*).
2. L'attacher à un cluster. **L'exécution de référence a été faite sur Databricks Serverless** —
   d'où l'absence de `.cache()`, non disponible dans ce mode, Spark Connect matérialisant seul les
   résultats intermédiaires.
3. *Run all*. Aucune clé API, aucune configuration : le bucket S3 est public.
4. Configurer les visualisations : chaque `display()` est précédé d'un commentaire indiquant le
   type de graphique conseillé.

Les captures de `screens_databricks/` correspondent exactement à cette exécution.

---

## 📁 Structure du dépôt

| Fichier | Contenu |
|---|---|
| `Steam_project_1.ipynb` | **le livrable** — 70 cellules, toutes sorties conservées, 344 Ko |
| `screens_databricks/` | 61 captures des sorties Databricks, dans l'ordre de lecture |
| `README.md` | ce document |
| `.gitignore` | exclut les artefacts Python, les fichiers OS et les documents de travail |

---

## 🛠 Stack

Databricks (Serverless) · Apache Spark / PySpark · `pyspark.sql.functions` · `pyspark.ml.feature` et
`pyspark.ml.stat` (corrélations) · visualisations natives Databricks
