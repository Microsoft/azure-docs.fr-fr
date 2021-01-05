---
title: Intégration des ressources
description: En savoir plus sur l’intégration de ressources à l’aide de fournisseurs personnalisés Azure pour appliquer la gestion ou la configuration à d’autres types de ressources Azure.
author: jjbfour
ms.topic: conceptual
ms.date: 09/06/2019
ms.author: jobreen
ms.openlocfilehash: 1846b036f12fe7e691021ec0248782cad946d9b7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75649181"
---
# <a name="azure-custom-providers-resource-onboarding-overview"></a>Vue d’ensemble de l’intégration des ressources des fournisseurs personnalisés Azure

L’intégration des ressources des fournisseurs personnalisés Azure est un modèle d’extensibilité pour les types de ressources Azure. Elle vous permet d’appliquer des opérations ou une gestion à des ressources Azure existantes à l’échelle. Pour plus d’informations, consultez [Comment les fournisseurs personnalisés Azure peuvent étendre Azure](overview.md). Cet article aborde les points suivants :

- Ce que peut faire l’intégration des ressources.
- Les notions de base sur l’intégration des ressources et comment l’utiliser.
- Où trouver des guides et exemples de code pour commencer.

> [!IMPORTANT]
> Les fournisseurs personnalisés sont actuellement en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="what-can-resource-onboarding-do"></a>Que fait l’intégration des ressources ?

Comme pour les [ressources personnalisées des fournisseurs personnalisés Azure](./custom-providers-resources-endpoint-how-to.md), l’intégration des ressources définit un contrat qui redirige les demandes d’intégration vers un point de terminaison. Contrairement aux ressources personnalisées, l’intégration de ressources ne crée pas de nouveau type de ressource. Au lieu de cela, elle autorise l’extension de types de ressources existants. Et l’intégration de ressources fonctionne avec Azure Policy ; ainsi, la gestion et la configuration des ressources peuvent être effectuées à grande échelle. Voici quelques exemples de flux de travail d’intégration de ressources :

- Installer et gérer sur les extensions de machine virtuelle.
- Télécharger et configurer les valeurs par défaut sur les comptes de stockage Azure.
- Activer les paramètres de diagnostic de base à l’échelle.

## <a name="resource-onboarding-basics"></a>Notions de base de l’intégration des ressources

Vous configurez l’intégration des ressources via des fournisseurs personnalisés Azure à l’aide des types de ressources Microsoft.CustomProviders/resourceProviders et Microsoft.CustomProviders/associations. Pour activer l’intégration des ressources pour un fournisseur personnalisé, pendant le processus de configuration, créez un **resourceType** appelé « associations » avec un **routingType** incluant « extension ». Microsoft.CustomProviders/associations et Microsoft.CustomProviders/resourceProviders n’ont pas besoin d’appartenir au même groupe de ressources.

Voici un exemple de fournisseur personnalisé Azure :

```JSON
{
  "properties": {
    "resourceTypes": [
      {
        "name": "associations",
        "routingType": "Proxy,Cache,Extension",
        "endpoint": "https://microsoft.com/"
      }
    ]
  },
  "location": "eastus"
}
```

Propriété | Requis ? | Description
---|---|---
name | Oui | Nom de la définition de point de terminaison. Pour l’intégration de ressources, le nom doit être « associations ».
routingType | Oui | Détermine le type de contrat avec le point de terminaison. Pour l’intégration des ressources, les **routingTypes** acceptables sont « Proxy,Cache,Extension » et « Webhook,Cache,Extension ».
endpoint | Oui | Point de terminaison vers lequel router les requêtes. Il gère la réponse, ainsi que tous les effets secondaires de la requête.

Après avoir créé le fournisseur personnalisé avec le type de ressource associations, vous pouvez cibler avec Microsoft.CustomProviders/associations. Microsoft.CustomProviders/associations est une ressource d’extension qui peut étendre n’importe quelle autre ressource Azure. Lorsqu’une instance de Microsoft.CustomProviders/associations est créée, elle prend une propriété **targetResourceId**, qui doit être un ID de ressource Microsoft.CustomProviders/resourceProviders ou Microsoft.Solutions/applications valide. Dans de tels cas, la requête est transférée au type de ressource associations sur l’instance Microsoft.CustomProviders/resourceProviders que vous avez créée.

> [!NOTE]
> Si un ID de ressource Microsoft.Solutions/applications est fourni en tant que **targetResourceId**, il doit y avoir un Microsoft.CustomProviders/resourceProviders déployé dans le groupe de ressources géré portant le nom « public ».

Exemple d’association de fournisseurs personnalisés Azure :

```JSON
{
  "properties": {
    "targetResourceId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}",
    ...
  }
}
```

Propriété | Requis ? | Description
---|---|---
targetResourceId | Oui | L’ID de la ressource Microsoft.CustomProviders/resourceProviders ou Microsoft.Solutions/applications.

## <a name="how-to-use-resource-onboarding"></a>Utilisation de l’intégration des ressources

L’intégration de ressources fonctionne en étendant d’autres ressources avec la ressource d’extension Microsoft.CustomProviders/associations. Dans l’exemple suivant, la requête est effectuée pour une machine virtuelle, mais toutes les ressources peuvent être étendues.

Tout d’abord, vous devez créer une ressource de fournisseur personnalisée avec un type de ressource associations. Cela permet de déclarer l’URL de rappel qui sera utilisée lors de la création d’une ressource Microsoft.CustomProviders/associations correspondante, qui cible le fournisseur personnalisé.

Exemple de demande de création de Microsoft.CustomProviders/resourceProviders :

``` HTTP
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}?api-version=2018-09-01-preview
Authorization: Bearer eyJ0e...
Content-Type: application/json

{
  "properties": {
    "resourceTypes": [
      {
        "name": "associations",
        "routingType": "Proxy,Cache,Extension",
        "endpoint": "https://{myCustomEndpoint}/"
      }
    ]
  },
  "location": "{location}"
}
```

Après avoir créé le fournisseur personnalisé, vous pouvez cibler d’autres ressources et appliquer les effets secondaires du fournisseur personnalisé à ceux-ci.

Exemple de demande de création de Microsoft.CustomProviders/associations :

``` HTTP
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{virtualMachineName}/providers/Microsoft.CustomProviders/associations/{associationName}?api-version=2018-09-01-preview
Authorization: Bearer eyJ0e...
Content-Type: application/json

{
  "properties": {
    "targetResourceId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}",
    "myProperty1": "myPropertyValue1",
    "myProperty2": {
        "myProperty3" : "myPropertyValue3"
    }
  }
}
```

Cette demande est ensuite transférée au point de terminaison spécifié dans le fournisseur personnalisé que vous avez créé, qui est référencé par le **targetResourceId** sous la forme suivante :

``` HTTP
PUT https://{endpointURL}/?api-version=2018-09-01-preview
Content-Type: application/json
X-MS-CustomProviders-RequestPath: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/associations/{associationName}
X-MS-CustomProviders-ExtensionPath: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{virtualMachineName}/providers/Microsoft.CustomProviders/associations/{associationName}
X-MS-CustomProviders-ExtendedResource: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{virtualMachineName}

{
  "properties": {
    "myProperty1": "myPropertyValue1",
    "myProperty2": {
        "myProperty3" : "myPropertyValue3"
    }
  }
}
```

Le point de terminaison doit répondre avec le `Content-Type` application/json et un corps de réponse JSON valide. Les champs qui sont renvoyés sous l’objet **properties** du JSON sont ajoutés à la réponse de retour de l’association.

## <a name="getting-help"></a>Obtenir de l’aide

Si vous avez des questions sur le développement de fournisseurs de ressources personnalisés Azure, postez-les sur [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-custom-providers). Avant de publier une question, vérifiez si une question similaire a déjà reçu une réponse. Ajoutez le mot clé ```azure-custom-providers``` pour obtenir une réponse rapide !

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert les fournisseurs personnalisés. Pour en savoir plus, consultez les articles suivants :

- [Tutoriel : Intégration des ressources avec des fournisseurs personnalisés](./tutorial-resource-onboarding.md)
- [Tutoriel : Créer des actions et des ressources personnalisées dans Azure](./tutorial-get-started-with-custom-providers.md)
- [Démarrage rapide : Créer un fournisseur de ressources personnalisées et déployer des ressources personnalisées](./create-custom-provider.md)
- [Procédure : ajouter des actions personnalisées à l’API REST Azure](./custom-providers-action-endpoint-how-to.md)
- [Procédure : ajouter des ressources personnalisées à l’API REST Azure](./custom-providers-resources-endpoint-how-to.md)
