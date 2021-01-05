---
title: Le serveur Apache ZooKeeper ne parvient pas à former un quorum dans Azure HDInsight
description: Le serveur Apache ZooKeeper ne parvient pas à former un quorum dans Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 05/20/2020
ms.openlocfilehash: 680ba2d0840b0f447f9a3cb99707b8357c68f14e
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545512"
---
# <a name="apache-zookeeper-server-fails-to-form-a-quorum-in-azure-hdinsight"></a>Le serveur Apache ZooKeeper ne parvient pas à former un quorum dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre des problèmes en lien avec Zookeeper dans des clusters Azure HDInsight.

## <a name="symptoms"></a>Symptômes

* Les deux gestionnaires des ressources passent en mode veille.
* Les deux éléments Namenodes sont en mode veille.
* Des travaux Spark, Hive et Yarn ou des requêtes Hive échouent en raison d’échecs de connexion de Zookeeper.
* Le démarrage des démons LLAP échoue sur des clusters sécurisés Hive interactifs ou Spark.

## <a name="sample-log"></a>Exemple de journal

Un message d’erreur semblable au suivant peut s’afficher dans les journaux Yarn (/var/log/hadoop-yarn/yarn/yarn-yarn*.log sur les nœuds principaux) :

```output
2020-05-05 03:17:18.3916720|Lost contact with Zookeeper. Transitioning to standby in 10000 ms if connection is not reestablished.
Message
2020-05-05 03:17:07.7924490|Received RMFatalEvent of type STATE_STORE_FENCED, caused by org.apache.zookeeper.KeeperException$NoAuthException: KeeperErrorCode = NoAuth
...
2020-05-05 03:17:08.3890350|State store operation failed 
2020-05-05 03:17:08.3890350|Transitioning to standby state
```

## <a name="related-issues"></a>Problèmes connexes

* Des services de haute disponibilité tels que les Yarn, NameNode et les Livy peuvent s’arrêter pour de nombreuses raisons.
* Vérifiez dans les journaux que cela est lié à des connexions de Zookeeper.
* Assurez-vous que le problème est récurrent (n’utilisez pas ces solutions dans des cas isolés).
* Des travaux peuvent échouer temporairement en raison de problèmes de connexion de Zookeeper.

## <a name="common-causes-for-zookeeper-failure"></a>Causes courantes d’échec de Zookeeper

* Utilisation élevée de l’UC sur les serveurs Zookeeper :
  * Dans l’interface utilisateur Ambari, si vous constatez une utilisation intensive, proche de 100 %, de l’UC sur les serveurs Zookeeper, des sessions Zookeeper ouvertes pendant cette période peuvent expirer.
* Les clients Zookeeper signalent des délais d’expiration fréquents :
  * Les journaux pour Resource Manager, Namenode et autres, indiquent de fréquentes délais d’expiration de clients.
  * Cela peut entraîner une perte de quorum, des basculements fréquents et d’autres problèmes.

## <a name="check-for-zookeeper-status"></a>Vérifier l’état de Zookeeper

* Recherchez les serveurs Zookeeper dans le fichier /etc/hosts ou dans l’interface utilisateur d’Ambari.
* Exécutez la commande suivante :
  * `echo stat | nc <ZOOKEEPER_HOST_IP> 2181` (ou 2182).  
  * Le port 2181 est l’instance Apache Zookeeper.
  * Le port 2182 est utilisé par le Zookeeper HDInsight (pour fournir une haute disponibilité pour des services qui n’offrent pas de haute disponibilité en mode natif).
  * Si la commande n’affiche aucune sortie, cela signifie que les serveurs Zookeeper ne sont pas en cours d’exécution.
  * Si les serveurs sont en cours d’exécution, le résultat inclut des statistiques de connexions clientes et autres statistiques.

```output
Zookeeper version: 3.4.6-8--1, built on 12/05/2019 12:55 GMT
Clients:
 /10.2.0.57:50988[1](queued=0,recved=715,sent=715)
 /10.2.0.57:46632[1](queued=0,recved=138340,sent=138347)
 /10.2.0.34:14688[1](queued=0,recved=264653,sent=353420)
 /10.2.0.52:49680[1](queued=0,recved=134812,sent=134814)
 /10.2.0.57:50614[1](queued=0,recved=19812,sent=19812)
 /10.2.0.56:35034[1](queued=0,recved=2586,sent=2586)
 /10.2.0.52:63982[1](queued=0,recved=72215,sent=72217)
 /10.2.0.57:53024[1](queued=0,recved=19805,sent=19805)
 /10.2.0.57:45126[1](queued=0,recved=19621,sent=19621)
 /10.2.0.56:41270[1](queued=0,recved=1348743,sent=1348788)
 /10.2.0.53:59097[1](queued=0,recved=72215,sent=72217)
 /10.2.0.56:41088[1](queued=0,recved=788,sent=802)
 /10.2.0.34:10246[1](queued=0,recved=19575,sent=19575)
 /10.2.0.56:40944[1](queued=0,recved=717,sent=717)
 /10.2.0.57:45466[1](queued=0,recved=19861,sent=19861)
 /10.2.0.57:59634[0](queued=0,recved=1,sent=0)
 /10.2.0.34:14704[1](queued=0,recved=264622,sent=353355)
 /10.2.0.57:42244[1](queued=0,recved=49245,sent=49248)

Latency min/avg/max: 0/3/14865
Received: 238606078
Sent: 239139381
Connections: 18
Outstanding: 0
Zxid: 0x1004f99be
Mode: follower
Node count: 133212
```

## <a name="cpu-load-peaks-up-every-hour"></a>Pics de charge du processeur toutes les heures

* Connectez-vous au serveur Zookeeper et vérifiez le fichier /etc/crontab.
* Si des tâches horaires sont en cours d’exécution à ce moment-là, randomisez l’heure de début sur différents serveurs Zookeeper.
  
## <a name="purging-old-snapshots"></a>Purge d’anciens instantanés

* Des Zookeepers sont configurés pour purger automatiquement d’anciens instantanés.
* Par défaut, les 30 derniers instantanés sont conservés.
* Le nombre d’instantanés conservés est contrôlé par la clé de configuration `autopurge.snapRetainCount`. Cette propriété se trouve dans les fichiers suivants :
  * `/etc/zookeeper/conf/zoo.cfg` pour le Zookeeper Hadoop.
  * `/etc/hdinsight-zookeeper/conf/zoo.cfg` pour le Zookeeper HDInsight.
* Affectez la valeur 3 à `autopurge.snapRetainCount` et redémarrez les serveurs Zookeeper.
  * La configuration du Zookeeper Hadoop peut être mise à jour et le service redémarré via Ambari.
  * Arrêtez et redémarrez manuellement le Zookeeper HDInsight.
    * `sudo lsof -i :2182` indique l’ID de processus à arrêter.
    * `sudo python /opt/startup_scripts/startup_hdinsight_zookeeper.py`
* Ne purgez pas les instantanés manuellement : la suppression manuelle d’instantanés peut entraîner une perte de données.

## <a name="cancelledkeyexception-in-the-zookeeper-server-log-doesnt-require-snapshot-cleanup"></a>CancelledKeyException dans le journal du serveur Zookeeper ne nécessite pas de nettoyage d’instantané.

* Cette exception apparaît sur les serveurs Zookeeper (fichiers /var/log/zookeeper/zookeeper-zookeeper-* ou /var/log/hdinsight-zookeeper/zookeeper*)
* Cette exception signifie généralement que le client n’est plus actif et que le serveur est incapable d’envoyer un message.
* Cette exception indique également que le client Zookeeper met fin aux sessions prématurément.
* Recherchez les autres symptômes décrits dans ce document.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

- Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).
- Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Connexion de la communauté Azure aux ressources appropriées : réponses, support technique et experts.
- Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support** . Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).