---
title: Interface CLI Azure Service Fabric - sfctl container
description: Apprenez-en plus sur sfctl, l’interface de ligne de commande d’Azure Service Fabric. Contient la liste des commandes pour les conteneurs.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: f82883b68ab911fb0b89fc117d9a9d77e05a781a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86245889"
---
# <a name="sfctl-container"></a>sfctl container
Exécutez des commandes relatives à un conteneur sur un nœud de cluster.

## <a name="commands"></a>Commandes

|Commande|Description|
| --- | --- |
| invoke-api | Appelle l’API de conteneur sur un conteneur déployé sur un nœud Service Fabric pour le package de code donné. |
| logs | Permet d’obtenir les journaux d’activité du conteneur déployé sur un nœud Service Fabric. |

## <a name="sfctl-container-invoke-api"></a>sfctl container invoke-api
Appelle l’API de conteneur sur un conteneur déployé sur un nœud Service Fabric pour le package de code donné.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id           [Requis] | Identité de l’application. <br><br> Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --code-package-instance-id [Requis] | ID qui identifie de façon unique une instance de package de code déployée sur un nœud Service Fabric. <br><br> Peut être récupéré par « service code-package-list ». |
| --code-package-name        [Requis] | Nom de package de code spécifié dans un manifeste de service inscrit dans le cadre d’un type d’application dans un cluster Service Fabric. |
| --container-api-uri-path   [Requis] | Chemin d’URI de l’API REST du conteneur, utilisez '{ID}' à la place du nom/ID de conteneur. |
| --node-name [Requis] | Nom du nœud. |
| --service-manifest-name [Requis] | Nom d’un manifeste de service inscrit dans le cadre d’un type d’application dans un cluster Service Fabric. |
| --container-api-body | Corps de la demande HTTP pour l’API REST de conteneur. |
| --container-api-content-type | Type de contenu pour l’API REST de conteneur, par défaut 'application/json'. |
| --container-api-http-verb | Verbe HTTP pour l’API REST de conteneur, par défaut GET. |
| --timeout -t | Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-container-logs"></a>journaux d’activité de conteneur sfctl
Permet d’obtenir les journaux d’activité du conteneur déployé sur un nœud Service Fabric.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id           [Requis] | Identité de l’application. <br><br> Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --code-package-instance-id [Requis] | ID d’instance de package de code, qui peut être récupéré avec 'service code-package-list'. |
| --code-package-name        [Requis] | Nom de package de code spécifié dans un manifeste de service inscrit dans le cadre d’un type d’application dans un cluster Service Fabric. |
| --node-name [Requis] | Nom du nœud. |
| --service-manifest-name [Requis] | Nom d’un manifeste de service inscrit dans le cadre d’un type d’application dans un cluster Service Fabric. |
| --tail | Nombre de lignes à afficher à partir de la fin des journaux d’activité. La valeur par défaut est 100. « all » pour afficher les journaux d’activité complets. |
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
