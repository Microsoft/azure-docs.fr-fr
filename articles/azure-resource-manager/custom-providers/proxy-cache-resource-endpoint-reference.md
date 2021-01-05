---
title: Informations de référence sur les caches de ressources personnalisées
description: Informations de référence sur le cache de ressources personnalisées des fournisseurs de ressources personnalisés Azure. Cet article passera par la configuration requise pour les points de terminaison mettant en œuvre des caches de ressources personnalisées.
ms.topic: conceptual
ms.author: jobreen
author: jjbfour
ms.date: 06/20/2019
ms.openlocfilehash: e1b8c44f020d18066423eed236018308fe88b607
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75649173"
---
# <a name="custom-resource-cache-reference"></a>Informations de référence sur les caches de ressources personnalisées

Cet article passera par la configuration requise pour les points de terminaison mettant en œuvre des caches de ressources personnalisées. Si vous n’êtes pas familiarisé avec les fournisseurs de ressources personnalisées Azure, consultez [la vue d’ensemble des fournisseurs de ressources personnalisées](overview.md).

## <a name="how-to-define-a-cache-resource-endpoint"></a>Comment définir un point de terminaison de cache de ressource

Vous pouvez créer une ressource de proxy en associant la valeur « Proxy, Cache » au paramètre **routingType**.

Exemple de fournisseur de ressources personnalisées :

```JSON
{
  "properties": {
    "resourceTypes": [
      {
        "name": "myCustomResources",
        "routingType": "Proxy, Cache",
        "endpoint": "https://{endpointURL}/"
      }
    ]
  },
  "location": "eastus",
  "type": "Microsoft.CustomProviders/resourceProviders",
  "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}",
  "name": "{resourceProviderName}"
}
```

## <a name="building-proxy-resource-endpoint"></a>création d’un point de terminaison de ressource proxy

Un **point de terminaison** qui implémente un **point de terminaison** de type « Proxy, Cache » doit gérer la requête et la réponse de la nouvelle API dans Azure. Dans ce cas, le paramètre **resourceType** génère une nouvelle API de ressource Azure pour `PUT`, `GET` et `DELETE` afin d’exécuter CRUD sur une ressource unique, ainsi que `GET` pour récupérer toutes les ressources existantes :

> [!NOTE]
> L’API Azure génère les méthodes de requête `PUT`, `GET`, et `DELETE`, mais le **point de terminaison** du cache doit uniquement gérer `PUT` et `DELETE`.
> Nous recommandons que le **point de terminaison** implémente également `GET`.

### <a name="create-a-custom-resource"></a>Créer une ressource personnalisée

Requête entrante de l’API Azure :

``` HTTP
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}?api-version=2018-09-01-preview
Authorization: Bearer eyJ0e...
Content-Type: application/json

{
    "properties": {
        "myProperty1": "myPropertyValue1",
        "myProperty2": {
            "myProperty3" : "myPropertyValue3"
        }
    }
}
```

Cette requête est ensuite transférée au **point de terminaison** sous la forme suivante :

``` HTTP
PUT https://{endpointURL}/?api-version=2018-09-01-preview
Content-Type: application/json
X-MS-CustomProviders-RequestPath: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}

{
    "properties": {
        "myProperty1": "myPropertyValue1",
        "myProperty2": {
            "myProperty3" : "myPropertyValue3"
        }
    }
}
```

De même, la réponse du **point de terminaison** est ensuite transmise au client. La réponse du point de terminaison doit renvoyer :

- Un document d’objet JSON valide. Tous les tableaux et chaînes doivent être imbriqués sous un objet de niveau supérieur.
- L’en-tête `Content-Type` doit être défini sur « application/json; charset=utf-8 ».
- Le fournisseur de ressources personnalisées remplace les champs `name`, `type`, et `id` pour la requête.
- Le fournisseur de ressources personnalisées renvoie uniquement les champs figurant sous l’objet `properties` pour un point de terminaison du cache.

Réponse du **point de terminaison** :

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "properties": {
        "myProperty1": "myPropertyValue1",
        "myProperty2": {
            "myProperty3" : "myPropertyValue3"
        }
    }
}
```

Les champs `name`, `id`, et `type` sont automatiquement générés par le fournisseur de ressources personnalisées pour la ressource personnalisée.

Réponse du fournisseur de ressources personnalisées Azure :

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "name": "{myCustomResourceName}",
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}",
    "type": "Microsoft.CustomProviders/resourceProviders/myCustomResources",
    "properties": {
        "myProperty1": "myPropertyValue1",
        "myProperty2": {
            "myProperty3" : "myPropertyValue3"
        }
    }
}
```

### <a name="remove-a-custom-resource"></a>Supprimer une ressource personnalisée

Requête entrante de l’API Azure :

``` HTTP
Delete https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}?api-version=2018-09-01-preview
Authorization: Bearer eyJ0e...
Content-Type: application/json
```

Cette requête est ensuite transférée au **point de terminaison** sous la forme suivante :

``` HTTP
Delete https://{endpointURL}/?api-version=2018-09-01-preview
Content-Type: application/json
X-MS-CustomProviders-RequestPath: /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}
```

De même, la réponse du **point de terminaison** est ensuite retransmise au client. La réponse du point de terminaison doit renvoyer :

- Un document d’objet JSON valide. Tous les tableaux et chaînes doivent être imbriqués sous un objet de niveau supérieur.
- L’en-tête `Content-Type` doit être défini sur « application/json; charset=utf-8 ».
- Le fournisseur de ressources personnalisées Azure supprime uniquement l’élément de son cache si une réponse de niveau 200 est renvoyée. Même si la ressource n’existe pas, le **point de terminaison** doit renvoyer la valeur 204.

Réponse du **point de terminaison** :

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

Réponse du fournisseur de ressources personnalisées Azure :

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
```

### <a name="retrieve-a-custom-resource"></a>Récupérer une ressource personnalisée

Requête entrante de l’API Azure :

``` HTTP
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}?api-version=2018-09-01-preview
Authorization: Bearer eyJ0e...
Content-Type: application/json
```

La requête ne sera **pas** transmise au **point de terminaison**.

Réponse du fournisseur de ressources personnalisées Azure :

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "name": "{myCustomResourceName}",
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}",
    "type": "Microsoft.CustomProviders/resourceProviders/myCustomResources",
    "properties": {
        "myProperty1": "myPropertyValue1",
        "myProperty2": {
            "myProperty3" : "myPropertyValue3"
        }
    }
}
```

### <a name="enumerate-all-custom-resources"></a>Énumérer toutes les ressources personnalisées

Requête entrante de l’API Azure :

``` HTTP
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources?api-version=2018-09-01-preview
Authorization: Bearer eyJ0e...
Content-Type: application/json
```

Cette requête ne sera **pas** transmise au **point de terminaison**.

Réponse du fournisseur de ressources personnalisées Azure :

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
    "value" : [
        {
            "name": "{myCustomResourceName}",
            "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.CustomProviders/resourceProviders/{resourceProviderName}/myCustomResources/{myCustomResourceName}",
            "type": "Microsoft.CustomProviders/resourceProviders/myCustomResources",
            "properties": {
                "myProperty1": "myPropertyValue1",
                "myProperty2": {
                    "myProperty3" : "myPropertyValue3"
                }
            }
        }
    ]
}
```

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des fournisseurs de ressources personnalisées Azure](overview.md)
- [Démarrage rapide : Créer un fournisseur de ressources personnalisées Azure et déployer des ressources personnalisées](./create-custom-provider.md)
- [Tutoriel : Créer des actions et des ressources personnalisées dans Azure](./tutorial-get-started-with-custom-providers.md)
- [Guide pratique pour Ajout d’actions personnalisées à l’API REST Azure](./custom-providers-action-endpoint-how-to.md)
- [Reference : Informations de référence sur les proxys de ressources personnalisées](proxy-resource-endpoint-reference.md)
