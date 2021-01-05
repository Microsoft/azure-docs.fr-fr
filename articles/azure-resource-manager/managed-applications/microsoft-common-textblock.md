---
title: Élément d’interface utilisateur TextBlock
description: Décrit l’élément d’interface utilisateur Microsoft.Common.TextBlock pour le portail Azure. Permet d’ajouter du texte à l’interface.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/27/2018
ms.author: tomfitz
ms.openlocfilehash: ba2c4d410891910c29ee1fda1065f8230ab171bf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87063862"
---
# <a name="microsoftcommontextblock-ui-element"></a>Élément d’interface utilisateur Microsoft.Common.TextBlock

Contrôle qui peut être utilisé pour ajouter du texte à l’interface du portail.

## <a name="ui-sample"></a>Exemple d’interface utilisateur

![Microsoft.Common.TextBox](./media/managed-application-elements/microsoft-common-textblock.png)

## <a name="schema"></a>schéma

```json
{
  "name": "text1",
  "type": "Microsoft.Common.TextBlock",
  "visible": true,
  "options": {
    "text": "Please provide the configuration values for your application.",
    "link": {
      "label": "Learn more",
      "uri": "https://www.microsoft.com"
    }
  }
}
```

## <a name="sample-output"></a>Exemple de sortie

```json
"Please provide the configuration values for your application. Learn more"
```

## <a name="next-steps"></a>Étapes suivantes

* Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
* Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).
