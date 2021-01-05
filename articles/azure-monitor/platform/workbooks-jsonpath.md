---
title: Classeurs Azure Monitor - Transformer des données JSON avec JSONPath
description: Comment utiliser JSONPath dans les classeurs Azure Monitor pour transformer les résultats des données JSON retournées par un point de terminaison interrogé au format souhaité.
services: azure-monitor
author: lgayhardt
manager: carmonm
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 05/06/2020
ms.author: lagayhar
ms.openlocfilehash: a2411d9257b1083cb2bcbfcad289813a6c062dff
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92143588"
---
# <a name="how-to-use-jsonpath-to-transform-json-data-in-workbooks"></a>Comment utiliser JSONPath pour transformer des données JSON dans des classeurs

Les classeurs sont en mesure d’interroger les données de nombreuses sources. Certains points de terminaison, tels que [Azure Resource Manager](../../azure-resource-manager/management/overview.md) ou un point de terminaison personnalisé, peuvent retourner des résultats au format JSON. Si les données JSON retournées par le point de terminaison interrogé ne sont pas configurées dans un format souhaité, JSONPath peut être utilisé pour transformer les résultats.

JSONPath est un langage de requête pour JSON qui est similaire à XPath pour XML. Comme XPath, JSONPath permet l’extraction et la filtration de données à partir de la structure JSON.

À l’aide de la transformation JSONPath, les auteurs de classeurs peuvent convertir JSON en une structure de table. La table peut ensuite être utilisée pour tracer des [visualisations de classeur](./workbooks-overview.md#visualizations).

## <a name="using-jsonpath"></a>Utilisation de JSONPath

1. Basculez le classeur en mode d’édition en cliquant sur l’élément *Modifier* de la barre d’outils.
2. Utilisez le lien *Ajouter* > *Ajouter une requête* pour ajouter un contrôle de requête au classeur.
3. Sélectionnez le type de source de données en tant que *JSON*.
4. Utiliser l’éditeur JSON pour entrer l’extrait de code JSON suivant
    ```json
    { "store": {
        "books": [ 
          { "category": "reference",
            "author": "Nigel Rees",
            "title": "Sayings of the Century",
            "price": 8.95
          },
          { "category": "fiction",
            "author": "Evelyn Waugh",
            "title": "Sword of Honour",
            "price": 12.99
          },
          { "category": "fiction",
            "author": "Herman Melville",
            "title": "Moby Dick",
            "isbn": "0-553-21311-3",
            "price": 8.99
          },
          { "category": "fiction",
            "author": "J. R. R. Tolkien",
            "title": "The Lord of the Rings",
            "isbn": "0-395-19395-8",
            "price": 22.99
          }
        ],
        "bicycle": {
          "color": "red",
          "price": 19.95
        }
      }
    }
    ```  

Supposons que nous avons reçu l’objet JSON ci-dessus en tant que représentation de l’inventaire d’un magasin. Notre tâche consiste à créer une table des livres disponibles dans le magasin en répertoriant leurs titres, leurs auteurs et leurs prix.

1. Sélectionnez l’onglet *Paramètres du résultat* et basculez le format des résultats sur *Chemin JSON*.
2. Appliquez les paramètres de chemin d’accès JSON suivants :

    Table de chemins JSON : `$.store.books`. Ce champ représente le chemin d’accès de la racine de la table. Dans ce cas, nous nous soucions de l’inventaire des livres du magasin. Le chemin d’accès de la table filtre le fichier JSON avec les informations du livre.

   | ID de colonne | Chemin JSON de colonne |
   |:-----------|:-----------------|
   | Intitulé      | `$.title`        |
   | Auteur     | `$.author`       |
   | Price      | `$.price`        |

    Les ID de colonne seront les en-têtes de colonnes. Les champs de chemins d’accès de colonne JSON représentent le chemin d’accès entre la racine de la table et la valeur de colonne.

1. Appliquer les paramètres ci-dessus en cliquant sur *Exécuter la requête*

![ Modification de l’élément de requête avec la source de données JSON et le format de résultat du chemin JSON](./media/workbooks-jsonpath/query-jsonpath.png)

## <a name="next-steps"></a>Étapes suivantes
- [Présentation des classeurs](workbooks-overview.md)
- [Groupes dans les classeurs Azure Monitor](workbooks-groups.md)