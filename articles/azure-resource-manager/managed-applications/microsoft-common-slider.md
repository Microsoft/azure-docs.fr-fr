---
title: Élément d’interface utilisateur Slider
description: Décrit l’élément d’interface utilisateur Microsoft.Common.Slider pour le portail Azure. Permet aux utilisateurs de définir une valeur à partir d’une plage d’options.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/10/2020
ms.author: tomfitz
ms.openlocfilehash: bb168a4bff6d2570f539307530232b5063bb535c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87094251"
---
# <a name="microsoftcommonslider-ui-element"></a>Élément d’interface utilisateur Microsoft.Common.Slider

Le contrôle Slider permet aux utilisateurs de sélectionner parmi une plage de valeurs autorisées.

## <a name="ui-sample"></a>Exemple d’interface utilisateur

:::image type="content" source="./media/managed-application-elements/microsoft-common-slider.png" alt-text="Microsoft.Common.Slider":::

## <a name="schema"></a>schéma

```json
{
    "name": "memorySize",
    "type": "Microsoft.Common.Slider",
    "min": 1,
    "max": 64,
    "label": "Memory",
    "subLabel": "MB",
    "defaultValue": 24,
    "showStepMarkers": false,
    "toolTip": "Pick the size in MB",
    "constraints": {
        "required": false
    },
    "visible": true
}
```

## <a name="sample-output"></a>Exemple de sortie

```json
26
```

## <a name="remarks"></a>Notes

- Les valeurs `min` et `max` sont requises. Elles définissent les points de début et de fin du curseur.
- La valeur par défaut de la propriété `showStepMarkers` est true. Les marqueurs de pas s’affichent uniquement lorsque la plage du minimum au maximum est inférieure ou égale à 100.


## <a name="next-steps"></a>Étapes suivantes

* Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
* Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).
