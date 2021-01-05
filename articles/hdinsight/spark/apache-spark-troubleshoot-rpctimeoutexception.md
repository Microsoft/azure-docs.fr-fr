---
title: RpcTimeoutException pour Apache Spark Thrift – Azure HDInsight
description: Vous constatez 502 erreurs lors du traitement de jeux de données volumineux à l’aide du serveur Thrift Apache Spark
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 07/29/2019
ms.openlocfilehash: b4b5268bef71bd6b23df6049fd3ac8af973d4385
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93287753"
---
# <a name="scenario-rpctimeoutexception-for-apache-spark-thrift-server-in-azure-hdinsight"></a>Scénario : RpcTimeoutException pour le serveur Thrift Apache Spark dans Azure HDInsight

Cet article décrit la procédure à suivre pour résoudre les problèmes rencontrés lors de l’utilisation de composants Apache Spark dans des clusters Azure HDInsight.

## <a name="issue"></a>Problème

L’application Spark échoue avec `org.apache.spark.rpc.RpcTimeoutException` une exception et un message : `Futures timed out`, comme le montre l’exemple suivant :

```
org.apache.spark.rpc.RpcTimeoutException: Futures timed out after [120 seconds]. This timeout is controlled by spark.rpc.askTimeout
 at org.apache.spark.rpc.RpcTimeout.org$apache$spark$rpc$RpcTimeout$$createRpcTimeoutException(RpcTimeout.scala:48)
```

`OutOfMemoryError` et `overhead limit exceeded` des erreurs peuvent également survenir dans le `sparkthriftdriver.log`, comme le montre l’exemple suivant :

```
WARN  [rpc-server-3-4] server.TransportChannelHandler: Exception in connection from /10.0.0.17:53218
java.lang.OutOfMemoryError: GC overhead limit exceeded
```

## <a name="cause"></a>Cause

Ces erreurs sont dues à un manque de ressources mémoire lors du traitement des données. Si le processus de nettoyage de la mémoire Java démarre, il peut causer l’arrêt du fonctionnement de l’application Spark. Les requêtes commencent à expirer et à arrêter le traitement. L’erreur `Futures timed out` indique qu’un cluster est soumis à de fortes contraintes.

## <a name="resolution"></a>Résolution

Augmentez la taille du cluster en ajoutant plus de nœuds Worker ou en augmentant la capacité de mémoire des nœuds de cluster existants. Vous pouvez également ajuster le pipeline des données pour réduire la quantité de données traitées en même temps.

Le `spark.network.timeout` contrôle le délai d’expiration pour toutes les connexions réseau. L’augmentation du délai d’expiration du réseau donne le temps nécessaires pour que certaines opérations critiques puissent se terminer, mais ne permettra pas de résoudre complètement le problème.

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]