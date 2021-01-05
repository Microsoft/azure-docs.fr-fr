---
title: 'Migration des données : Apache Hadoop en local vers Azure HDInsight'
description: Découvrez les meilleures pratiques concernant la migration de données afin de migrer des clusters Hadoop locaux vers Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: ashishth
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/22/2019
ms.openlocfilehash: d2a2c734b256ad934b7a17d7cefd1783b406e766
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92537182"
---
# <a name="migrate-on-premises-apache-hadoop-clusters-to-azure-hdinsight---data-migration-best-practices"></a>Migrer des clusters Apache Hadoop locaux vers Azure HDInsight – meilleures pratiques concernant la migration de données

Cet article donne des conseils pour la migration de données vers Azure HDInsight. Il fait partie d’une série qui propose des bonnes pratiques pour aider à la migration des systèmes Apache Hadoop locaux vers Azure HDInsight.

## <a name="migrate-on-premises-data-to-azure"></a>Migrer des données locales vers Azure

Il existe deux options principales pour migrer des données d’un environnement local vers l’environnement Azure :

* Transférer des données sur le réseau avec TLS
    * Sur Internet : vous pouvez transférer des données vers le stockage Azure via une connexion Internet standard à l'aide d'un outil comme : l'Explorateur Stockage Azure, AzCopy, Azure Powershell et Azure CLI. Pour plus d’informations, consultez [Déplacer des données vers et à partir du Stockage Azure](../../storage/common/storage-choose-data-transfer-solution.md).

    * Express Route : ExpressRoute est un service Azure qui vous permet de créer des connexions privées entre les centres de données Microsoft et une infrastructure locale ou une installation de colocalisation. Les connexions ExpressRoute ne sont pas établies via le réseau public Internet ; elles offrent plus de sécurité, de fiabilité et de rapidité, mais moins de latences que les connexions classiques sur Internet. Pour plus d’informations, consultez l’article [Créer et modifier un circuit ExpressRoute](../../expressroute/expressroute-howto-circuit-portal-resource-manager.md).

    * Transfert de données en ligne Data Box : Data Box Edge et Data Box Gateway sont des produits de transfert de données en ligne, qui font office de passerelles de stockage réseau pour gérer les données transitant entre votre site et Azure. Data Box Edge est un appareil réseau local, qui échange des données avec Azure et utilise le computing en périphérie basé sur l’intelligence artificielle pour traiter les données. Data Box Gateway est une appliance virtuelle dotée de fonctionnalités de passerelle de stockage. Pour plus d’informations, consultez [Documentation Azure Data Box - Transfert en ligne](../../databox-online/index.yml).

* Expédition de données hors connexion

    Transfert de données hors connexion Data Box : les appareils Data Box, Data Box Disk et Data Box Heavy vous permettent de transférer de grandes quantités de données vers Azure quand l’utilisation du réseau n’est pas envisageable. Ces appareils de transfert de données hors connexion sont échangés entre votre organisation et le centre de données Azure. Ils utilisent un chiffrement AES pour protéger vos données en transit et font l’objet d’un processus complet d’assainissement après téléchargement, destiné à supprimer vos données. Pour plus d’informations sur les appareils de transfert Data Box, consultez [Documentation Azure Data Box - Transfert hors connexion](../../databox/index.yml). Pour plus d’informations sur la migration des clusters Hadoop, consultez [Utiliser Azure Data Box pour migrer des données à partir d’un magasin HDFS en local vers le stockage Azure](../../storage/blobs/data-lake-storage-migrate-on-premises-hdfs-cluster.md).

Le tableau suivant indique les durées moyennes de transfert de données en fonction du volume de données et de la bande passante réseau. Utilise une Data Box si la migration des données est censée prendre plus de trois semaines.

|Quantité de données | Bande passante réseau<br>sur<br>**45 Mbits/s (T3)**|Bande passante réseau<br>sur<br>**100 Mbits/s**|Bande passante réseau<br>sur<br>**1 Gbit/s**|Bande passante réseau<br>sur<br>**10 Gbits/s**|
|---|:---:|:---:|:---:|:---:|
|1 To|2 jours|1 jour| 2 heures|14 minutes|
|10 To|22 jours|10 jours|1 jour|2 heures|
|35 To|76 jours|34 jours|3 jours|8 heures|
|80 To|173 jours|78 jours|8 jours|19 heures|
|100 To|216 jours|97 jours|10 jours|1 jour|
|200 To|1 an|194 jours|19 jours|2 jours|
|500 To|3 ans|1 an|49 jours|5 jours|
|1 Po|6 ans|3 ans|97 jours|10 jours|
|2 Po|12 ans|5 ans|194 jours|19 jours|

Des outils natifs d'Azure, comme Apache Hadoop DistCp, Azure Data Factory et AzureCp, peuvent être utilisés pour transférer des données sur le réseau. L’outil tiers WANDisco peut également être utilisé dans le même but. Apache Kafka Mirrormaker et Apache Sqoop peuvent être utilisés pour le transfert de données en cours depuis l'environnement local vers le système de stockage Azure.

## <a name="performance-considerations-when-using-apache-hadoop-distcp"></a>Considérations relatives aux performances lors de l'utilisation d'Apache Hadoop DistCp

DistCp est un projet Apache qui utilise un travail de mappage MapReduce pour transférer des données, gérer des erreurs et résoudre ces dernières. Ce projet affecte une liste de fichiers sources à chaque tâche de mappage. Ensuite, la tâche de mappage copie tous les fichiers affectés dans la destination. Il existe plusieurs techniques pour améliorer les performances de DistCp.

### <a name="increase-the-number-of-mappers"></a>Augmenter le nombre de mappeurs

DistCp tente de créer des tâches de mappage afin que chacune d’entre elles copie à peu près le même nombre d’octets. Par défaut, les travaux DistCp utilisent 20 mappeurs. En utilisant plus de mappeurs pour Distcp (avec le paramètre « m » au niveau de la ligne de commande), vous augmentez le parallélisme lors du transfert de données et vous réduisez la longueur de ce processus. Toutefois, il existe deux choses à prendre en compte lors de l’augmentation du nombre de mappeurs :

* La granularité la plus basse de DistCp est un fichier unique. Indiquer un nombre de mappeurs plus élevé que le nombre de fichiers sources est inutile et gaspille les ressources de cluster disponibles.

* Prenez en considération la mémoire Yarn disponible sur le cluster pour définir le nombre de mappeurs. Chaque tâche de mappage est lancée en tant que conteneur Yarn. En partant du principe qu’aucune autre charge de travail lourde ne soit en cours d’exécution sur le cluster, le nombre de mappeurs peut être déterminé par la formule suivante : m = (nombre de nœuds worker \* mémoire YARN pour chaque nœud worker) / taille du conteneur YARN. Toutefois, si d’autres applications utilisent de la mémoire, choisissez d’utiliser uniquement une partie de la mémoire YARN pour les travaux DistCp.

### <a name="use-more-than-one-distcp-job"></a>Utiliser plusieurs travaux DistCp

Lorsque la taille du jeu de données à déplacer est supérieure à 1 To, utilisez plusieurs travaux DistCp. En utilisant plusieurs travaux, vous limitez l’impact des échecs. Si un travail échoue, il vous suffit de redémarrer celui-ci, et pas l’ensemble des travaux.

### <a name="consider-splitting-files"></a>Envisager le fractionnement des fichiers

Si vous disposez d’un petit nombre de fichiers volumineux, envisagez de les fractionner en blocs de fichiers de 256 Mo afin d’obtenir plus de concurrence potentielle avec plus de mappeurs.

### <a name="use-the-strategy-command-line-parameter"></a>Utiliser le paramètre de ligne de commande « strategy »

Envisagez d’utiliser le paramètre `strategy = dynamic` dans la ligne de commande. La valeur par défaut du paramètre `strategy` est `uniform size`. Dans ce cas, chaque mappage copie à peu près le même nombre d’octets. Quand ce paramètre est défini sur `dynamic`, le fichier listing est fractionné en plusieurs « blocs-de-fichiers » (chunk-files). Le nombre de blocs de fichiers est un multiple du nombre de mappages. Chaque tâche de mappage est affectée à un des blocs de fichiers. Une fois que tous les chemins d’accès d’un bloc ont été traités, le bloc en question est supprimé et un nouveau bloc est acquis. Le processus se poursuit jusqu’à ce qu’il n’y ait plus de bloc disponible. Cette approche « dynamique » permet aux tâches de mappage rapides de consommer plus de chemins d’accès que les tâches lentes, ce qui permet d’accélérer le travail DistCp dans sa totalité.

### <a name="increase-the-number-of-threads"></a>Augmenter le nombre de threads

Regardez si l’augmentation du paramètre `-numListstatusThreads` permet d’améliorer les performances. Ce paramètre contrôle le nombre de threads à utiliser pour la création de listes de fichiers. La valeur maximale de ce paramètre est de 40.

### <a name="use-the-output-committer-algorithm"></a>Utiliser l’algorithme de validateur de sortie

Regardez si le transfert du paramètre `-Dmapreduce.fileoutputcommitter.algorithm.version=2` permet d’améliorer les performances. Cet algorithme de validateur de sortie permet d’optimiser l’écriture de fichiers de sortie dans la destination. La commande suivante est un exemple qui illustre l’utilisation de différents paramètres :

```bash
hadoop distcp -Dmapreduce.fileoutputcommitter.algorithm.version=2 -numListstatusThreads 30 -m 100 -strategy dynamic hdfs://nn1:8020/foo/bar wasb://<container_name>@<storage_account_name>.blob.core.windows.net/foo/
```

## <a name="metadata-migration"></a>Migration de métadonnées

### <a name="apache-hive"></a>Apache Hive

Le metastore Hive peut être migré en utilisant soit les scripts soit la réplication de base de données.

#### <a name="hive-metastore-migration-using-scripts"></a>Migration du metastore Hive à l’aide des scripts

1. Générez les langages de définition de données (DDL) Hive à partir du metastore Hive local. Cette étape peut être effectuée à l'aide d'un [wrapper bash script](https://github.com/hdinsight/hdinsight.github.io/blob/master/hive/hive-export-import-metastore.md).
1. Modifiez le DDL généré pour remplacer l'URL HDFS par des URL WASB/ADLS/ABFS.
1. Exécutez le DDL mis à jour sur le metastore à partir du cluster HDInsight.
1. Assurez-vous que la version du metastore Hive est compatible avec l'environnement local et cloud.

#### <a name="hive-metastore-migration-using-db-replication"></a>Migration du metastore Hive à l’aide de la réplication de base de données

- Configurez la réplication de base de données entre la base de données du metastore Hive local et la base de données du metastore HDInsight.
- Utilisez « Hive MetaTool » pour remplacer l’URL HDFS par les URL WASB/ADLS/ABFS. Par exemple :

    ```bash
    ./hive --service metatool -updateLocation hdfs://nn1:8020/ wasb://<container_name>@<storage_account_name>.blob.core.windows.net/
    ```

### <a name="apache-ranger"></a>Apache Ranger

- Exportez les stratégies Ranger locales dans des fichiers xml.
- Transformez des chemins locaux basés sur HDFS en WASB/ADLS, à l’aide d’un outil tel que XSLT.
- Importez les stratégies sur l’exécution Ranger dans HDInsight.

## <a name="next-steps"></a>Étapes suivantes

Lisez l’article suivant de cette série :

- [Bonnes pratiques concernant la sécurité pour une migration locale vers Azure HDInsight Hadoop](apache-hadoop-on-premises-migration-best-practices-security-devops.md)