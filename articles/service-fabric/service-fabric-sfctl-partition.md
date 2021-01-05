---
title: 'Interface CLI Azure Service Fabric : sfctl partition'
description: Apprenez-en davantage sur sfctl, l’interface de ligne de commande d’Azure Service Fabric. Inclut une liste des commandes liées à la gestion des partitions pour un service.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: fff569c586548d84ed55018764363ad7f05e526d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86257085"
---
# <a name="sfctl-partition"></a>sfctl partition
Interroge et gère des partitions pour tout service.

## <a name="commands"></a>Commandes

|Commande|Description|
| --- | --- |
| data-loss | Cette API provoque une perte de données pour la partition spécifiée. |
| data-loss-status | Permet d’obtenir la progression d’une opération de perte de données dans une partition démarrée à l’aide de l’API StartDataLoss. |
| health | Permet d’obtenir l’intégrité de la partition Service Fabric spécifiée. |
| info | Permet d’obtenir des informations sur une partition Service Fabric. |
| list | Permet d’obtenir la liste des partitions d’un service Service Fabric. |
| load | Permet d’obtenir les informations de chargement de la partition Service Fabric spécifiée. |
| load-reset | Réinitialise le chargement actuel d’une partition Service Fabric. |
| quorum-loss | Entraîne une perte de quorum pour une partition de service avec état. |
| quorum-loss-status | Permet d’obtenir la progression d’une opération de perte de quorum dans une partition démarrée à l’aide de l’API StartQuorumLoss. |
| recover | Indique au cluster Service Fabric qu’il doit tenter de récupérer une partition spécifique qui est actuellement bloquée dans la perte de quorum. |
| recover-all | Indique au cluster Service Fabric qu’il doit tenter de récupérer les services (notamment les services système) qui sont actuellement bloqués dans la perte de quorum. |
| report-health | Envoie un rapport d’intégrité sur la partition Service Fabric. |
| restart | Cette API redémarre tout ou partie des réplicas ou instances de la partition spécifiée. |
| restart-status | Permet d’obtenir la progression d’une opération de redémarrage de partition démarrée à l’aide de l’API StartPartitionRestart. |
| svc-name | Permet d’obtenir le nom du service Service Fabric d’une partition. |

## <a name="sfctl-partition-data-loss"></a>sfctl partition restart
Cette API provoque une perte de données pour la partition spécifiée.

Elle déclenche l’appel de l’API OnDataLossAsync de la partition.  Cette API provoque une perte de données pour la partition spécifiée. Elle déclenche l’appel de l’API OnDataLoss de la partition. La perte de données réelle dépend du paramètre DataLossMode spécifié.
- PartialDataLoss : seul un quorum de réplicas est supprimé, et l’API OnDataLoss est déclenchée pour la partition, mais la perte de données réelle dépend de la présence de la réplication en cours.  
- FullDataLoss : tous les réplicas étant supprimés, toutes les données sont perdues et l’API OnDataLoss est déclenchée. Cette API doit uniquement être appelée avec un service avec état comme cible. Il n’est pas conseillé d’appeler cette API avec un service système comme cible.

> [!NOTE]   
> Une fois que cette API a été appelée, elle ne peut pas être annulée. L’appel de CancelOperation arrête uniquement l’exécution et nettoie l’état du système interne. Cette API ne restaure pas les données si la commande a progressé suffisamment pour entraîner une perte de données. Appelez l’API GetDataLossProgress avec le même ID OperationId pour retourner des informations sur l’opération démarrée avec cette API.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --data-loss-mode [requis] | Cette énumération est transmise à l’API StartDataLoss pour indiquer le type de perte de données à déclencher. |
| --operation-id [requis] | GUID qui identifie un appel de cette API.  Celui-ci est transmis à l’API GetProgress correspondante. |
| --partition-id [requis] | Identité de la partition. |
| --service-id [Requis] | Identité du service. Cet ID est généralement le nom complet du service sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si un service est nommé « fabric\:/myapp/app1/svc1 », son identité est « myapp\~app1\~svc1 » dans les versions 6.0 et ultérieures, et « myapp/app1/svc1 » dans les versions précédentes. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-partition-data-loss-status"></a>sfctl partition data-loss-status
Permet d’obtenir la progression d’une opération de perte de données dans une partition démarrée à l’aide de l’API StartDataLoss.

Permet d’obtenir la progression d’une opération de perte de données démarrée à l’aide de l’API StartDataLoss en utilisant l’ID OperationId.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --operation-id [Requis] | GUID qui identifie un appel de cette API.  Celui-ci est transmis à l’API GetProgress correspondante. |
| --partition-id [Requis] | Identité de la partition. |
| --service-id [Requis] | Identité du service. Cet ID est généralement le nom complet du service sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si un service est nommé « fabric\:/myapp/app1/svc1 », son identité est « myapp\~app1\~svc1 » dans les versions 6.0 et ultérieures, et « myapp/app1/svc1 » dans les versions précédentes. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-partition-health"></a>sfctl partition health
Permet d’obtenir l’intégrité de la partition Service Fabric spécifiée.

EventsHealthStateFilter permet de filtrer la collection d’événements d’intégrité signalés dans le service en fonction de l’état d’intégrité. ReplicasHealthStateFilter permet de filtrer la collection d’objets ReplicaHealthState de la partition. Si vous spécifiez une partition qui n’existe pas dans le magasin d’intégrité, cette demande retourne une erreur.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --partition-id [Requis] | Identité de la partition. |
| --events-health-state-filter | Permet de filtrer la collection d’objets HealthEvent retournés en fonction de leur état d’intégrité. Les valeurs possibles de ce paramètre incluent la valeur entière de l’un des états d’intégrité suivants. Seuls les événements qui correspondent au filtre sont renvoyés. Tous les événements sont utilisés pour évaluer l’état d’intégrité agrégé. Si cet argument n’est pas spécifié, toutes les entrées sont retournées. Les valeurs d’état correspondent à une énumération basée sur des indicateurs. La valeur peut donc être une combinaison de ces valeurs obtenue à l’aide de l’opérateur « OR » au niveau du bit. Par exemple, si la valeur indiquée est 6, tous les événements dont la valeur HealthState est OK (2) et Warning (4) sont retournés.  <br> - Default : valeur par défaut. Correspond à toute valeur HealthState. La valeur est égale à zéro.  <br> - None : filtre qui ne correspond à aucune valeur HealthState. Permet de ne retourner aucun résultat sur une collection donnée d’états. La valeur est égale à 1.  <br> - OK : filtre qui correspond à l’entrée ayant OK comme valeur HealthState. La valeur est égale à 2.  <br> - Warning : filtre qui correspond à l’entrée ayant Warning comme valeur HealthState. La valeur est égale à 4.  <br> - Error : filtre qui correspond à l’entrée ayant Error comme valeur HealthState. La valeur est égale à 8.  <br> - All : filtre qui correspond à l’entrée ayant n’importe quelle valeur HealthState. La valeur est égale à 65535. |
| --exclude-health-statistics | Indique si les statistiques d’intégrité doivent être retournées dans le cadre du résultat de la requête. False par défaut. Les statistiques affichent le nombre d’entités enfants dont l’état d’intégrité est OK, Warning et Error. |
| --replicas-health-state-filter | Permet de filtrer la collection d’objets ReplicaHealthState de la partition. La valeur peut être obtenue à partir des membres ou des opérations au niveau du bit sur les membres de HealthStateFilter. Seuls les réplicas qui correspondent au filtre sont retournés. Tous les réplicas sont utilisés pour évaluer l’état d’intégrité agrégé. Si cet argument n’est pas spécifié, toutes les entrées sont retournées. Les valeurs d’état correspondent à une énumération basée sur des indicateurs. La valeur peut donc être une combinaison de ces valeurs obtenue à l’aide de l’opérateur « OR » au niveau du bit. Par exemple, si la valeur indiquée est 6, tous les événements dont la valeur HealthState est OK (2) et Warning (4) sont retournés. Les valeurs possibles de ce paramètre incluent la valeur entière de l’un des états d’intégrité suivants.  <br> - Default : valeur par défaut. Correspond à toute valeur HealthState. La valeur est égale à zéro.  <br> - None : filtre qui ne correspond à aucune valeur HealthState. Permet de ne retourner aucun résultat sur une collection donnée d’états. La valeur est égale à 1.  <br> - OK : filtre qui correspond à l’entrée ayant OK comme valeur HealthState. La valeur est égale à 2.  <br> - Warning : filtre qui correspond à l’entrée ayant Warning comme valeur HealthState. La valeur est égale à 4.  <br> - Error : filtre qui correspond à l’entrée ayant Error comme valeur HealthState. La valeur est égale à 8.  <br> - All : filtre qui correspond à l’entrée ayant n’importe quelle valeur HealthState. La valeur est égale à 65535. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-partition-info"></a>sfctl partition info
Permet d’obtenir des informations sur une partition Service Fabric.

Permet d’obtenir des informations sur la partition spécifiée. La réponse inclut l’ID de la partition, les informations de schéma de partitionnement, les clés prises en charge par la partition, l’état, l’intégrité et d’autres détails sur la partition.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --partition-id [Requis] | Identité de la partition. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-partition-list"></a>sfctl partition list
Permet d’obtenir la liste des partitions d’un service Service Fabric.

La réponse inclut l’ID de la partition, les informations de schéma de partitionnement, les clés prises en charge par la partition, l’état, l’intégrité et d’autres détails sur la partition.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --service-id [Requis] | Identité du service. Cet ID est généralement le nom complet du service sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si un service est nommé « fabric\:/myapp/app1/svc1 », son identité est « myapp\~app1\~svc1 » dans les versions 6.0 et ultérieures, et « myapp/app1/svc1 » dans les versions précédentes. |
| --continuation-token | Le paramètre de jeton de liaison permet d’obtenir le jeu de résultats suivant. Un jeton de liaison avec une valeur non vide est inclus dans la réponse de l’API quand les résultats du système ne tiennent pas dans une seule réponse. Lorsque cette valeur est transmise à l’appel d’API suivant, l’API retourne le jeu de résultats suivant. S’il n’existe pas de résultats supplémentaires, le jeton de liaison ne contient pas de valeur. La valeur de ce paramètre ne doit pas être codée URL. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-partition-load"></a>sfctl partition load
Permet d’obtenir les informations de chargement de la partition Service Fabric spécifiée.

Retourne des informations sur le chargement de la partition spécifiée. La réponse inclut la liste des rapports de chargement d’une partition Service Fabric. Chaque rapport comprend le nom de métrique du chargement, la valeur et la dernière heure de rapport (UTC).

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --partition-id [Requis] | Identité de la partition. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-partition-load-reset"></a>sfctl partition load-reset
Réinitialise le chargement actuel d’une partition Service Fabric.

Redéfinit le chargement actuel d’une partition Service Fabric sur le chargement par défaut du service.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --partition-id [Requis] | Identité de la partition. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-partition-quorum-loss"></a>sfctl partition quorum-loss
Entraîne une perte de quorum pour une partition de service avec état.

Cette API est utile dans une situation de perte de quorum temporaire dans votre service. Appelez l’API GetQuorumLossProgress avec le même ID OperationId pour retourner des informations sur l’opération démarrée avec cette API. Cette API peut être appelée uniquement dans les services persistants avec état (HasPersistedState==true).  N’utilisez pas cette API dans les services sans état ni les services uniquement en mémoire avec état.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --operation-id [requis] | GUID qui identifie un appel de cette API.  Celui-ci est transmis à l’API GetProgress correspondante. |
| --partition-id [requis] | Identité de la partition. |
| --quorum-loss-duration [requis] | Durée pendant laquelle la partition est conservée dans la perte de quorum.  Cette durée doit être spécifiée en secondes. |
| --quorum-loss-mode [requis] | Cette énumération est transmise à l’API StartQuorumLoss pour indiquer le type de perte de quorum à déclencher. |
| --service-id [requis] | Identité du service. Cet ID est généralement le nom complet du service sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si un service est nommé « fabric\:/myapp/app1/svc1 », son identité est « myapp\~app1\~svc1 » dans les versions 6.0 et ultérieures, et « myapp/app1/svc1 » dans les versions précédentes. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-partition-quorum-loss-status"></a>sfctl partition quorum-loss-status
Permet d’obtenir la progression d’une opération de perte de quorum dans une partition démarrée à l’aide de l’API StartQuorumLoss.

Permet d’obtenir la progression d’une opération de perte de quorum démarrée à l’aide de l’API StartQuorumLoss en utilisant l’ID OperationId indiqué.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --operation-id [Requis] | GUID qui identifie un appel de cette API.  Celui-ci est transmis à l’API GetProgress correspondante. |
| --partition-id [Requis] | Identité de la partition. |
| --service-id [Requis] | Identité du service. Cet ID est généralement le nom complet du service sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si un service est nommé « fabric\:/myapp/app1/svc1 », son identité est « myapp\~app1\~svc1 » dans les versions 6.0 et ultérieures, et « myapp/app1/svc1 » dans les versions précédentes. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-partition-recover"></a>sfctl partition recover
Indique au cluster Service Fabric qu’il doit tenter de récupérer une partition spécifique qui est actuellement bloquée dans la perte de quorum.

Cette opération doit être effectuée uniquement s’il est avéré que les réplicas arrêtés ne peuvent pas être récupérés. Une utilisation incorrecte de cette API peut entraîner une perte de données potentielle.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --partition-id [Requis] | Identité de la partition. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-partition-recover-all"></a>sfctl partition recover-all
Indique au cluster Service Fabric qu’il doit tenter de récupérer les services (notamment les services système) qui sont actuellement bloqués dans la perte de quorum.

Cette opération doit être effectuée uniquement s’il est avéré que les réplicas arrêtés ne peuvent pas être récupérés. Une utilisation incorrecte de cette API peut entraîner une perte de données potentielle.

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

## <a name="sfctl-partition-report-health"></a>sfctl partition report-health
Envoie un rapport d’intégrité sur la partition Service Fabric.

Signale l’état d’intégrité de la partition Service Fabric spécifiée. Le rapport doit contenir les informations sur la source du rapport d’intégrité et sur la propriété dont il fait état. Il est envoyé à une partition de passerelle Service Fabric, qui opère son transfert vers le magasin d’intégrité. Le rapport peut être accepté par la passerelle, mais rejeté par le magasin d’intégrité après une validation supplémentaire. Par exemple, le magasin d’intégrité peut rejeter le rapport en raison d’un paramètre non valide, comme un numéro de séquence obsolète. Pour voir si le rapport a été appliqué dans le magasin d’intégrité, vérifiez qu’il s’affiche dans la section des événements.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --health-property [obligatoire] | Propriété des informations d’intégrité. <br><br> Une entité peut avoir des rapports d’intégrité pour différentes propriétés. La propriété est une chaîne et non une énumération fixe pour offrir au rapporteur la possibilité de catégoriser la condition d’état qui déclenche le rapport. Par exemple, un rapporteur avec l’ID source (SourceId) « LocalWatchdog » peut surveiller l’état du disque disponible sur un nœud. Il peut donc signaler la propriété « AvailableDisk » sur ce nœud. Le même rapporteur peut surveiller la connectivité du nœud. Il peut donc signaler une propriété « Connectivity » sur le même nœud. Dans le magasin d’intégrité, ces rapports sont traités comme des événements d’intégrité distincts pour le nœud spécifié. Avec l’ID source (SourceId), la propriété identifie de façon unique les informations d’intégrité. |
| --health-state    [obligatoire] | Les valeurs possibles sont \: « Invalid », « Ok », « Warning », « Error », « Unknown ». |
| --partition-id [Requis] | Identité de la partition. |
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

## <a name="sfctl-partition-restart"></a>sfctl partition restart
Cette API redémarre tout ou partie des réplicas ou instances de la partition spécifiée.

Cette API est particulièrement utile pour tester le basculement. Si elle est utilisée pour cibler une partition de service sans état, l’API RestartPartitionMode doit être définie sur AllReplicasOrInstances. Appelez l’API GetPartitionRestartProgress à l’aide du même identifiant OperationId pour obtenir la progression.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --operation-id [Requis] | GUID qui identifie un appel de cette API.  Celui-ci est transmis à l’API GetProgress correspondante. |
| --partition-id [Requis] | Identité de la partition. |
| --restart-partition-mode [Requis] | Définit les partitions à redémarrer. |
| --service-id [Requis] | Identité du service. Cet ID est généralement le nom complet du service sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si un service est nommé « fabric\:/myapp/app1/svc1 », son identité est « myapp\~app1\~svc1 » dans les versions 6.0 et ultérieures, et « myapp/app1/svc1 » dans les versions précédentes. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-partition-restart-status"></a>sfctl partition restart-status
Permet d’obtenir la progression d’une opération de redémarrage de partition démarrée à l’aide de l’API StartPartitionRestart.

Permet d’obtenir la progression d’une opération de redémarrage de partition démarrée à l’aide de l’API StartPartitionRestart en utilisant l’ID OperationId fourni.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --operation-id [Requis] | GUID qui identifie un appel de cette API.  Celui-ci est transmis à l’API GetProgress correspondante. |
| --partition-id [Requis] | Identité de la partition. |
| --service-id [Requis] | Identité du service. Cet ID est généralement le nom complet du service sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si un service est nommé « fabric\:/myapp/app1/svc1 », son identité est « myapp\~app1\~svc1 » dans les versions 6.0 et ultérieures, et « myapp/app1/svc1 » dans les versions précédentes. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-partition-svc-name"></a>sfctl partition svc-name
Permet d’obtenir le nom du service Service Fabric d’une partition.

Permet d’obtenir le nom du service de la partition spécifiée. Une erreur 404 est retournée si l’ID de partition n’existe pas dans le cluster.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --partition-id [Requis] | Identité de la partition. |
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
