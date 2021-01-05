---
title: Interface CLI Azure Service Fabric - sfctl application
description: Apprenez-en plus sur sfctl, l’interface de ligne de commande d’Azure Service Fabric. Inclut la liste des commandes liées à la gestion des applications.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: 880770345eb7d65850db322bd97d64c60b6681ee
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86260863"
---
# <a name="sfctl-application"></a>sfctl application
Permet de créer, de supprimer et de gérer les applications et les types d’application.

## <a name="commands"></a>Commandes

|Commande|Description|
| --- | --- |
| create | Permet de créer une application Service Fabric à l’aide de la description spécifiée. |
| delete | Supprime une application Service Fabric existante. |
| deployed | Permet d’obtenir les informations relatives à une application déployée sur un nœud Service Fabric. |
| deployed-health | Permet d’obtenir les informations relatives à l’intégrité d’une application déployée sur un nœud Service Fabric. |
| deployed-list | Permet d’obtenir la liste des applications déployées sur un nœud Service Fabric. |
| health | Permet d’obtenir l’intégrité de l’application Service Fabric. |
| info | Permet d’obtenir des informations sur une application Service Fabric. |
| list | Permet d’obtenir la liste des applications créées dans le cluster Service Fabric qui correspondent aux filtres spécifiés. |
| load | Permet d’obtenir les informations relatives à une application Service Fabric. |
| manifest | Permet d’obtenir le manifeste qui décrit un type d’application. |
| provision | Permet d’approvisionner ou d’inscrire un type d’application Service Fabric auprès du cluster à l’aide du package « .sfpkg » dans le magasin externe ou du package d’application dans le magasin d’images. |
| report-health | Permet d’envoyer un rapport d’intégrité sur l’application Service Fabric. |
| type | Permet d’obtenir la liste des types d’applications du cluster Service Fabric qui correspondent exactement au nom spécifié. |
| type-list | Permet d’obtenir la liste des types d’applications du cluster Service Fabric. |
| unprovision | Permet de supprimer ou d’annuler l’inscription d’un type d’application Service Fabric dans le cluster. |
| upgrade | Commence la mise à niveau d’une application dans le cluster Service Fabric. |
| upgrade-resume | Commence la mise à niveau d’une application dans le cluster Service Fabric. |
| upgrade-rollback | Annule la mise à niveau en cours d’une application du cluster Service Fabric. |
| upgrade-status | Permet d’obtenir les détails de la dernière mise à jour effectuée sur cette application. |
| upload | Copie un package d’application Service Fabric dans le magasin d’images. |

## <a name="sfctl-application-create"></a>sfctl application create
Permet de créer une application Service Fabric à l’aide de la description spécifiée.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --app-name    [Requis] | Nom de l’application, y compris le schéma d’URI « fabric\: ». |
| --app-type    [Requis] | Nom du type d’application trouvé dans le manifeste d’application. |
| --app-version [Requis] | Version du type d’application indiquée dans le manifeste de l’application. |
| --max-node-count | Nombre maximal de nœuds sur lesquels Service Fabric réserve de la capacité pour cette application. Notez que cela ne signifie pas que les services de cette application sont placés sur tous les nœuds. |
| --metrics | Liste encodée au format JSON des descriptions des mesures de capacité de l’application. Une métrique est définie comme un nom, associé à un ensemble de capacités pour chaque nœud sur lequel l’application existe. |
| --min-node-count | Nombre minimal de nœuds sur lesquels Service Fabric réserve de la capacité pour cette application. Notez que cela ne signifie pas que les services de cette application sont placés sur tous les nœuds. |
| --parameters | Liste encodée au format JSON des remplacements de paramètres de l’application à appliquer lors de la création de l’application. |
| --timeout -t | Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-delete"></a>sfctl application delete
Supprime une application Service Fabric existante.

Une application doit être créée avant de pouvoir être supprimée. La suppression d’une application entraîne la suppression de tous les services qui font partie de l’application. Par défaut, Service Fabric essaie de fermer les réplicas de service sans perte de données, puis supprime le service. Toutefois, si un service rencontre des problèmes de fermeture normale des réplicas, l’opération de suppression peut prendre un certain temps ou bloquer. Utilisez l’indicateur ForceRemove facultatif pour ignorer la séquence de fermeture normale et forcer la suppression de l’application et de tous ses services.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
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

## <a name="sfctl-application-deployed"></a>sfctl application deployed
Permet d’obtenir les informations relatives à une application déployée sur un nœud Service Fabric.

Cette requête renvoie des informations sur l’application système si l’ID d’application fourni est celui de l’application système. Les résultats incluent les applications déployées dont l’état est défini sur active, activating (activation) ou downloading (téléchargement). Pour cette requête, le nom de nœud doit obligatoirement correspondre à un nœud du cluster. Si le nom de nœud fourni ne pointe pas vers un nœud Service Fabric actif du cluster, la requête échoue.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --node-name      [obligatoire] | Nom du nœud. |
| --include-health-state | Inclut l’état d’intégrité d’une entité. Si ce paramètre est défini sur false ou n’est pas spécifié, l’état d’intégrité retourné est « Inconnu ». Lorsque la valeur est true, la requête s’applique en parallèle au nœud et au service système d’intégrité avant la fusion des résultats. Par conséquent, la requête est plus coûteuse et peut prendre plus de temps. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-deployed-health"></a>sfctl application deployed-health
Permet d’obtenir les informations relatives à l’intégrité d’une application déployée sur un nœud Service Fabric.

Permet d’obtenir les informations relatives à l’intégrité d’une application déployée sur un nœud Service Fabric. Utilisez EventsHealthStateFilter pour filtrer éventuellement la collection d’objets HealthEvent signalés sur l’application déployée en fonction d’un état d’intégrité. Utilisez DeployedServicePackagesHealthStateFilter pour filtrer éventuellement les enfants de DeployedServicePackageHealth en fonction d’un état d’intégrité.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id                     [obligatoire] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --node-name                          [obligatoire] | Nom du nœud. |
| --deployed-service-packages-health-state-filter | Permet de filtrer, par état d’intégrité, les objets d’état d’intégrité des packages de services déployés qui sont retournés dans le résultat de la requête d’intégrité de l’application déployée. Les valeurs possibles de ce paramètre incluent la valeur entière de l’un des états d’intégrité suivants. Seuls les packages de services déployés qui correspondent au filtre sont retournés. Tous les packages de services déployés sont utilisés pour évaluer l’état d’intégrité agrégé de l’application déployée. Si cet argument n’est pas spécifié, toutes les entrées sont retournées. Les valeurs d’état correspondent à une énumération basée sur des indicateurs. La valeur peut donc être une combinaison de ces valeurs obtenue à l’aide de l’opérateur « OR » au niveau du bit. Par exemple, si la valeur indiquée est 6, l’état d’intégrité des packages de services dont la valeur HealthState est OK (2) et Warning (4) est retourné.  <br> - Default : valeur par défaut. Correspond à toute valeur HealthState. La valeur est égale à zéro.  <br> - None : filtre qui ne correspond à aucune valeur HealthState. Permet de ne retourner aucun résultat sur une collection donnée d’états. La valeur est égale à 1.  <br> - OK : filtre qui correspond à l’entrée ayant OK comme valeur HealthState. La valeur est égale à 2.  <br> - Warning : filtre qui correspond à l’entrée ayant Warning comme valeur HealthState. La valeur est égale à 4.  <br> - Error : filtre qui correspond à l’entrée ayant Error comme valeur HealthState. La valeur est égale à 8.  <br> - All : filtre qui correspond à l’entrée ayant n’importe quelle valeur HealthState. La valeur est égale à 65535. |
| --events-health-state-filter | Permet de filtrer la collection d’objets HealthEvent retournés en fonction de leur état d’intégrité. Les valeurs possibles de ce paramètre incluent la valeur entière de l’un des états d’intégrité suivants. Seuls les événements qui correspondent au filtre sont renvoyés. Tous les événements sont utilisés pour évaluer l’état d’intégrité agrégé. Si cet argument n’est pas spécifié, toutes les entrées sont retournées. Les valeurs d’état correspondent à une énumération basée sur des indicateurs. La valeur peut donc être une combinaison de ces valeurs obtenue à l’aide de l’opérateur « OR » au niveau du bit. Par exemple, si la valeur indiquée est 6, tous les événements dont la valeur HealthState est OK (2) et Warning (4) sont retournés.  <br> - Default : valeur par défaut. Correspond à toute valeur HealthState. La valeur est égale à zéro.  <br> - None : filtre qui ne correspond à aucune valeur HealthState. Permet de ne retourner aucun résultat sur une collection donnée d’états. La valeur est égale à 1.  <br> - OK : filtre qui correspond à l’entrée ayant OK comme valeur HealthState. La valeur est égale à 2.  <br> - Warning : filtre qui correspond à l’entrée ayant Warning comme valeur HealthState. La valeur est égale à 4.  <br> - Error : filtre qui correspond à l’entrée ayant Error comme valeur HealthState. La valeur est égale à 8.  <br> - All : filtre qui correspond à l’entrée ayant n’importe quelle valeur HealthState. La valeur est égale à 65535. |
| --exclude-health-statistics | Indique si les statistiques d’intégrité doivent être retournées dans le cadre du résultat de la requête. False par défaut. Les statistiques affichent le nombre d’entités enfants dont l’état d’intégrité est OK, Warning et Error. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-deployed-list"></a>sfctl application deployed-list
Permet d’obtenir la liste des applications déployées sur un nœud Service Fabric.

Permet d’obtenir la liste des applications déployées sur un nœud Service Fabric. Les résultats n’incluent pas d’informations sur les applications système déployées, sauf si elles sont explicitement recherchées par ID. Les résultats incluent les applications déployées dont l’état est défini sur active, activating (activation) ou downloading (téléchargement). Pour cette requête, le nom de nœud doit obligatoirement correspondre à un nœud du cluster. Si le nom de nœud fourni ne pointe pas vers un nœud Service Fabric actif du cluster, la requête échoue.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --node-name [Requis] | Nom du nœud. |
| --continuation-token | Le paramètre de jeton de liaison permet d’obtenir le jeu de résultats suivant. Un jeton de liaison avec une valeur non vide est inclus dans la réponse de l’API quand les résultats du système ne tiennent pas dans une seule réponse. Lorsque cette valeur est transmise à l’appel d’API suivant, l’API retourne le jeu de résultats suivant. S’il n’existe pas de résultats supplémentaires, le jeton de liaison ne contient pas de valeur. La valeur de ce paramètre ne doit pas être codée URL. |
| --include-health-state | Inclut l’état d’intégrité d’une entité. Si ce paramètre est défini sur false ou n’est pas spécifié, l’état d’intégrité retourné est « Inconnu ». Lorsque la valeur est true, la requête s’applique en parallèle au nœud et au service système d’intégrité avant la fusion des résultats. Par conséquent, la requête est plus coûteuse et peut prendre plus de temps. |
| --max-results | Nombre maximal de résultats à renvoyer dans le cadre des requêtes paginées. Ce paramètre définit la limite supérieure du nombre de résultats renvoyés. Le nombre de résultats renvoyés peut être inférieur au nombre maximal de résultats spécifié s’ils ne tiennent pas dans le message conformément aux restrictions de taille maximale définies dans la configuration. Si ce paramètre est défini sur zéro ou n’est pas spécifié, la requête paginée comprend le nombre maximal de résultats pouvant tenir dans le message renvoyé. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-health"></a>sfctl application health
Permet d’obtenir l’intégrité de l’application Service Fabric.

Permet d’obtenir l’état d’intégrité de l’application Service Fabric. La réponse indique l’état d’intégrité Ok, Error ou Warning. Si l’entité est introuvable dans le magasin d’intégrité, il renvoie des erreurs.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id                 [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --deployed-applications-health-state-filter | Permet de filtrer, par état d’intégrité, les objets d’état d’intégrité des applications déployées qui sont retournés dans le résultat de la requête d’intégrité de l’application. Les valeurs possibles de ce paramètre incluent la valeur entière de l’un des états d’intégrité suivants. Seules les applications déployées qui correspondent au filtre sont retournées. Toutes les applications déployées sont utilisées pour évaluer l’état d’intégrité agrégé. Si cet argument n’est pas spécifié, toutes les entrées sont retournées. Les valeurs d’état correspondent à une énumération basée sur des indicateurs. La valeur peut donc être une combinaison de ces valeurs obtenue à l’aide de l’opérateur « OR » au niveau du bit. Par exemple, si la valeur indiquée est 6, l’état d’intégrité des applications déployées dont la valeur HealthState est OK (2) et Warning (4) est retourné.  <br> - Default : valeur par défaut. Correspond à toute valeur HealthState. La valeur est égale à zéro.  <br> - None : filtre qui ne correspond à aucune valeur HealthState. Permet de ne retourner aucun résultat sur une collection donnée d’états. La valeur est égale à 1.  <br> - OK : filtre qui correspond à l’entrée ayant OK comme valeur HealthState. La valeur est égale à 2.  <br> - Warning : filtre qui correspond à l’entrée ayant Warning comme valeur HealthState. La valeur est égale à 4.  <br> - Error : filtre qui correspond à l’entrée ayant Error comme valeur HealthState. La valeur est égale à 8.  <br> - All : filtre qui correspond à l’entrée ayant n’importe quelle valeur HealthState. La valeur est égale à 65535. |
| --events-health-state-filter | Permet de filtrer la collection d’objets HealthEvent retournés en fonction de leur état d’intégrité. Les valeurs possibles de ce paramètre incluent la valeur entière de l’un des états d’intégrité suivants. Seuls les événements qui correspondent au filtre sont renvoyés. Tous les événements sont utilisés pour évaluer l’état d’intégrité agrégé. Si cet argument n’est pas spécifié, toutes les entrées sont retournées. Les valeurs d’état correspondent à une énumération basée sur des indicateurs. La valeur peut donc être une combinaison de ces valeurs obtenue à l’aide de l’opérateur « OR » au niveau du bit. Par exemple, si la valeur indiquée est 6, tous les événements dont la valeur HealthState est OK (2) et Warning (4) sont retournés.  <br> - Default : valeur par défaut. Correspond à toute valeur HealthState. La valeur est égale à zéro.  <br> - None : filtre qui ne correspond à aucune valeur HealthState. Permet de ne retourner aucun résultat sur une collection donnée d’états. La valeur est égale à 1.  <br> - OK : filtre qui correspond à l’entrée ayant OK comme valeur HealthState. La valeur est égale à 2.  <br> - Warning : filtre qui correspond à l’entrée ayant Warning comme valeur HealthState. La valeur est égale à 4.  <br> - Error : filtre qui correspond à l’entrée ayant Error comme valeur HealthState. La valeur est égale à 8.  <br> - All : filtre qui correspond à l’entrée ayant n’importe quelle valeur HealthState. La valeur est égale à 65535. |
| --exclude-health-statistics | Indique si les statistiques d’intégrité doivent être retournées dans le cadre du résultat de la requête. False par défaut. Les statistiques affichent le nombre d’entités enfants dont l’état d’intégrité est OK, Warning et Error. |
| --services-health-state-filter | Permet de filtrer, par état d’intégrité, les objets d’état d’intégrité des services qui sont retournés dans le résultat de la requête d’intégrité des services. Les valeurs possibles de ce paramètre incluent la valeur entière de l’un des états d’intégrité suivants. Seuls les services qui correspondent au filtre sont retournés. Tous les services sont utilisés pour évaluer l’état d’intégrité agrégé. Si cet argument n’est pas spécifié, toutes les entrées sont retournées. Les valeurs d’état correspondent à une énumération basée sur des indicateurs. La valeur peut donc être une combinaison de ces valeurs obtenue à l’aide de l’opérateur « OR » au niveau du bit. Par exemple, si la valeur indiquée est 6, l’état d’intégrité des services dont la valeur HealthState est OK (2) et Warning (4) est retourné.  <br> - Default : valeur par défaut. Correspond à toute valeur HealthState. La valeur est égale à zéro.  <br> - None : filtre qui ne correspond à aucune valeur HealthState. Permet de ne retourner aucun résultat sur une collection donnée d’états. La valeur est égale à 1.  <br> - OK : filtre qui correspond à l’entrée ayant OK comme valeur HealthState. La valeur est égale à 2.  <br> - Warning : filtre qui correspond à l’entrée ayant Warning comme valeur HealthState. La valeur est égale à 4.  <br> - Error : filtre qui correspond à l’entrée ayant Error comme valeur HealthState. La valeur est égale à 8.  <br> - All : filtre qui correspond à l’entrée ayant n’importe quelle valeur HealthState. La valeur est égale à 65535. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-info"></a>sfctl application info
Permet d’obtenir des informations sur une application Service Fabric.

Renvoie les informations sur l’application qui a été créée ou en cours de création dans le cluster Service Fabric et dont le nom correspond à celui spécifié comme paramètre. La réponse comprend le nom, le type, l’état, les paramètres et d’autres détails sur l’application.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id      [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --exclude-application-parameters | Indicateur qui spécifie si les paramètres de l’application doivent être exclus du résultat. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-list"></a>sfctl application list
Permet d’obtenir la liste des applications créées dans le cluster Service Fabric qui correspondent aux filtres spécifiés.

Permet d’obtenir les informations sur les applications qui ont été créées ou sont en cours de création dans le cluster Service Fabric et qui correspondent aux filtres spécifiés. La réponse comprend le nom, le type, l’état, les paramètres et d’autres détails sur l’application. Si les applications ne tiennent pas dans une page, une page de résultats est renvoyée, ainsi que d’un jeton de liaison, qui peut être utilisé pour obtenir la page suivante. Les filtres ApplicationTypeName et ApplicationDefinitionKindFilter ne peuvent pas être spécifiés en même temps.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-definition-kind-filter | Permet de filtrer sur ApplicationDefinitionKind, qui est le mécanisme utilisé pour définir une application Service Fabric.  <br> - Default - Valeur par défaut, qui a le même effet que la sélection de « All ». La valeur est égale à 0.  <br> - All - Filtre qui correspond aux entrées ayant n’importe quelle valeur ApplicationDefinitionKind. La valeur est égale à 65535.  <br> - ServiceFabricApplicationDescription - Filtre qui correspond aux entrées dont la valeur d’ApplicationDefinitionKind est égale à ServiceFabricApplicationDescription. La valeur est égale à 1.  <br> - Compose - Filtre qui correspond aux entrées dont la valeur d’ApplicationDefinitionKind est égale à Compose. La valeur est égale à 2. |
| --application-type-name | Nom du type d’application utilisé pour filtrer les applications à interroger. Cette valeur ne doit pas contenir la version du type d’application. |
| --continuation-token | Le paramètre de jeton de liaison permet d’obtenir le jeu de résultats suivant. Un jeton de liaison avec une valeur non vide est inclus dans la réponse de l’API quand les résultats du système ne tiennent pas dans une seule réponse. Lorsque cette valeur est transmise à l’appel d’API suivant, l’API retourne le jeu de résultats suivant. S’il n’existe pas de résultats supplémentaires, le jeton de liaison ne contient pas de valeur. La valeur de ce paramètre ne doit pas être codée URL. |
| --exclude-application-parameters | Indicateur qui spécifie si les paramètres de l’application doivent être exclus du résultat. |
| --max-results | Nombre maximal de résultats à renvoyer dans le cadre des requêtes paginées. Ce paramètre définit la limite supérieure du nombre de résultats renvoyés. Le nombre de résultats renvoyés peut être inférieur au nombre maximal de résultats spécifié s’ils ne tiennent pas dans le message conformément aux restrictions de taille maximale définies dans la configuration. Si ce paramètre est défini sur zéro ou n’est pas spécifié, la requête paginée comprend le nombre maximal de résultats pouvant tenir dans le message renvoyé. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-load"></a>sfctl application load
Permet d’obtenir les informations relatives à une application Service Fabric.

Retourne des informations de chargement concernant l’application qui a été créée ou est en cours de création dans le cluster Service Fabric, et dont le nom correspond à celui spécifié comme paramètre. La réponse comprend le nom, le nombre minimal de nœuds, le nombre maximal de nœuds, le nombre de nœuds que l’application occupe actuellement, ainsi que les métriques de chargement relatives à l’application.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-manifest"></a>sfctl application manifest
Permet d’obtenir le manifeste qui décrit un type d’application.

La réponse contient le code XML du manifeste de l’application sous forme de chaîne.

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

## <a name="sfctl-application-provision"></a>sfctl application provision
Permet d’approvisionner ou d’inscrire un type d’application Service Fabric auprès du cluster à l’aide du package « .sfpkg » dans le magasin externe ou du package d’application dans le magasin d’images.

Permet d’approvisionner un type d’application Service Fabric auprès du cluster. L’approvisionnement est nécessaire avant toute instanciation de nouvelles applications. L’opération d’approvisionnement peut être effectuée sur le package d’application spécifié par relativePathInImageStore ou à l’aide de l’URI du package « .sfpkg » externe. À moins que --external-provision soit défini, cette commande exige un approvisionnement du magasin d’images.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-package-download-uri | Chemin d’accès du package d’application .sfpkg à partir duquel le package d’application peut être téléchargé à l’aide des protocoles HTTP ou HTTPS. <br><br> Pour l’approvisionnement à partir d’un magasin d’images externe uniquement. Le package d’application peut être stocké dans un magasin externe qui fournit l’opération GET pour télécharger le fichier. Les protocoles pris en charge sont HTTP et HTTPS, et le chemin d’accès doit autoriser l’accès en lecture. |
| --application-type-build-path | Pour l’approvisionnement à partir d’un magasin d’images uniquement. Chemin d’accès relatif du package d’application dans le magasin d’images spécifié lors de l’opération de chargement préalable. |
| --application-type-name | Pour l’approvisionnement à partir d’un magasin d’images externe uniquement. Le nom du type d’application correspond à celui indiqué dans le manifeste de l’application. |
| --application-type-version | Pour l’approvisionnement à partir d’un magasin d’images externe uniquement. La version du type d’application correspond à celle indiquée dans le manifeste de l’application. |
| --external-provision | Emplacement à partir duquel le package d’application peut être inscrit ou approvisionné. Indique que l’approvisionnement concerne un package d’application précédemment chargé dans un magasin externe. Le package d’application présente l’extension *.sfpkg. |
| --no-wait | Indique si l’approvisionnement doit se faire de façon asynchrone. <br><br> Lorsque la valeur est définie sur true, l’opération d’approvisionnement s’exécute quand la requête est acceptée par le système et l’opération d’approvisionnement se poursuit sans délai d’expiration. La valeur par défaut est false. Pour les packages d’application volumineux, nous vous recommandons de définir la valeur sur true. |
| --timeout -t | Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-report-health"></a>sfctl application report-health
Permet d’envoyer un rapport d’intégrité sur l’application Service Fabric.

Signale l’état d’intégrité de l’application Service Fabric spécifiée. Le rapport doit contenir les informations sur la source du rapport d’intégrité et sur la propriété dont il fait état. Il est envoyé à une application de passerelle Service Fabric, qui opère son transfert vers le magasin d’intégrité. Le rapport peut être accepté par la passerelle, mais rejeté par le magasin d’intégrité après une validation supplémentaire. Par exemple, le magasin d’intégrité peut rejeter le rapport en raison d’un paramètre non valide, comme un numéro de séquence obsolète. Pour voir si le rapport a été appliqué dans le magasin d’intégrité, obtenez l’intégrité de l’application et vérifiez que le rapport s’affiche.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id  [obligatoire] | Identité de l’application. <br><br> Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
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

## <a name="sfctl-application-type"></a>sfctl application type
Permet d’obtenir la liste des types d’applications du cluster Service Fabric qui correspondent exactement au nom spécifié.

Renvoie les informations sur les types d’application qui sont approvisionnés ou en cours d’approvisionnement dans le cluster Service Fabric. Ces résultats sont des types d’applications dont le nom correspond exactement à celui spécifié comme paramètre et qui sont conformes aux paramètres de requête donnés. Toutes les versions du type d’application correspondant au nom de type d’application sont renvoyées avec chaque version renvoyée sous la forme d’un type d’application. La réponse comprend le nom, la version, l’état et d’autres informations sur le type d’application. Il s’agit d’une requête paginée, ce qui signifie que si les applications ne tiennent pas dans une page, une page de résultats est renvoyée, ainsi que d’un jeton de liaison, qui peut être utilisé pour obtenir la page suivante. Par exemple, s’il existe 10 types d’application mais qu’une page ne peut en contenir que les trois premiers, ou si le maximum de résultats est défini sur 3, alors la valeur 3 est retournée. Pour accéder au reste des résultats, récupérez les pages suivantes en utilisant le jeton de liaison renvoyé dans la requête suivante. Un jeton de liaison vide est renvoyé s’il n’existe aucune autre page.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-type-name [Requis] | Nom du type d’application. |
| --application-type-version | Version du type d’application. |
| --continuation-token | Le paramètre de jeton de liaison permet d’obtenir le jeu de résultats suivant. Un jeton de liaison avec une valeur non vide est inclus dans la réponse de l’API quand les résultats du système ne tiennent pas dans une seule réponse. Lorsque cette valeur est transmise à l’appel d’API suivant, l’API retourne le jeu de résultats suivant. S’il n’existe pas de résultats supplémentaires, le jeton de liaison ne contient pas de valeur. La valeur de ce paramètre ne doit pas être codée URL. |
| --exclude-application-parameters | Indicateur qui spécifie si les paramètres de l’application doivent être exclus du résultat. |
| --max-results | Nombre maximal de résultats à renvoyer dans le cadre des requêtes paginées. Ce paramètre définit la limite supérieure du nombre de résultats renvoyés. Le nombre de résultats renvoyés peut être inférieur au nombre maximal de résultats spécifié s’ils ne tiennent pas dans le message conformément aux restrictions de taille maximale définies dans la configuration. Si ce paramètre est défini sur zéro ou n’est pas spécifié, la requête paginée comprend le nombre maximal de résultats pouvant tenir dans le message renvoyé. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-type-list"></a>sfctl application type-list
Permet d’obtenir la liste des types d’applications du cluster Service Fabric.

Renvoie les informations sur les types d’application qui sont approvisionnés ou en cours d’approvisionnement dans le cluster Service Fabric. Chaque version d’un type d’application est retournée en tant que type d’une application. La réponse comprend le nom, la version, l’état et d’autres informations sur le type d’application. Il s’agit d’une requête paginée, ce qui signifie que si les applications ne tiennent pas dans une page, une page de résultats est renvoyée, ainsi que d’un jeton de liaison, qui peut être utilisé pour obtenir la page suivante. Par exemple, s’il existe 10 types d’application mais qu’une page ne peut en contenir que les trois premiers, ou si le maximum de résultats est défini sur 3, alors la valeur 3 est retournée. Pour accéder au reste des résultats, récupérez les pages suivantes en utilisant le jeton de liaison renvoyé dans la requête suivante. Un jeton de liaison vide est renvoyé s’il n’existe aucune autre page.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-type-definition-kind-filter | Utilisé pour filtrer sur ApplicationTypeDefinitionKind, qui est le mécanisme utilisé pour définir un type d’application Service Fabric.  <br> - Default - Valeur par défaut, qui a le même effet que la sélection de « All ». La valeur est égale à 0.  <br> - All - Filtre qui correspond aux entrées ayant n’importe quelle valeur ApplicationTypeDefinitionKind. La valeur est égale à 65535.  <br> - ServiceFabricApplicationPackage - Filtre qui correspond aux entrées dont l’argument ApplicationTypeDefinitionKind présente la valeur ServiceFabricApplicationPackage. La valeur est égale à 1.  <br> - Compose - Filtre qui correspond aux entrées dont la valeur ApplicationTypeDefinitionKind est Compose. La valeur est égale à 2. |
| --continuation-token | Le paramètre de jeton de liaison permet d’obtenir le jeu de résultats suivant. Un jeton de liaison avec une valeur non vide est inclus dans la réponse de l’API quand les résultats du système ne tiennent pas dans une seule réponse. Lorsque cette valeur est transmise à l’appel d’API suivant, l’API retourne le jeu de résultats suivant. S’il n’existe pas de résultats supplémentaires, le jeton de liaison ne contient pas de valeur. La valeur de ce paramètre ne doit pas être codée URL. |
| --exclude-application-parameters | Indicateur qui spécifie si les paramètres de l’application doivent être exclus du résultat. |
| --max-results | Nombre maximal de résultats à renvoyer dans le cadre des requêtes paginées. Ce paramètre définit la limite supérieure du nombre de résultats renvoyés. Le nombre de résultats renvoyés peut être inférieur au nombre maximal de résultats spécifié s’ils ne tiennent pas dans le message conformément aux restrictions de taille maximale définies dans la configuration. Si ce paramètre est défini sur zéro ou n’est pas spécifié, la requête paginée comprend le nombre maximal de résultats pouvant tenir dans le message renvoyé. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-unprovision"></a>sfctl application unprovision
Permet de supprimer ou d’annuler l’inscription d’un type d’application Service Fabric dans le cluster.

Cette opération n’est possible que si toutes les instances d’application du type d’application ont été supprimées. Une fois l’enregistrement du type d’application annulé, aucune nouvelle instance de l’application ne peut être créée pour ce type d’application particulier.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-type-name [Requis] | Nom du type d’application. |
| --application-type-version [Requis] | Version du type d’application indiquée dans le manifeste de l’application. |
| --async-parameter | Indique si l’annulation de l’approvisionnement doit se faire de façon asynchrone. Lorsque la valeur est définie sur true, l’opération d’annulation de l’approvisionnement s’exécute quand la requête est acceptée par le système et l’opération d’annulation de l’approvisionnement se poursuit sans délai d’expiration. La valeur par défaut est false. Toutefois, nous vous recommandons de définir cette valeur sur true pour les packages d’application volumineux qui ont été approvisionnés. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-upgrade"></a>sfctl application upgrade
Commence la mise à niveau d’une application dans le cluster Service Fabric.

Valide les paramètres de mise à niveau d’application fournis et commence la mise à niveau de l’application si les paramètres sont valides. Notez que la description de la mise à niveau remplace la description de l’application existante. Cela signifie que si les paramètres ne sont pas spécifiés, les paramètres existants sur les applications seront remplacés par la liste de paramètres vide. Cela se traduirait par l’utilisation par l’application de la valeur par défaut des paramètres à partir du manifeste d’application.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id      [Requis] | Identité de l’application. <br><br> Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --application-version [obligatoire] | La version du type d’application cible (trouvée dans le manifeste de l’application) pour la mise à niveau de l’application. |
| --parameters          [obligatoire] | Liste JSON des remplacements de paramètres d’application à appliquer lors de la création de l’application. |
| --default-service-health-policy | Spécification JSON de la stratégie de contrôle d’intégrité utilisée par défaut pour évaluer l’intégrité d’un type de service. |
| --failure-action | Action à effectuer lorsqu’une mise à niveau surveillée détecte des violations de stratégie de surveillance ou de stratégie d’intégrité. |
| --force-restart | Force le redémarrage des processus pendant la mise à jour, même si la version du code n’a pas changé. |
| --health-check-retry-timeout | Délai séparant les tentatives d'exécution des contrôles d'intégrité si l'application ou le cluster n'est pas sain.  Valeur par défaut \: PT0H10M0S. |
| --health-check-stable-duration | Durée pendant laquelle l’application ou le cluster doivent rester sains avant que la mise à niveau ne passe au domaine de mise à niveau suivant.  Valeur par défaut \: PT0H2M0S. <br><br> Elle est d’abord interprétée en tant que chaîne représentant une durée ISO 8601. Si cette tentative échoue, elle est interprétée comme un nombre représentant le nombre total de millisecondes. |
| --health-check-wait-duration | Délai d’attente entre l’achèvement d’un domaine de mise à niveau et le démarrage du processus des contrôles d’intégrité.  Valeur par défaut \: 0. |
| --max-unhealthy-apps | Pourcentage maximal autorisé d’applications déployées non saines. Représenté par un nombre compris entre 0 et 100. |
| --mode | Mode utilisé pour surveiller l’intégrité pendant une mise à niveau propagée.  Valeur par défaut \: UnmonitoredAuto. |
| --replica-set-check-timeout | Durée maximale pendant laquelle bloquer le traitement d’un domaine de mise à niveau et éviter la perte de disponibilité en cas de problèmes inattendus. Mesurée en secondes. |
| --service-health-policy | Mappage JSON avec stratégie d’intégrité de type de service par nom de type de service. Par défaut, le mappage est vide. |
| --timeout -t | Valeur par défaut \: 60. |
| --upgrade-domain-timeout | Durée d’exécution de chaque domaine de mise à niveau avant l’exécution de FailureAction.  Valeur par défaut \: P10675199DT02H48M05.4775807S. <br><br> Elle est d’abord interprétée en tant que chaîne représentant une durée ISO 8601. Si cette tentative échoue, elle est interprétée comme un nombre représentant le nombre total de millisecondes. |
| --upgrade-timeout | Durée d’exécution de l’ensemble de la mise à niveau avant l’exécution de FailureAction.  Valeur par défaut \: P10675199DT02H48M05.4775807S. <br><br> Elle est d’abord interprétée en tant que chaîne représentant une durée ISO 8601. Si cette tentative échoue, elle est interprétée comme un nombre représentant le nombre total de millisecondes. |
| --warning-as-error | Indique si les avertissements sont traités avec le même niveau de gravité que les erreurs. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-upgrade-resume"></a>sfctl application upgrade-resume
Commence la mise à niveau d’une application dans le cluster Service Fabric.

Reprend la mise à niveau d’une application Service Fabric manuelle non contrôlée. Service Fabric met à niveau un domaine de mise à niveau à la fois. Pour les mises à niveau manuelles non contrôlées, lorsque Service Fabric termine un domaine de mise à niveau, il attend que vous appeliez cette API avant de passer au domaine de mise à niveau suivant.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id      [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --upgrade-domain-name [obligatoire] | Nom du domaine de mise à niveau dans lequel reprendre la mise à niveau. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-upgrade-rollback"></a>sfctl application upgrade-rollback
Annule la mise à niveau en cours d’une application du cluster Service Fabric.

Annule la mise à niveau de l’application actuelle vers la version précédente. Cette API peut uniquement être utilisée pour annuler la mise à niveau en cours d’exécution actuelle qui restaure par progression vers la nouvelle version. Si l’application n’est pas actuellement en cours de mise à niveau, utilisez l’API StartApplicationUpgrade pour la mettre à niveau vers la version souhaitée, y compris l’annulation vers une version précédente.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-upgrade-status"></a>sfctl application upgrade-status
Permet d’obtenir les détails de la dernière mise à jour effectuée sur cette application.

Renvoie les informations concernant l’état de la dernière mise à niveau d’application, ainsi que des détails facilitant le débogage des problèmes d’intégrité des applications.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-application-upload"></a>sfctl application upload
Copie un package d’application Service Fabric dans le magasin d’images.

Permet d’afficher éventuellement la progression du chargement pour chaque fichier du package. La progression du chargement est envoyée à `stderr`.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --path     [Obligatoire] | Chemin d’accès au package d’application local. |
| --compress | S’applique uniquement aux packages d’applications Service Fabric. Créez un dossier contenant le package d’application compressé à l’emplacement par défaut ou à l’emplacement spécifié par le paramètre compressed-location, puis chargez le dossier nouvellement créé. <br><br> S’il existe un fichier compressé généré par sfctl, il est remplacé si cet indicateur est défini. Une erreur est retournée si le répertoire n’est pas un package d’application. S’il s’agit déjà d’un package d’application compressé, le dossier est copié dessus tel quel. Par défaut, le package d’application compressé nouvellement créé est supprimé après un chargement réussi. Si le chargement échoue, veuillez nettoyer manuellement le package compressé si nécessaire. La suppression ne supprime aucun répertoire vide qui a pu être créé si le paramètre d’emplacement compressé fait référence à des répertoires inexistants. |
| --compressed-location | Emplacement où placer le package d’application compressé. <br><br> Si aucun emplacement n’est fourni, le package compressé est placé dans un dossier nouvellement créé appelé sfctl_compressed_temp sous le répertoire parent spécifié dans l’argument path. Par exemple, si l’argument path a la valeur C\:/FolderA/AppPkg, le package compressé est ajouté à C\:/FolderA/sfctl_compressed_temp/AppPkg. |
| --imagestore-string | Magasin d’images de destination dans lequel charger le package d’application.  Valeur par défaut \: fabric\:ImageStore. <br><br> Pour effectuer un chargement vers un emplacement de fichier, faites commencer ce paramètre par « file\:». Dans le cas contraire, la valeur doit être la chaîne de connexion du magasin d’images, par exemple la valeur par défaut. |
| --keep-compressed | Indique s’il faut ou non conserver le package compressé généré en cas de réussite du chargement. <br><br> Si la valeur n’est pas définie, en cas de réussite de l’opération, les packages d’applications compressés sont supprimés. Si le chargement a échoué, le package d’application est toujours conservé dans le répertoire de sortie pour re-chargement. |
| --show-progress | Permet d’afficher la progression du chargement pour les packages volumineux. |
| --timeout -t | Délai d’attente total en secondes. Le chargement échoue et retourne une erreur une fois la durée du délai d’attente écoulée. Ce délai d’attente s’applique à l’ensemble du package d’application, et les délais d’attente des différents fichiers correspondent à la durée restante du délai d’attente. Le délai d’attente n’inclut pas le temps nécessaire pour compresser le package d’application.  Par défaut \: 300. |

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
