---
title: Élément d’interface utilisateur TagsByResource
description: Décrit l’élément d’interface utilisateur Microsoft.Common.TagsByResource pour le portail Azure. Il permet d’appliquer des balises à une ressource pendant le déploiement.
author: tfitzmac
ms.topic: conceptual
ms.date: 11/11/2019
ms.author: tomfitz
ms.openlocfilehash: e730201812005a9b469a964e4acd081ebe86b100
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87063947"
---
# <a name="microsoftcommontagsbyresource-ui-element"></a>Élément d’interface utilisateur Microsoft.Common.TagsByResource

Contrôle permettant d’associer des [balises](../management/tag-resources.md) aux ressources d’un déploiement.

## <a name="ui-sample"></a>Exemple d’interface utilisateur

![Microsoft.Common.DropDown](./media/managed-application-elements/microsoft-common-tagsbyresource.png)

## <a name="schema"></a>schéma

```json
{
  "name": "element1",
  "type": "Microsoft.Common.TagsByResource",
  "resources": [
    "Microsoft.Storage/storageAccounts",
    "Microsoft.Compute/virtualMachines"
  ]
}
```

## <a name="sample-output"></a>Exemple de sortie

```json
{
  "Microsoft.Storage/storageAccounts": {
    "Dept": "Finance",
    "Environment": "Production"
  },
  "Microsoft.Compute/virtualMachines": {
    "Dept": "Finance"
  }
}
```

## <a name="remarks"></a>Notes

- Au moins un élément du tableau `resources` doit être spécifié.
- Chaque élément de `resources` doit être un type de ressource complet. Ces éléments apparaissent dans la liste déroulante **Ressources** et l’utilisateur peut y assigner des balises.
- La sortie du contrôle est mise en forme afin de faciliter l’attribution de valeurs de balise dans un modèle Azure Resource Manager. Pour recevoir la sortie du contrôle dans un modèle, ajoutez un paramètre dans votre modèle comme indiqué dans l’exemple suivant :

  ```json
  "parameters": {
    "tagsByResource": { "type": "object", "defaultValue": {} }
  }
  ```

  Pour chaque ressource pouvant être marquée par une balise, assignez la propriété de balises à la valeur de paramètre pour le type de ressource concerné :

  ```json
  {
    "name": "saName1",
    "type": "Microsoft.Storage/storageAccounts",
    "tags": "[ if(contains(parameters('tagsByResource'), 'Microsoft.Storage/storageAccounts'), parameters('tagsByResource')['Microsoft.Storage/storageAccounts'], json('{}')) ]",
    ...
  ```

- Utilisez la fonction [si](../templates/template-functions-logical.md#if) lorsque vous accédez au paramètre tagsByResource. Celle-ci vous permet d’assigner un objet vide lorsqu’aucune balise n’est assignée au type de ressource donné.

## <a name="next-steps"></a>Étapes suivantes

- Pour voir une présentation de la création de définitions d’interface utilisateur, consultez la page [Prise en main de CreateUiDefinition](create-uidefinition-overview.md).
- Pour obtenir une description des propriétés communes des éléments d’interface utilisateur, consultez la page [Éléments de CreateUiDefinition](create-uidefinition-elements.md).
