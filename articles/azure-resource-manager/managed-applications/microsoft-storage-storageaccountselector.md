---
title: Élément d'interface utilisateur StorageAccountSelector
description: Décrit l’élément d’interface utilisateur Microsoft.Storage.StorageAccountSelector pour le Portail Azure.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/28/2018
ms.author: tomfitz
ms.openlocfilehash: fe70c42387e9dedea8943b5dcce04a1f9ded5190
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87033283"
---
# <a name="microsoftstoragestorageaccountselector-ui-element"></a>Élément d’interface utilisateur Microsoft.Storage.StorageAccountSelector

Contrôle permettant de sélectionner un compte de stockage nouveau ou existant.

## <a name="ui-sample"></a>Exemple d’interface utilisateur

Le contrôle affiche la valeur par défaut.

![Microsoft.Storage.StorageAccountSelector](./media/managed-application-elements/microsoft-storage-storageaccountselector.png)

Le contrôle permet à l’utilisateur de créer un nouveau compte de stockage ou de sélectionner un compte de stockage existant.

![Microsoft.Storage.StorageAccountSelector nouveau](./media/managed-application-elements/microsoft-storage-storageaccountselector-new.png)

## <a name="schema"></a>schéma

```json
{
  "name": "element1",
  "type": "Microsoft.Storage.StorageAccountSelector",
  "label": "Storage account",
  "toolTip": "",
  "defaultValue": {
    "name": "storageaccount01",
    "type": "Premium_LRS"
  },
  "constraints": {
    "allowedTypes": [],
    "excludedTypes": []
  },
  "options": {
    "hideExisting": false
  },
  "visible": true
}
```

## <a name="sample-output"></a>Exemple de sortie

```json
{
  "name": "storageaccount01",
  "resourceGroup": "rg01",
  "type": "Premium_LRS",
  "newOrExisting": "new"
}
```

## <a name="remarks"></a>Notes

- S’il est spécifié, le caractère unique de `defaultValue.name` est automatiquement validé. Si le nom de compte de stockage n’est pas unique, l’utilisateur doit indiquer un autre nom ou choisir un compte de stockage existant.
- La valeur par défaut pour `defaultValue.type` est **Premium_LRS**.
- Tout type non spécifié dans `constraints.allowedTypes` est masqué et tout type non spécifié dans `constraints.excludedTypes` s’affiche. `constraints.allowedTypes` et `constraints.excludedTypes` sont tous deux facultatifs, mais ne peuvent pas être utilisés simultanément.
- Si `options.hideExisting` est défini sur **true**, l’utilisateur ne peut pas choisir de compte de stockage existant. La valeur par défaut est **false**.

## <a name="next-steps"></a>Étapes suivantes
* Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
* Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).
