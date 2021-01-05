---
title: Élément d’interface utilisateur ResourceSelector
description: Décrit l’élément d’interface utilisateur Microsoft.Solutions.ResourceSelector pour le portail Azure. Utilisé pour obtenir une liste des ressources existantes.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: d4b21f092733f6ebb3c2bf5aa06c1d2782323730
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87094338"
---
# <a name="microsoftsolutionsresourceselector-ui-element"></a>Élément d’interface utilisateur Microsoft.Solutions.ResourceSelector

ResourceSelector permet aux utilisateurs de sélectionner une ressource existante dans un abonnement.

## <a name="ui-sample"></a>Exemple d’interface utilisateur

![Microsoft.Solutions.ResourceSelector](./media/managed-application-elements/microsoft-solutions-resourceselector.png)

## <a name="schema"></a>schéma

```json
{
    "name": "storageSelector",
    "type": "Microsoft.Solutions.ResourceSelector",
    "label": "Select storage accounts",
    "resourceType": "Microsoft.Storage/storageAccounts",
    "options": {
        "filter": {
            "subscription": "onBasics",
            "location": "onBasics"
        }
    }
}
```

## <a name="sample-output"></a>Exemple de sortie

```json
"name": "{resource-name}",
"id": "/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/{resource-provider-namespace}/{resource-type}/{resource-name}",
"location": "{deployed-location}"
```

## <a name="remarks"></a>Notes

Dans la propriété `resourceType`, spécifiez l’espace de noms du fournisseur de ressources et le nom du type de ressource pour la ressource que vous souhaitez afficher dans la liste.

La propriété `filter` limite les options disponibles pour les ressources. Vous pouvez limiter les résultats par localisation ou par abonnement. Pour montrer uniquement les ressources qui correspondent à la sélection dans l’étape Principes de base, utilisez `onBasics`. Pour montrer toutes les ressources, utilisez `all`. La valeur par défaut est `all`.

## <a name="next-steps"></a>Étapes suivantes

* Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
* Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).
