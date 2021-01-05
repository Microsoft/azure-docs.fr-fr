---
title: 'Régler les performances : Storm, HDInsight et Azure Data Lake Storage Gen2 | Microsoft Docs'
description: Comprendre les recommandations relatives au réglage des performances d’une topologie Azure Storm sur un cluster Azure HDInsight et Azure Data Lake Storage Gen2.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 11/18/2019
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: 4db85357ee970d13d6b4fcce195cae66932bed18
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95912788"
---
# <a name="tune-performance-storm-hdinsight--azure-data-lake-storage-gen2"></a>Régler les performances : Storm, HDInsight et Azure Data Lake Storage Gen2

Comprendre les facteurs à prendre en compte lorsque vous optimisez les performances d’une topologie Storm dans Azure. Par exemple, il est important de comprendre les caractéristiques du travail effectué par les Spouts et les Bolts (si le travail est intensif en E/S ou en mémoire). Cet article aborde diverses recommandations d’optimisation des performances, y compris la résolution des problèmes courants.

## <a name="prerequisites"></a>Prérequis

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).
* **Un compte Azure Data Lake Storage Gen2**. Pour obtenir les instructions de création de compte, consultez [Démarrage rapide : Créer un compte de stockage pour l’analyse](../common/storage-account-create.md).
* Un **cluster Azure HDInsight** avec un accès à un compte Data Lake Storage Gen2. Consultez [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md). Veillez à activer le Bureau à distance pour le cluster.
* **Exécution d’un cluster Storm sur Data Lake Storage Gen2**. Pour plus d’informations, consultez [Présentation d’Apache Storm sur HDInsight : analyse en temps réel pour Hadoop](../../hdinsight/storm/apache-storm-overview.md).
* **Conseils de réglage des performances sur Data Lake Storage Gen2**.  Pour en savoir plus sur les concepts généraux relatifs aux performances, consultez [Recommandations en matière de réglage des performances de Data Lake Storage Gen2](data-lake-storage-performance-tuning-guidance.md).   

## <a name="tune-the-parallelism-of-the-topology"></a>Ajuster le parallélisme de la topologie

Vous pouvez améliorer les performances en augmentant l’accès concurrentiel des E/S vers et à partir de Data Lake Storage Gen2. Une topologie Storm possède un ensemble de configurations qui déterminent le parallélisme :
* Nombre de processus Worker (les Workers sont répartis uniformément sur les machines virtuelles).
* Nombre d’instances d’exécuteur de Spout.
* Nombre d’instances d’exécuteur de Bolt.
* Nombre de tâches de Spout.
* Nombre de tâches de Bolt.

Par exemple, sur un cluster avec 4 machines virtuelles et 4 processus Worker, 32 exécuteurs de Spout et 32 tâches de Spout, 256 exécuteurs de Bolt et 512 tâches de Bolt, prenez en compte les aspects suivants :

Chaque superviseur, qui est un nœud Worker, a un seul processus Worker de machine virtuelle Java. Ce processus de machine virtuelle Java gère 4 threads de Spout et 64 threads de Bolt. Au sein de chaque thread, les tâches sont exécutées séquentiellement. Avec la configuration précédente, chaque thread de Spout comporte 1 tâche, et chaque thread de Bolt comporte 2 tâches.

Dans Storm, voici les différents composants impliqués et leur impact sur le niveau de parallélisme que vous obtenez :
* Le nœud principal (appelé Nimbus dans Storm) est utilisé pour envoyer et gérer les travaux. Ces nœuds n’ont aucun impact sur le degré de parallélisme.
* Les nœuds superviseurs. Dans HDInsight, ils correspondent aux nœuds Worker de machine virtuelle Azure.
* Les tâches worker sont des processus Storm s’exécutant sur des machines virtuelles. Chaque tâche Worker correspond à une instance de machine virtuelle Java. Storm distribue le nombre de processus worker que vous spécifiez de manière aussi égale que possible sur les nœuds de travail.
* Instances d’exécuteur de Spout et de Bolt. Chaque instance d’exécuteur correspond à un thread s’exécutant dans les processus Worker (machine virtuelle Java).
* Tâches de Storm. Il s’agit des tâches logiques exécutées par chacun de ces threads. Cela ne modifie pas le niveau de parallélisme. Vous devez donc évaluer si vous avez besoin de plusieurs tâches par exécuteur ou non.

### <a name="get-the-best-performance-from-data-lake-storage-gen2"></a>Obtenir des performances optimales pour Data Lake Storage Gen2

Quand vous travaillez avec Data Lake Storage Gen2, vous pouvez optimiser les performances en adoptant les pratiques suivantes :
* Fusionnez vos petits éléments dans des tailles supérieures.
* Effectuez autant de requêtes simultanées que vous le pouvez. Étant donné que chaque thread de Bolt effectue des lectures bloquantes, il est préférable d’avoir un nombre de threads dans la plage de 8 à 12 par cœur. Cela permet de garantir une utilisation correcte de la carte réseau et du processeur. Une machine virtuelle plus grande permet d’avoir plus de demandes simultanées.  

### <a name="example-topology"></a>Exemple de topologie

Supposons que vous disposez d’un cluster de 8 nœuds Worker avec une machine virtuelle Azure D13v2. Cette machine virtuelle a 8 cœurs, donc sur les 8 nœuds Worker, vous avez un total de 64 cœurs.

Supposons que nous utilisons 8 threads bolt par cœur. Avec 64 cœurs, nous voulons 512 instances (c’est-à-dire les threads) d’exécuteur de Bolt au total. Dans ce cas, supposons que nous commençons avec une machine virtuelle Java par machine virtuelle et utilisons principalement l’accès concurrentiel des threads dans la machine virtuelle Java pour obtenir l’accès concurrentiel. Cela signifie que nous avons besoins de 8 tâches worker (une par machine virtuelle Azure) et 512 exécuteurs bolt. Avec cette configuration, Storm va tenter de distribuer les Workers uniformément entre les nœuds Worker (également appelés nœuds superviseurs), en donnant à chaque nœud Worker une machine virtuelle Java. À présent dans l’exemple, Storm tente de distribuer les exécuteurs uniformément entre les superviseurs, en donnant à chaque superviseur (c’est-à-dire chaque machine virtuelle Java) 8 threads chacun.

## <a name="tune-additional-parameters"></a>Ajuster les paramètres supplémentaires
Une fois que vous avez la topologie de base, vous pouvez envisager de modifier les paramètres :
* **Number of JVMs per worker node (Nombre de machines virtuelles Java par nœud Worker).** Si vous avez une structure de données de grande taille (par exemple, une table de recherche) que vous hébergez dans la mémoire, chaque machine virtuelle Java requiert une copie distincte. Vous pouvez également utiliser la structure de données entre plusieurs threads si vous avez moins de machines virtuelles Java. Pour les E/S de Bolt, le nombre de machines virtuelles Java ne fait pas beaucoup de différence pour le nombre de threads ajoutés sur ces machines virtuelles Java. Pour plus de simplicité, il est judicieux d’avoir une machine virtuelle Java par Worker. En fonction de ce que fait votre Bolt ou du type de traitement d’application nécessaire, vous devrez peut-être modifier ce nombre.
* **Number of spout executors (Nombre d’exécuteurs de Spout).** Étant donné que l’exemple précédent utilise les Bolts pour écrire dans Data Lake Storage Gen2, le nombre de Spouts ne concerne pas directement les performances Bolt. Toutefois, selon la quantité de traitement ou d’E/S se produisant dans le Spout, il est judicieux d’ajuster les Spouts pour de meilleures performances. Assurez-vous d’avoir suffisamment de Spouts pour occuper les Bolts. Les taux de sortie des Spouts doivent correspondre au débit des Bolts. La configuration réelle varie selon le Spout.
* **Nombre de tâches.** Chaque Bolt s’exécute en tant que thread unique. Les tâches supplémentaires par Bolt n’apportent pas d’accès concurrentiel supplémentaire. Le seul moment où elles sont utiles est si votre processus d’accusé de réception de tuple représente une grande partie du temps d’exécution de votre bolt. Il est judicieux de rassembler plusieurs tuples dans un élément plus grand avant d’envoyer un accusé de réception à partir du Bolt. Par conséquent, dans la plupart des cas, plusieurs tâches ne fournissent pas d’avantage supplémentaire.
* **Local or shuffle grouping (Groupage local ou aléatoire).** Lorsque ce paramètre est activé, les tuples sont envoyés vers des Bolts dans le même processus Worker. Cela réduit les communications entre processus et les appels réseau. Cela est recommandé pour la plupart des topologies.

Ce scénario de base est un bon point de départ. Effectuez un test avec vos propres données pour régler les paramètres précédents et ainsi optimiser les performances.

## <a name="tune-the-spout"></a>Ajuster le Spout

Vous pouvez modifier les paramètres suivants pour ajuster le Spout.

- **Délai d’expiration de tuple : topology.message.timeout.secs**. Ce paramètre détermine la durée d’attente de fin et d’accusé de réception d’un message avant de le considérer comme ayant échoué.

- **Mémoire maximale par le processus worker : worker.childopts**. Ce paramètre permet de spécifier des paramètres de ligne de commande supplémentaires pour les Workers Java. Le paramètre le plus couramment utilisé ici est XmX, qui détermine la quantité maximale de mémoire allouée au segment de mémoire d’une machine virtuelle Java.

- **Nombre max. de spouts en attente : topology.max.spout.pending**. Ce paramètre détermine le nombre de tuples qui peuvent être en cours de transmission (pas encore acceptés sur tous les nœuds de la topologie) par thread de Spout à tout moment.

  Un bon calcul à faire est l’estimation de la taille de chacun de vos tuples. Ensuite, calculez la quantité de mémoire par thread spout. La mémoire totale allouée à un thread divisée par cette valeur devrait vous donner la limite supérieure pour le paramètre de nombre maximal de Spouts en attente.

Le Bolt Storm par défaut de Data Lake Storage Gen2 a un paramètre de stratégie de synchronisation de taille (fileBufferSize) que vous pouvez utiliser pour ajuster ce paramètre.

Dans les topologies intensives en E/S, il est judicieux que chaque thread de Bolt écrive dans son propre fichier et définisse une stratégie de rotation de fichiers (fileRotationSize). Lorsque le fichier atteint une certaine taille, le flux est automatiquement vidé, et un nouveau fichier est utilisé pour l’écriture. La taille de fichier recommandée pour la rotation est de 1 Go.

## <a name="monitor-your-topology-in-storm"></a>Surveiller votre topologie dans Storm  
Lorsque votre topologie est en cours d’exécution, vous pouvez la surveiller dans l’interface utilisateur de Storm. Voici les principaux paramètres à examiner :

* **Total process execution latency (Latence totale de l’exécution du processus).** Il s’agit de la durée moyenne nécessaire à un tuple pour être émis par le Spout, traité par le Bolt et être accepté.

* **Total bolt process latency (Latence totale du processus Bolt).** Il s’agit du temps moyen passé par le tuple sur le Bolt jusqu’à réception d’un accusé de réception.

* **Total bolt execute latency (Latence totale d’exécution de Bolt).** Il s’agit du temps moyen passé par le Bolt dans la méthode execute.

* **Nombre d’échecs.** Cela fait référence au nombre de tuples qui n’ont pas pu être entièrement traités avant expiration.

* **Capacité.** Il s’agit d’une mesure du niveau d’activité de votre système. Si ce nombre a la valeur 1, vos bolts fonctionnent aussi rapidement que possible. S’il est inférieur à 1, augmentez le parallélisme. S’il est supérieur à 1, réduisez le parallélisme.

## <a name="troubleshoot-common-problems"></a>Résoudre les problèmes courants
Voici quelques scénarios courants de résolution des problèmes.
* **Grand nombre de tuples expirés.** Examinez chaque nœud de la topologie pour déterminer où se trouve le goulet d’étranglement. La raison la plus courante est que les Bolts ne sont pas en mesure de suivre le rythme des Spouts. La conséquence est que des tuples bouchent les mémoires tampon internes en attendant d’être traités. Envisagez d’augmenter la valeur de délai d’attente ou de diminuer le nombre maximal de Spouts en attente.

* **Latence totale d’exécution de processus élevée, mais latence de traitement des Bolts faible.** Dans ce cas, il est possible que les tuples ne soient pas reconnus suffisamment vite. Vérifiez qu’il existe un nombre suffisant de validateurs. Une autre possibilité est qu’ils restent en file d’attente trop longtemps avant que les Bolts lancent leur traitement. Réduisez le nombre maximal de spouts en attente.

* **Latence d’exécution des Bolts élevée.** Cela signifie que la méthode execute() de votre Bolt prend trop de temps. Optimisez le code ou examinez le comportement de vidage et les tailles d’écriture.

### <a name="data-lake-storage-gen2-throttling"></a>Limitation de bande passante Data Lake Storage Gen2
Vous pouvez constater des échecs de tâche si vous atteignez les limites de la bande passante fournie par Data Lake Storage Gen2. Consultez les journaux d’activité des tâches pour les erreurs de limitation. Vous pouvez réduire le parallélisme en augmentant la taille de conteneur.    

Pour vérifier si une limitation est appliquée, activez la journalisation du débogage côté client :

1. Dans **Ambari** > **Storm** > **Config** > **Avancé storm-worker-log4j**, remplacez **&lt;root level="info"&gt;** par **&lt;root level=”debug”&gt;** . Redémarrez tous les nœuds/le service pour que la configuration prenne effet.
2. Surveillez l’éventuelle présence d’exceptions de limitation de Data Lake Storage Gen2 dans les journaux d’activité de topologie Storm sur les nœuds Worker (sous /var/log/storm/worker-artifacts/&lt;NomTopologie&gt;/&lt;port&gt;/worker.log).

## <a name="next-steps"></a>Étapes suivantes
Pour optimiser davantage les performances, consultez [ce blog](/archive/blogs/shanyu/performance-tuning-for-hdinsight-storm-and-microsoft-azure-eventhubs).

Pour exécuter un exemple supplémentaire, consultez [celui-ci sur GitHub](https://github.com/hdinsight/storm-performance-automation).