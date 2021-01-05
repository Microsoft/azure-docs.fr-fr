---
title: 'BindException : adresse déjà utilisée dans Azure HDInsight'
description: 'BindException : adresse déjà utilisée dans Azure HDInsight'
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 08/16/2019
ms.openlocfilehash: 60f95292d517277f607c968629c55641e0eecce1
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92540259"
---
# <a name="scenario-bindexception---address-already-in-use-in-azure-hdinsight"></a>Scénario : BindException : adresse déjà utilisée dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Échec de l’opération de redémarrage du serveur de région Apache HBase. Dans le journal `region-server.log` dans le répertoire `/var/log/hbase` des nœuds worker où le démarrage des serveurs de région échoue, vous pouvez voir un message d’erreur semblable à ce qui suit :

```
Caused by: java.net.BindException: Problem binding to /10.2.0.4:16020 : Address already in use
...

Caused by: java.net.BindException: Address already in use
...
```

## <a name="cause"></a>Cause

Redémarrage des serveurs de région Apache HBase pendant une activité impliquant de lourdes charges de travail. Voici ce qui se passe en arrière-plan quand un utilisateur lance l’opération de redémarrage sur le serveur de région HBase à partir de l’interface utilisateur Apache Ambari :

1. Ambari Agent envoie une demande d’arrêt au serveur régional.

1. L’agent attend 30 secondes que le serveur s’arrête de manière appropriée

1. Si votre application continue à se connecter au serveur régional, ce dernier ne sera pas arrêté immédiatement. Le délai de 30 secondes expire avant l’arrêt.

1. Après 30 secondes, Ambari Agent force l’arrêt du serveur régional en lui envoyant une commande force-kill (`kill -9`).

1. En raison de cet arrêt brutal et même si le processus du serveur de région est arrêté, le port associé au processus peut ne pas être libéré, ce qui aboutit en fin de compte à `AddressBindException`.

## <a name="resolution"></a>Résolution

Réduisez la charge sur les serveurs de région HBase avant de lancer un redémarrage. En outre, il est judicieux de commencer par vider toutes les tables. Pour des informations de référence sur le vidage des tables, consultez [HDInsight HBase : Comment améliorer le délai de redémarrage de cluster Apache HBase en vidant les tables](https://web.archive.org/web/20190112153155/https://blogs.msdn.microsoft.com/azuredatalake/2016/09/19/hdinsight-hbase-how-to-improve-hbase-cluster-restart-time-by-flushing-tables/).

Vous pouvez également essayer de redémarrer manuellement les serveurs de région sur les nœuds Worker à l’aide des commandes suivantes :

```bash
sudo su - hbase -c "/usr/hdp/current/hbase-regionserver/bin/hbase-daemon.sh stop regionserver"
sudo su - hbase -c "/usr/hdp/current/hbase-regionserver/bin/hbase-daemon.sh start regionserver"
```

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

* Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Connexion de la communauté Azure aux ressources appropriées : réponses, support technique et experts.

* Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support** . Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).