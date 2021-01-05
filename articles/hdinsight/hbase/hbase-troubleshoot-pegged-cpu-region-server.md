---
title: Processeur invariable dans un cluster Apache HBase - Azure HDInsight
description: Résoudre les problèmes de processeur invariable sur un serveur de région d’un cluster Apache HBase sur Azure HDInsight
ms.service: hdinsight
ms.topic: troubleshooting
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.date: 08/01/2019
ms.openlocfilehash: bed73c3ccc7f514ffc9ff8f97534ae4b249834ce
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017013"
---
# <a name="scenario-pegged-cpu-on-region-server-in-apache-hbase-cluster-in-azure-hdinsight"></a>Scénario : Processeur invariable sur le serveur de région dans le cluster Apache HBase d'Azure HDInsight

Cet article décrit les éventuelles solutions à appliquer pour résoudre les problèmes rencontrés lors d’interactions avec des clusters Azure HDInsight.

## <a name="issue"></a>Problème

Le processus du serveur de région Apache HBase commence à occuper près de 200 % du processeur, ce qui entraîne le déclenchement d'alertes sur le processus et le cluster HBase Master, qui ne fonctionnent pas à pleine capacité.

## <a name="cause"></a>Cause

Si vous exécutez la version 3.4 du cluster HBase, vous avez peut-être rencontré un bogue provoqué par la mise à niveau du JDK vers la version 1.7.0_151. Le symptôme est le suivant : le processus du serveur de région commence à occuper près de 200 % du processeur (pour le vérifier, exécutez la commande `top` ; si un processus occupe près de 200 % du processeur, relevez son PID et vérifiez qu'il s'agit bien du processus du serveur de région en exécutant `ps -aux | grep`).

## <a name="resolution"></a>Résolution

1. Installez le JDK 1.8 sur TOUS les nœuds du cluster, comme indiqué ci-dessous :

    * Exécutez l'action de script `https://raw.githubusercontent.com/Azure/hbase-utils/master/scripts/upgradetojdk18allnodes.sh`. Veillez à sélectionner l'option d'exécution sur tous les nœuds.

    * Vous pouvez également vous connecter à chaque nœud individuel et exécuter la commande `sudo add-apt-repository ppa:openjdk-r/ppa -y && sudo apt-get -y update && sudo apt-get install -y openjdk-8-jdk`.

1. Accédez à l'interface utilisateur Ambari - `https://<clusterdnsname>.azurehdinsight.net`.

1. Accédez à **HBase -> Configurations -> Avancé -> Avancé** `hbase-env configs` et remplacez la variable `JAVA_HOME` par `export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64`. Enregistrez le changement de configuration.

1. [Facultatif mais recommandé] [Videz toutes les tables du cluster](/archive/blogs/azuredatalake/hdinsight-hbase-how-to-improve-hbase-cluster-restart-time-by-flushing-tables).

1. À partir de l'interface utilisateur Ambari, redémarrez tous les services HBase qui doivent l'être.

1. En fonction des données contenues sur le cluster, le délai nécessaire pour que celui-ci atteigne un état stable peut aller de quelques minutes à une heure. Pour vérifier que l'état du cluster est stable, consultez l'interface utilisateur HMaster (tous les serveurs de région doivent être actifs) à partir d'Ambari (actualiser) ou, depuis Headnode, exécutez l'interpréteur de commandes HBase puis la commande status.

Pour vérifier que la mise à niveau a bien été effectuée, vérifiez que les processus HBase appropriés sont lancés à l'aide de la version Java qui convient. Par exemple, pour le serveur de région, vérifiez ce qui suit :

```
ps -aux | grep regionserver, and verify the version like '''/usr/lib/jvm/java-8-openjdk-amd64/bin/java
```

## <a name="next-steps"></a>Étapes suivantes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]