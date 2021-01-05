---
title: Interface CLI Azure Service Fabric - sfctl rpm
description: Apprenez-en davantage sur sfctl, l’interface de ligne de commande d’Azure Service Fabric. Contient la liste des commandes liées au service gestionnaire de réparation.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: 7317fd66303aaabf5232106aa7391439880bebaf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86260288"
---
# <a name="sfctl-rpm"></a>sfctl rpm
Interroge et envoie des commandes vers le service gestionnaire de réparation.

## <a name="commands"></a>Commandes

|Commande|Description|
| --- | --- |
| approve-force | Force l’approbation de la tâche de réparation donnée. |
| supprimer | Supprime une tâche de réparation terminée. |
| list | Permet d’obtenir une liste de tâches de réparation correspondant aux filtres donnés. |

## <a name="sfctl-rpm-approve-force"></a>sfctl rpm approve-force
Force l’approbation de la tâche de réparation donnée.

Cette API prend en charge la plateforme Service Fabric ; elle n’est pas censée être utilisée directement à partir de votre code.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --task-id [Requis] | ID de la tâche de réparation. |
| --version | Numéro de version actuelle de la tâche de réparation. Si autre que zéro, la requête aboutit uniquement si cette valeur correspond à la version actuelle réelle de la tâche de réparation. Si égale à zéro, aucune vérification de version n’est effectuée. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-rpm-delete"></a>sfctl rpm delete
Supprime une tâche de réparation terminée.

Cette API prend en charge la plateforme Service Fabric ; elle n’est pas censée être utilisée directement à partir de votre code.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --task-id [Requis] | ID de la tâche de réparation terminée à supprimer. |
| --version | Numéro de version actuelle de la tâche de réparation. Si autre que zéro, la requête aboutit uniquement si cette valeur correspond à la version actuelle réelle de la tâche de réparation. Si égale à zéro, aucune vérification de version n’est effectuée. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-rpm-list"></a>sfctl rpm list
Permet d’obtenir une liste de tâches de réparation correspondant aux filtres donnés.

Cette API prend en charge la plateforme Service Fabric ; elle n’est pas censée être utilisée directement à partir de votre code.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --executor-filter | Nom de l’Exécuteur de réparations dont les tâches revendiquées doivent être incluses dans la liste. |
| --state-filter | OR au niveau du bit des valeurs suivantes, qui spécifie les états de tâches qu’il convient d’inclure dans la liste des résultats. <ul><li>1 - Créé</li><li>2 – Revendiquée</li><li>4 – En cours de préparation</li><li>8 – Approuvée</li><li>16 – En cours d’exécution</li><li>32 – En cours de restauration</li><li>64 - Terminé</li></ul>
| --task-id-filter | Préfixe d’ID de tâche de réparation à mettre en correspondance. |

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
