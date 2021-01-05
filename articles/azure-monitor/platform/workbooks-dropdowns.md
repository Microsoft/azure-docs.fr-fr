---
title: Paramètres de liste déroulante des classeurs Azure Monitor
description: Créez des rapports complexes en toute simplicité grâce à des classeurs paramétrables prédéfinis et personnalisés contenant des paramètres de liste déroulante.
services: azure-monitor
manager: carmonm
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 10/23/2019
ms.openlocfilehash: 07a8ece76bffe0fbbcbc211e83730433cdeda2c4
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92143808"
---
# <a name="workbook-drop-down-parameters"></a>Paramètres de liste déroulante des classeurs

Les listes déroulantes permettent à l’utilisateur de collecter une ou plusieurs valeurs d’entrée à partir d’un ensemble connu (par exemple, en sélectionnant l’une des demandes de son application). Elles représentent un moyen convivial de recueillir des entrées arbitraires des utilisateurs. Elles sont particulièrement utiles pour permettre le filtrage dans des rapports interactifs. 

Le moyen le plus simple de spécifier une liste déroulante consiste à fournir une liste statique dans le paramètre. Il est plus intéressant de récupérer la liste de manière dynamique avec une requête KQL. Les paramètres permettent également de spécifier s’il s’agit d’une sélection unique ou multiple et, dans le deuxième cas, comment le jeu de résultats doit être mis en forme (délimiteur, guillemets, etc.).

## <a name="creating-a-static-drop-down-parameter"></a>Créer un paramètre de liste déroulante statique

1. Démarrez avec un classeur vide en mode d’édition.
2. Choisissez _Ajouter des paramètres_ dans les liens du classeur.
3. Cliquez sur le bouton bleu _Ajouter un paramètre_.
4. Dans le volet Nouveau paramètre qui apparaît, entrez :
    1. Nom du paramètre : `Environment`
    2. Type de paramètre : `Drop down`
    3. Obligatoire : `checked`
    4. Autoriser `multiple selection` : `unchecked`
    5. Récupérer des données à partir de : `JSON`
5. Dans le bloc de texte de l’entrée JSON, insérez cet extrait de code JSON :
    ```json
    [
        { "value":"dev", "label":"Development" },
        { "value":"ppe", "label":"Pre-production" },
        { "value":"prod", "label":"Production", "selected":true }
    ]
    ```
6. Appuyez sur le bouton bleu `Update`.
7. Choisissez « Enregistrer » dans la barre d’outils pour créer le paramètre.
8. Le paramètre Environment est une liste déroulante à trois valeurs.

    ![Image représentant la création d’une liste déroulante statique](./media/workbook-dropdowns/dropdown-create.png)

## <a name="creating-a-static-dropdown-with-groups-of-items"></a>Créer une liste déroulante statique comportant des groupes d’éléments

Si le résultat/JSON de votre requête contient un champ « group », la liste déroulante affiche des groupes de valeurs. Suivez l’exemple ci-dessus, en utilisant cette fois le JSON suivant :

```json
[
    { "value":"dev", "label":"Development", "group":"Development" },
    { "value":"dev-cloud", "label":"Development (Cloud)", "group":"Development" },
    { "value":"ppe", "label":"Pre-production", "group":"Test" },
    { "value":"ppe-test", "label":"Pre-production (Test)", "group":"Test" },
    { "value":"prod1", "label":"Prod 1", "selected":true, "group":"Production" },
    { "value":"prod2", "label":"Prod 2", "group":"Production" }
]
```

![Image montrant un exemple de liste déroulante groupée](./media/workbook-dropdowns/grouped-dropDown.png)


## <a name="creating-a-dynamic-drop-down-parameter"></a>Créer un paramètre de liste déroulante dynamique
1. Démarrez avec un classeur vide en mode d’édition.
2. Choisissez _Ajouter des paramètres_ dans les liens du classeur.
3. Cliquez sur le bouton bleu _Ajouter un paramètre_.
4. Dans le volet Nouveau paramètre qui apparaît, entrez :
    1. Nom du paramètre : `RequestName`
    2. Type de paramètre : `Drop down`
    3. Obligatoire : `checked`
    4. Autoriser `multiple selection` : `unchecked`
    5. Récupérer des données à partir de : `Query`
5. Dans le bloc de texte de l’entrée JSON, insérez cet extrait de code JSON :

    ```kusto
        requests
        | summarize by name
        | order by name asc
    ```
1. Appuyez sur le bouton bleu `Run Query`.
2. Choisissez « Enregistrer » dans la barre d’outils pour créer le paramètre.
3. Le paramètre RequestName est la liste déroulante des noms de toutes les demandes de l’application.

    ![Image représentant la création d’une liste déroulante dynamique](./media/workbook-dropdowns/dropdown-dynamic.png)

## <a name="referencing-drop-down-parameter"></a>Faire référence à un paramètre de liste déroulante

### <a name="in-kql"></a>En KQL
1. Ajoutez un contrôle de requête au classeur et sélectionnez une ressource Application Insights.
2. Dans l’éditeur KQL, entrez cet extrait de code :

    ```kusto
        requests
        | where name == '{RequestName}'
        | summarize Requests = count() by bin(timestamp, 1h)

    ```
3. Cela développe ainsi l’heure d’évaluation de la requête :

    ```kusto
        requests
        | where name == 'GET Home/Index'
        | summarize Requests = count() by bin(timestamp, 1h)
    ```

4. Exécutez la requête pour afficher les résultats. Si vous le souhaitez, présentez-les sous forme de graphique.

    ![Image représentant une référence à une liste déroulante en KQL](./media/workbook-dropdowns/dropdown-reference.png)


## <a name="parameter-value-label-selection-and-group"></a>Valeur, étiquette, sélection et groupe de paramètres
La requête utilisée dans le paramètre de liste déroulante dynamique ci-dessus retourne simplement une liste de valeurs fidèlement restituées dans la liste déroulante. Mais comment faire pour choisir un autre nom d’affichage ou en sélectionner un ? Les paramètres de liste déroulante gèrent ces possibilités grâce aux colonnes valeur, étiquette, sélection et groupe.

L’exemple ci-dessous montre comment récupérer une liste de dépendances Application Insights dont les noms d’affichage sont stylisés avec un emoji, dont le premier nom d’affichage est sélectionné et qui est groupée par nom d’opération.

```kusto
dependencies
| summarize by operation_Name, name
| where name !contains ('.')
| order by name asc
| serialize Rank = row_number()
| project value = name, label = strcat('🌐 ', name), selected = iff(Rank == 1, true, false), group = operation_Name
```

![Image montrant un paramètre de liste déroulante utilisant les options de valeur, d’étiquette, de sélection et de groupe](./media/workbook-dropdowns/dropdown-more-options.png)


## <a name="drop-down-parameter-options"></a>Options des paramètres de liste déroulante
| Paramètre | Explication | Exemple |
| ------------- |:-------------|:-------------|
| `{DependencyName}` | Valeur sélectionnée | GET fabrikamaccount |
| `{DependencyName:label}` | Étiquette sélectionnée | 🌐 GET fabrikamaccount |
| `{DependencyName:value}` | Valeur sélectionnée | GET fabrikamaccount |

## <a name="multiple-selection"></a>Sélection multiple
Les exemples donnés jusqu’ici définissent explicitement le paramètre de façon à sélectionner une seule valeur dans la liste déroulante. Les paramètres de liste déroulante prennent également en charge `multiple selection` : il suffit pour l’activer de cocher l’option `Allow multiple selection`. 

L’utilisateur a également la possibilité de spécifier le format du jeu de résultats avec les paramètres `delimiter` et `quote with`. Par défaut, les valeurs sont simplement retournées sous la forme d’une collection au format : ’a’, ’b’, ’c’. L’utilisateur peut aussi limiter le nombre de sélections.

Il faut modifier le code KQL qui fait référence au paramètre pour le faire fonctionner avec le format du résultat. Le moyen le plus courant d’y parvenir consiste à utiliser l’opérateur `in`.

```kusto
dependencies
| where name in ({DependencyName})
| summarize Requests = count() by bin(timestamp, 1h), name
```

Voici un exemple de liste déroulante à sélection multiple :

![Image représentant un paramètre de liste déroulante à sélection multiple](./media/workbook-dropdowns/dropdown-multiselect.png)

## <a name="next-steps"></a>Étapes suivantes

* [Commencez](./workbooks-overview.md#visualizations) à en apprendre davantage sur les nombreuses options pour les visualisations enrichies des classeurs.
* [Contrôlez](workbooks-access-control.md) et partagez l’accès à vos ressources de classeur.