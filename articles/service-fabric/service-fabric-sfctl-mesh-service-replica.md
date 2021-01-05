---
title: 'CLI Azure Service Fabric : sfctl mesh service-replica'
description: Apprenez-en plus sur sfctl, l'interface de ligne de commande d'Azure Service Fabric. Comprend une liste de commandes permettant d'obtenir les détails du réplica pour les ressources d'une application.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: cbfdba30663e2aa531ab1db955b0e035a0588709
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86245719"
---
# <a name="sfctl-mesh-service-replica"></a>sfctl mesh service-replica
Obtenir les détails du réplica et répertorier les réplicas d’un service donné dans une ressource d’application.

## <a name="commands"></a>Commandes

|Commande|Description|
| --- | --- |
| list | Répertorie tous les réplicas d’un service. |
| show | Permet d’obtenir le réplica donné du service d’une application. |

## <a name="sfctl-mesh-service-replica-list"></a>sfctl mesh service-replica list
Répertorie tous les réplicas d’un service.

Permet d’obtenir des informations sur tous les réplicas d’un service. Les informations incluent la description et d’autres propriétés du réplica du service.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --app-name --application-name [obligatoire] | Le nom de l’application. |
| --service-name                [obligatoire] | Nom du service. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-mesh-service-replica-show"></a>sfctl mesh service-replica show
Permet d’obtenir le réplica donné du service d’une application.

Récupère les informations relatives au réplica du service portant le nom spécifié. Les informations incluent la description et d’autres propriétés du réplica du service.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --app-name --application-name [obligatoire] | Le nom de l’application. |
| --name -n                     [obligatoire] | Le nom du réplica du service. |
| --service-name                [obligatoire] | Nom du service. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |


## <a name="next-steps"></a>Étapes suivantes
- [Configurez](service-fabric-cli.md) l’interface de ligne de commande Service Fabric.
- Découvrez comment utiliser l’interface de ligne de commande (CLI) Service Fabric à l’aide d’[exemples de scripts](./scripts/sfctl-upgrade-application.md).
