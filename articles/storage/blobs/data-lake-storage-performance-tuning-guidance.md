---
title: Optimiser Azure Data Lake Storage Gen2 pour les performances | Microsoft Docs
description: Découvrez comment optimiser les performances d’Azure Data Lake Storage Gen2. Ingérez des données, structurez votre jeu de données, et bien plus encore.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 11/18/2019
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: 2011fa8e85f10f12ae914b02710bbd65f5700403
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95913043"
---
# <a name="optimize-azure-data-lake-storage-gen2-for-performance"></a>Optimiser Azure Data Lake Storage Gen2 pour les performances

Azure Data Lake Storage Gen2 prend en charge un débit élevé pour l’analytique intensive des E/S et le déplacement des données.  Dans Data Lake Storage Gen2, il est important d’utiliser tout le débit disponible (la quantité de données qui peuvent être lues ou écrites par seconde) pour optimiser les performances.  Pour cela, il convient d’effectuer le plus possible de lectures et d’écritures en parallèle.

![Performances de Data Lake Storage Gen2](./media/data-lake-storage-performance-tuning-guidance/throughput.png)

Data Lake Storage Gen2 peut être mis à l’échelle afin de fournir le débit nécessaire pour tous les scénarios d’analytique. Par défaut, un compte Data Lake Storage Gen2 fournit automatiquement suffisamment de débit pour répondre aux besoins d’une vaste catégorie de cas d’usage. Pour les cas où les clients atteignent la limite par défaut, vous pouvez configurer le compte Data Lake Storage Gen2 de manière à fournir un débit plus important en contactant le [support Azure](https://azure.microsoft.com/support/faq/).

## <a name="data-ingestion"></a>Ingestion de données

Lors de l’ingestion de données à partir d’un système source dans Data Lake Storage Gen2, vous devez impérativement tenir compte du fait que le matériel source, le matériel réseau source et la connectivité réseau à Data Lake Storage Gen2 peuvent agir comme goulot d’étranglement.  

![Diagramme montrant les facteurs à prendre en compte lors de l’ingestion de données d’un système source dans Data Lake Storage Gen2.](./media/data-lake-storage-performance-tuning-guidance/bottleneck.png)

Vous devez impérativement vous assurer que le déplacement des données n’est pas affecté par ces facteurs.

### <a name="source-hardware"></a>Matériel source

Que vous utilisiez des machines locales ou des machines virtuelles dans Azure, vous devez sélectionner avec soin le matériel adapté. Pour le matériel du disque source, préférez les disques SSD aux disques durs HDD et choisissez les disques avec les broches les plus rapides. Pour le matériel réseau source, utilisez la carte réseau la plus rapide possible.  Sur Azure, nous vous recommandons des machines virtuelles Azure D14 qui sont équipées de disques et du matériel de mise en réseau suffisamment puissants.

### <a name="network-connectivity-to-data-lake-storage-gen2"></a>Connectivité réseau à Data Lake Storage Gen2

La connectivité réseau entre vos données sources et Data Lake Storage Gen2 peut parfois être le goulot d’étranglement. Lorsque vos données sources sont locales, envisagez d’utiliser une liaison dédiée avec [Azure ExpressRoute](https://azure.microsoft.com/services/expressroute/). Si vos données sources sont dans Azure, les performances seront meilleures si les données sont dans la même région Azure que le compte Data Lake Storage Gen2.

### <a name="configure-data-ingestion-tools-for-maximum-parallelization"></a>Configurer les outils d’ingestion des données pour une parallélisation maximale

Après avoir traité les goulots d’étranglement du matériel source et de la connectivité réseau susmentionnés, vous pouvez configurer vos outils d’ingestion. Le tableau suivant résume les paramètres clés de plusieurs outils d’ingestion populaires et fournit des articles détaillés concernant le réglage des performances.  Pour en savoir plus sur l’outil à utiliser pour votre scénario, consultez cet [article](data-lake-storage-data-scenarios.md).

| Outil               | Paramètres     | Détails supplémentaires                                                                 |
|--------------------|------------------------------------------------------|------------------------------|
| DistCp            | -m (mappeur)   | [Lien](data-lake-storage-use-distcp.md#performance-considerations-while-using-distcp)                             |
| Azure Data Factory| parallelCopies    | [Lien](../../data-factory/copy-activity-performance.md)                          |
| Sqoop           | fs.azure.block.size, -m (mappeur)    |   [Lien](/archive/blogs/shanyu/performance-tuning-for-hdinsight-storm-and-microsoft-azure-eventhubs)        |

## <a name="structure-your-data-set"></a>Structure de votre jeu de données

Quand les données sont stockées dans Data Lake Storage Gen2, la taille du fichier, le nombre de fichiers et la structure de dossiers ont un impact sur les performances.  La section suivante décrit les meilleures pratiques en la matière.  

### <a name="file-size"></a>Taille du fichier

En règle générale, les moteurs d’analytique comme HDInsight et Azure Data Lake Analytics affichent une surcharge par fichier. Si vous stockez vos données sous forme de petits fichiers nombreux, cela peut nuire aux performances. De manière générale, organisez vos données dans de grands fichiers pour des performances optimales (taille allant de 256 Mo à 100 Go). Certains moteurs et applications peuvent avoir des difficultés à traiter efficacement les fichiers supérieurs à 100 Go.

Parfois, les pipelines de données ont un contrôle limité sur les données brutes qui possèdent un grand nombre de petits fichiers. En général, nous recommandons que votre système dispose d’une sorte de processus permettant d’agréger des fichiers de petite taille en fichiers plus volumineux pour les utiliser dans des applications en aval.

### <a name="organizing-time-series-data-in-folders"></a>Organisation des données de série chronologique dans des dossiers

Pour les charges de travail Hive, le nettoyage de partition de données de série chronologique peut aider certaines requêtes à lire uniquement un sous-ensemble de données, ce qui améliore les performances.    

Ces pipelines d’ingestion des données de série chronologique, organisent souvent leurs fichiers selon une structure d’appellation très structurée pour les fichiers et les dossiers. Découvrez ci-après un exemple très courant de données structurées par date :

*\DataSet\YYYY\MM\DD\datafile_YYYY_MM_DD.tsv*

Notez que les informations de date / heure s’affichent à la fois en tant que dossiers et dans le nom de fichier.

Pour la date et l’heure, le modèle suivant est récurrent.

*\DataSet\YYYY\MM\DD\HH\mm\datafile_YYYY_MM_DD_HH_mm.tsv*

Là encore, le type d’organisation des dossiers et fichiers sélectionné doit optimiser les plus gros fichiers et un nombre raisonnable de fichiers par dossier.

## <a name="optimizing-io-intensive-jobs-on-hadoop-and-spark-workloads-on-hdinsight"></a>Optimisation des tâches à usage intensif d’E/S sur les charges de travail Hadoop et Spark sur HDInsight

Ces tâches appartiennent à l’une des trois catégories suivantes :

* **Usage intensif du processeur.**  Ces tâches nécessitent de longues heures de calcul avec un minimum de temps d’E/S.  L’exemple inclut les tâches de traitement en langage naturel et Machine Learning.  
* **Utilisation intensive de la mémoire.**  Ces tâches utilisent beaucoup de mémoire.  Exemples : PageRank et travaux d’analytique en temps réel.  
* **Usage intensif des E/S.**  La plus grande partie du temps de ces tâches est consacrée aux E/S.  Exemple courant : tâche de copie qui ne traite que les opérations de lecture et écriture.  Autres exemples : tâches de préparation de données qui lisent une grande quantité de données, effectuent une transformation de données et réécrivent les données dans le magasin.  

Les instructions suivantes ne s’appliquent qu’aux tâches avec un usage intensif d’E/S.

## <a name="general-considerations"></a>Considérations d’ordre général

Vous pouvez avoir une tâche qui lit ou écrit jusqu’à 100 Mo en une seule opération, mais une mémoire tampon de cette taille risque de compromettre les performances.
Pour optimiser les performances, essayez de maintenir la taille des opérations d’E/S entre 4 et 16 Mo.

### <a name="general-considerations-for-an-hdinsight-cluster"></a>Considérations générales pour les clusters HDInsight

* **Versions HDInsight.** Pour des performances optimales, utilisez la dernière version de HDInsight.
* **Régions.** Placez le compte Data Lake Storage Gen2 dans la même région que le cluster HDInsight.  

Un cluster HDInsight est composé de deux nœuds principaux et de nœuds Worker. Chaque nœud Worker fournit une quantité spécifique de cœurs et de mémoire, déterminée par le type de machine virtuelle.  Lorsque vous exécutez une tâche, YARN est le négociateur de ressource qui alloue la mémoire et les cœurs disponible pour créer des conteneurs.  Chaque conteneur exécute les tâches nécessaires pour terminer la tâche.  Les conteneurs sont exécutés en parallèle pour traiter rapidement les tâches. Par conséquent, l’exécution du plus grand nombre possible de conteneurs en parallèle permet d’améliorer les performances.

Il existe trois couches au sein d’un cluster HDInsight qui peuvent être ajustées pour augmenter le nombre de conteneurs et utiliser tout le débit disponible.  

* **Couche physique**
* **Couche YARN**
* **Couche de charge de travail**

### <a name="physical-layer"></a>Couche physique

**Exécutez le cluster avec le plus de nœuds et/ou les plus grosses machines virtuelles.**  Un plus grand cluster permet d’exécuter plus de conteneurs YARN, comme le montre l’image ci-dessous.

![Diagramme illustrant la façon dont un cluster de plus grande taille vous permet d’exécuter davantage de conteneurs YARN.](./media/data-lake-storage-performance-tuning-guidance/VM.png)

**Utilisez des machines virtuelles avec davantage de bande passante réseau.**  La quantité de bande passante réseau peut être un goulot d’étranglement si elle moins importante que le débit de Data Lake Storage Gen2.  La taille de la bande passante réseau varie en fonction des machines virtuelles.  Choisissez un type de machine virtuelle qui possède la plus grande bande passante possible.

### <a name="yarn-layer"></a>Couche YARN

**Utilisez des conteneurs YARN plus petits.**  Réduisez la taille de chaque conteneur YARN pour créer plusieurs conteneurs avec la même quantité de ressources.

![Diagramme montrant le résultat de la réduction de la taille de chaque conteneur YARN pour créer d’autres conteneurs.](./media/data-lake-storage-performance-tuning-guidance/small-containers.png)

En fonction de votre charge de travail, il y aura toujours une taille de conteneur YARN minimale requise. Si vous sélectionnez un conteneur trop petit, vos tâches rencontreront des problèmes de mémoire insuffisante. En général, les conteneurs YARN ne doivent pas être inférieurs à 1 Go. Les conteneurs YARN de 3 Go sont courants. Pour certaines charges de travail, des conteneurs YARN plus grands peuvent être nécessaires.  

**Augmentez le nombre de cœurs par conteneur YARN.**  Augmentez le nombre de cœurs alloués à chaque conteneur pour augmenter le nombre de tâches parallèles qui s’exécutent dans chaque conteneur.  Cela fonctionne pour des applications comme Spark qui exécutent plusieurs tâches par conteneur.  Pour des applications comme Hive qui exécutent un seul thread par conteneur, il est préférable d’avoir plus de conteneurs plutôt que plus de cœurs par conteneur.

### <a name="workload-layer"></a>Couche de charge de travail

**Utilisez tous les conteneurs disponibles.**  Définissez le nombre de tâches de manière à ce qu’il soit égal ou supérieur au nombre de conteneurs disponibles. Ainsi, toutes les ressources sont utilisées.

![Diagramme montrant l’utilisation de tous les conteneurs.](./media/data-lake-storage-performance-tuning-guidance/use-containers.png)

**L’échec des tâches est coûteux.** Si chaque tâche doit traiter une grande quantité de données, l’échec d’une tâche entraîne une nouvelle tentative coûteuse.  Par conséquent, il est préférable de créer davantage de tâches, chacune d’elle traitant une petite quantité de données.

Outre les consignes générales ci-dessus, chaque application possède des paramètres différents à configurer pour cette application spécifique. Le tableau suivant répertorie certains des paramètres et des liens pour paramétrer les performances pour chaque application.

| Charge de travail | Paramètre de définition des tâches |
|----------|------------------------|
| [Spark sur HDInsight](data-lake-storage-performance-tuning-spark.md) | <ul><li>Num-executors</li><li>Executor-memory</li><li>Executor-cores</li></ul> |
| [Hive sur HDInsight](data-lake-storage-performance-tuning-hive.md) | <ul><li>hive.tez.container.size</li></ul> |
| [MapReduce sur HDInsight](data-lake-storage-performance-tuning-mapreduce.md) | <ul><li>Mapreduce.map.memory</li><li>Mapreduce.job.maps</li><li>Mapreduce.reduce.memory</li><li>Mapreduce.job.reduces</li></ul> |
| [Storm sur HDInsight](data-lake-storage-performance-tuning-storm.md)| <ul><li>Nombre de processus de travail</li><li>Nombre d’instances d’exécuteur de spout</li><li>Nombre d’instances d’exécuteur de bolt </li><li>Nombre de tâches de spout</li><li>Nombre de tâches de bolt</li></ul>|

## <a name="see-also"></a>Voir aussi
* [Présentation d’Azure Data Lake Storage Gen2](data-lake-storage-introduction.md)