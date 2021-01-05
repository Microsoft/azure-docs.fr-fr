---
title: Élément d’interface utilisateur IdentitySelector
description: Décrit l’élément d’interface utilisateur Microsoft.ManagedIdentity.IdentitySelector pour le portail Azure. À utiliser pour attribuer des identités managées à une ressource.
author: tfitzmac
ms.topic: conceptual
ms.date: 02/06/2020
ms.author: tomfitz
ms.openlocfilehash: 91e3cd0b99825fd72eb342ce7a8555b046455538
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87063358"
---
# <a name="microsoftmanagedidentityidentityselector-ui-element"></a>Élément d’interface utilisateur Microsoft.ManagedIdentity.IdentitySelector

Contrôle permettant d’attribuer des [identités managées](../../active-directory/managed-identities-azure-resources/overview.md) pour une ressource dans un déploiement.

## <a name="ui-sample"></a>Exemple d’interface utilisateur

Cette règle se compose des éléments suivants :

![Première étape de Microsoft.ManagedIdentity.IdentitySelector](./media/managed-application-elements/microsoft-managedidentity-identityselector-1.png)

Quand l’utilisateur sélectionne **Ajouter**, le formulaire suivant s’ouvre. L’utilisateur peut sélectionner une ou plusieurs identités attribuées par l’utilisateur pour la ressource.

![Deuxième étape de Microsoft.ManagedIdentity.IdentitySelector](./media/managed-application-elements/microsoft-managedidentity-identityselector-2.png)

Les identités sélectionnées sont affichées dans le tableau. L’utilisateur peut ajouter ou supprimer des éléments de ce tableau.

![Troisième étape de Microsoft.ManagedIdentity.IdentitySelector](./media/managed-application-elements/microsoft-managedidentity-identityselector-3.png)

## <a name="schema"></a>schéma

```json
{
  "name": "identity",
  "type": "Microsoft.ManagedIdentity.IdentitySelector",
  "label": "Managed Identity Configuration",
  "toolTip": {
    "systemAssignedIdentity": "Enable system assigned identity to grant the resource access to other existing resources.",
    "userAssignedIdentity": "Add user assigned identities to grant the resource access to other existing resources."
  },
  "defaultValue": {
    "systemAssignedIdentity": "Off"
  },
  "options": {
    "hideSystemAssignedIdentity": false,
    "hideUserAssignedIdentity": false
  },
  "visible": true
}
```

## <a name="sample-output"></a>Exemple de sortie

```json
{
  "identity": {
    "value": {
      "type": "UserAssigned",
      "userAssignedIdentities": {
        "/subscriptions/xxxx/resourceGroups/TestResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/TestUserIdentity1": {}
      }
    }
  }
}
```

## <a name="remarks"></a>Notes

- Utilisez **defaultValue.systemAssignedIdentity** pour définir une valeur initiale pour le contrôle des options d’identité attribuées par le système. La valeur par défaut est **Off**. Les valeurs autorisées sont les suivantes :
  - **On** : une identité attribuée par le système est attribuée à la ressource.
  - **Off** : une identité attribuée par le système n’est pas attribuée à la ressource.
  - **OnOnly** : une identité attribuée par le système est attribuée à la ressource. Les utilisateurs ne peuvent pas modifier cette valeur lors du déploiement.
  - **OffOnly** : une identité attribuée par le système n’est pas attribuée à la ressource. Les utilisateurs ne peuvent pas modifier cette valeur lors du déploiement.

- Si **options.hideSystemAssignedIdentity** a la valeur **true**, l’interface utilisateur permettant de configurer l’identité attribuée par le système n’est pas affichée. La valeur par défaut de cette option est **false**.
- Si **options.hideSUserAssignedIdentity** a la valeur **true**, l’interface utilisateur permettant de configurer l’identité attribuée par l’utilisateur n’est pas affichée. Une identité attribuée par l’utilisateur n’est pas attribuée à la ressource. La valeur par défaut de cette option est **false**.

## <a name="next-steps"></a>Étapes suivantes

- Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
- Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).