---
title: Élément d’interface utilisateur OptionsGroup
description: Décrit l’élément d’interface utilisateur Microsoft.Common.OptionsGroup pour le portail Azure. Permet aux utilisateurs de sélectionner les options disponibles lors du déploiement d’une application managée.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/09/2020
ms.author: tomfitz
ms.openlocfilehash: aa73b4cbded98291a14792a7151df9fdfb885b53
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87004186"
---
# <a name="microsoftcommonoptionsgroup-ui-element"></a>Élément d’interface utilisateur Microsoft.Common.OptionsGroup

Le contrôle OptionsGroup permet aux utilisateurs de sélectionner une option parmi au moins deux choix. Un utilisateur ne peut sélectionner qu’une seule option.

> [!NOTE]
> Dans le passé, ce contrôle affichait les options horizontalement. À présent, le contrôle présente les options verticalement sous forme de boutons radio.

## <a name="ui-sample"></a>Exemple d’interface utilisateur

:::image type="content" source="./media/managed-application-elements/microsoft-common-optionsgroup-2.png" alt-text="Microsoft.Common.OptionsGroup":::

## <a name="schema"></a>schéma

```json
{
  "name": "element1",
  "type": "Microsoft.Common.OptionsGroup",
  "label": "Some options group",
  "defaultValue": "Value two",
  "toolTip": "",
  "constraints": {
    "allowedValues": [
      {
        "label": "Value one",
        "value": "one"
      },
      {
        "label": "Value two",
        "value": "two"
      }
    ],
    "required": true
  },
  "visible": true
}
```

## <a name="sample-output"></a>Exemple de sortie

```json
"two"
```

## <a name="remarks"></a>Notes

- L’étiquette de `constraints.allowedValues` est le texte qui s’affiche pour un élément, et sa valeur est la valeur de sortie de l’élément sélectionné lors de la sélection.
- Si elle est spécifiée, la valeur par défaut doit être une étiquette présente dans `constraints.allowedValues`. Dans le cas contraire, le premier élément de `constraints.allowedValues` est sélectionné par défaut. La valeur par défaut est **null**.
- `constraints.allowedValues` doit avoir au moins un élément.

## <a name="next-steps"></a>Étapes suivantes

* Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
* Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).
