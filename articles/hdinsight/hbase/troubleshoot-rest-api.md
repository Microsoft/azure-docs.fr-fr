---
title: API REST pour interroger Apache HBase dans Azure HDInsight
description: Cet article décrit les étapes de résolution des problèmes lors de l’interaction avec des composants Apache HBase sur des clusters Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 04/08/2020
ms.openlocfilehash: 1091366ac4d0445a465bc75913baf0f812e6a635
ms.sourcegitcommit: 3e8058f0c075f8ce34a6da8db92ae006cc64151a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92629287"
---
# <a name="rest-api-to-query-apache-hbase-in-azure-hdinsight"></a>API REST pour interroger Apache HBase dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue"></a>Problème

L’utilisation de l’interface REST d’Apache HBase pour interroger une table sous un espace de noms autre que celui par défaut entraîne une erreur de runtime (état HTTP 500).

## <a name="cause"></a>Cause :

L’API REST HBase est uniquement prise en charge lors de l’utilisation de l’espace de noms par défaut. Il s’agit d’un problème connu en ce qui concerne l’utilisation d’espaces de noms HBase ou l’exécution d’appels qui font référence à des commandes GET spécifiques sur des colonnes avec des familles de colonnes au moyen du serveur REST sur HDInsight. Cela est dû à un problème de sécurité avec la passerelle HDInsight. Lorsque vous utilisez l’API pour créer une table avec un espace de noms en accédant aux colonnes via des familles de colonnes, vous devez spécifier le caractère `:`, qui est considéré comme un problème de sécurité dans le module IIS Gateway.

## <a name="mitigation"></a>Limitation des risques

Contournez la passerelle ou le serveur REST en déployant votre application sur une machine virtuelle qui se trouve dans le même réseau virtuel Azure que le cluster HDInsight. Vous pouvez ensuite communiquer avec HBase directement via RPC (en ignorant entièrement le serveur REST) ou sur des serveurs REST individuels qui s’exécutent sur les nœuds Worker en contournant les passerelles HDInsight.

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’aide :

* Obtenez des réponses de la part d’experts Azure en faisant appel au [Support de la communauté Azure](https://azure.microsoft.com/support/community/).

* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Connexion de la communauté Azure aux ressources appropriées : réponses, support technique et experts.

* Si vous avez besoin d’une aide supplémentaire, vous pouvez envoyer une requête de support à partir du [Portail Microsoft Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade/). Sélectionnez **Support** dans la barre de menus, ou ouvrez le hub **Aide + Support**. Pour plus d’informations, consultez [Création d’une demande de support Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md). L’accès au support relatif à la gestion et à la facturation des abonnements est inclus avec votre abonnement Microsoft Azure. En outre, le support technique est fourni avec l’un des [plans de support Azure](https://azure.microsoft.com/support/plans/).