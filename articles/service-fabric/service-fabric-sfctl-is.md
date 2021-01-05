---
title: 'Interface CLI Azure Service Fabric : sfctl is'
description: Apprenez-en davantage sur sfctl, l'interface de ligne de commande d'Azure Service Fabric. Contient une liste de commandes pour la gestion de l’infrastructure.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: 349f70c32ea4ebb4559f053d5ef05b4b37b6480f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86260377"
---
# <a name="sfctl-is"></a>sfctl is
Interroge et envoie des commandes vers le service d’infrastructure.

## <a name="commands"></a>Commandes

|Commande|Description|
| --- | --- |
| command | Appelle une commande d’administration dans l’instance de service d’infrastructure donnée. |
| query | Appelle une requête en lecture seule dans l’instance de service d’infrastructure donnée. |

## <a name="sfctl-is-command"></a>sfctl is command
Appelle une commande d’administration dans l’instance de service d’infrastructure donnée.

Pour les clusters pour lesquels une ou plusieurs instances du service d’infrastructure sont configurées, cette API permet d’envoyer des commandes propres à l’infrastructure vers une instance particulière du service d’infrastructure. Les commandes disponibles et leurs formats de réponse correspondants varient en fonction de l’infrastructure sur laquelle le cluster s’exécute. Cette API prend en charge la plateforme Service Fabric ; elle n’est pas censée être utilisée directement à partir de votre code.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --command [Requis] | Texte de la commande à appeler. Le contenu de la commande dépend de l’infrastructure. |
| --service-id | Identité du service d’infrastructure. <br><br> Il s’agit du nom complet du service d’infrastructure sans le schéma d’URI « fabric »\:. Ce paramètre est obligatoire uniquement pour les clusters pour lesquels plusieurs instances du service d’infrastructure s’exécutent. |
| --timeout -t | Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-is-query"></a>sfctl is query
Appelle une requête en lecture seule dans l’instance de service d’infrastructure donnée.

Pour les clusters pour lesquels une ou plusieurs instances du service d’infrastructure sont configurées, cette API permet d’envoyer des requêtes propres à l’infrastructure vers une instance particulière du service d’infrastructure. Les commandes disponibles et leurs formats de réponse correspondants varient en fonction de l’infrastructure sur laquelle le cluster s’exécute. Cette API prend en charge la plateforme Service Fabric ; elle n’est pas censée être utilisée directement à partir de votre code.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --command [Requis] | Texte de la commande à appeler. Le contenu de la commande dépend de l’infrastructure. |
| --service-id | Identité du service d’infrastructure. <br><br> Il s’agit du nom complet du service d’infrastructure sans le schéma d’URI « fabric »\:. Ce paramètre est obligatoire uniquement pour les clusters pour lesquels plusieurs instances du service d’infrastructure s’exécutent. |
| --timeout -t | Valeur par défaut \: 60. |

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
