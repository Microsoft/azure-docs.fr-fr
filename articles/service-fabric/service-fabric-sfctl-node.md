---
title: Interface CLI Azure Service Fabric - sfctl node
description: Apprenez-en davantage sur sfctl, l’interface de ligne de commande d’Azure Service Fabric. Contient la liste des commandes liées à la gestion des nœuds de cluster.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: f2cf1011db37892f71bdd06f409cec1a76463507
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86257151"
---
# <a name="sfctl-node"></a>sfctl node
Permet de gérer les nœuds qui forment un cluster.

## <a name="commands"></a>Commandes

|Commande|Description|
| --- | --- |
| add-configuration-parameter-overrides | Ajoute la liste des substitutions de configuration sur le nœud spécifié. |
| disable | Désactive un nœud de cluster Service Fabric avec l’intention de désactivation spécifiée. |
| enable | Active un nœud de cluster Service Fabric actuellement désactivé. |
| get-configuration-overrides | Obtient la liste des substitutions de configuration sur le nœud spécifié. |
| health | Permet d’obtenir l’intégrité d’un nœud Service Fabric. |
| info | Permet d’obtenir des informations sur un nœud spécifique du cluster Service Fabric. |
| list | Permet d’obtenir la liste des nœuds du cluster Service Fabric. |
| load | Permet d’obtenir les informations de chargement d’un nœud Service Fabric. |
| remove-configuration-overrides | Supprime les substitutions de configuration sur le nœud spécifié. |
| remove-state | Informe Service Fabric que l’état persistant d’un nœud a été définitivement supprimé ou perdu. |
| report-health | Envoie un rapport d’intégrité sur le nœud Service Fabric. |
| restart | Redémarre un nœud de cluster Service Fabric. |
| transition | Démarre ou arrête un nœud de cluster. |
| transition-status | Permet d’obtenir la progression d’une opération démarrée à l’aide de StartNodeTransition. |

## <a name="sfctl-node-add-configuration-parameter-overrides"></a>sfctl node add-configuration-parameter-overrides
Ajoute la liste des substitutions de configuration sur le nœud spécifié.

Cette API permet d’ajouter toutes les substitutions de configuration existantes sur le nœud spécifié.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --config-parameter-override-list [Requis] | Description de l’ajout de la liste des substitutions de configuration. |
| --node-name                          [Requis] | Nom du nœud. |
| --force | Force l’ajout de substitutions de configuration sur des nœuds spécifiés. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-node-disable"></a>sfctl node disable
Désactive un nœud de cluster Service Fabric avec l’intention de désactivation spécifiée.

Désactive un nœud de cluster Service Fabric avec l’intention de désactivation spécifiée. Une fois la désactivation en cours, son intention peut être augmentée, mais pas réduite (par exemple, un nœud qui est désactivé avec l’intention Pause peut continuer à être désactivé avec Restart, mais l’inverse n’est pas vrai). Des nœuds peuvent être réactivés à l’aide de l’opération d’activation de nœud à tout moment une fois qu’ils sont désactivés. Si la désactivation n’est pas terminée, cette commande l’annule. Un nœud qui tombe en panne et redevient opérationnel en cours de désactivation doit toujours être réactivé avant que des services soient placés sur ce nœud.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --node-name [Requis] | Nom du nœud. |
| --deactivation-intent | Décrit l’intention ou le motif de la désactivation du nœud. Les valeurs possibles sont les suivantes. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-node-enable"></a>sfctl node enable
Active un nœud de cluster Service Fabric actuellement désactivé.

Active un nœud de cluster Service Fabric actuellement désactivé. Une fois activé, le nœud redevient une cible viable pour placer de nouveaux réplicas, et tout réplica désactivé restant sur ce nœud est réactivé.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --node-name [Requis] | Nom du nœud. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-node-get-configuration-overrides"></a>sfctl node get-configuration-overrides
Obtient la liste des substitutions de configuration sur le nœud spécifié.

Cette API permet d’obtenir toutes les substitutions de configuration existantes sur le nœud spécifié.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --node-name [Requis] | Nom du nœud. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-node-health"></a>sfctl node health
Permet d’obtenir l’intégrité d’un nœud Service Fabric.

Permet d’obtenir l’intégrité d’un nœud Service Fabric. EventsHealthStateFilter permet de filtrer la collection d’événements d’intégrité signalés dans le nœud en fonction de l’état d’intégrité. Si vous spécifiez un nœud qui n’existe pas dans le magasin d’intégrité, cette cmdlet retourne une erreur.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --node-name [Requis] | Nom du nœud. |
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

## <a name="sfctl-node-info"></a>sfctl node info
Permet d’obtenir des informations sur un nœud spécifique du cluster Service Fabric.

La réponse comprend le nom, l’état, l’ID, l’intégrité, la durée de fonctionnement et d’autres détails sur le nœud.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --node-name [Requis] | Nom du nœud. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-node-list"></a>sfctl node list
Permet d’obtenir la liste des nœuds du cluster Service Fabric.

La réponse comprend le nom, l’état, l’ID, l’intégrité, la durée de fonctionnement et d’autres détails sur le nœud.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --continuation-token | Le paramètre de jeton de liaison permet d’obtenir le jeu de résultats suivant. Un jeton de liaison avec une valeur non vide est inclus dans la réponse de l’API quand les résultats du système ne tiennent pas dans une seule réponse. Lorsque cette valeur est transmise à l’appel d’API suivant, l’API retourne le jeu de résultats suivant. S’il n’existe pas de résultats supplémentaires, le jeton de liaison ne contient pas de valeur. La valeur de ce paramètre ne doit pas être codée URL. |
| --max-results | Nombre maximal de résultats à renvoyer dans le cadre des requêtes paginées. Ce paramètre définit la limite supérieure du nombre de résultats renvoyés. Le nombre de résultats renvoyés peut être inférieur au nombre maximal de résultats spécifié s’ils ne tiennent pas dans le message conformément aux restrictions de taille maximale définies dans la configuration. Si ce paramètre est défini sur zéro ou n’est pas spécifié, la requête paginée comprend le nombre maximal de résultats pouvant tenir dans le message renvoyé. |
| --node-status-filter | Permet de filtrer les nœuds en fonction de NodeStatus. Seuls les nœuds qui correspondent à la valeur de filtre spécifiée sont retournés. Les valeurs possibles sont les suivantes.  Par défaut \: par défaut. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-node-load"></a>sfctl node load
Permet d’obtenir les informations de chargement d’un nœud Service Fabric.

Permet de récupérer les informations sur le chargement d’un nœud Service Fabric pour toutes les mesures dont la charge ou la capacité est définie.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --node-name [Requis] | Nom du nœud. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-node-remove-configuration-overrides"></a>sfctl node remove-configuration-overrides
Supprime les substitutions de configuration sur le nœud spécifié.

Cette API permet de supprimer toutes les substitutions de configuration existantes sur un nœud spécifié.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --node-name [Requis] | Nom du nœud. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-node-remove-state"></a>sfctl node remove-state
Informe Service Fabric que l’état persistant d’un nœud a été définitivement supprimé ou perdu.

Cela implique qu’il n’est pas possible de récupérer l’état persistant de ce nœud. Cela se produit généralement lorsqu’un disque dur est nettoyé ou tombe en panne. Le nœud doit être arrêté pour que cette opération réussisse. Cette opération indique à Service Fabric que les réplicas sur ce nœud n’existent plus, et que Service Fabric ne doit plus attendre que ces réplicas redeviennent opérationnels. N’exécutez pas cet applet de commande si l’état sur le nœud n’a pas été supprimé et que le nœud peut revenir opérationnel sans modification de son état. À partir de Service Fabric 6.5, pour utiliser cette API pour des nœuds seed, transformez ces nœuds seed en nœuds classiques (non initiaux), puis appelez l’API afin de supprimer l’état des nœuds. Si le cluster s'exécute sur Azure, Service Fabric tente automatiquement de transformer un nœud seed qui vient de s’arrêter en nœud non seed. Pour le permettre, assurez-vous que le nombre de nœuds non seed dans le type de nœud principal est supérieur au nombre de nœuds seed avec l’état Arrêté. Le cas échéant, ajoutez des nœuds supplémentaires au type de nœud principal. Pour un cluster autonome, si le nœud initial inactif n’est pas supposé retrouver pas son état opérationnel, supprimez-le du cluster. Pour ce faire, voir https\://docs.microsoft.com/fr-fr/azure/service-fabric/service-fabric-cluster-windows-server-add-remove-nodes.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --node-name [Requis] | Nom du nœud. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-node-report-health"></a>sfctl node report-health
Envoie un rapport d’intégrité sur le nœud Service Fabric.

Signale l’état d’intégrité du nœud Service Fabric spécifié. Le rapport doit contenir les informations sur la source du rapport d’intégrité et sur la propriété dont il fait état. Il est envoyé à un nœud de passerelle Service Fabric, qui opère son transfert vers le magasin d’intégrité. Le rapport peut être accepté par la passerelle, mais rejeté par le magasin d’intégrité après une validation supplémentaire. Par exemple, le magasin d’intégrité peut rejeter le rapport en raison d’un paramètre non valide, comme un numéro de séquence obsolète. Pour voir si le rapport a été appliqué dans le magasin d’intégrité, vérifiez qu’il s’affiche dans la section des événements d’intégrité.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --health-property [obligatoire] | Propriété des informations d’intégrité. <br><br> Une entité peut avoir des rapports d’intégrité pour différentes propriétés. La propriété est une chaîne et non une énumération fixe pour offrir au rapporteur la possibilité de catégoriser la condition d’état qui déclenche le rapport. Par exemple, un rapporteur avec l’ID source (SourceId) « LocalWatchdog » peut surveiller l’état du disque disponible sur un nœud. Il peut donc signaler la propriété « AvailableDisk » sur ce nœud. Le même rapporteur peut surveiller la connectivité du nœud. Il peut donc signaler une propriété « Connectivity » sur le même nœud. Dans le magasin d’intégrité, ces rapports sont traités comme des événements d’intégrité distincts pour le nœud spécifié. Avec l’ID source (SourceId), la propriété identifie de façon unique les informations d’intégrité. |
| --health-state    [obligatoire] | Les valeurs possibles sont \: « Invalid », « Ok », « Warning », « Error », « Unknown ». |
| --node-name [Requis] | Nom du nœud. |
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

## <a name="sfctl-node-restart"></a>sfctl node restart
Redémarre un nœud de cluster Service Fabric.

Redémarre un nœud de cluster Service Fabric déjà démarré.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --node-name [Requis] | Nom du nœud. |
| --create-fabric-dump | Spécifiez True pour créer une image mémoire du processus du nœud Fabric. Cette valeur respecte la casse.  Par défaut \: false. |
| --node-instance-id | ID d’instance du nœud cible. Si un ID d’instance est spécifié, le nœud est redémarré uniquement s’il correspond à l’instance actuelle du nœud. La valeur par défaut 0 correspond à tout ID d’instance. L’ID d’instance peut être obtenu à l’aide de la requête d’obtention de nœud.  Valeur par défaut \: 0. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-node-transition"></a>sfctl node transition
Démarre ou arrête un nœud de cluster.

Démarre ou arrête un nœud de cluster.  Un nœud de cluster est un processus, pas l’instance de système d’exploitation proprement dite.  Pour démarrer un nœud, définissez le paramètre NodeTransitionType sur « Start ». Pour arrêter un nœud, définissez le paramètre NodeTransitionType sur « Stop ». Cette API démarre l’opération (lorsque l’API retourne du contenu, il se peut que le nœud n’ait pas encore terminé la transition). Appelez l’API GetNodeTransitionProgress avec le même identifiant OperationId pour obtenir la progression de l’opération.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --node-instance-id [Requis] | ID d’instance du nœud cible. Il peut être déterminé via l’API GetNodeInfo. |
| --node-name [Requis] | Nom du nœud. |
| --node-transition-type [Requis] | Indique le type de transition à effectuer.  NodeTransitionType.Start démarre un nœud arrêté. NodeTransitionType.Stop arrête un nœud opérationnel. |
| --operation-id [Requis] | GUID qui identifie un appel de cette API.  Celui-ci est transmis à l’API GetProgress correspondante. |
| --stop-duration-in-seconds [Requis] | Durée, en secondes, pendant laquelle conserver le nœud arrêté.  La valeur minimale est égale à 600 et la valeur maximale est égale à 14400.  À l’expiration de ce délai, le nœud redevient automatiquement opérationnel. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-node-transition-status"></a>sfctl node transition-status
Permet d’obtenir la progression d’une opération démarrée à l’aide de StartNodeTransition.

Permet d’obtenir la progression d’une opération démarrée à l’aide de StartNodeTransition en utilisant l’ID OperationId indiqué.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --node-name [Requis] | Nom du nœud. |
| --operation-id [Requis] | GUID qui identifie un appel de cette API.  Celui-ci est transmis à l’API GetProgress correspondante. |
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
