---
title: Interface de ligne de commande Azure Service Fabric – sfctl compose
description: Apprenez-en plus sur sfctl, l’interface de ligne de commande d’Azure Service Fabric. Contient une liste de commandes pour les applications Docker Compose.
author: jeffj6123
ms.topic: reference
ms.date: 1/16/2020
ms.author: jejarry
ms.openlocfilehash: e84b35ff3dfa7be0c848a4d7b906c622d9b88914
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86257346"
---
# <a name="sfctl-compose"></a>sfctl compose
Permet de créer, de supprimer et de gérer les applications Docker Compose.

## <a name="commands"></a>Commandes

|Commande|Description|
| --- | --- |
| create | Permet de créer un déploiement compose Service Fabric. |
| list | Permet d’obtenir la liste des déploiements compose créés dans le cluster Service Fabric. |
| remove | Permet de supprimer un déploiement compose Service Fabric existant du cluster. |
| status | Permet d’obtenir des informations sur un déploiement compose Service Fabric. |
| upgrade | Permet de commencer la mise à niveau d’un déploiement compose dans le cluster Service Fabric. |
| upgrade-rollback | Permet de commencer la restauration d’une mise à niveau de déploiement compose dans le cluster Service Fabric. |
| upgrade-status | Permet d’obtenir des informations sur la dernière mise à niveau effectuée sur le déploiement Compose Service Fabric. |

## <a name="sfctl-compose-create"></a>sfctl compose create
Permet de créer un déploiement compose Service Fabric.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --deployment-name [Requis] | Le nom du déploiement. |
| --file-path       [Requis] | Chemin d’accès au fichier Docker Compose cible. |
| --encrypted-pass | Au lieu de demander un mot de passe de registre de conteneurs, utilisez une phrase secrète déjà chiffrée. |
| --has-pass | Demande un mot de passe pour accéder au registre de conteneurs. |
| --timeout -t | Valeur par défaut \: 60. |
| --user | Nom d’utilisateur pour se connecter au Registre du conteneur. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-compose-list"></a>sfctl compose list
Permet d’obtenir la liste des déploiements compose créés dans le cluster Service Fabric.

Permet d’obtenir l’état des déploiements compose créés ou qui sont en cours de création dans le cluster Service Fabric. La réponse comprend le nom, l’état et d’autres détails sur les déploiements compose. Si la liste des déploiements ne tient pas sur une page, une page de résultats est renvoyée, ainsi qu’un jeton de liaison qui peut être utilisé pour accéder à la page suivante.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --continuation-token | Le paramètre de jeton de liaison permet d’obtenir le jeu de résultats suivant. Un jeton de liaison avec une valeur non vide est inclus dans la réponse de l’API quand les résultats du système ne tiennent pas dans une seule réponse. Lorsque cette valeur est transmise à l’appel d’API suivant, l’API retourne le jeu de résultats suivant. S’il n’existe pas de résultats supplémentaires, le jeton de liaison ne contient pas de valeur. La valeur de ce paramètre ne doit pas être codée URL. |
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

## <a name="sfctl-compose-remove"></a>sfctl compose remove
Permet de supprimer un déploiement compose Service Fabric existant du cluster.

Permet de supprimer un déploiement compose Service Fabric existant.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --deployment-name [Requis] | Identité du déploiement. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-compose-status"></a>sfctl compose status
Permet d’obtenir des informations sur un déploiement compose Service Fabric.

Renvoie l’état du déploiement compose créé ou en cours de création dans le cluster Service Fabric et dont le nom correspond à celui spécifié comme paramètre. La réponse comprend le nom, l’état et d’autres détails sur le déploiement.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --deployment-name [Requis] | Identité du déploiement. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-compose-upgrade"></a>sfctl compose upgrade
Permet de commencer la mise à niveau d’un déploiement compose dans le cluster Service Fabric.

Permet de valider les paramètres de mise à niveau fournis et de commencer la mise à niveau du déploiement si les paramètres sont valides.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --deployment-name [Requis] | Le nom du déploiement. |
| --file-path        [Requis] | Chemin d’accès au fichier Docker Compose cible. |
| --default-svc-type-health-map | Dictionnaire encodé JSON qui décrit la stratégie de contrôle d’intégrité utilisée pour évaluer l’intégrité des services. |
| --encrypted-pass | Au lieu de demander un mot de passe de registre de conteneurs, utilisez une phrase secrète déjà chiffrée. |
| --failure-action | Les valeurs possibles sont \: « Invalid », « Rollback », « Manual ». |
| --force-restart | Force le redémarrage des processus pendant la mise à jour, même si la version du code n’a pas changé. <br><br> La mise à niveau modifie uniquement la configuration ou les données. |
| --has-pass | Demande un mot de passe pour accéder au registre de conteneurs. |
| --health-check-retry | Délai séparant les tentatives d'exécution des contrôles d'intégrité si l'application ou le cluster n'est pas sain. |
| --health-check-stable | Durée pendant laquelle l’application ou le cluster doivent rester sains avant que la mise à niveau ne passe au domaine de mise à niveau suivant. <br><br> Elle est d’abord interprétée en tant que chaîne représentant une durée ISO 8601. Si cette tentative échoue, elle est interprétée comme un nombre représentant le nombre total de millisecondes. |
| --health-check-wait | Délai d’attente entre l’achèvement d’un domaine de mise à niveau et le démarrage du processus des contrôles d’intégrité. |
| --replica-set-check | Durée maximale pendant laquelle bloquer le traitement d’un domaine de mise à niveau et éviter la perte de disponibilité en cas de problèmes inattendus. <br><br> Lorsque ce délai d’attente expire, le traitement du domaine de mise à niveau se poursuit, indépendamment des problèmes de perte de disponibilité. Le délai d’expiration est réinitialisé au début de chaque domaine de mise à niveau. Les valeurs valides sont comprises entre 0 et 42949672925 inclus. |
| --svc-type-health-map | Liste encodée JSON d’objets décrivant les stratégies de contrôle d’intégrité utilisées pour évaluer l’intégrité de différents types de services. |
| --timeout -t | Valeur par défaut \: 60. |
| --unhealthy-app | Pourcentage maximal autorisé d’applications non saines avant signalement d’une erreur. <br><br> Par exemple, pour autoriser 10 % des applications pouvant être défectueuses, cette valeur serait de 10. Il s’agit du pourcentage maximum toléré d’applications pouvant être défectueuses avant que l’intégrité du cluster ne soit considérée comme étant à l’état Error. Si le pourcentage est respecté mais qu’il existe au moins une application pouvant être défectueuse, l’état d’intégrité est Warning. Ce pourcentage est calculé en divisant le nombre d’applications pouvant être défectueuses par le nombre total d’instances d’application du cluster. |
| --upgrade-domain-timeout | Durée d’exécution de chaque domaine de mise à niveau avant l’exécution de FailureAction. <br><br> Elle est d’abord interprétée en tant que chaîne représentant une durée ISO 8601. Si cette tentative échoue, elle est interprétée comme un nombre représentant le nombre total de millisecondes. |
| --upgrade-kind | Valeur par défaut \: Rolling. |
| --upgrade-mode | Les valeurs possibles sont \: « Invalid », « UnmonitoredAuto », « UnmonitoredManual », « Monitored ».  Valeur par défaut \: UnmonitoredAuto. |
| --upgrade-timeout | Durée d’exécution de l’ensemble de la mise à niveau avant l’exécution de FailureAction. <br><br> Elle est d’abord interprétée en tant que chaîne représentant une durée ISO 8601. Si cette tentative échoue, elle est interprétée comme un nombre représentant le nombre total de millisecondes. |
| --user | Nom d’utilisateur pour se connecter au Registre du conteneur. |
| --warning-as-error | Indique si les avertissements sont traités avec le même niveau de gravité que les erreurs. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-compose-upgrade-rollback"></a>sfctl compose upgrade-rollback
Permet de commencer la restauration d’une mise à niveau de déploiement compose dans le cluster Service Fabric.

Permet de restaurer une mise à niveau de déploiement compose Service Fabric.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --deployment-name [Requis] | Identité du déploiement. |
| --timeout -t | Délai d’attente du serveur pour l’exécution de l’opération en secondes. Il spécifie la durée pendant laquelle le client attend la fin de l’opération demandée. La valeur par défaut de ce paramètre est de 60 secondes.  Valeur par défaut \: 60. |

### <a name="global-arguments"></a>Arguments globaux

|Argument|Description|
| --- | --- |
| --debug | Augmentez le niveau de détail de la journalisation pour afficher tous les journaux de débogage. |
| --help -h | Affichez ce message d’aide et quittez. |
| --output -o | Format de sortie.  Valeurs autorisées \: json, jsonc, table, tsv.  Valeur par défaut \: json. |
| --query | Chaîne de requêtes JMESPath. Pour obtenir plus d’informations et des exemples, consultez : http\://jmespath.org/. |
| --verbose | Augmentez le niveau de détail de la journalisation. Utilisez --debug pour des journaux de débogage complets. |

## <a name="sfctl-compose-upgrade-status"></a>sfctl compose upgrade-status
Permet d’obtenir des informations sur la dernière mise à niveau effectuée sur le déploiement Compose Service Fabric.

Renvoie les informations concernant l’état de la mise à niveau du déploiement compose, ainsi que des détails facilitant le débogage des problèmes d’intégrité des applications.

### <a name="arguments"></a>Arguments

|Argument|Description|
| --- | --- |
| --deployment-name [Requis] | Identité du déploiement. |
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
- [Configurez](service-fabric-cli.md) l’interface de ligne de commande Service Fabric.
- Découvrez comment utiliser l’interface de ligne de commande (CLI) Service Fabric à l’aide d’[exemples de scripts](./scripts/sfctl-upgrade-application.md).
