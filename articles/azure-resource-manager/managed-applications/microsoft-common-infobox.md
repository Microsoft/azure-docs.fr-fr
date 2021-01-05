---
title: Élément d’interface utilisateur InfoBox
description: Décrit l’élément d’interface utilisateur Microsoft.Common.InfoBox pour le Portail Azure. Permet d’ajouter du texte ou des avertissements lors du déploiement d’une application managée.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/15/2018
ms.author: tomfitz
ms.openlocfilehash: 7580ac0650706d6aee49bbf0e8235e8c5dab33f4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87033346"
---
# <a name="microsoftcommoninfobox-ui-element"></a>Élément d’interface utilisateur Microsoft.Common.InfoBox

Contrôle qui ajoute une zone d’information. La zone contient du texte important ou des avertissements qui aident les utilisateurs à comprendre les valeurs qu’ils fournissent. Elle peut également comporter un lien vers une URI proposant des informations supplémentaires.

## <a name="ui-sample"></a>Exemple d’interface utilisateur

![Microsoft.Common.InfoBox](./media/managed-application-elements/microsoft-common-infobox.png)


## <a name="schema"></a>schéma

```json
{
  "name": "text1",
  "type": "Microsoft.Common.InfoBox",
  "visible": true,
  "options": {
    "icon": "None",
    "text": "Nullam eros mi, mollis in sollicitudin non, tincidunt sed enim. Sed et felis metus, rhoncus ornare nibh. Ut at magna leo.",
    "uri": "https://www.microsoft.com"
  }
}
```

## <a name="sample-output"></a>Exemple de sortie

```json
"Nullam eros mi, mollis in sollicitudin non, tincidunt sed enim. Sed et felis metus, rhoncus ornare nibh. Ut at magna leo."
```

## <a name="remarks"></a>Notes

* Pour `icon`, utilisez **Aucune**, **Info**, **Avertissement** ou **Erreur**.
* La propriété `uri` est facultative.

## <a name="next-steps"></a>Étapes suivantes

* Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
* Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).
