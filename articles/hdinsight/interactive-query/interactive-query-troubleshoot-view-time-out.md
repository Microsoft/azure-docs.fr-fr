---
title: La vue Apache Hive expire à partir du résultat de requête - Azure HDInsight
description: La vue Apache Hive expire lors de la récupération d’un résultat de requête dans Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 07/30/2019
ms.openlocfilehash: c642760a5f6bfa7e59d42237e1583617e322ece3
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93288834"
---
# <a name="scenario-apache-hive-view-times-out-when-fetching-a-query-result-in-azure-hdinsight"></a>Scénario : La vue Apache Hive expire lors de la récupération d’un résultat de requête dans Azure HDInsight

Cet article décrit la procédure à suivre pour résoudre les problèmes rencontrés lors de l’utilisation de composants Interactive Query dans des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Lors de l’exécution de certaines requêtes à partir de la vue Apache Hive, l’erreur suivante peut se produire :

```
result fetch timed out
java.util.concurrent.TimeoutException: deadline passed
```

## <a name="cause"></a>Cause

La valeur du délai d’expiration par défaut de la vue Hive peut ne pas convenir à la requête que vous exécutez. La période spécifiée est trop courte pour que la vue Hive récupère le résultat de la requête.

## <a name="resolution"></a>Résolution

Augmentez les délais d’expiration de la vue Apache Ambari Hive en définissant les propriétés suivantes dans `/etc/ambari-server/conf/ambari.properties`.

```
views.ambari.request.read.timeout.millis=300000
views.request.read.timeout.millis=300000
views.ambari.hive.<HIVE_VIEW_INSTANCE_NAME>.result.fetch.timeout=300000
```

La valeur de `HIVE_VIEW_INSTANCE_NAME` est disponible à la fin de l’URL de la vue Hive.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]