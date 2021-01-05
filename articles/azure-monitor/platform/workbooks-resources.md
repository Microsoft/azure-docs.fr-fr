---
title: Paramètres de ressources des classeurs Azure Monitor
description: Découvrez comment utiliser les paramètres de ressources pour choisir des ressources dans les classeurs. Utilisez les paramètres des ressources pour définir l’étendue à partir de laquelle les données sont extraites.
services: azure-monitor
manager: carmonm
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 10/23/2019
ms.openlocfilehash: 27e98ae51cbea654ca0f06979b289a497a5aedf3
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92143514"
---
# <a name="workbook-resource-parameters"></a>Paramètres de ressources des classeurs

Les paramètres de ressources permettent de choisir des ressources dans les classeurs, ce qui est utile pour définir dans quelle étendue les données sont extraites. Il peut s’agir par exemple de permettre aux utilisateurs de sélectionner l’ensemble de machines virtuelles qui sera utilisé par les graphiques pour la présentation des données.

La valeur des sélecteurs de ressources peut provenir du contexte du classeur, d’une liste statique ou de requêtes Azure Resource Graph.

## <a name="creating-a-resource-parameter-workbook-resources"></a>Créer un paramètre de ressource (ressources de classeur)
1. Démarrez avec un classeur vide en mode d’édition.
2. Choisissez _Ajouter des paramètres_ dans les liens du classeur.
3. Cliquez sur le bouton bleu _Ajouter un paramètre_.
4. Dans le volet Nouveau paramètre qui apparaît, entrez :
    1. Nom du paramètre : `Applications`
    2. Type de paramètre : `Resource picker`
    3. Obligatoire : `checked`
    4. Autoriser les sélections multiples : `checked`
5. Récupérer des données à partir de : `Workbook Resources`
6. Inclure uniquement les types de ressources : `Application Insights`
7. Choisissez « Enregistrer » dans la barre d’outils pour créer le paramètre.

![Image représentant la création d’un paramètre de ressource à l’aide des ressources de classeur](./media/workbooks-resources/resource-create.png)

## <a name="creating-a-resource-parameter-azure-resource-graph"></a>Créer un paramètre de ressource (Azure Resource Graph)
1. Démarrez avec un classeur vide en mode d’édition.
2. Choisissez _Ajouter des paramètres_ dans les liens du classeur.
3. Cliquez sur le bouton bleu _Ajouter un paramètre_.
4. Dans le volet Nouveau paramètre qui apparaît, entrez :
    1. Nom du paramètre : `Applications`
    2. Type de paramètre : `Resource picker`
    3. Obligatoire : `checked`
    4. Autoriser les sélections multiples : `checked`
5. Récupérer des données à partir de : `Query`
    1. Type de requête : `Azure Resource Graph`
    2. Abonnements : `Use default subscriptions`
    3. Dans le contrôle de requête, ajoutez cet extrait de code :
    ```kusto
    where type == 'microsoft.insights/components'
    | project value = id, label = name, selected = false, group = resourceGroup
    ```
7. Choisissez « Enregistrer » dans la barre d’outils pour créer le paramètre.

![Image représentant la création d’un paramètre de ressource à l’aide d’Azure Resource Graph](./media/workbooks-resources/resource-query.png)

> [!NOTE]
> Azure Resource Graph n’est pas encore disponible dans tous les clouds. Si vous choisissez cette approche, vérifiez qu’elle est prise en charge dans votre cloud cible.

[Documentation Azure Resource Graph](../../governance/resource-graph/overview.md)

## <a name="creating-a-resource-parameter--json-list"></a>Créer un paramètre de ressource (liste JSON)
1. Démarrez avec un classeur vide en mode d’édition.
2. Choisissez _Ajouter des paramètres_ dans les liens du classeur.
3. Cliquez sur le bouton bleu _Ajouter un paramètre_.
4. Dans le volet Nouveau paramètre qui apparaît, entrez :
    1. Nom du paramètre : `Applications`
    2. Type de paramètre : `Resource picker`
    3. Obligatoire : `checked`
    4. Autoriser les sélections multiples : `checked`
5. Récupérer des données à partir de : `JSON`
    1. Dans le contrôle de contenu, ajoutez cet extrait de code JSON :
    ```json
    [
        { "value":"/subscriptions/<sub-id>/resourceGroups/<resource-group>/providers/<resource-type>/acmeauthentication", "label": "acmeauthentication", "selected":true, "group":"Acme Backend" },
        { "value":"/subscriptions/<sub-id>/resourceGroups/<resource-group>/providers/<resource-type>/acmeweb", "label": "acmeweb", "selected":false, "group":"Acme Frontend" }
    ]
    ```
    2. Appuyez sur le bouton bleu _Mettre à jour_.
6. Si vous le souhaitez, définissez `Include only resource types` sur _Application Insights_.
7. Choisissez « Enregistrer » dans la barre d’outils pour créer le paramètre.

## <a name="referencing-a-resource-parameter"></a>Faire référence à un paramètre de ressource
1. Ajoutez un contrôle de requête au classeur et sélectionnez une ressource Application Insights.
2. Utilisez la liste déroulante _Application Insights_ pour lier le paramètre au contrôle. Cela permet de définir l’étendue de la requête sur les ressources retournées par le paramètre à l’exécution.
4. Dans le contrôle KQL, ajoutez cet extrait de code :
    ```kusto
    requests
    | summarize Requests = count() by appName, name
    | order by Requests desc
    ```
5. Exécutez la requête pour afficher les résultats. 

![Image représentant une référence à un paramètre de ressource dans un contrôle de requête](./media/workbooks-resources/resource-reference.png)

> Cette approche peut permettre de lier des ressources à d’autres contrôles, comme des métriques.

## <a name="resource-parameter-options"></a>Options des paramètres de ressources
| Paramètre | Explication | Exemple |
| ------------- |:-------------|:-------------|
| `{Applications}` | ID de la ressource sélectionnée | _/subscriptions/<ID-abonnement>/resourceGroups/<groupe-ressources>/providers/<type-ressource>/acmeauthentication_ |
| `{Applications:label}` | Étiquette de la ressource sélectionnée | `acmefrontend` |
| `{Applications:value}` | Valeur de la ressource sélectionnée | _’/subscriptions/<ID-abonnement>/resourceGroups/<groupe-ressources>/providers/<type-ressource>/acmeauthentication’_ |
| `{Applications:name}` | Nom de la ressource sélectionnée | `acmefrontend` |
| `{Applications:resourceGroup}` | Groupe de ressources de la ressource sélectionnée | `acmegroup` |
| `{Applications:resourceType}` | Type de la ressource sélectionnée | _microsoft.insights/components_ |
| `{Applications:subscription}` | Abonnement de la ressource sélectionnée |  |
| `{Applications:grid}` | Grille présentant les propriétés de la ressource, utile pour le rendu dans un bloc de texte lors du débogage  |  |

## <a name="next-steps"></a>Étapes suivantes

* [Commencez](./workbooks-overview.md#visualizations) à en apprendre davantage sur les nombreuses options pour les visualisations enrichies des classeurs.
* [Contrôlez](workbooks-access-control.md) et partagez l’accès à vos ressources de classeur.