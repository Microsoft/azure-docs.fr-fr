---
title: Délais d’attente avec la commande « HBase Hbck » dans Azure HDInsight
description: Problème d’expiration de délai d’attente avec la commande « hbase hbck » lors de la correction d’affectations de régions
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 08/16/2019
ms.openlocfilehash: c1269f159a23d5fd23e08e7abfbd9a05ddd533ba
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92539936"
---
# <a name="scenario-timeouts-with-hbase-hbck-command-in-azure-hdinsight"></a>Scénario : Délais d’attente avec la commande « HBase Hbck » dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue"></a>Problème

La commande `hbase hbck` rencontre des délais d’attente lors de la résolution d’attributions de région.

## <a name="cause"></a>Cause

Les problèmes de délai d’expiration rencontrés lors de l’utilisation de la commande `hbck` peuvent être dus à la présence de plusieurs régions en état de transition pendant un certain temps. Ces régions apparaissent comme étant hors connexion dans l’interface utilisateur HBase Master. En raison du grand nombre de régions en tentative de transition, HBase Master pourrait connaître un problème de délai d’expiration et ne pas parvenir à remettre ces régions en ligne.

## <a name="resolution"></a>Résolution

1. Connectez-vous au cluster HDInsight HBase à l’aide de SSH.

1. Exécutez la commande `hbase zkcli` pour vous connecter au shell Apache ZooKeeper.

1. Exécutez la commande `rmr /hbase/regions-in-transition` ou `rmr /hbase-unsecure/regions-in-transition`.

1. Quittez l’interpréteur de commandes `hbase zkcli` à l’aide de la commande `exit`.

1. Ouvrez l’interface utilisateur d’Apache Ambari et redémarrez le service HBase Master actif.

1. Exécutez la commande `hbase hbck -fixAssignments`.

1. Surveillez la section « region in transition » (région en transition) de l’interface utilisateur HBase Master pour vous assurer qu’aucune région n’est bloquée.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

- Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

- Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Connexion de la communauté Azure aux ressources appropriées : réponses, support technique et experts.

- Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support** . Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).