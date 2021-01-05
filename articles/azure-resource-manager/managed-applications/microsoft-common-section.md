---
title: Élément d’interface utilisateur Section
description: Décrit l’élément d’interface utilisateur Microsoft.Common.Section pour le portail Azure. Permet de regrouper des éléments dans le portail pour le déploiement d’applications managées.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: tomfitz
ms.openlocfilehash: 924aff8f2ba3d796b65f52494845f3b10018065c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87063985"
---
# <a name="microsoftcommonsection-ui-element"></a>Élément d’interface utilisateur Microsoft.Common.Section

Contrôle qui regroupe un ou plusieurs éléments sous un titre.

## <a name="ui-sample"></a>Exemple d’interface utilisateur

![Microsoft.Common.Section](./media/managed-application-elements/microsoft-common-section.png)

## <a name="schema"></a>schéma

```json
{
  "name": "section1",
  "type": "Microsoft.Common.Section",
  "label": "Example section",
  "elements": [
    {
      "name": "text1",
      "type": "Microsoft.Common.TextBox",
      "label": "Example text box 1"
    },
    {
      "name": "text2",
      "type": "Microsoft.Common.TextBox",
      "label": "Example text box 2"
    }
  ],
  "visible": true
}
```

## <a name="remarks"></a>Notes

- `elements` doit avoir au moins un élément et peut avoir tous les types d’éléments à l’exception de `Microsoft.Common.Section`.
- Cet élément ne prend pas en charge la propriété `toolTip`.

## <a name="sample-output"></a>Exemple de sortie
Pour accéder aux valeurs de sortie des éléments de `elements`, utilisez les fonctions [basics()](create-ui-definition-referencing-functions.md#basics) ou [steps()](create-ui-definition-referencing-functions.md#steps) et la notation sous forme de points :

```json
steps('configuration').section1.text1
```

Les éléments de type `Microsoft.Common.Section` n’ont eux-mêmes aucune valeur de sortie.

## <a name="next-steps"></a>Étapes suivantes

* Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
* Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).
