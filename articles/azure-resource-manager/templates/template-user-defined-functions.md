---
title: Fonctions définies par l’utilisateur dans les modèles
description: Explique comment définir et utiliser des fonctions définies par l'utilisateur dans un modèle Azure Resource Manager.
ms.topic: conceptual
ms.date: 03/09/2020
ms.openlocfilehash: 12199aca788438e4558ed2c607b866f0dc723e2a
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96436961"
---
# <a name="user-defined-functions-in-arm-template"></a>Fonctions définies par l'utilisateur dans un modèle Azure Resource Manager

Dans votre modèle, vous pouvez créer vos propres fonctions. Ces fonctions peuvent être utilisées dans votre modèle. Les fonctions définies par l’utilisateur sont distinctes des [fonctions de modèles standard](template-functions.md) qui sont automatiquement disponibles dans votre modèle. Créez vos propres fonctions quand vous avez des expressions complexes qui sont utilisées de manière répétitive dans votre modèle.

Cet article vous explique comment ajouter des fonctions définies par l'utilisateur dans votre modèle Azure Resource Manager.

## <a name="define-the-function"></a>Définir la fonction

Vos fonctions requièrent une valeur pour l’espace de noms afin d’éviter tout conflit avec les fonctions de modèle. L'exemple suivant illustre une fonction qui renvoie un nom unique :

```json
"functions": [
  {
    "namespace": "contoso",
    "members": {
      "uniqueName": {
        "parameters": [
          {
            "name": "namePrefix",
            "type": "string"
          }
        ],
        "output": {
          "type": "string",
          "value": "[concat(toLower(parameters('namePrefix')), uniqueString(resourceGroup().id))]"
        }
      }
    }
  }
],
```

## <a name="use-the-function"></a>Utiliser la fonction

L'exemple suivant illustre un modèle qui comprend une fonction définie par l'utilisateur. Il utilise cette fonction afin d'obtenir un nom unique pour un compte de stockage. Le modèle comprend un paramètre **storageNamePrefix** qu'il transmet à la fonction sous forme de paramètre.

```json
{
 "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
 "contentVersion": "1.0.0.0",
 "parameters": {
   "storageNamePrefix": {
     "type": "string",
     "maxLength": 11
   }
 },
 "functions": [
  {
    "namespace": "contoso",
    "members": {
      "uniqueName": {
        "parameters": [
          {
            "name": "namePrefix",
            "type": "string"
          }
        ],
        "output": {
          "type": "string",
          "value": "[concat(toLower(parameters('namePrefix')), uniqueString(resourceGroup().id))]"
        }
      }
    }
  }
],
 "resources": [
   {
     "type": "Microsoft.Storage/storageAccounts",
     "apiVersion": "2019-04-01",
     "name": "[contoso.uniqueName(parameters('storageNamePrefix'))]",
     "location": "South Central US",
     "sku": {
       "name": "Standard_LRS"
     },
     "kind": "StorageV2",
     "properties": {
       "supportsHttpsTrafficOnly": true
     }
   }
 ]
}
```

## <a name="limitations"></a>Limites

La définition d’une fonction utilisateur est soumise à certaines restrictions :

* La fonction ne peut pas accéder aux variables.
* La fonction ne peut utiliser que des paramètres définis dans l’autre fonction. Quand vous utilisez la fonction [parameters](template-functions-deployment.md#parameters) dans une fonction définie par l’utilisateur, vous êtes limité aux paramètres de cette fonction.
* La fonction ne peut pas appeler d’autres fonctions définies par l’utilisateur.
* La fonction ne peut pas utiliser la fonction [reference](template-functions-resource.md#reference) ni aucune des fonctions [list](template-functions-resource.md#list).
* Les paramètres de la fonction ne peuvent pas avoir de valeur par défaut.

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur les propriétés disponibles pour les fonctions définies par l'utilisateur, consultez [Comprendre la structure et la syntaxe des modèles Azure Resource Manager](template-syntax.md).
* Pour obtenir la liste des fonctions de modèle disponibles, consultez [Fonctions de modèle Azure Resource Manager](template-functions.md).
