---
title: Interface de ligne de commande Azure Service Fabric - Événements sfctl
description: Décrit les commandes d’événements sfctl de l’interface de ligne de commande Service Fabric.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: 635bddef1a98d6ed2b112662d8c9c44fe65e9e32
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76906094"
---
# <a name="sfctl-events"></a>Événements sfctl
Récupérez les événements du magasin d’événements (si le service EventStore est déjà installé).

Le service système EventStore peut être ajouté via une mise à niveau de configuration vers n’importe quel cluster SFRP exécutant >=6,4. Vérifiez l’URL suivante\: https\://docs.microsoft.com/azure/service-fabric/service-fabric-diagnostics-eventstore.

## <a name="commands"></a>Commandes

|Commande|Description|
| --- | --- |
| all-applications-list | Obtient tous les événements liés aux applications. |
| all-nodes-list | Obtient tous les événements liés aux nœuds. |
| all-partitions-list | Obtient tous les événements liés aux partitions. |
| all-services-list | Obtient tous les événements liés aux services. |
| application-list | Obtient un événement lié à une application. |
| cluster-list | Obtient tous les événements liés au cluster. |
| node-list | Obtient un événement liés à un nœud. |
| partition-all-replicas-list | Obtient tous les événements liés aux réplicas pour une partition. |
| partition-list | Obtient un événement lié à une partition. |
| partition-replica-list | Obtient un événement lié à un réplica de partition. |
| service-list | Obtient un événement liés à un service. |

## <a name="sfctl-events-all-applications-list"></a>sfctl events all-applications-list
Obtient tous les événements liés aux applications.

La réponse est une liste d’objets ApplicationEvent.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --end-time-utc   [requis] | Heure de fin d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| --start-time-utc [requis] | Heure de début d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| Events-types-filtre | Chaîne séparée par des virgules spécifiant les types de FabricEvents qui doivent être inclus uniquement dans la réponse. |
| --exclude-analysis-events | Ce paramètre désactive la récupération d’AnalysisEvents si la valeur true est transmise. |
| --skip-correlation-lookup | Ce paramètre désactive la recherche d’informations CorrelatedEvents si la valeur true est transmise. Dans le cas contraire, CorrelationEvents est traité et le champ HasCorrelatedEvents dans chaque FabricEvent est rempli. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-events-all-nodes-list"></a>sfctl events all-nodes-list
Obtient tous les événements liés aux nœuds.

La réponse est la liste d’objets NodeEvent.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --end-time-utc   [requis] | Heure de fin d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| --start-time-utc [requis] | Heure de début d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| Events-types-filtre | Chaîne séparée par des virgules spécifiant les types de FabricEvents qui doivent être inclus uniquement dans la réponse. |
| --exclude-analysis-events | Ce paramètre désactive la récupération d’AnalysisEvents si la valeur true est transmise. |
| --skip-correlation-lookup | Ce paramètre désactive la recherche d’informations CorrelatedEvents si la valeur true est transmise. Dans le cas contraire, CorrelationEvents est traité et le champ HasCorrelatedEvents dans chaque FabricEvent est rempli. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-events-all-partitions-list"></a>sfctl events all-partitions-list
Obtient tous les événements liés aux partitions.

La réponse est une liste d’objets PartitionEvent.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --end-time-utc   [requis] | Heure de fin d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| --start-time-utc [requis] | Heure de début d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| Events-types-filtre | Chaîne séparée par des virgules spécifiant les types de FabricEvents qui doivent être inclus uniquement dans la réponse. |
| --exclude-analysis-events | Ce paramètre désactive la récupération d’AnalysisEvents si la valeur true est transmise. |
| --skip-correlation-lookup | Ce paramètre désactive la recherche d’informations CorrelatedEvents si la valeur true est transmise. Dans le cas contraire, CorrelationEvents est traité et le champ HasCorrelatedEvents dans chaque FabricEvent est rempli. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-events-all-services-list"></a>sfctl events all-services-list
Obtient tous les événements liés aux services.

La réponse est une liste d’objets ServiceEvent.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --end-time-utc   [requis] | Heure de fin d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| --start-time-utc [requis] | Heure de début d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| Events-types-filtre | Chaîne séparée par des virgules spécifiant les types de FabricEvents qui doivent être inclus uniquement dans la réponse. |
| --exclude-analysis-events | Ce paramètre désactive la récupération d’AnalysisEvents si la valeur true est transmise. |
| --skip-correlation-lookup | Ce paramètre désactive la recherche d’informations CorrelatedEvents si la valeur true est transmise. Dans le cas contraire, CorrelationEvents est traité et le champ HasCorrelatedEvents dans chaque FabricEvent est rempli. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-events-application-list"></a>sfctl events application-list
Obtient un événement lié à une application.

La réponse est une liste d’objets ApplicationEvent.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --end-time-utc   [requis] | Heure de fin d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| --start-time-utc [requis] | Heure de début d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| Events-types-filtre | Chaîne séparée par des virgules spécifiant les types de FabricEvents qui doivent être inclus uniquement dans la réponse. |
| --exclude-analysis-events | Ce paramètre désactive la récupération d’AnalysisEvents si la valeur true est transmise. |
| --skip-correlation-lookup | Ce paramètre désactive la recherche d’informations CorrelatedEvents si la valeur true est transmise. Dans le cas contraire, CorrelationEvents est traité et le champ HasCorrelatedEvents dans chaque FabricEvent est rempli. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-events-cluster-list"></a>sfctl events cluster-list
Obtient tous les événements liés au cluster.

La réponse est une liste d’objets ClusterEvent.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --end-time-utc   [requis] | Heure de fin d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| --start-time-utc [requis] | Heure de début d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| Events-types-filtre | Chaîne séparée par des virgules spécifiant les types de FabricEvents qui doivent être inclus uniquement dans la réponse. |
| --exclude-analysis-events | Ce paramètre désactive la récupération d’AnalysisEvents si la valeur true est transmise. |
| --skip-correlation-lookup | Ce paramètre désactive la recherche d’informations CorrelatedEvents si la valeur true est transmise. Dans le cas contraire, CorrelationEvents est traité et le champ HasCorrelatedEvents dans chaque FabricEvent est rempli. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-events-node-list"></a>sfctl events node-list
Obtient un événement liés à un nœud.

La réponse est la liste d’objets NodeEvent.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --end-time-utc   [requis] | Heure de fin d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| --node-name      [obligatoire] | Nom du nœud. |
| --start-time-utc [requis] | Heure de début d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| Events-types-filtre | Chaîne séparée par des virgules spécifiant les types de FabricEvents qui doivent être inclus uniquement dans la réponse. |
| --exclude-analysis-events | Ce paramètre désactive la récupération d’AnalysisEvents si la valeur true est transmise. |
| --skip-correlation-lookup | Ce paramètre désactive la recherche d’informations CorrelatedEvents si la valeur true est transmise. Dans le cas contraire, CorrelationEvents est traité et le champ HasCorrelatedEvents dans chaque FabricEvent est rempli. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-events-partition-all-replicas-list"></a>sfctl events partition-all-replicas-list
Obtient tous les événements liés aux réplicas pour une partition.

La réponse est une liste d’objets ReplicaEvent.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --end-time-utc   [requis] | Heure de fin d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| --partition-id [requis] | Identité de la partition. |
| --start-time-utc [requis] | Heure de début d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| Events-types-filtre | Chaîne séparée par des virgules spécifiant les types de FabricEvents qui doivent être inclus uniquement dans la réponse. |
| --exclude-analysis-events | Ce paramètre désactive la récupération d’AnalysisEvents si la valeur true est transmise. |
| --skip-correlation-lookup | Ce paramètre désactive la recherche d’informations CorrelatedEvents si la valeur true est transmise. Dans le cas contraire, CorrelationEvents est traité et le champ HasCorrelatedEvents dans chaque FabricEvent est rempli. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-events-partition-list"></a>sfctl events partition-list
Obtient un événement lié à une partition.

La réponse est une liste d’objets PartitionEvent.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --end-time-utc   [requis] | Heure de fin d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| --partition-id [requis] | Identité de la partition. |
| --start-time-utc [requis] | Heure de début d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| Events-types-filtre | Chaîne séparée par des virgules spécifiant les types de FabricEvents qui doivent être inclus uniquement dans la réponse. |
| --exclude-analysis-events | Ce paramètre désactive la récupération d’AnalysisEvents si la valeur true est transmise. |
| --skip-correlation-lookup | Ce paramètre désactive la recherche d’informations CorrelatedEvents si la valeur true est transmise. Dans le cas contraire, CorrelationEvents est traité et le champ HasCorrelatedEvents dans chaque FabricEvent est rempli. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-events-partition-replica-list"></a>sfctl events partition-replica-list
Obtient un événement lié à un réplica de partition.

La réponse est une liste d’objets ReplicaEvent.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --end-time-utc   [requis] | Heure de fin d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| --partition-id [requis] | Identité de la partition. |
| --replica-id     [requis] | Identificateur du réplica. |
| --start-time-utc [requis] | Heure de début d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| Events-types-filtre | Chaîne séparée par des virgules spécifiant les types de FabricEvents qui doivent être inclus uniquement dans la réponse. |
| --exclude-analysis-events | Ce paramètre désactive la récupération d’AnalysisEvents si la valeur true est transmise. |
| --skip-correlation-lookup | Ce paramètre désactive la recherche d’informations CorrelatedEvents si la valeur true est transmise. Dans le cas contraire, CorrelationEvents est traité et le champ HasCorrelatedEvents dans chaque FabricEvent est rempli. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-events-service-list"></a>sfctl events service-list
Obtient un événement liés à un service.

La réponse est une liste d’objets ServiceEvent.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --end-time-utc   [requis] | Heure de fin d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| --service-id [Requis] | Identité du service. Cet ID est généralement le nom complet du service sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si un service est nommé « fabric\:/myapp/app1/svc1 », son identité est « myapp\~app1\~svc1 » dans les versions 6.0 et ultérieures, et « myapp/app1/svc1 » dans les versions précédentes. |
| --start-time-utc [requis] | Heure de début d’une requête de recherche en ISO UTC aaaa-MM-jjTHH\:mm\:ssZ. |
| Events-types-filtre | Chaîne séparée par des virgules spécifiant les types de FabricEvents qui doivent être inclus uniquement dans la réponse. |
| --exclude-analysis-events | Ce paramètre désactive la récupération d’AnalysisEvents si la valeur true est transmise. |
| --skip-correlation-lookup | Ce paramètre désactive la recherche d’informations CorrelatedEvents si la valeur true est transmise. Dans le cas contraire, CorrelationEvents est traité et le champ HasCorrelatedEvents dans chaque FabricEvent est rempli. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

