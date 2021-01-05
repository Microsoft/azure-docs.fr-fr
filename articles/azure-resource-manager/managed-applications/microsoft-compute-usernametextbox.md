---
title: Élément d’interface utilisateur UserNameTextBox
description: Décrit l’élément d’interface utilisateur Microsoft.Compute.UserNameTextBox pour le portail Azure. Permet aux utilisateurs de fournir des noms d’utilisateur Windows ou Linux.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: tomfitz
ms.openlocfilehash: 82478f322e1df22bde50769b90f0424140920e9a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87063571"
---
# <a name="microsoftcomputeusernametextbox-ui-element"></a>Élément d’interface utilisateur Microsoft.Compute.UserNameTextBox

Contrôle de zone de texte avec validation intégrée des noms d’utilisateur Windows et Linux.

## <a name="ui-sample"></a>Exemple d’interface utilisateur

![Microsoft.Compute.UserNameTextBox](./media/managed-application-elements/microsoft-compute-usernametextbox.png)

## <a name="schema"></a>schéma

```json
{
  "name": "element1",
  "type": "Microsoft.Compute.UserNameTextBox",
  "label": "User name",
  "defaultValue": "",
  "toolTip": "",
  "constraints": {
    "required": true,
    "regex": "^[a-z0-9A-Z]{1,30}$",
    "validationMessage": "Only alphanumeric characters are allowed, and the value must be 1-30 characters long."
  },
  "osPlatform": "Windows",
  "visible": true
}
```

## <a name="sample-output"></a>Exemple de sortie

```json
"Example name"
```

## <a name="remarks"></a>Notes

- Si `constraints.required` est défini sur **true**, la zone de texte doit contenir une valeur pour permettre la réussite de la validation. La valeur par défaut est **true**.
- `osPlatform` doit être spécifié. Il peut s’agir de **Windows** ou de **Linux**.
- `constraints.regex` est un modèle d’expression régulière JavaScript. S’il est spécifié, la valeur de la zone de texte doit correspondre au modèle pour permettre la réussite de la validation. La valeur par défaut est **null**.
- `constraints.validationMessage` est une chaîne à afficher en cas d’échec de la validation de la valeur de la zone de texte spécifiée par `constraints.regex`. Si elle n’est pas spécifiée, les messages de validation intégrés de la zone de texte sont utilisés. La valeur par défaut est **null**.
- Cet élément dispose d’une validation intégrée basée sur la valeur spécifiée pour `osPlatform`. La validation intégrée est utilisable avec une expression régulière personnalisée. Si une valeur est spécifiée pour `constraints.regex`, les validations intégrées et personnalisées sont déclenchées.

## <a name="next-steps"></a>Étapes suivantes

* Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
* Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).
