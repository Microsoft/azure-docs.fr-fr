---
title: Problèmes de pulsations Apache Ambari dans Azure HDInsight
description: Examiner les diverses raisons à l’origine des problèmes de pulsation Apache Ambari dans Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 02/06/2020
ms.openlocfilehash: adc5dfcef8cce269b6b6d982178433b8ee163f92
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95998264"
---
# <a name="apache-ambari-heartbeat-issues-in-azure-hdinsight"></a>Problèmes de pulsations Apache Ambari dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="scenario-high-cpu-utilization"></a>Scénario : Utilisation élevée du processeur

### <a name="issue"></a>Problème

L’agent Ambari a une utilisation élevée du processeur, ce qui génère dans l’interface utilisateur Ambari des alertes pour certains nœuds indiquant que la pulsation de l’agent Ambari est perdue. L’alerte de pulsation perdue est généralement temporaire.

### <a name="cause"></a>Cause

En raison de divers bogues, dans de rares cas, votre agent Ambari peut avoir un pourcentage élevé (proche de 100) d’utilisation du processeur.

### <a name="resolution"></a>Résolution

1. Identifiez l’ID de processus (PID) de l’agent Ambari :

    ```bash
    ps -ef | grep ambari_agent
    ```

1. Exécutez ensuite la commande suivante pour afficher l’utilisation de l’UC :

    ```bash
    top -p <ambari-agent-pid>
    ```

1. Redémarrez l’agent Ambari pour atténuer le problème :

    ```bash
    service ambari-agent restart
    ```

1. Si le redémarrage ne fonctionne pas, arrêtez le processus de l’agent Ambari, puis démarrez-le :

    ```bash
    kill -9 <ambari-agent-pid>
    service ambari-agent start
    ```

---

## <a name="scenario-ambari-agent-not-started"></a>Scénario : Agent Ambari non démarré

### <a name="issue"></a>Problème

L’agent Ambari n’a pas démarré, ce qui génère dans l’interface utilisateur Ambari des alertes pour certains nœuds indiquant que la pulsation de l’agent Ambari est perdue.

### <a name="cause"></a>Cause

Les alertes sont provoquées par l’agent Ambari qui n’est pas en cours d’exécution.

### <a name="resolution"></a>Résolution

1. Confirmez l’état de l’agent Ambari :

    ```bash
    service ambari-agent status
    ```

1. Vérifiez si les services de contrôleur de basculement sont en cours d’exécution :

    ```bash
    ps -ef | grep failover
    ```

    Si les services du contrôleur de basculement ne sont pas en cours d’exécution, il est probable qu’un problème empêche l’agent HDInsight de démarrer le contrôleur de basculement. Vérifiez le journal de l’agent HDInsight à partir du fichier `/var/log/hdinsight-agent/hdinsight-agent.out`.

## <a name="scenario-heartbeat-lost-for-ambari"></a>Scénario : Pulsation perdue pour Ambari

### <a name="issue"></a>Problème

L’agent de pulsation Ambari a été perdu.

### <a name="cause"></a>Cause

Les journaux OMS sont à l’origine d’une utilisation élevée de l’UC.

### <a name="resolution"></a>Résolution

* Désactivez la journalisation Azure Monitor à l’aide de l’applet de commande PowerShell [Disable-AzHDInsightMonitoring](/powershell/module/az.hdinsight/disable-azhdinsightmonitoring).
* Supprimez le fichier journal `mdsd.warn`

---

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]