---
title: Créer des fonctions de référencement de définition de l’interface utilisateur
description: Décrit les fonctions à utiliser pour la construction de définitions d’interface utilisateur pour le portail Azure, référençant d’autres objets.
author: tfitzmac
ms.topic: conceptual
ms.date: 07/13/2020
ms.author: tomfitz
ms.openlocfilehash: ad21c5b34a58c35b2cef5e430be7cb8cd1296402
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87094187"
---
# <a name="createuidefinition-referencing-functions"></a>Fonctions de référencement CreateUiDefinition

Fonctions à utiliser pour référencer les sorties à partir des propriétés ou du contexte d’un fichier CreateUiDefinition

## <a name="basics"></a>basics

Retourne les valeurs de sortie d’un élément qui est défini dans l’étape [Principes de base](create-uidefinition-overview.md#basics). Spécifiez le nom de l’élément comme paramètre de la fonction.

Pour récupérer les valeurs de sortie des éléments dans d’autres étapes, utilisez la fonction [steps()](#steps).

L’exemple suivant retourne la sortie de l’élément nommé `clusterName` dans l’étape relative aux principes de base :

```json
"[basics('clusterName')]"
```

Les valeurs retournées varient en fonction du type d’élément récupéré.

## <a name="location"></a>location

Retourne l’emplacement sélectionné dans l’étape relative aux principes de base ou le contexte actuel.

L’exemple suivant retourne une valeur telle que `"westus"` :

```json
"[location()]"
```

## <a name="resourcegroup"></a>resourceGroup

Retourne des informations détaillées sur le resourceGroup sélectionné dans l’étape Principes de base ou le contexte actuel.

L’exemple suivant :

```json
"[resourceGroup()]"
```

Retourne les propriétés suivantes :

```json
{
    "mode": "New" or "Existing",
    "name": "{resourceGroupName}",
    "location": "{resourceGroupLocation}"
}
```

Vous pouvez obtenir une valeur particulière avec la notation par points.

```json
"[resourceGroup().name]"
```

## <a name="steps"></a>steps

Retourne les éléments relatifs à une étape spécifiée. Spécifiez le nom de l’étape comme paramètre de la fonction. À partir des éléments retournés, vous pouvez obtenir des valeurs de propriété particulières.

Pour obtenir les valeurs de sortie des éléments dans l’étape Principes de base, utilisez la fonction [basics()](#basics).

L’exemple suivant retourne l’étape nommée `vmParameters`, incluant l’élément nommé `adminUsername`.

```json
"[steps('vmParameters').adminUsername]"
```

## <a name="subscription"></a>subscription

Retourne les propriétés de l’abonnement sélectionné dans l’étape Principes de base ou le contexte actuel.

L’exemple suivant :

```json
"[subscription()]"
```

Retourne les propriétés suivantes :

```json
{
    "id": "/subscriptions/{subscription-id}",
    "subscriptionId": "{subscription-id}",
    "tenantId": "{tenant-id}",
    "displayName": "{name-of-subscription}"
}
```

## <a name="next-steps"></a>Étapes suivantes

* Pour une introduction au développement de l’interface du portail, consultez [CreateUiDefinition.json pour une expérience de création d’applications managées Azure](create-uidefinition-overview.md).
