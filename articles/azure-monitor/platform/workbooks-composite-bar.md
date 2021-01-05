---
title: Renderer de barre composite des classeurs Azure Monitor
description: Découvrez toutes les visualisations du renderer de barre composite des classeurs Azure Monitor.
services: azure-monitor
author: lgayhardt
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 9/04/2020
ms.author: lagayhar
ms.openlocfilehash: 80846ecb1ad48d9f8ba49f0025772b4e131c23e2
ms.sourcegitcommit: ef69245ca06aa16775d4232b790b142b53a0c248
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/06/2020
ms.locfileid: "91776355"
---
# <a name="composite-bar-renderer"></a>Renderer de barre composite

Le classeur permet la modification de l’affichage des données à l’aide d’une barre composite, une barre composée de plusieurs barres.

L’image ci-dessous montre la barre composite pour l’état de la base de données qui représente le nombre de serveurs à l’état en ligne, hors connexion et en cours de récupération.

![Capture d’écran de la barre composite pour l’état de la base de données.](./media/workbooks-composite-bar/database-status.png)

Le renderer de barre composite est pris en charge pour les visualisations de grilles, de vignettes et de graphiques.

## <a name="adding-composite-bar-renderer"></a>Ajout d’un renderer de barre composite

1. Basculez le classeur en mode d’édition en sélectionnant l’élément *Modifier* de la barre d’outils.
2. Sélectionnez *Ajouter*, puis *Ajouter une requête*.
3. Définissez *Source de données* sur « JSON » et *Visualisation* sur « Grille ».
4. Ajoutez les données JSON suivantes.

```json
[
    {"sub":"X", "server": "A", "online": 20, "recovering": 3, "offline": 4, "total": 27},
    {"sub":"X", "server": "B", "online": 15, "recovering": 8, "offline": 5, "total": 28},
    {"sub":"Y", "server": "C", "online": 25, "recovering": 4, "offline": 5, "total": 34},
    {"sub":"Y", "server": "D", "online": 18, "recovering": 6, "offline": 9, "total": 33}
]
```

5. Exécutez la requête.
6. Sélectionnez **Paramètres de colonne** pour ouvrir les paramètres.
7. Sélectionnez « total » dans *Colonnes* et choisissez « barre composite » pour *Renderer de colonne*.
8. Définissez les paramètres suivants sous *Paramètres de barre composite*.

| Nom de la colonne | Couleur        |
|-------------|--------------|
| online      | Vert        |
| récupération  | Jaune       |
| hors connexion     | Rouge (vif) |

9. Ajouter l’étiquette `["online"] of ["total"] are healthy`.
10. Dans les paramètres de colonne pour les états en ligne, hors connexion et en cours de récupération, vous pouvez définir le renderer de colonne sur « Masqué » (facultatif).
11. Sélectionnez *Étiquettes* en haut et mettez à jour l’étiquette de la colonne Total sur « État de la base de données » (facultatif).
12. Sélectionnez **Appliquer**.

Les paramètres de la barre composite se présentent comme la capture d’écran ci-dessous :

![Capture d’écran des paramètres de colonne de la barre composite avec les paramètres décrits ci-dessus.](./media/workbooks-composite-bar/composite-bar-settings.png)

Voici la barre composite avec les paramètres définis ci-dessus :

![Capture d’écran de la barre composite.](./media/workbooks-composite-bar/composite-bar.png)

## <a name="composite-bar-settings"></a>Paramètres de la barre composite

Sélectionnez le nom de la colonne et la couleur correspondante pour afficher cette colonne dans cette couleur dans le cadre de la barre composite. Vous pouvez insérer et supprimer des lignes et les déplacer vers le haut ou vers le bas.

### <a name="label"></a>Etiquette

L’étiquette de la barre composite s’affiche en haut de la barre composite. Vous pouvez utiliser un mélange de texte statique, de colonnes et de paramètres.  Si Étiquette est vide, la valeur des colonnes actuelles est affichée en tant qu’étiquette. Dans l’exemple précédent, si nous laissons le champ de l’étiquette noir, la valeur du total des colonnes s’affiche.

Faites référence aux colonnes avec `["columnName"]`.

Faites référence aux paramètres avec `{paramName}`.

Le nom de colonne et le nom de paramètre respectent tous deux la casse. Vous pouvez également créer un lien à partir des étiquettes en sélectionnant « Définir cet élément comme lien », puis en ajoutant des paramètres de lien.

### <a name="aggregation"></a>Agrégation

Les agrégations sont utiles pour les visualisations Arborescence/Regrouper par. Les données d’une colonne pour la ligne de groupe sont déterminées par l’agrégation définie pour cette colonne. Il existe trois types d’agrégations applicables pour les barres composites : None, Sum et Inherit.

Pour ajouter des paramètres Regrouper par :

1. Dans les paramètres de colonne, accédez à la colonne à laquelle vous souhaitez ajouter des paramètres.
2. Dans *Paramètres d’arborescence/Regrouper par*, sous *Type d’arborescence*, sélectionnez **Regrouper par**.
3. Sélectionnez le champ avec lequel vous souhaitez effectuer le regroupement.

![Capture d’écran des paramètres Regrouper par.](./media/workbooks-composite-bar/group-by-settings.png)

#### <a name="none"></a>None

L’agrégation None signifie qu’aucun résultat n’est affiché pour cette colonne pour les lignes de groupe.

![Capture d’écran de la barre composite avec l’agrégation None.](./media/workbooks-composite-bar/none.png)

#### <a name="sum"></a>SUM

Si l’agrégation est définie sur Sum, la colonne dans la ligne de groupe affiche la barre composite en utilisant la somme des colonnes utilisées pour l’afficher. L’étiquette utilise également la somme des colonnes qui y sont mentionnées.

Dans l’exemple ci-dessous, l’agrégation maximum est définie sur les états en ligne, hors connexion et en cours de récupération, et l’agrégation de la colonne Total est Sum.

![Capture d’écran de la barre composite avec l’agrégation Sum.](./media/workbooks-composite-bar/sum.png)

#### <a name="inherit"></a>Hériter

Si l’agrégation est définie sur Inherit, la colonne dans la ligne de groupe affiche la barre composite en utilisant l’agrégation définie par les utilisateurs pour les colonnes utilisées pour l’afficher. Les colonnes utilisées dans l’étiquette utilisent également l’agrégation définie par l’utilisateur. Si le renderer de colonne actuel est une barre composite et qu’il est utilisé dans l’étiquette (comme « total » dans l’exemple ci-dessus), Sum est utilisé comme agrégation pour cette colonne.

Dans l’exemple ci-dessous, l’agrégation maximum est définie sur les états en ligne, hors connexion et en cours de récupération, et l’agrégation de la colonne Total est Inherit.

![Capture d’écran de la barre composite avec l’agrégation Inherit.](./media/workbooks-composite-bar/inherit.png)

## <a name="sorting"></a>Tri

Pour les visualisations de grilles, le tri des lignes de la colonne avec le renderer de barre composite repose sur la somme des colonnes utilisées pour afficher dynamiquement l’ordinateur des barres composites. Dans les exemples précédents, la valeur utilisée pour le tri est la somme des colonnes des états en ligne, hors connexion et en cours de récupération pour cette ligne particulière.

## <a name="tiles-visualization"></a>Visualisation des vignettes

1. Sélectionnez **Ajouter**, puis *Ajouter une requête*.
2. Changez la source de données en JSON, puis entrez les données de l’[exemple précédent](#adding-composite-bar-renderer).
3. Choisissez la visualisation de type *Vignettes*.
4. Exécutez la requête.
5. Sélectionnez **Paramètres de la vignette**.
6. Sélectionnez *Gauche* dans les champs de vignette.
7. Entrez les paramètres ci-dessous sous *Paramètres de champ*.
    1. Utiliser la colonne : « serveur ».
    2. Renderer de colonne : « Texte ».
8. Sélectionnez *Bas* dans les champs de vignette.
9. Entrez les paramètres ci-dessous sous *Paramètres de champ*.
    1. Utiliser la colonne : « total ».
    2. Renderer de colonne : « Barre composite ».
    3. Définissez les paramètres suivants sous « Paramètres de barre composite ».

    | Nom de la colonne | Couleur        |
    |-------------|--------------|
    | online      | Vert        |
    | récupération  | Jaune       |
    | hors connexion     | Rouge (vif) |

    4. Ajoutez l’étiquette `["online"] of ["total"] are healthy`.
10. Sélectionnez **Appliquer**.

Paramètres de barre composite pour les vignettes :

![Capture d’écran des paramètres de barre composite pour les vignettes avec les paramètres décrits ci-dessus.](./media/workbooks-composite-bar/tiles-settings.png)

L’affichage de la barre composite pour les vignettes avec les paramètres ci-dessus se présente comme suit :

![Capture d’écran des vignettes de barre composite.](./media/workbooks-composite-bar/composite-bar-tiles.png)

## <a name="graphs-visualization"></a>Visualisation des graphiques

Afin de créer un renderer de barre composite pour la visualisation des graphiques (de type clusters Hive), suivez les instructions ci-dessous.

1. Sélectionnez **Ajouter**, puis *Ajouter une requête*.
2. Changez la source de données en JSON, puis entrez les données de l’[exemple précédent](#adding-composite-bar-renderer).
3. Choisissez la visualisation de type *Graphiques*.
4. Exécutez la requête.
5. Sélectionnez **Paramètres du graphe**.
6. Sélectionnez *Contenu au centre* dans les paramètres de format de nœud.
7. Entrez les paramètres ci-dessous sous *Paramètres de champ*.
    1. Utiliser la colonne : « total ».
    2. Renderer de colonne : « Barre composite ».
    3. Entrez les paramètres suivants sous *Paramètres de barre composite*.

    |Nom de la colonne  |     Couleur    |
    |-------------|--------------|
    | online      | Vert        |
    | récupération  | Jaune       |
    | hors connexion     | Rouge (vif) |

   4. Ajoutez l’étiquette `["online"] of ["total"] are healthy`.
9. Entrez les paramètres ci-dessous sous *Paramètres de mise en page*.
    1. Type de graphique : **Clusters Hive**.
    2. ID de nœud : « serveur ».
    3. Champ Regrouper par : « Aucun ».
    4. Taille du nœud : 100.
    5. Marge entre les hexagones : 5.
    6. Type de coloration : **Aucun**.
1. Sélectionnez **Appliquer**.
    
Paramètres de barre composite pour les graphiques :

![Capture d’écran des paramètres de barre composite pour les graphiques avec les paramètres décrits ci-dessus.](./media/workbooks-composite-bar/graphs-settings.png)

L’affichage de la barre composite pour les graphiques avec les paramètres ci-dessus se présente comme suit :

![Capture d’écran des graphiques à barres composites avec des clusters Hive.](./media/workbooks-composite-bar/composite-bar-graphs.png)

## <a name="next-steps"></a>Étapes suivantes

* [Déployez](workbooks-automate.md) des classeurs avec Azure Resource Manager.
* [Contrôlez](workbooks-access-control.md) et partagez l’accès à vos ressources de classeur.
