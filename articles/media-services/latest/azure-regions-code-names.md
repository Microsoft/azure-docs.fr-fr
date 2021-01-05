---
title: Clouds et régions dans lesquels Azure Media Services v3 est disponible
description: Cet article parle des URL utilisées pour les points de terminaison et le code pour les régions.
services: media-services
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: reference
ms.date: 10/28/2020
ms.author: inhenkel
ms.openlocfilehash: c17eeda91af46c4159f7c4b1747416298f24c703
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93319638"
---
# <a name="regional-code-names-and-endpoints"></a>Points de terminaison et noms de code régionaux

### <a name="region-code-name"></a>Nom de code de la région

Lorsque le paramètre **location** est utilisé dans une commande ou une requête, vous devez fournir le nom de code de la région comme valeur **location**. Pour obtenir le nom de code de la région dans laquelle se trouve votre compte et vers laquelle votre appel doit être routé, vous pouvez exécuter la ligne suivante dans l’interface de ligne de commande Azure.

```azurecli-interactive
az account list-locations
```

Une fois que vous avez exécuté la ligne ci-dessus, vous obtenez une liste de toutes les régions Azure. Accédez à la région Azure qui présente la valeur *displayName* que vous recherchez et utilisez sa valeur *name* pour le paramètre **location**.

Par exemple, pour la région Azure USA Ouest 2 affichée ci-dessous, vous allez utiliser « westus2 » pour le paramètre **location**.

```json
   {
      "displayName": "West US 2",
      "id": "/subscriptions/00000000-23da-4fce-b59c-f6fb9513eeeb/locations/westus2",
      "latitude": "47.233",
      "longitude": "-119.852",
      "name": "westus2",
      "subscriptionId": null
    }
```

## <a name="endpoints"></a>Points de terminaison  

Il est important de connaître les points de terminaison suivants lors de la connexion à des comptes Media Services à partir de différents clouds Azure nationaux.

### <a name="global-azure"></a>Azure global

| Service | Point de terminaison |
| ------- | -------- |
| Azure Resource Manager |  `https://management.azure.com/` |
| Authentification | `https://login.microsoftonline.com/` |
| Audience du jeton | `https://management.core.windows.net/` |

### <a name="azure-government"></a>Azure Government

| Service | Point de terminaison |
| ------- | -------- |
| Azure Resource Manager |  `https://management.usgovcloudapi.net/` |
| Authentification | `https://login.microsoftonline.us/` |
| Audience du jeton | `https://management.core.usgovcloudapi.net/` |

[!INCLUDE [Widevine is not available in the GovCloud region.](./includes/widevine-not-available-govcloud.md)]

### <a name="azure-germany"></a>Azure Germany

> [!NOTE]
> Les points de terminaison Azure Allemagne s’appliquent uniquement aux clouds souverains en Allemagne.

| Service | Point de terminaison |
| ------- | -------- |
| Azure Resource Manager | `https://management.cloudapi.de/` |
| Authentification | `https://login.microsoftonline.de/` |
| Audience du jeton | `https://management.core.cloudapi.de/`|

### <a name="azure-china-21vianet"></a>Azure China 21Vianet

| Service | Point de terminaison |
| ------- | -------- |
| Azure Resource Manager | `https://management.chinacloudapi.cn/` |
| Authentification | `https://login.chinacloudapi.cn/` |
| Audience du jeton |  `https://management.core.chinacloudapi.cn/` |

## <a name="see-also"></a>Voir aussi

* [Régions Azure](https://azure.microsoft.com/global-infrastructure/regions/)
* [Points de terminaison et noms de code régionaux](azure-regions-code-names.md)
* [Zones géographiques Azure](https://azure.microsoft.com/global-infrastructure/geographies/)
* [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/)

## <a name="next-steps"></a>Étapes suivantes

[Vue d’ensemble de Media Services v3](media-services-overview.md)
