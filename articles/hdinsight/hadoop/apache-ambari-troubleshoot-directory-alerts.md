---
title: Alertes de répertoire Apache Ambari dans Azure HDInsight
description: Discussion et analyse des causes et solutions possibles pour les alertes de répertoire Apache Ambari dans HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 01/22/2020
ms.openlocfilehash: d8b1f705eb08d1c58b32e0cbd7c57722a6de93cc
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95998309"
---
# <a name="scenario-apache-ambari-directory-alerts-in-azure-hdinsight"></a>Scénario : Alertes de répertoire Apache Ambari dans Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Vous recevez des erreurs d’Apache Ambari semblables à celles-ci :

```
1/1 local-dirs have errors: [ /mnt/resource/hadoop/yarn/local : Cannot create directory: /mnt/resource/hadoop/yarn/local ]
1/1 log-dirs have errors: [ /mnt/resource/hadoop/yarn/log : Cannot create directory: /mnt/resource/hadoop/yarn/log ]
```

## <a name="cause"></a>Cause

Les répertoires indiqués dans l’alerte Ambari sont absents sur les nœuds Worker concernés.

## <a name="resolution"></a>Résolution

Créez manuellement les répertoires manquants sur les nœuds Worker concernés.

1. Utilisez le protocole SSH pour le nœud Worker approprié.

1. Obtenez l’utilisateur racine : `sudo su`.

1. Créez les répertoires nécessaires de manière récursive.

1. Modifiez le propriétaire et le groupe pour ces répertoires.

    ```bash
    chown -R yarn /mnt/resource/hadoop/yarn/local
    chgrp -R hadoop /mnt/resource/hadoop/yarn/local
    chown -R yarn /mnt/resource/hadoop/yarn/log
    chgrp -R hadoop /mnt/resource/hadoop/yarn/log
    ```

1. À partir de l’interface utilisateur Apache Ambari, désactivez, puis activez l’alerte.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]