---
title: Interface CLI Azure Service Fabric - Réplica sfctl
description: Apprenez-en davantage sur sfctl, l’interface de ligne de commande d’Azure Service Fabric. Contient une liste de commandes pour la gestion des réplicas.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: 3ed1ecca686471e052f07e945738ad1a49e68464
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86260308"
---
# <a name="sfctl-replica"></a>sfctl replica
Permet de gérer les réplicas qui font partie des partitions de service.

## <a name="commands"></a>Commandes

|Commande|Description|
| --- | --- |
| deployed | Permet d’obtenir les détails d’un réplica déployé sur un nœud Service Fabric. |
| deployed-list | Permet d’obtenir la liste des réplicas déployés sur un nœud Service Fabric. |
| health | Permet d’obtenir l’intégrité d’une instance de service sans état ou d’un réplica de service avec état Service Fabric. |
| info | Permet d’obtenir les informations sur un réplica d’une partition Service Fabric. |
| list | Permet d’obtenir les informations sur des réplicas d’une partition de service Service Fabric. |
| remove | Supprime un réplica de service exécuté sur un nœud. |
| report-health | Envoie un rapport d’intégrité sur le réplica Service Fabric. |
| restart | Redémarre un réplica de service d’un service persistant exécuté sur un nœud. |

## <a name="sfctl-replica-deployed"></a>sfctl replica deployed
Permet d’obtenir les détails d’un réplica déployé sur un nœud Service Fabric.

Permet d’obtenir les détails du réplica déployé sur un nœud Service Fabric. Les informations incluent le type de service, le nom de service, l’opération de service en cours, l’heure/la date de démarrage de l’opération de service en cours, l’ID de partition, l’ID de réplica/instance, le chargement signalé ainsi que d’autres informations.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --node-name [Requis] | Nom du nœud. |
| --partition-id [Requis] | Identité de la partition. |
| --replica-id [Requis] | Identificateur du réplica. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-replica-deployed-list"></a>sfctl replica deployed-list
Permet d’obtenir la liste des réplicas déployés sur un nœud Service Fabric.

Permet d’obtenir la liste contenant les informations sur les réplicas déployés sur un nœud Service Fabric. Les informations comprennent l’ID de partition, l’ID de réplica, l’état du réplica, le nom du service, le nom du type de service et autres. Utilisez les paramètres de requête PartitionId ou ServiceManifestName pour retourner des informations sur les réplicas déployés qui correspondent aux valeurs spécifiées de ces paramètres.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --application-id [Requis] | Identité de l’application. Il s’agit généralement du nom complet de l’application, sans le schéma d’URI « fabric\: ». À compter de la version 6.0, les noms hiérarchiques sont délimités par le caractère « \~ ». Par exemple, si l’application est nommée « fabric\:/myapp/app1 », son identité est « myapp\~app1 » dans les versions 6.0 et ultérieures, et « myapp/app1 » dans les versions précédentes. |
| --node-name      [obligatoire] | Nom du nœud. |
| --partition-id | Identité de la partition. |
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

## <a name="sfctl-replica-health"></a>sfctl replica health
Permet d’obtenir l’intégrité d’une instance de service sans état ou d’un réplica de service avec état Service Fabric.

Permet d’obtenir l’intégrité d’un réplica Service Fabric. EventsHealthStateFilter permet de filtrer la collection d’événements d’intégrité signalés dans le réplica en fonction de l’état d’intégrité.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --partition-id [Requis] | Identité de la partition. |
| --replica-id [Requis] | Identificateur du réplica. |
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

## <a name="sfctl-replica-info"></a>sfctl replica info
Permet d’obtenir les informations sur un réplica d’une partition Service Fabric.

La réponse inclut l’ID, le rôle, l’état, l’intégrité, le nom du nœud, la durée de fonctionnement et d’autres détails sur le réplica.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --partition-id [Requis] | Identité de la partition. |
| --replica-id [Requis] | Identificateur du réplica. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-replica-list"></a>sfctl replica list
Permet d’obtenir les informations sur des réplicas d’une partition de service Service Fabric.

Le point de terminaison GetReplicas retourne des informations sur la partition spécifiée. La réponse inclut l’ID, le rôle, l’état, l’intégrité, le nom du nœud, la durée de fonctionnement et d’autres détails sur le réplica.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --partition-id [Requis] | Identité de la partition. |
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

## <a name="sfctl-replica-remove"></a>sfctl replica remove
Supprime un réplica de service exécuté sur un nœud.

Cette API simule un échec de réplica Service Fabric en supprimant un réplica dans un cluster Service Fabric. Cette suppression ferme le réplica, effectue la transition du réplica vers le rôle Aucun, puis supprime toutes les informations sur l’état du réplica dans le cluster. Cette API teste le chemin d’accès de suppression d’état du réplica, puis simule le chemin d’accès permanent du rapport d’erreurs via les API clientes. Avertissement : aucune vérification de sécurité n’est effectuée lorsque cette API est utilisée. Une utilisation incorrecte de cette API peut entraîner une perte des données des services avec état. De plus, l’indicateur forceRemove a un impact sur tous les autres réplicas hébergés dans le même processus.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --node-name [Requis] | Nom du nœud. |
| --partition-id [Requis] | Identité de la partition. |
| --replica-id [Requis] | Identificateur du réplica. |
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

## <a name="sfctl-replica-report-health"></a>sfctl replica report-health
Envoie un rapport d’intégrité sur le réplica Service Fabric.

Signale l’état d’intégrité du réplica Service Fabric spécifié. Le rapport doit contenir les informations sur la source du rapport d’intégrité et sur la propriété dont il fait état. Il est envoyé à un réplica de passerelle Service Fabric, qui opère son transfert vers le magasin d’intégrité. Le rapport peut être accepté par la passerelle, mais rejeté par le magasin d’intégrité après une validation supplémentaire. Par exemple, le magasin d’intégrité peut rejeter le rapport en raison d’un paramètre non valide, comme un numéro de séquence obsolète. Pour voir si le rapport a été appliqué dans le magasin d’intégrité, récupérez l’état d’intégrité du réplica et vérifiez qu’il s’affiche dans la section des événements d’intégrité.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --health-property [obligatoire] | Propriété des informations d’intégrité. <br><br> Une entité peut avoir des rapports d’intégrité pour différentes propriétés. La propriété est une chaîne et non une énumération fixe pour offrir au rapporteur la possibilité de catégoriser la condition d’état qui déclenche le rapport. Par exemple, un rapporteur avec l’ID source (SourceId) « LocalWatchdog » peut surveiller l’état du disque disponible sur un nœud. Il peut donc signaler la propriété « AvailableDisk » sur ce nœud. Le même rapporteur peut surveiller la connectivité du nœud. Il peut donc signaler une propriété « Connectivity » sur le même nœud. Dans le magasin d’intégrité, ces rapports sont traités comme des événements d’intégrité distincts pour le nœud spécifié. Avec l’ID source (SourceId), la propriété identifie de façon unique les informations d’intégrité. |
| --health-state    [obligatoire] | Les valeurs possibles sont \: « Invalid », « Ok », « Warning », « Error », « Unknown ». |
| --partition-id [Requis] | Identité de la partition. |
| --replica-id [Requis] | Identité de la partition. |
| --source-id       [obligatoire] | Nom source qui identifie le composant client/espion/système qui a généré les informations d’intégrité. |
| --description | Description des informations d’intégrité. <br><br> Il s’agit de texte libre utilisé pour ajouter des informations sur le rapport lisibles par l’homme. La longueur de chaîne maximale de la description est de 4 096 caractères. Si la chaîne fournie est plus longue, elle est automatiquement tronquée. Si elle est tronquée, les derniers caractères de la description contiennent un marqueur « [Tronqué] », et la taille totale de la chaîne est de 4 096 caractères. La présence du marqueur indique aux utilisateurs qu’une troncation a été effectuée. Notez que si elle est tronquée, la description a moins que les 4 096 caractères de la chaîne d’origine. |
| --immediate | Indicateur qui spécifie si le rapport doit être envoyé immédiatement. <br><br> Un rapport d’intégrité est envoyé à une application de passerelle Service Fabric, qui opère son transfert vers le magasin d’intégrité. Si le paramètre immediate est défini sur true, le rapport est immédiatement envoyé de la passerelle HTTP au magasin d’intégrité, quels que soient les paramètres du client Fabric qu’utilise l’application de passerelle HTTP. Cela est utile pour les rapports critiques qui doivent être envoyés dès que possible. En fonction du minutage et d’autres conditions, l’envoi du rapport peut quand même échouer, par exemple si la passerelle HTTP est fermée ou si le message n’atteint pas la passerelle. Si le paramètre immediate est défini sur false, le rapport est envoyé en fonction des paramètres du client d’intégrité de la passerelle HTTP. C’est pourquoi il est traité par lot selon la configuration HealthReportSendInterval. Il s’agit du paramètre recommandé, car il permet au client d’intégrité d’optimiser les messages de rapport d’intégrité envoyés au magasin d’intégrité, ainsi que le traitement des rapports d’intégrité. Par défaut, les rapports ne sont pas envoyés immédiatement. |
| --remove-when-expired | Valeur qui indique si le rapport est supprimé du magasin d’intégrité quand il expire. <br><br> Si la valeur définie est true, le rapport est supprimé du magasin d’intégrité après son expiration. Si la valeur définie est false, le rapport est traité comme une erreur quand il expire. La valeur de cette propriété est false par défaut. Quand les clients créent un rapport régulièrement, ils doivent définir RemoveWhenExpired sur false (valeur par défaut). De cette manière, si le rapporteur rencontre des problèmes (par exemple, un interblocage) et qu’il ne peut pas créer de rapport, l’entité est évaluée comme erreur quand le rapport d’intégrité expire. L’entité est marquée comme étant dans l’état d’intégrité Erreur. |
| --sequence-number | Numéro de séquence de ce rapport d’intégrité sous forme de chaîne numérique. <br><br> Le numéro de séquence de rapport est utilisé par le magasin d’intégrité pour détecter les rapports obsolètes. S’il n’est pas spécifié, un numéro de séquence est généré automatiquement par le client d’intégrité quand un rapport est ajouté. |
| --service-kind | Genre de réplica de service (avec ou sans état) pour lequel l’intégrité est signalée. Voici les valeurs possibles \: « Stateless », « Stateful ».  État par défaut \: Stateful. |
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

## <a name="sfctl-replica-restart"></a>sfctl replica restart
Redémarre un réplica de service d’un service persistant exécuté sur un nœud.

Redémarre un réplica de service d’un service persistant exécuté sur un nœud. Avertissement : aucune vérification de sécurité n’est effectuée lorsque cette API est utilisée. Une utilisation incorrecte de cette API peut entraîner une perte de disponibilité des services avec état.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --node-name [Requis] | Nom du nœud. |
| --partition-id [Requis] | Identité de la partition. |
| --replica-id [Requis] | Identificateur du réplica. |
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
