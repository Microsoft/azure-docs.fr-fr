---
title: L’interface utilisateur Apache Ambari affiche les hôtes et les services dans Azure HDInsight
description: Dépannage d’un problème d’interface utilisateur Apache Ambari qui affiche les hôtes et les services dans Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 08/02/2019
ms.openlocfilehash: 9c9676fa1457be014098b8190986c617262e628d
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93286495"
---
# <a name="scenario-apache-ambari-ui-shows-down-hosts-and-services-in-azure-hdinsight"></a>Scénario : L’interface utilisateur Apache Ambari affiche les hôtes et les services dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue"></a>Problème

L’interface utilisateur Apache Ambari est accessible, mais l’interface utilisateur indique que presque tous les services sont en panne, tous les ordinateurs hôtes présentent une perte de pulsations.

## <a name="cause"></a>Cause

Dans la plupart des scénarios, il s’agit d’un problème avec le serveur Ambari qui ne s’exécute pas sur le nœud principal actif. Vérifiez quel nœud principal est le nœud principal actif et assurez-vous que votre serveur Ambari s’exécute sur le serveur approprié. Ne démarrez pas manuellement ambari-server, laissez le service de contrôleur de basculement responsable du démarrage de ambari-Server sur le nœud principal de droite. Redémarrez le nœud principal actif pour forcer un basculement.

Des problèmes de mise en réseau peuvent également être à l’origine de ce problème. À partir de chaque nœud de cluster, vérifiez si vous pouvez exécuter une commande ping sur `headnodehost`. Il existe une situation rare dans laquelle aucun nœud de cluster ne peut se connecter à `headnodehost`:

```
$>telnet headnodehost 8440
... No route to host
```

## <a name="resolution"></a>Résolution

En général, le redémarrage du nœud principal actif atténue ce problème. Si ce n’est pas le cas, contactez l’équipe d’assistance technique HDInsight.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]