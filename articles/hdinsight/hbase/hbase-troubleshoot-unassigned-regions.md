---
title: Problèmes avec les serveurs de région dans Azure HDInsight
description: Problèmes avec les serveurs de région dans Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 08/16/2019
ms.openlocfilehash: 6225e51a7707a926df91e68fbb27bfb56f89a3ee
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92539919"
---
# <a name="issues-with-region-servers-in-azure-hdinsight"></a>Problèmes avec les serveurs de région dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="scenario-unassigned-regions"></a>Scénario : Régions non affectées

### <a name="issue"></a>Problème

Lors de l’exécution de la commande `hbase hbck`, vous voyez un message d’erreur semblable à celui-ci :

```
multiple regions being unassigned or holes in the chain of regions
```

Dans l’interface utilisateur Apache HBase Master, vous pouvez voir le nombre de régions en état de déséquilibre sur tous les serveurs régionaux. Vous pouvez ensuite exécuter la commande `hbase hbck` pour afficher les trous dans la chaîne de régions.

### <a name="cause"></a>Cause

Les espaces peuvent être le résultat des régions hors connexion.

### <a name="resolution"></a>Résolution

Corrigez les attributions. Suivez les étapes ci-dessous pour rétablir les régions non attribuées à leur état normal :

1. Connectez-vous au cluster HDInsight HBase à l’aide de SSH.

1. Exécutez la commande `hbase zkcli` pour vous connecter au shell ZooKeeper.

1. Exécutez la commande `rmr /hbase/regions-in-transition` ou `rmr /hbase-unsecure/regions-in-transition`.

1. Quittez le shell zookeeper à l’aide de la commande `exit`.

1. Ouvrez l’interface utilisateur d’Apache Ambari et redémarrez le service HBase Master actif.

1. Exécutez de nouveau la commande `hbase hbck` (sans autre option). Vérifiez la sortie et assurez-vous que toutes les régions sont assignées.

---

## <a name="scenario-dead-region-servers"></a>Scénario : Serveurs de région inactifs

### <a name="issue"></a>Problème

Échec du démarrage des serveurs de région.

### <a name="cause"></a>Cause

Plusieurs répertoires WAL de fractionnement.

1. Obtenez la liste des répertoires WAL actuels : `hadoop fs -ls -R /hbase/WALs/ > /tmp/wals.out`.

1. Examinez le fichier `wals.out`. Si le nombre de répertoires de fractionnement (commençant par *-splitting) est trop important, le serveur de région est probablement défaillant en raison de ces répertoires.

### <a name="resolution"></a>Résolution

1. Arrêtez HBase à partir du portail Ambari.

1. Exécutez `hadoop fs -ls -R /hbase/WALs/ > /tmp/wals.out` pour afficher une liste actualisée des répertoires WAL.

1. Déplacez les répertoires *-splitting dans un dossier temporaire, `splitWAL`, et supprimez les répertoires *-splitting.

1. Exécutez la commande `hbase zkcli` pour vous connecter au shell zookeeper.

1. Exécutez `rmr /hbase-unsecure/splitWAL`.

1. Redémarrez le service HBase.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

* Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Connexion de la communauté Azure aux ressources appropriées : réponses, support technique et experts.

* Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support** . Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).