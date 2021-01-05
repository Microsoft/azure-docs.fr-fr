---
title: Interface CLI Azure Service Fabric - sfctl cluster
description: Apprenez-en davantage sur sfctl, l’interface de ligne de commande d’Azure Service Fabric. Inclut la liste des commandes liées à la gestion des clusters.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: 05132e4c7e644e681e2e4cfdedaa2b04a066ebeb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86259966"
---
# <a name="sfctl-cluster"></a>sfctl cluster
Permet de sélectionner, de gérer et d’utiliser les clusters Service Fabric.

## <a name="commands"></a>Commandes

|Commande|Description|
| --- | --- |
| code-versions | Obtient une liste des versions de code fabric approvisionnées dans un cluster Service Fabric. |
| config-versions | Obtient une liste des versions de configuration fabric approvisionnées dans un cluster Service Fabric. |
| health | Obtient l’intégrité d’un cluster Service Fabric. |
| manifest | Obtient le manifeste du cluster Service Fabric. |
| operation-cancel | Annule une opération d’erreur induite par l’utilisateur. |
| operation-list | Obtient une liste des opérations d’erreurs induites par l’utilisateur filtrées par l’entrée fournie. |
| provision | Approvisionner le code ou les packages de configuration d’un cluster Service Fabric. |
| recover-system | Indique au cluster Service Fabric qu’il doit tenter de récupérer les services système qui sont actuellement bloqués dans la perte de quorum. |
| report-health | Envoie un rapport d’intégrité sur le cluster Service Fabric. |
| select | Se connecte à un point de terminaison du cluster Service Fabric. |
| show-connection | Indique le cluster Service Fabric auquel cette instance sfctl est connectée. |
| unprovision | Annule l’approvisionnement du code ou des packages de configuration d’un cluster Service Fabric. |
| upgrade | Commence la mise à niveau du code ou de la version de configuration d’un cluster Service Fabric. |
| upgrade-resume | Effectue le déplacement de la mise à niveau de cluster vers le domaine de mise à niveau suivant. |
| upgrade-rollback | Restaure la mise à niveau d’un cluster Service Fabric. |
| upgrade-status | Obtient la progression de la mise à niveau de cluster actuelle. |
| upgrade-update | Met à jour les paramètres de mise à niveau d’une mise à niveau du cluster Service Fabric. |

## <a name="sfctl-cluster-code-versions"></a>sfctl cluster code-versions
Obtient une liste des versions de code fabric approvisionnées dans un cluster Service Fabric.

Obtient une liste des informations sur les versions de code fabric approvisionnées dans le cluster. Le paramètre CodeVersion peut être utilisé pour filtrer la sortie uniquement vers cette version spécifique.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --code-version | La version du produit Service Fabric. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-cluster-config-versions"></a>sfctl cluster config-versions
Obtient une liste des versions de configuration fabric approvisionnées dans un cluster Service Fabric.

Obtient une liste des informations sur les versions de configuration fabric approvisionnées dans le cluster. Le paramètre ConfigVersion peut être utilisé pour filtrer la sortie uniquement vers cette version spécifique.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --config-version | La version de configuration Service Fabric. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-cluster-health"></a>sfctl cluster health
Obtient l’intégrité d’un cluster Service Fabric.

EventsHealthStateFilter permet de filtrer la collecte d’événements de contrôle d’intégrité signalés sur le cluster en fonction de l’état d’intégrité. De même, utilisez NodesHealthStateFilter et ApplicationsHealthStateFilter pour filtrer la collection de nœuds et d’applications renvoyées en fonction de leur état d’intégrité agrégé.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --applications-health-state-filter | Permet de filtrer, par état d’intégrité, les objets d’état d’intégrité des applications qui sont retournés dans les résultats de la requête d’intégrité du cluster. Les valeurs possibles pour ce paramètre incluent la valeur entière qui est obtenue à partir des membres ou des opérations au niveau du bit effectuées sur les membres de l’énumération HealthStateFilter. Seules les applications qui correspondent au filtre sont retournées. Toutes les applications sont utilisées pour évaluer l’état d’intégrité agrégé. Si cet argument n’est pas spécifié, toutes les entrées sont retournées. Les valeurs d’état sont une énumération basée sur des indicateurs. La valeur peut donc être une combinaison de ces valeurs obtenue à l’aide de l’opérateur « OR » au niveau du bit. Par exemple, si la valeur indiquée est 6, l’état d’intégrité des applications dont la valeur HealthState est OK (2) et Warning (4) est retourné.  <br> - Default : valeur par défaut. Correspond à toute valeur HealthState. La valeur est égale à zéro.  <br> - None : filtre qui ne correspond à aucune valeur HealthState. Permet de ne retourner aucun résultat sur une collection donnée d’états. La valeur est égale à 1.  <br> - OK : filtre qui correspond à l’entrée ayant OK comme valeur HealthState. La valeur est égale à 2.  <br> - Warning : filtre qui correspond à l’entrée ayant Warning comme valeur HealthState. La valeur est égale à 4.  <br> - Error : filtre qui correspond à l’entrée ayant Error comme valeur HealthState. La valeur est égale à 8.  <br> - All : filtre qui correspond à l’entrée ayant n’importe quelle valeur HealthState. La valeur est égale à 65535. |
| --events-health-state-filter | Permet de filtrer la collection d’objets HealthEvent retournés en fonction de leur état d’intégrité. Les valeurs possibles de ce paramètre incluent la valeur entière de l’un des états d’intégrité suivants. Seuls les événements qui correspondent au filtre sont renvoyés. Tous les événements sont utilisés pour évaluer l’état d’intégrité agrégé. Si cet argument n’est pas spécifié, toutes les entrées sont retournées. Les valeurs d’état correspondent à une énumération basée sur des indicateurs. La valeur peut donc être une combinaison de ces valeurs obtenue à l’aide de l’opérateur « OR » au niveau du bit. Par exemple, si la valeur indiquée est 6, tous les événements dont la valeur HealthState est OK (2) et Warning (4) sont retournés.  <br> - Default : valeur par défaut. Correspond à toute valeur HealthState. La valeur est égale à zéro.  <br> - None : filtre qui ne correspond à aucune valeur HealthState. Permet de ne retourner aucun résultat sur une collection donnée d’états. La valeur est égale à 1.  <br> - OK : filtre qui correspond à l’entrée ayant OK comme valeur HealthState. La valeur est égale à 2.  <br> - Warning : filtre qui correspond à l’entrée ayant Warning comme valeur HealthState. La valeur est égale à 4.  <br> - Error : filtre qui correspond à l’entrée ayant Error comme valeur HealthState. La valeur est égale à 8.  <br> - All : filtre qui correspond à l’entrée ayant n’importe quelle valeur HealthState. La valeur est égale à 65535. |
| --exclude-health-statistics | Indique si les statistiques d’intégrité doivent être retournées dans le cadre du résultat de la requête. False par défaut. Les statistiques affichent le nombre d’entités enfants dont l’état d’intégrité est OK, Warning et Error. |
| --include-system-application-health-statistics | Indique si les statistiques d’intégrité doivent inclure celles des applications fabric\:/System. False par défaut. Si IncludeSystemApplicationHealthStatistics est défini sur true, les statistiques d’intégrité incluent les entités qui appartiennent à l’application fabric\:/System. Dans le cas contraire, le résultat de la requête inclut uniquement les statistiques d’intégrité pour les applications utilisateur. Pour que ce paramètre soit appliqué, les statistiques d’intégrité doivent être incluses dans le résultat de la requête. |
| --nodes-health-state-filter | Permet de filtrer, par état d’intégrité, les objets d’état d’intégrité des nœuds qui sont retournés dans les résultats de la requête d’intégrité du cluster. Les valeurs possibles de ce paramètre incluent la valeur entière de l’un des états d’intégrité suivants. Seuls les nœuds qui correspondent au filtre sont renvoyés. Tous les nœuds sont utilisés pour évaluer l’état d’intégrité agrégé. Si cet argument n’est pas spécifié, toutes les entrées sont retournées. Les valeurs d’état sont une énumération basée sur des indicateurs. La valeur peut donc être une combinaison de ces valeurs obtenue à l’aide de l’opérateur « OR » au niveau du bit. Par exemple, si la valeur indiquée est 6, l’état d’intégrité des nœuds dont la valeur HealthState est OK (2) et Warning (4) est retourné.  <br> - Default : valeur par défaut. Correspond à toute valeur HealthState. La valeur est égale à zéro.  <br> - None : filtre qui ne correspond à aucune valeur HealthState. Permet de ne retourner aucun résultat sur une collection donnée d’états. La valeur est égale à 1.  <br> - OK : filtre qui correspond à l’entrée ayant OK comme valeur HealthState. La valeur est égale à 2.  <br> - Warning : filtre qui correspond à l’entrée ayant Warning comme valeur HealthState. La valeur est égale à 4.  <br> - Error : filtre qui correspond à l’entrée ayant Error comme valeur HealthState. La valeur est égale à 8.  <br> - All : filtre qui correspond à l’entrée ayant n’importe quelle valeur HealthState. La valeur est égale à 65535. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-cluster-manifest"></a>sfctl cluster manifest
Obtient le manifeste du cluster Service Fabric.

Obtient le manifeste du cluster Service Fabric. Le manifeste du cluster contient les propriétés du cluster qui incluent des types de nœuds différents sur le cluster, les configurations de sécurité, l’erreur et les topologies des domaines de mise à niveau, etc. Ces propriétés sont spécifiées dans le fichier ClusterConfig.JSON lors du déploiement d’un cluster autonome. Toutefois, la plupart des informations du manifeste du cluster sont générées en interne par service fabric durant le déploiement de cluster dans d’autres scénarios de déploiement (par exemple, lorsque vous utilisez le portail Azure). Le contenu du manifeste de cluster est à titre d’information uniquement et les utilisateurs ne sont pas supposés dépendre du format du contenu du fichier ou de son interprétation.

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

## <a name="sfctl-cluster-operation-cancel"></a>sfctl cluster operation-cancel
Annule une opération d’erreur induite par l’utilisateur.

Les API suivantes démarrent des opérations d’erreurs qui peuvent être annulées au moyen de CancelOperation\: StartDataLoss, StartQuorumLoss, StartPartitionRestart, StartNodeTransition. Si force est défini sur false, l’opération induite par l’utilisateur spécifiée est correctement arrêtée et nettoyée.  Si force est défini sur true, la commande va être abandonnée et un état interne peut éventuellement être conservé.  La définition de force sur true doit être utilisée avec précaution. Vous n’êtes pas autorisé à appeler cette API avec force défini sur true tant que cette API n’a pas encore été appelée sur la même commande de test avec force défini sur false, ou sauf si la commande de test a déjà un OperationState OperationState.RollingBack. Clarification \: OperationState.RollingBack signifie que le système nettoiera/nettoie l’état du système interne généré par l’exécution de la commande.  Cela ne restaurera pas les données si la commande de test engendre une perte de données.  Par exemple, si vous appelez StartDataLoss, puis cette API, le système nettoiera seulement l’état interne lié à l’exécution de la commande. Il ne restaurera pas les données de la partition cible si la commande a progressé suffisamment pour entraîner une perte de données. Remarque importante\:  Si cette API est appelée avec force == true, l’état interne peut être conservé.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --operation-id [Requis] | GUID qui identifie un appel de cette API.  Celui-ci est transmis à l’API GetProgress correspondante. |
| --force | Indique s’il faut restaurer et nettoyer l’état du système interne modifié par l’exécution de l’opération induite par l’utilisateur. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-cluster-operation-list"></a>sfctl cluster operation-list
Obtient une liste des opérations d’erreurs induites par l’utilisateur filtrées par l’entrée fournie.

Obtient la liste des opérations d’erreurs induites par l’utilisateur filtrées par l’entrée fournie.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --state-filter | Permet de filtrer sur OperationState pour les opérations induites par l’utilisateur. - 65535 - sélectionner All - 1 - sélectionner Running - 2 - sélectionner RollingBack - 8 - sélectionner Completed - 16 - sélectionner Faulted - 32 - sélectionner Cancelled - 64 - sélectionner ForceCancelled.  Par défaut \: 65535. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |
| --type-filter | Permet de filtrer sur OperationType pour les opérations induites par l’utilisateur. - 65535 - sélectionner All - 1 - sélectionner PartitionDataLoss. - 2 - sélectionner PartitionQuorumLoss. - 4 - sélectionner PartitionRestart. - 8 - sélectionner NodeTransition.  Par défaut \: 65535. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-cluster-provision"></a>sfctl cluster provision
Approvisionner le code ou les packages de configuration d’un cluster Service Fabric.

Valide et provisionne le code ou les packages de configuration d’un cluster Service Fabric.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --cluster-manifest-file-path | Chemin d’accès au fichier du manifeste de cluster. |
| --code-file-path | Chemin d’accès au fichier du package de code de cluster. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-cluster-recover-system"></a>sfctl cluster recover-system
Indique au cluster Service Fabric qu’il doit tenter de récupérer les services système qui sont actuellement bloqués dans la perte de quorum.

Indique au cluster Service Fabric qu’il doit tenter de récupérer les services système qui sont actuellement bloqués dans la perte de quorum. Cette opération doit être effectuée uniquement s’il est avéré que les réplicas arrêtés ne peuvent pas être récupérés. Une utilisation incorrecte de cette API peut entraîner une perte de données potentielle.

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

## <a name="sfctl-cluster-report-health"></a>sfctl cluster report-health
Envoie un rapport d’intégrité sur le cluster Service Fabric.

Envoie un rapport d’intégrité sur un cluster Service Fabric. Le rapport doit contenir les informations sur la source du rapport d’intégrité et sur la propriété dont il fait état. Il est envoyé à un nœud de passerelle Service Fabric, qui opère son transfert vers le magasin d’intégrité. Le rapport peut être accepté par la passerelle, mais rejeté par le magasin d’intégrité après une validation supplémentaire. Par exemple, le magasin d’intégrité peut rejeter le rapport en raison d’un paramètre non valide, comme un numéro de séquence obsolète. Pour voir si le rapport a été appliqué dans le magasin d’intégrité, exécutez GetClusterHealth et vérifiez qu’il s’affiche dans la section des événements d’intégrité.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --health-property [obligatoire] | Propriété des informations d’intégrité. <br><br> Une entité peut avoir des rapports d’intégrité pour différentes propriétés. La propriété est une chaîne et non une énumération fixe pour offrir au rapporteur la possibilité de catégoriser la condition d’état qui déclenche le rapport. Par exemple, un rapporteur avec l’ID source (SourceId) « LocalWatchdog » peut surveiller l’état du disque disponible sur un nœud. Il peut donc signaler la propriété « AvailableDisk » sur ce nœud. Le même rapporteur peut surveiller la connectivité du nœud. Il peut donc signaler une propriété « Connectivity » sur le même nœud. Dans le magasin d’intégrité, ces rapports sont traités comme des événements d’intégrité distincts pour le nœud spécifié. Avec l’ID source (SourceId), la propriété identifie de façon unique les informations d’intégrité. |
| --health-state    [obligatoire] | Les valeurs possibles sont \: « Invalid », « Ok », « Warning », « Error », « Unknown ». |
| --source-id       [obligatoire] | Nom source qui identifie le composant client/espion/système qui a généré les informations d’intégrité. |
| --description | Description des informations d’intégrité. <br><br> Il s’agit de texte libre utilisé pour ajouter des informations sur le rapport lisibles par l’homme. La longueur de chaîne maximale de la description est de 4 096 caractères. Si la chaîne fournie est plus longue, elle est automatiquement tronquée. Si elle est tronquée, les derniers caractères de la description contiennent un marqueur « [Tronqué] », et la taille totale de la chaîne est de 4 096 caractères. La présence du marqueur indique aux utilisateurs qu’une troncation a été effectuée. Notez que si elle est tronquée, la description a moins que les 4 096 caractères de la chaîne d’origine. |
| --immediate | Indicateur qui spécifie si le rapport doit être envoyé immédiatement. <br><br> Un rapport d’intégrité est envoyé à une application de passerelle Service Fabric, qui opère son transfert vers le magasin d’intégrité. Si le paramètre immediate est défini sur true, le rapport est immédiatement envoyé de la passerelle HTTP au magasin d’intégrité, quels que soient les paramètres du client Fabric qu’utilise l’application de passerelle HTTP. Cela est utile pour les rapports critiques qui doivent être envoyés dès que possible. En fonction du minutage et d’autres conditions, l’envoi du rapport peut quand même échouer, par exemple si la passerelle HTTP est fermée ou si le message n’atteint pas la passerelle. Si le paramètre immediate est défini sur false, le rapport est envoyé en fonction des paramètres du client d’intégrité de la passerelle HTTP. C’est pourquoi il est traité par lot selon la configuration HealthReportSendInterval. Il s’agit du paramètre recommandé, car il permet au client d’intégrité d’optimiser les messages de rapport d’intégrité envoyés au magasin d’intégrité, ainsi que le traitement des rapports d’intégrité. Par défaut, les rapports ne sont pas envoyés immédiatement. |
| --remove-when-expired | Valeur qui indique si le rapport est supprimé du magasin d’intégrité quand il expire. <br><br> Si la valeur définie est true, le rapport est supprimé du magasin d’intégrité après son expiration. Si la valeur définie est false, le rapport est traité comme une erreur quand il expire. La valeur de cette propriété est false par défaut. Quand les clients créent un rapport régulièrement, ils doivent définir RemoveWhenExpired sur false (valeur par défaut). De cette manière, si le rapporteur rencontre des problèmes (par exemple, un interblocage) et qu’il ne peut pas créer de rapport, l’entité est évaluée comme erreur quand le rapport d’intégrité expire. L’entité est marquée comme étant dans l’état d’intégrité Erreur. |
| --sequence-number | Numéro de séquence de ce rapport d’intégrité sous forme de chaîne numérique. <br><br> Le numéro de séquence de rapport est utilisé par le magasin d’intégrité pour détecter les rapports obsolètes. S’il n’est pas spécifié, un numéro de séquence est généré automatiquement par le client d’intégrité quand un rapport est ajouté. |
| --timeout -t | Valeur par défaut \: 60. |
| --ttl | Durée pendant laquelle ce rapport d’intégrité est valide. Ce champ utilise le format ISO8601 pour spécifier la durée. <br><br> Quand les clients créent régulièrement des rapports, ils doivent les envoyer avec une fréquence supérieure à la durée de vie. Si les clients créent des rapports lors d’une transition, ils peuvent définir la durée de vie sur Infinite (illimitée). Quand la durée de vie expire, l’événement d’intégrité qui contient les informations d’intégrité est supprimé du magasin d’intégrité si RemoveWhenExpired est true, ou évalué comme erreur si RemoveWhenExpired est false. Si cet argument n’est pas spécifié, la valeur de durée de vie est par défaut Infinite (illimitée). |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-cluster-select"></a>sfctl cluster select
Se connecte à un point de terminaison du cluster Service Fabric.

Si vous vous connectez au cluster sécurisé, vous devez spécifier un chemin d’accès absolu à un fichier de certificat (.crt) et de clé (.key) ou à un seul fichier comportant les deux (.pem). Ne spécifiez pas les deux. Si vous le souhaitez, lorsque vous vous connectez à un cluster sécurisé, spécifiez également un chemin d’accès absolu à un fichier de groupement d’autorités de certification ou à un répertoire de certificats d’autorités de certification approuvés.  Il n’y a pas de connexion à un cluster sans exécuter d’abord cette commande, y compris une connexion à localhost. Cependant, aucun point de terminaison explicite n’est nécessaire pour se connecter à un cluster local.  Si vous utilisez un certificat auto-signé, ou un autre certificat qui n'a pas été signé par une autorité de certification connue, utilisez le paramètre --ca pour veiller à ce que la validation aboutisse. Hors cluster de production, pour contourner la validation côté client (utile pour un certificat auto-signé ou un certificat provenant d'une autorité de certification non connue), utilisez l'option --no-verify. Bien que son utilisation soit possible, cette option n'est pas recommandée pour les clusters de production. Une erreur pourrait se produire lors de la vérification du certificat.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --aad | Utilisez Azure Active Directory pour l’authentification. |
| --ca | Chemin d’accès absolu au répertoire de certificats d’autorités de certification à traiter comme étant valides ou au fichier de groupement d’autorités de certification. Si vous utilisez un répertoire de certificats d’autorité de certification, le paramètre `c_rehash <directory>` fourni par OpenSSL doit d’abord être exécuté pour calculer les hachages de certificat et créer les liens symboliques appropriés. Cela permet de vérifier que le certificat retourné par le cluster est valide. |
| --cert | Chemin d’accès absolu à un fichier de certificat client. |
| --endpoint | URL de point de terminaison du cluster, y compris le port et le préfixe HTTP ou HTTPS. En règle générale, le point de terminaison se présente sous la forme https\://<votre-URL>\:19080. Si aucun point de terminaison n’est spécifié, la valeur par défaut est http\://localhost\:19080.  Par défaut\: http\://localhost\:19080. |
| --key | Chemin d’accès absolu à un fichier de clé de certificat client. |
| --no-verify | Désactivez la vérification des certificats lors de l’utilisation de HTTPS. Remarque \: il s’agit d’une option non sécurisée à ne pas utiliser pour les environnements de production. |
| --pem | Chemin d’accès absolu à un fichier de certificat client au format .pem. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-cluster-show-connection"></a>sfctl cluster show-connection
Indique le cluster Service Fabric auquel cette instance sfctl est connectée.

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-cluster-unprovision"></a>sfctl cluster unprovision
Annule l’approvisionnement du code ou des packages de configuration d’un cluster Service Fabric.

Il est possible d’annuler la mise en service du code et de la configuration séparément.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --code-version | Version du package de code de cluster. |
| --config-version | Version du manifeste de cluster. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-cluster-upgrade"></a>sfctl cluster upgrade
Commence la mise à niveau du code ou de la version de configuration d’un cluster Service Fabric.

Valide les paramètres de mise à niveau fournis et démarre la mise à niveau de la version du code ou de la configuration d’un cluster Service Fabric, si les paramètres sont valides.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --app-health-map | Dictionnaire au format JSON comprenant des paires Nom d’application/Pourcentage maximal de non intégrité avant déclenchement d’une erreur. |
| --app-type-health-map | Dictionnaire au format JSON comprenant des paires Nom de type d’application/Pourcentage maximal de non intégrité avant déclenchement d’une erreur. |
| --code-version | Version du code du cluster. |
| --config-version | Version de la configuration du cluster. |
| --delta-health-evaluation | Permet l’évaluation de l’intégrité delta plutôt que l’évaluation de l’intégrité absolue après chaque mise à niveau de domaine de mise à niveau. |
| --delta-unhealthy-nodes | Pourcentage maximal autorisé de dégradation de l’intégrité des nœuds pendant les mises à niveau de clusters.  Valeur par défaut \: 10. <br><br> Le delta est mesuré entre l’état des nœuds au début de la mise à niveau et l’état des nœuds au moment de l’évaluation de l’intégrité. Cette vérification est effectuée après chaque mise à niveau de domaine de mise à niveau afin de vérifier que l’état global du cluster se trouve dans les limites autorisées. |
| --failure-action | Les valeurs possibles sont \: « Invalid », « Rollback », « Manual ». |
| --force-restart | Force le redémarrage des processus pendant la mise à jour, même si la version du code n’a pas changé. <br><br> La mise à niveau modifie uniquement la configuration ou les données. |
| --health-check-retry | Délai séparant les tentatives d'exécution des contrôles d'intégrité si l'application ou le cluster n'est pas sain. |
| --health-check-stable | Durée pendant laquelle l’application ou le cluster doivent rester sains avant que la mise à niveau ne passe au domaine de mise à niveau suivant. <br><br> Elle est d’abord interprétée en tant que chaîne représentant une durée ISO 8601. Si cette tentative échoue, elle est interprétée comme un nombre représentant le nombre total de millisecondes. |
| --health-check-wait | Délai d’attente entre l’achèvement d’un domaine de mise à niveau et le démarrage du processus des contrôles d’intégrité. |
| --replica-set-check-timeout | Durée maximale pendant laquelle bloquer le traitement d’un domaine de mise à niveau et éviter la perte de disponibilité en cas de problèmes inattendus. <br><br> Lorsque ce délai d’attente expire, le traitement du domaine de mise à niveau se poursuit, indépendamment des problèmes de perte de disponibilité. Le délai d’expiration est réinitialisé au début de chaque domaine de mise à niveau. Les valeurs valides sont comprises entre 0 et 42949672925 inclus. |
| --rolling-upgrade-mode | Les valeurs possibles sont \: « Invalid », « UnmonitoredAuto », « UnmonitoredManual », « Monitored ».  Valeur par défaut \: UnmonitoredAuto. |
| --timeout -t | Valeur par défaut \: 60. |
| --unhealthy-applications | Pourcentage maximal autorisé d’applications non saines avant signalement d’une erreur. <br><br> Par exemple, pour autoriser 10 % des applications pouvant être défectueuses, cette valeur serait de 10. Il s’agit du pourcentage maximum toléré d’applications pouvant être défectueuses avant que l’intégrité du cluster ne soit considérée comme étant à l’état Error. Si le pourcentage est respecté mais qu’il existe au moins une application pouvant être défectueuse, l’état d’intégrité est Warning. Ce calcul est effectué en divisant le nombre d’applications pouvant être défectueuses par le nombre total d’instances de l’application dans le cluster, à l’exception des types d’applications inclus dans le ApplicationTypeHealthPolicyMap. Le calcul est arrondi pour tolérer la défaillance d’un petit nombre d’applications. |
| --unhealthy-nodes | Pourcentage maximal autorisé de nœuds non sains avant signalement d’une erreur. <br><br> Par exemple, pour autoriser 10 % de nœuds défectueux, cette valeur serait de 10. Il s’agit du pourcentage maximum toléré de nœuds pouvant être défectueux avant que l’intégrité du cluster ne soit considérée comme étant à l’état Error. Si le pourcentage est respecté mais qu’il existe au moins un nœud pouvant être défectueux, l’état d’intégrité est Warning. Le pourcentage est calculé en divisant le nombre de nœuds défectueux par le nombre total de nœuds du cluster. Le calcul est arrondi pour tolérer une défaillance sur un petit nombre de nœuds. Dans les clusters de grande taille, certains nœuds sont toujours inactifs ou en réparation. Ce pourcentage doit donc être configuré pour tolérer cette condition. |
| --upgrade-domain-delta-unhealthy-nodes | Pourcentage maximal autorisé de dégradation de l’intégrité des nœuds de domaine de mise à niveau pendant les mises à niveau de clusters.  Valeur par défaut \: 15. <br><br> Le delta est mesuré entre l’état des nœuds de domaine de mise à niveau au début de la mise à niveau et leur état au moment de l’évaluation de l’intégrité. Cette vérification est effectuée après chaque mise à niveau de domaine de mise à niveau (pour l’ensemble des domaines de mise à niveau) afin de vérifier que l’état des domaines de mise à niveau se trouve dans les limites autorisées. |
| --upgrade-domain-timeout | Durée d’exécution de chaque domaine de mise à niveau avant l’exécution de FailureAction. <br><br> Elle est d’abord interprétée en tant que chaîne représentant une durée ISO 8601. Si cette tentative échoue, elle est interprétée comme un nombre représentant le nombre total de millisecondes. |
| --upgrade-timeout | Durée d’exécution de l’ensemble de la mise à niveau avant l’exécution de FailureAction. <br><br> Elle est d’abord interprétée en tant que chaîne représentant une durée ISO 8601. Si cette tentative échoue, elle est interprétée comme un nombre représentant le nombre total de millisecondes. |
| --warning-as-error | Indique si les avertissements sont traités avec le même niveau de gravité que les erreurs. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-cluster-upgrade-resume"></a>sfctl cluster upgrade-resume
Effectue le déplacement de la mise à niveau de cluster vers le domaine de mise à niveau suivant.

Le cas échéant, effectue le déplacement de la mise à niveau de code de cluster ou de la configuration vers le domaine de mise à niveau suivant.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --upgrade-domain [Requis] | Domaine de mise à niveau suivant pour cette mise à niveau de cluster. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-cluster-upgrade-rollback"></a>sfctl cluster upgrade-rollback
Restaure la mise à niveau d’un cluster Service Fabric.

Restaure la mise à niveau du code ou de la configuration d’un cluster Service Fabric.

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

## <a name="sfctl-cluster-upgrade-status"></a>sfctl cluster upgrade-status
Obtient la progression de la mise à niveau de cluster actuelle.

Obtient la progression actuelle de la mise à niveau de cluster en cours. Si aucune mise à niveau n’est actuellement en cours, obtient le dernier état de la précédente mise à niveau de cluster.

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

## <a name="sfctl-cluster-upgrade-update"></a>sfctl cluster upgrade-update
Met à jour les paramètres de mise à niveau d’une mise à niveau du cluster Service Fabric.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --app-health-map | Dictionnaire au format JSON comprenant des paires Nom d’application/Pourcentage maximal de non intégrité avant déclenchement d’une erreur. |
| --app-type-health-map | Dictionnaire au format JSON comprenant des paires Nom de type d’application/Pourcentage maximal de non intégrité avant déclenchement d’une erreur. |
| --delta-health-evaluation | Permet l’évaluation de l’intégrité delta plutôt que l’évaluation de l’intégrité absolue après chaque mise à niveau de domaine de mise à niveau. |
| --delta-unhealthy-nodes | Pourcentage maximal autorisé de dégradation de l’intégrité des nœuds pendant les mises à niveau de clusters.  Valeur par défaut \: 10. <br><br> Le delta est mesuré entre l’état des nœuds au début de la mise à niveau et l’état des nœuds au moment de l’évaluation de l’intégrité. Cette vérification est effectuée après chaque mise à niveau de domaine de mise à niveau afin de vérifier que l’état global du cluster se trouve dans les limites autorisées. |
| --failure-action | Les valeurs possibles sont \: « Invalid », « Rollback », « Manual ». |
| --force-restart | Force le redémarrage des processus pendant la mise à jour, même si la version du code n’a pas changé. <br><br> La mise à niveau modifie uniquement la configuration ou les données. |
| --health-check-retry | Délai séparant les tentatives d'exécution des contrôles d'intégrité si l'application ou le cluster n'est pas sain. |
| --health-check-stable | Durée pendant laquelle l’application ou le cluster doivent rester sains avant que la mise à niveau ne passe au domaine de mise à niveau suivant. <br><br> Elle est d’abord interprétée en tant que chaîne représentant une durée ISO 8601. Si cette tentative échoue, elle est interprétée comme un nombre représentant le nombre total de millisecondes. |
| --health-check-wait | Délai d’attente entre l’achèvement d’un domaine de mise à niveau et le démarrage du processus des contrôles d’intégrité. |
| --replica-set-check-timeout | Durée maximale pendant laquelle bloquer le traitement d’un domaine de mise à niveau et éviter la perte de disponibilité en cas de problèmes inattendus. <br><br> Lorsque ce délai d’attente expire, le traitement du domaine de mise à niveau se poursuit, indépendamment des problèmes de perte de disponibilité. Le délai d’expiration est réinitialisé au début de chaque domaine de mise à niveau. Les valeurs valides sont comprises entre 0 et 42949672925 inclus. |
| --rolling-upgrade-mode | Les valeurs possibles sont \: « Invalid », « UnmonitoredAuto », « UnmonitoredManual », « Monitored ».  Valeur par défaut \: UnmonitoredAuto. |
| --timeout -t | Valeur par défaut \: 60. |
| --unhealthy-applications | Pourcentage maximal autorisé d’applications non saines avant signalement d’une erreur. <br><br> Par exemple, pour autoriser 10 % des applications pouvant être défectueuses, cette valeur serait de 10. Il s’agit du pourcentage maximum toléré d’applications pouvant être défectueuses avant que l’intégrité du cluster ne soit considérée comme étant à l’état Error. Si le pourcentage est respecté mais qu’il existe au moins une application pouvant être défectueuse, l’état d’intégrité est Warning. Ce calcul est effectué en divisant le nombre d’applications pouvant être défectueuses par le nombre total d’instances de l’application dans le cluster, à l’exception des types d’applications inclus dans le ApplicationTypeHealthPolicyMap. Le calcul est arrondi pour tolérer la défaillance d’un petit nombre d’applications. |
| --unhealthy-nodes | Pourcentage maximal autorisé de nœuds non sains avant signalement d’une erreur. <br><br> Par exemple, pour autoriser 10 % de nœuds défectueux, cette valeur serait de 10. Il s’agit du pourcentage maximum toléré de nœuds pouvant être défectueux avant que l’intégrité du cluster ne soit considérée comme étant à l’état Error. Si le pourcentage est respecté mais qu’il existe au moins un nœud pouvant être défectueux, l’état d’intégrité est Warning. Le pourcentage est calculé en divisant le nombre de nœuds défectueux par le nombre total de nœuds du cluster. Le calcul est arrondi pour tolérer une défaillance sur un petit nombre de nœuds. Dans les clusters de grande taille, certains nœuds sont toujours inactifs ou en réparation. Ce pourcentage doit donc être configuré pour tolérer cette condition. |
| --upgrade-domain-delta-unhealthy-nodes | Pourcentage maximal autorisé de dégradation de l’intégrité des nœuds de domaine de mise à niveau pendant les mises à niveau de clusters.  Valeur par défaut \: 15. <br><br> Le delta est mesuré entre l’état des nœuds de domaine de mise à niveau au début de la mise à niveau et leur état au moment de l’évaluation de l’intégrité. Cette vérification est effectuée après chaque mise à niveau de domaine de mise à niveau (pour l’ensemble des domaines de mise à niveau) afin de vérifier que l’état des domaines de mise à niveau se trouve dans les limites autorisées. |
| --upgrade-domain-timeout | Durée d’exécution de chaque domaine de mise à niveau avant l’exécution de FailureAction. <br><br> Elle est d’abord interprétée en tant que chaîne représentant une durée ISO 8601. Si cette tentative échoue, elle est interprétée comme un nombre représentant le nombre total de millisecondes. |
| --upgrade-kind | Les valeurs possibles sont \: « Invalid », « Rolling », « Rolling_ForceRestart ».  Valeur par défaut \: Rolling. |
| --upgrade-timeout | Durée d’exécution de l’ensemble de la mise à niveau avant l’exécution de FailureAction. <br><br> Elle est d’abord interprétée en tant que chaîne représentant une durée ISO 8601. Si cette tentative échoue, elle est interprétée comme un nombre représentant le nombre total de millisecondes. |
| --warning-as-error | Indique si les avertissements sont traités avec le même niveau de gravité que les erreurs. |

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
