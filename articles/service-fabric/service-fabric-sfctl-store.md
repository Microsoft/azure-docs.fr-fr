---
title: Interface CLI Azure Service Fabric - sfctl store
description: Apprenez-en davantage sur sfctl, l’interface de ligne de commande d’Azure Service Fabric. Inclut la liste des commandes permettant d’effectuer des opérations au niveau des fichiers sur le magasin d’images de cluster.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: af5399be44d2946775622598c5c1db3d8355f7cd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86257068"
---
# <a name="sfctl-store"></a>sfctl store
Effectue des opérations élémentaires au niveau des fichiers dans le magasin d’images de cluster.

## <a name="commands"></a>Commandes

|Commande|Description|
| --- | --- |
| supprimer | Supprime le contenu du magasin d’images. |
| root-info | Permet d’obtenir les informations de contenu à la racine du magasin d’images. |
| stat | Permet d’obtenir les informations de contenu du magasin d’images. |

## <a name="sfctl-store-delete"></a>sfctl store delete
Supprime le contenu du magasin d’images.

Supprime le contenu du magasin d’images trouvé dans le chemin d’accès relatif du magasin d’images donné. Cette commande permet de supprimer les packages d’application chargés une fois qu’ils ont été approvisionnés.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --content-path [Requis] | Chemin d’accès relatif au fichier ou au dossier dans le magasin d’images à partir de sa racine. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-store-root-info"></a>sfctl store root-info
Permet d’obtenir les informations de contenu à la racine du magasin d’images.

Retourne les informations sur le contenu du magasin d’images à la racine du magasin d’images.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-store-stat"></a>sfctl store stat
Permet d’obtenir les informations de contenu du magasin d’images.

Retourne les informations sur le contenu du magasin d’images dans le chemin d’accès de contenu spécifié. Le chemin d’accès de contenu varie selon la racine du magasin d’images.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --content-path [Requis] | Chemin d’accès relatif au fichier ou au dossier dans le magasin d’images à partir de sa racine. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |


## <a name="next-steps"></a>Étapes suivantes
- [Configurez](service-fabric-cli.md) l’interface de ligne de commande (CLI) Service Fabric.
- Découvrez comment utiliser l’interface de ligne de commande (CLI) Service Fabric à l’aide d’[exemples de scripts](./scripts/sfctl-upgrade-application.md).
