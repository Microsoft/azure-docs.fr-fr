---
title: Présentation et résolution des erreurs WebHCat sur HDInsight - Azure
description: Découvrez quelles sont les erreurs courantes renvoyées par WebHCat sur HDInsight et comment les résoudre.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.custom: hdinsightactive
ms.date: 04/14/2020
ms.openlocfilehash: 951dba6e64561301dc7dbb6ebd6fd6b641c90a47
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93285676"
---
# <a name="understand-and-resolve-errors-received-from-webhcat-on-hdinsight"></a>Compréhension et résolution des erreurs reçues à partir de WebHCat sur HDInsight

Découvrez les erreurs reçues lors de l’utilisation de WebHCat avec HDInsight et comment les résoudre. WebHCat est utilisé en interne par les outils côté client tels qu’Azure PowerShell et Data Lake Tools pour Visual Studio.

## <a name="what-is-webhcat"></a>Présentation de WebHCat

[WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat) est une API REST pour [HCatalog](https://cwiki.apache.org/confluence/display/Hive/HCatalog), une couche de gestion du stockage et des tables pour Apache Hadoop. WebHCat est activé par défaut sur les clusters HDInsight et est utilisé par différents outils pour envoyer des tâches, obtenir le statut d’une tâche, etc. sans se connecter au cluster.

## <a name="modifying-configuration"></a>Modification de la configuration

Plusieurs des erreurs répertoriées dans ce document se produisent car une limite maximale configurée a été dépassée. Lorsque l’étape de résolution mentionne que vous pouvez modifier une valeur, utilisez Apache Ambari (web ou API REST) pour modifier la valeur. Pour plus d’informations, consultez [Gestion des clusters HDInsight à l’aide d’Apache Ambari](hdinsight-hadoop-manage-ambari.md)

### <a name="default-configuration"></a>Configuration par défaut

Le dépassement des valeurs par défaut suivantes peut entraîner une baisse des performances de WebHCat ou des erreurs :

| Paramètre | Résultat | Valeur par défaut |
| --- | --- | --- |
| [yarn.scheduler.capacity.maximum-applications][maximum-applications] |Nombre maximal de tâches pouvant être actives simultanément (en attente ou en cours d’exécution) |10 000 |
| [templeton.exec.max-procs][max-procs] |Nombre maximal de demandes pouvant être traitées simultanément |20 |
| [mapreduce.jobhistory.max-age-ms][max-age-ms] |Nombre de jours pendant lesquels l’historique des tâches est conservé. |7 jours |

## <a name="too-many-requests"></a>Trop de demandes

**Code d’état HTTP** : 429

| Cause | Résolution |
| --- | --- |
| Vous avez dépassé le nombre maximal de demandes simultanées prises en charge par WebHCat par minute (20 par défaut) |Réduisez votre charge de travail pour vérifier que vous n’avez pas dépassé le nombre maximal de demandes simultanées ou pour augmenter la limite de demandes simultanées en modifiant `templeton.exec.max-procs`. Pour en savoir plus, consultez la section [Modification de la configuration](#modifying-configuration) |

## <a name="server-unavailable"></a>Serveur non disponible

**Code d’état HTTP** : 503

| Cause | Résolution |
| --- | --- |
| Ce code d’état se produit généralement lors du basculement entre le HeadNode principal et secondaire du cluster. |Veuillez patienter deux minutes, puis recommencez l’opération. |

## <a name="bad-request-content-could-not-find-job"></a>Contenu de requête erroné : travail introuvable

**Code d’état HTTP** : 400

| Cause | Résolution |
| --- | --- |
| Les informations détaillées de la tâche ont été nettoyées par la tâche de nettoyage de l’historique |La période de conservation par défaut de l’historique des tâches est de 7 jours. La période de rétention par défaut peut être changée en modifiant `mapreduce.jobhistory.max-age-ms`. Pour en savoir plus, consultez la section [Modification de la configuration](#modifying-configuration) |
| Le travail a été supprimé en raison d’un basculement |Veuillez patienter deux minutes avant de renvoyer la tâche |
| ID de travail utilisé non valide |Vérifiez l’ID de travail |

## <a name="bad-gateway"></a>Passerelle incorrecte

**Code d’état HTTP** : 502

| Cause | Résolution |
| --- | --- |
| Le nettoyage de la mémoire interne coïncide avec le processus de WebHCat |Veuillez attendre que le nettoyage de la mémoire se termine ou redémarrez le service WebHCat |
| Le délai d’attente d’une réponse du service ResourceManager a expiré. Cette erreur peut se produire lorsque le nombre d’applications actives dépasse le nombre maximal configuré (10 000 par défaut) |Veuillez attendre que les tâches en cours d’exécution se terminent ou augmentez la limite de tâches simultanées en modifiant `yarn.scheduler.capacity.maximum-applications`. Pour en savoir plus, consultez la section [Modification de la configuration](#modifying-configuration). |
| Tentative de récupération de toutes les tâches par le biais de l’appel [GET /jobs](https://cwiki.apache.org/confluence/display/Hive/WebHCat+Reference+Jobs) lorsque `Fields` est défini sur `*` |Ne récupérez pas *tous* les détails des tâches. Utilisez plutôt `jobid` pour récupérer uniquement les détails des tâches dont l’ID a une valeur supérieure à un ID de tâche particulier. Ou, n’utilisez pas `Fields` |
| Le service WebHCat est arrêté pendant le basculement du HeadNode |Veuillez patienter deux minutes, puis recommencez l’opération |
| Plus de 500 tâches en attente ont été envoyées via WebHCat |Veuillez patienter le temps que les tâches en attente se terminent avant d’envoyer d’autres tâches |

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../includes/hdinsight-troubleshooting-next-steps.md)]

[maximum-applications]: https://docs.cloudera.com/HDPDocuments/HDP2/HDP-2.1.3/bk_system-admin-guide/content/setting_application_limits.html
[max-procs]: https://cwiki.apache.org/confluence/display/Hive/WebHCat+Configure#WebHCatConfigure-WebHCatConfiguration
[max-age-ms]: https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.6.0/ds_Hadoop/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml