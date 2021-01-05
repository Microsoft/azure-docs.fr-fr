---
title: Interface CLI Azure Service Fabric - sfctl service
description: Apprenez-en plus sur sfctl, l’interface de ligne de commande d’Azure Service Fabric. Contient la liste des commandes permettant de gérer les services, les types de services et les packages de services.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: 426220f38e6ddfaaf8b24bf4f7d34473d881ae56
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86245600"
---
# <a name="sfctl-service"></a>sfctl service
Permet de créer, de supprimer et de gérer le service, les types de service et les packages de services.

## <a name="commands"></a>Commandes

|Commande|Description|
| --- | --- |
| app-name | Permet d’obtenir le nom de l’application Service Fabric d’un service. |
| code-package-list | Permet d’obtenir la liste des packages de code déployés sur un nœud Service Fabric. |
| create | Permet de créer le service Service Fabric spécifié. |
| delete | Supprime un service Service Fabric existant. |
| deployed-type | Permet d’obtenir les informations relatives à un type de service spécifié de l’application déployée sur un nœud dans un cluster Service Fabric. |
| deployed-type-list | Permet d’obtenir la liste contenant les informations sur les types de services à partir des applications déployées sur un nœud d’un cluster Service Fabric. |
| description | Permet d’obtenir la description d’un service Service Fabric existant. |
| get-container-logs | Permet d’obtenir les journaux d’activité du conteneur déployé sur un nœud Service Fabric. |
| health | Permet d’obtenir l’intégrité du service Service Fabric spécifié. |
| info | Permet d’obtenir les informations sur le service spécifique appartenant à l’application Service Fabric. |
| list | Permet d’obtenir les informations sur tous les services faisant partie de l’application spécifiée par l’ID d’application. |
| manifest | Permet d’obtenir le manifeste qui décrit un type de service. |
| package-deploy | Permet de télécharger les packages associés au manifeste de service spécifié dans le cache d’images du nœud spécifié. |
| package-health | Permet d’obtenir les informations sur l’intégrité d’un package de services d’une application spécifique déployée pour un nœud et une application Service Fabric. |
| package-info | Permet d’obtenir la liste des packages de service déployés sur un nœud Service Fabric correspondant exactement au nom spécifié. |
| package-list | Permet d’obtenir la liste des packages de services déployés sur un nœud Service Fabric. |
| recover | Indique au cluster Service Fabric qu’il doit tenter de récupérer le service spécifié qui est actuellement bloqué dans la perte de quorum. |
| report-health | Envoie un rapport d’intégrité sur le service Service Fabric. |
| resolve | Résout une partition Service Fabric. |
| type-list | Permet d’obtenir la liste contenant les informations sur les types de services pris en charge par un type d’application provisionné dans un cluster Service Fabric. |
| update | Met à jour le service spécifié à l’aide de la description de la mise à jour donnée. |

## <a name="sfctl-service-app-name"></a>sfctl service app-name
Permet d’obtenir le nom de l’application Service Fabric d’un service.

Permet d’obtenir le nom de l’application pour le service spécifié. Une erreur 404 FABRIC_E_SERVICE_DOES_NOT_EXIST est retournée si un service avec l’ID de service fourni n’existe pas.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
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

## <a name="sfctl-service-code-package-list"></a>sfctl service code-package-list
Permet d’obtenir la liste des packages de code déployés sur un nœud Service Fabric.

Permet d’obtenir la liste des packages de code déployés sur un nœud Service Fabric pour l’application donnée.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --node-name      [obligatoire] | Nom du nœud. |
| --code-package-name | Nom de package de code spécifié dans un manifeste de service inscrit dans le cadre d’un type d’application dans un cluster Service Fabric. |
| --service-manifest-name | Nom d’un manifeste de service inscrit dans le cadre d’un type d’application dans un cluster Service Fabric. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-service-create"></a>sfctl service create
Permet de créer le service Service Fabric spécifié.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --app-id [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --name [Requis] | Nom du service. Ce doit être un enfant de l’ID d’application. Il doit s’agir du nom complet incluant l’URI `fabric\:`. Par exemple, le service `fabric\:/A/B` est un enfant de l’application `fabric\:/A`. |
| --service-type [Requis] | Nom du type de service. |
| --activation-mode | Mode d’activation pour le package de services. |
| --constraints | Contraintes de placement sous forme de chaîne. Les contraintes de placement sont des expressions booléennes sur les propriétés de nœud et permettent de restreindre un service à des nœuds particuliers en fonction des besoins du service. Par exemple, pour placer un service sur des nœuds où la valeur NodeType est blue, spécifiez \: « NodeColor == blue ». |
| --correlated-service | Nom du service cible avec lequel effectuer une mise en corrélation. |
| --correlation | Permet de mettre le service en corrélation avec un service existant à l’aide d’une affinité d’alignement. |
| --dns-name | Nom DNS du service à créer. Le service système DNS de Service Fabric doit être activé pour ce paramètre. |
| --instance-count | Nombre d’instances. Cela s’applique uniquement aux services sans état. |
| --int-scheme | Indique que le service doit être partitionné de manière uniforme dans une plage d’entiers non signés. |
| --int-scheme-count | Nombre de partitions dans la plage de clés de type entier à créer, si vous utilisez un schéma de partition d’entier uniforme. |
| --int-scheme-high | Fin de la plage de clés de type entier, si vous utilisez un schéma de partition d’entier uniforme. |
| --int-scheme-low | Début de la plage de clés de type entier, si vous utilisez un schéma de partition d’entier uniforme. |
| --load-metrics | Liste de mesures, encodée au format JSON, utilisée lors de l’équilibrage de charge des services entre les nœuds. |
| --min-replica-set-size | Taille minimale du jeu de réplicas en tant que nombre. Cela s’applique uniquement aux services avec état. |
| --move-cost | Spécifie le coût du déplacement du service. Les valeurs possibles sont\: « Zero », « Low », « Medium », « High » et « VeryHigh ». |
| --named-scheme | Indique que le service doit comporter plusieurs partitions nommées. |
| --named-scheme-list | Liste de noms, encodée au format JSON, de partition du service si vous utilisez le schéma de partition nommé. |
| --no-persisted-state | Si cet argument est défini sur True, cela indique que le service n’a pas d’état persistant stocké sur le disque local ou qu’il stocke l’état uniquement en mémoire. |
| --placement-policy-list | Liste, encodée au format JSON, de stratégies de placement pour le service, et les noms de domaine associés. Les stratégies peuvent être les suivantes (une ou plusieurs) \: `NonPartiallyPlaceService`, `PreferPrimaryDomain`, `RequireDomain`, `RequireDomainDistribution`. |
| --quorum-loss-wait | Durée maximale, en secondes, pendant laquelle une partition est autorisée à être à l’état de perte de quorum. Cela s’applique uniquement aux services avec état. |
| --replica-restart-wait | Durée, en secondes, entre l’arrêt d’un réplica et la création d’un nouveau réplica. Cela s’applique uniquement aux services avec état. |
| --scaling-policies | Liste, encodée au format JSON, des stratégies de mise à l’échelle pour ce service. |
| --service-placement-time | Durée pendant laquelle les réplicas peuvent rester InBuild avant d'indiquer que la build est bloquée. Cela s’applique uniquement aux services avec état. |
| --singleton-scheme | Indique que le service doit avoir une seule partition ou être un service non partitionné. |
| --stand-by-replica-keep | Durée maximale, en secondes, pendant laquelle les réplicas StandBy sont conservés avant d’être supprimés. Cela s’applique uniquement aux services avec état. |
| --stateful | Indique qu’il s’agit d’un service avec état. |
| --stateless | Indique qu’il s’agit d’un service sans état. |
| --target-replica-set-size | Taille cible du jeu de réplicas en tant que nombre. Cela s’applique uniquement aux services avec état. |
| --timeout -t | Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-service-delete"></a>sfctl service delete
Supprime un service Service Fabric existant.

Un service doit être créé avant de pouvoir être supprimé. Par défaut, Service Fabric essaie de fermer les réplicas de service sans perte de données, puis supprime le service. Toutefois, si le service rencontre des problèmes de fermeture normale des réplicas, l’opération de suppression peut prendre un certain temps ou bloquer. Utilisez l’indicateur ForceRemove facultatif pour ignorer la séquence de fermeture normale et forcer la suppression du service.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --service-id [Requis] | Identité du service. Cet ID est généralement le nom complet du service sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si un service est nommé « fabric\:/myapp/app1/svc1 », son identité est « myapp\~app1\~svc1 » dans les versions 6.0 et ultérieures, et « myapp/app1/svc1 » dans les versions précédentes. |
| --force-remove | Force la suppression d’un service ou d’une application Service Fabric, sans passer par la séquence d’arrêt normale. Ce paramètre permet de forcer la suppression d’une application ou d’un service pour lesquels le délai de suppression expire à cause de problèmes dans le code de service qui empêchent la fermeture normale des réplicas. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-service-deployed-type"></a>sfctl service deployed-type
Permet d’obtenir les informations relatives à un type de service spécifié de l’application déployée sur un nœud dans un cluster Service Fabric.

Permet d’obtenir la liste contenant les informations sur un type de service spécifique à partir des applications déployées sur un nœud d’un cluster Service Fabric. La réponse inclut le nom du type de service, l’état de son inscription, le package de code l’ayant inscrit et l’ID d’activation du package de service. Chaque entrée représente une activation d’un type de service, différentiée par l’ID d’activation.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id    [obligatoire] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --node-name      [obligatoire] | Nom du nœud. |
| --service-type-name [obligatoire] | Spécifie le nom d’un type de service Service Fabric. |
| --service-manifest-name | Nom du manifeste de service utilisé pour filtrer la liste des informations sur le type de service déployé. Si cet argument spécifié, la réponse contient uniquement les informations sur les types de services qui sont définis dans ce manifeste de service. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-service-deployed-type-list"></a>sfctl service deployed-type-list
Permet d’obtenir la liste contenant les informations sur les types de services à partir des applications déployées sur un nœud d’un cluster Service Fabric.

Permet d’obtenir la liste contenant les informations sur les types de services à partir des applications déployées sur un nœud d’un cluster Service Fabric. La réponse inclut le nom du type de service, l’état de son inscription, le package de code l’ayant inscrit et l’ID d’activation du package de service.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --node-name      [obligatoire] | Nom du nœud. |
| --service-manifest-name | Nom du manifeste de service utilisé pour filtrer la liste des informations sur le type de service déployé. Si cet argument spécifié, la réponse contient uniquement les informations sur les types de services qui sont définis dans ce manifeste de service. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-service-description"></a>sfctl service description
Permet d’obtenir la description d’un service Service Fabric existant.

Permet d’obtenir la description d’un service Service Fabric existant. Vous devez créer un service avant de pouvoir obtenir sa description.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
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

## <a name="sfctl-service-get-container-logs"></a>sfctl service get-container-logs
Permet d’obtenir les journaux d’activité du conteneur déployé sur un nœud Service Fabric.

Permet d’obtenir les journaux d’activité du conteneur déployé sur un nœud Service Fabric pour le package de code donné.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id        [obligatoire] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --code-package-name     [obligatoire] | Nom de package de code spécifié dans un manifeste de service inscrit dans le cadre d’un type d’application dans un cluster Service Fabric. |
| --node-name             [obligatoire] | Nom du nœud. |
| --service-manifest-name [obligatoire] | Nom d’un manifeste de service inscrit dans le cadre d’un type d’application dans un cluster Service Fabric. |
| --previous | Indique s’il faut récupérer les journaux d’activité des conteneurs quittés/morts de l’instance de package de code. |
| --tail | Nombre de lignes à afficher à partir de la fin des journaux d’activité. La valeur par défaut est 100. « all » pour afficher les journaux d’activité complets. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-service-health"></a>sfctl service health
Permet d’obtenir l’intégrité du service Service Fabric spécifié.

Permet d’obtenir les informations d’intégrité du service spécifié. EventsHealthStateFilter permet de filtrer la collection d’événements d’intégrité signalés dans le service en fonction de l’état d’intégrité. Utilisez PartitionsHealthStateFilter pour filtrer la collection de partitions retournée. Si vous spécifiez un service qui n’existe pas dans le magasin d’intégrité, cette requête retourne une erreur.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --service-id [Requis] | Identité du service. Cet ID est généralement le nom complet du service sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si un service est nommé « fabric\:/myapp/app1/svc1 », son identité est « myapp\~app1\~svc1 » dans les versions 6.0 et ultérieures, et « myapp/app1/svc1 » dans les versions précédentes. |
| --events-health-state-filter | Permet de filtrer la collection d’objets HealthEvent retournés en fonction de leur état d’intégrité. Les valeurs possibles de ce paramètre incluent la valeur entière de l’un des états d’intégrité suivants. Seuls les événements qui correspondent au filtre sont renvoyés. Tous les événements sont utilisés pour évaluer l’état d’intégrité agrégé. Si cet argument n’est pas spécifié, toutes les entrées sont retournées. Les valeurs d’état correspondent à une énumération basée sur des indicateurs. La valeur peut donc être une combinaison de ces valeurs obtenue à l’aide de l’opérateur « OR » au niveau du bit. Par exemple, si la valeur indiquée est 6, tous les événements dont la valeur HealthState est OK (2) et Warning (4) sont retournés.  <br> - Default : valeur par défaut. Correspond à toute valeur HealthState. La valeur est égale à zéro.  <br> - None : filtre qui ne correspond à aucune valeur HealthState. Permet de ne retourner aucun résultat sur une collection donnée d’états. La valeur est égale à 1.  <br> - OK : filtre qui correspond à l’entrée ayant OK comme valeur HealthState. La valeur est égale à 2.  <br> - Warning : filtre qui correspond à l’entrée ayant Warning comme valeur HealthState. La valeur est égale à 4.  <br> - Error : filtre qui correspond à l’entrée ayant Error comme valeur HealthState. La valeur est égale à 8.  <br> - All : filtre qui correspond à l’entrée ayant n’importe quelle valeur HealthState. La valeur est égale à 65535. |
| --exclude-health-statistics | Indique si les statistiques d’intégrité doivent être retournées dans le cadre du résultat de la requête. False par défaut. Les statistiques affichent le nombre d’entités enfants dont l’état d’intégrité est OK, Warning et Error. |
| --partitions-health-state-filter | Permet de filtrer les objets d’état d’intégrité des partitions retournés dans le résultat de la requête d’intégrité du service en fonction de leur état d’intégrité. Les valeurs possibles de ce paramètre incluent la valeur entière de l’un des états d’intégrité suivants. Seules les partitions qui correspondent au filtre sont retournées. Toutes les partitions sont utilisées pour évaluer l’état d’intégrité agrégé. Si cet argument n’est pas spécifié, toutes les entrées sont retournées. Les valeurs d’état correspondent à une énumération basée sur des indicateurs. La valeur peut donc être une combinaison de ces valeurs obtenue à l’aide de l’opérateur « OR » au niveau du bit. Par exemple, si la valeur indiquée est 6, l’état d’intégrité des partitions dont la valeur HealthState est OK (2) et Warning (4) est retourné.  <br> - Default : valeur par défaut. Correspond à toute valeur HealthState. La valeur est égale à zéro.  <br> - None : filtre qui ne correspond à aucune valeur HealthState. Permet de ne retourner aucun résultat sur une collection donnée d’états. La valeur est égale à 1.  <br> - OK : filtre qui correspond à l’entrée ayant OK comme valeur HealthState. La valeur est égale à 2.  <br> - Warning : filtre qui correspond à l’entrée ayant Warning comme valeur HealthState. La valeur est égale à 4.  <br> - Error : filtre qui correspond à l’entrée ayant Error comme valeur HealthState. La valeur est égale à 8.  <br> - All : filtre qui correspond à l’entrée ayant n’importe quelle valeur HealthState. La valeur est égale à 65535. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-service-info"></a>sfctl service info
Permet d’obtenir les informations sur le service spécifique appartenant à l’application Service Fabric.

Retourne les informations sur le service spécifié appartenant à l’application Service Fabric indiquée.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
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

## <a name="sfctl-service-list"></a>sfctl service list
Permet d’obtenir les informations sur tous les services faisant partie de l’application spécifiée par l’ID d’application.

Retourne les informations sur tous les services faisant partie de l’application spécifiée par l’ID d’application.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --continuation-token | Le paramètre de jeton de liaison permet d’obtenir le jeu de résultats suivant. Un jeton de liaison avec une valeur non vide est inclus dans la réponse de l’API quand les résultats du système ne tiennent pas dans une seule réponse. Lorsque cette valeur est transmise à l’appel d’API suivant, l’API retourne le jeu de résultats suivant. S’il n’existe pas de résultats supplémentaires, le jeton de liaison ne contient pas de valeur. La valeur de ce paramètre ne doit pas être codée URL. |
| --service-type-name | Nom du type de service utilisé pour filtrer les services correspondant à la requête. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-service-manifest"></a>sfctl service manifest
Permet d’obtenir le manifeste qui décrit un type de service.

Permet d’obtenir le manifeste qui décrit un type de service. La réponse contient le code XML du manifeste de service sous forme de chaîne.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-type-name [Requis] | Nom du type d’application. |
| --application-type-version [Requis] | Version du type d’application. |
| --service-manifest-name [Requis] | Nom d’un manifeste de service inscrit dans le cadre d’un type d’application dans un cluster Service Fabric. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-service-package-deploy"></a>sfctl service package-deploy
Permet de télécharger les packages associés au manifeste de service spécifié dans le cache d’images du nœud spécifié.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --app-type-name         [obligatoire] | Nom du manifeste d’application pour le manifeste de service demandé correspondant. |
| --app-type-version      [obligatoire] | Version du manifeste d’application pour le manifeste de service demandé correspondant. |
| --node-name             [obligatoire] | Nom du nœud. |
| --service-manifest-name [obligatoire] | Nom du manifeste de service associé aux packages à télécharger. |
| --share-policy | Liste, encodée au format JSON, des stratégies de partage. Chaque élément de stratégie de partage est composé d’un nom (« name ») et d’une étendue (« scope »). Le nom correspond au nom du code, de la configuration ou du package de données à partager. L’étendue peut être « None », « All », « Code », « Config » ou « Data ». |
| --timeout -t | Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-service-package-health"></a>sfctl service package-health
Permet d’obtenir les informations sur l’intégrité d’un package de services d’une application spécifique déployée pour un nœud et une application Service Fabric.

Permet d’obtenir les informations sur l’intégrité d’un package de services d’une application spécifique déployée sur un nœud Service Fabric. Utilisez EventsHealthStateFilter pour filtrer éventuellement la collection d’objets HealthEvent signalés sur le package de service déployé en fonction d’un état d’intégrité.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id       [obligatoire] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --node-name            [obligatoire] | Nom du nœud. |
| --service-package-name [obligatoire] | Nom du package de service. |
| --events-health-state-filter | Permet de filtrer la collection d’objets HealthEvent retournés en fonction de leur état d’intégrité. Les valeurs possibles de ce paramètre incluent la valeur entière de l’un des états d’intégrité suivants. Seuls les événements qui correspondent au filtre sont renvoyés. Tous les événements sont utilisés pour évaluer l’état d’intégrité agrégé. Si cet argument n’est pas spécifié, toutes les entrées sont retournées. Les valeurs d’état correspondent à une énumération basée sur des indicateurs. La valeur peut donc être une combinaison de ces valeurs obtenue à l’aide de l’opérateur « OR » au niveau du bit. Par exemple, si la valeur indiquée est 6, tous les événements dont la valeur HealthState est OK (2) et Warning (4) sont retournés.  <br> - Default : valeur par défaut. Correspond à toute valeur HealthState. La valeur est égale à zéro.  <br> - None : filtre qui ne correspond à aucune valeur HealthState. Permet de ne retourner aucun résultat sur une collection donnée d’états. La valeur est égale à 1.  <br> - OK : filtre qui correspond à l’entrée ayant OK comme valeur HealthState. La valeur est égale à 2.  <br> - Warning : filtre qui correspond à l’entrée ayant Warning comme valeur HealthState. La valeur est égale à 4.  <br> - Error : filtre qui correspond à l’entrée ayant Error comme valeur HealthState. La valeur est égale à 8.  <br> - All : filtre qui correspond à l’entrée ayant n’importe quelle valeur HealthState. La valeur est égale à 65535. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-service-package-info"></a>sfctl service package-info
Permet d’obtenir la liste des packages de service déployés sur un nœud Service Fabric correspondant exactement au nom spécifié.

Retourne les informations sur les packages de code déployés sur un nœud Service Fabric pour l’application donnée. Ces résultats sont des packages de service dont le nom correspond exactement au nom de package de service spécifié comme paramètre.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id       [obligatoire] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --node-name            [obligatoire] | Nom du nœud. |
| --service-package-name [obligatoire] | Nom du package de service. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-service-package-list"></a>sfctl service package-list
Permet d’obtenir la liste des packages de services déployés sur un nœud Service Fabric.

Retourne les informations sur les packages de code déployés sur un nœud Service Fabric pour l’application donnée.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --node-name      [obligatoire] | Nom du nœud. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-service-recover"></a>sfctl service recover
Indique au cluster Service Fabric qu’il doit tenter de récupérer le service spécifié qui est actuellement bloqué dans la perte de quorum.

Indique au cluster Service Fabric qu’il doit tenter de récupérer le service spécifié qui est actuellement bloqué dans la perte de quorum. Cette opération doit être effectuée uniquement s’il est avéré que les réplicas arrêtés ne peuvent pas être récupérés. Une utilisation incorrecte de cette API peut entraîner une perte de données potentielle.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
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

## <a name="sfctl-service-report-health"></a>sfctl service report-health
Envoie un rapport d’intégrité sur le service Service Fabric.

Signale l’état d’intégrité du service Service Fabric spécifié. Le rapport doit contenir les informations sur la source du rapport d’intégrité et sur la propriété dont il fait état. Il est envoyé à un service de passerelle Service Fabric, qui opère son transfert vers le magasin d’intégrité. Le rapport peut être accepté par la passerelle, mais rejeté par le magasin d’intégrité après une validation supplémentaire. Par exemple, le magasin d’intégrité peut rejeter le rapport en raison d’un paramètre non valide, comme un numéro de séquence obsolète. Pour voir si le rapport a été appliqué dans le magasin d’intégrité, vérifiez qu’il s’affiche dans les événements d’intégrité du service.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --health-property [obligatoire] | Propriété des informations d’intégrité. <br><br> Une entité peut avoir des rapports d’intégrité pour différentes propriétés. La propriété est une chaîne et non une énumération fixe pour offrir au rapporteur la possibilité de catégoriser la condition d’état qui déclenche le rapport. Par exemple, un rapporteur avec l’ID source (SourceId) « LocalWatchdog » peut surveiller l’état du disque disponible sur un nœud. Il peut donc signaler la propriété « AvailableDisk » sur ce nœud. Le même rapporteur peut surveiller la connectivité du nœud. Il peut donc signaler une propriété « Connectivity » sur le même nœud. Dans le magasin d’intégrité, ces rapports sont traités comme des événements d’intégrité distincts pour le nœud spécifié. Avec l’ID source (SourceId), la propriété identifie de façon unique les informations d’intégrité. |
| --health-state    [obligatoire] | Les valeurs possibles sont \: « Invalid », « Ok », « Warning », « Error », « Unknown ». |
| --service-id      [obligatoire] | Identité du service. <br><br> Il s’agit généralement du nom complet du service sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si un service est nommé « fabric\:/myapp/app1/svc1 », son identité est « myapp\~app1\~svc1 » dans les versions 6.0 et ultérieures, et « myapp/app1/svc1 » dans les versions précédentes. |
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

## <a name="sfctl-service-resolve"></a>sfctl service resolve
Résout une partition Service Fabric.

Résout une partition de service Service Fabric pour obtenir les points de terminaison des réplicas de service.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --service-id [Requis] | Identité du service. Cet ID est généralement le nom complet du service sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si un service est nommé « fabric\:/myapp/app1/svc1 », son identité est « myapp\~app1\~svc1 » dans les versions 6.0 et ultérieures, et « myapp/app1/svc1 » dans les versions précédentes. |
| --partition-key-type | Type de clé de la partition. Ce paramètre est obligatoire si le schéma de partition du service est Int64Range ou Named. Les valeurs possibles sont les suivantes. - None (1) : indique que le paramètre PartitionKeyValue n’est pas spécifié. Cela concerne les partitions dont le schéma de partitionnement est Singleton. Il s’agit de la valeur par défaut. La valeur est égale à 1. - Int64Range (2) : indique que le paramètre PartitionKeyValue est une clé de partition int64. Cela concerne les partitions dont le schéma de partitionnement est Int64Range. La valeur est égale à 2. - Named (3) : indique que le paramètre PartitionKeyValue est le nom de la partition. Cela concerne les partitions dont le schéma de partitionnement est Named. La valeur est 3. |
| --partition-key-value | Clé de partition. Ce paramètre est obligatoire si le schéma de partition du service est Int64Range ou Named. Il ne s’agit pas de l’ID de partition, mais de la valeur de clé de type entier ou du nom de l’ID de partition. Par exemple, si votre service utilise des partitions par spécification de plages de valeurs de 0 à 10, alors PartitionKeyValue est un entier dans cette plage. Envoyez une requête de description du service pour afficher la plage ou le nom correspondant. |
| --previous-rsp-version | Valeur du champ Version de la réponse qui a été reçue précédemment. Cet argument est obligatoire si l’utilisateur sait que le résultat obtenu précédemment est périmé. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-service-type-list"></a>sfctl service type-list
Permet d’obtenir la liste contenant les informations sur les types de services pris en charge par un type d’application provisionné dans un cluster Service Fabric.

Permet d’obtenir la liste contenant les informations sur les types de services pris en charge par un type d’application provisionné dans un cluster Service Fabric. Le type d’application fourni doit exister. Sinon, un état 404 est retourné.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-type-name [Requis] | Nom du type d’application. |
| --application-type-version [Requis] | Version du type d’application. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-service-update"></a>sfctl service update
Met à jour le service spécifié à l’aide de la description de la mise à jour donnée.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --service-id [Requis] | Identité du service. Il s’agit généralement du nom complet du service sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si un service est nommé « fabric\:/myapp/app1/svc1 », son identité est « myapp\~app1\~svc1 » dans les versions 6.0 et ultérieures, et « myapp/app1/svc1 » dans les versions précédentes. |
| --constraints | Contraintes de placement sous forme de chaîne. Les contraintes de placement sont des expressions booléennes sur les propriétés de nœud et permettent de restreindre un service à des nœuds particuliers en fonction des besoins du service. Par exemple, pour placer un service sur des nœuds où NodeType est blue, spécifiez \: « NodeColor == blue ». |
| --correlated-service | Nom du service cible avec lequel effectuer une mise en corrélation. |
| --correlation | Permet de mettre le service en corrélation avec un service existant à l’aide d’une affinité d’alignement. |
| --instance-count | Nombre d’instances. Cela s’applique uniquement aux services sans état. |
| --load-metrics | Liste de métriques, encodée au format JSON, utilisée lors de l’équilibrage de charge des services entre les nœuds. |
| --min-replica-set-size | Taille minimale du jeu de réplicas en tant que nombre. Cela s’applique uniquement aux services avec état. |
| --move-cost | Spécifie le coût du déplacement du service. Les valeurs possibles sont\: « Zero », « Low », « Medium », « High » et « VeryHigh ». |
| --placement-policy-list | Liste, encodée au format JSON, de stratégies de placement pour le service, et les noms de domaine associés. Les stratégies peuvent être les suivantes (une ou plusieurs) \: `NonPartiallyPlaceService`, `PreferPrimaryDomain`, `RequireDomain`, `RequireDomainDistribution`. |
| --quorum-loss-wait | Durée maximale, en secondes, pendant laquelle une partition est autorisée à être à l’état de perte de quorum. Cela s’applique uniquement aux services avec état. |
| --replica-restart-wait | Durée, en secondes, entre l’arrêt d’un réplica et la création d’un nouveau réplica. Cela s’applique uniquement aux services avec état. |
| --scaling-policies | Liste, encodée au format JSON, des stratégies de mise à l’échelle pour ce service. |
| --service-placement-time | Durée pendant laquelle les réplicas peuvent rester InBuild avant d'indiquer que la build est bloquée. Cela s’applique uniquement aux services avec état. |
| --stand-by-replica-keep | Durée maximale, en secondes, pendant laquelle les réplicas StandBy sont conservés avant d’être supprimés. Cela s’applique uniquement aux services avec état. |
| --stateful | Indique que le service cible est un service avec état. |
| --stateless | Indique que le service cible est un service sans état. |
| --target-replica-set-size | Taille cible du jeu de réplicas en tant que nombre. Cela s’applique uniquement aux services avec état. |
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
