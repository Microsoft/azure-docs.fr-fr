---
title: Sauvegarde et réplication pour Apache HBase – Phoenix – Azure HDInsight
description: Configurer la sauvegarde et la réplication pour Apache HBase et Apache Phoenix dans Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/19/2019
ms.openlocfilehash: 3ed55387034a383e402d027fd5cab60c4a59c23c
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94657038"
---
# <a name="set-up-backup-and-replication-for-apache-hbase-and-apache-phoenix-on-hdinsight"></a>Configurer la sauvegarde et la réplication pour Apache HBase et Apache Phoenix sur HDInsight

Apache HBase prend en charge plusieurs approches pour la protection contre les pertes de données :

* Copier le dossier `hbase`
* Exporter puis importer
* Copier des tables
* Instantanés
* Réplication

> [!NOTE]  
> Apache Phoenix stocke ses métadonnées dans les tables HBase, ceci pour que les données soient sauvegardées lors de la sauvegarde des tables de catalogue de système HBase.

Les sections suivantes décrivent le scénario d’utilisation de chacune de ces approches.

## <a name="copy-the-hbase-folder"></a>Copier le dossier HBase

Avec cette approche, vous copiez l’ensemble des données HBase, sans pouvoir sélectionner de sous-ensemble de tables ou de familles de colonnes. Les approches suivantes procurent un contrôle supérieur.

HBase dans HDInsight a recours au stockage par défaut sélectionné lors de la création du cluster, qu’il s’agisse d’objets blob du stockage Azure ou d’une instance Azure Data Lake Storage. Dans les deux cas, HBase stocke ses données et ses fichiers de métadonnées sous le chemin d’accès suivant :

`/hbase`

* Dans un compte Stockage Azure, le dossier `hbase` est hébergé à la racine du conteneur d’objets blob :

  `wasbs://<containername>@<accountname>.blob.core.windows.net/hbase`

* Dans Azure Data Lake Storage, le dossier `hbase` est hébergé sous le chemin racine spécifié lors du provisionnement d’un cluster. Ce chemin d’accès racine comporte généralement un dossier `clusters`, avec un sous-dossier portant le nom de votre cluster HDInsight :

  `/clusters/<clusterName>/hbase`

Dans les deux cas, le dossier `hbase` comporte l’ensemble des données que HBase a transféré sur le disque, mais il ne peut toutefois pas contenir les données en mémoire. Avant que vous ne puissiez considérer ce dossier comme une représentation précise des données HBase, vous devez fermer ce cluster.

Une fois que vous avez supprimé le cluster, vous pouvez laisser les données telles quelles, ou les copier vers un nouvel emplacement :

* Créez une nouvelle instance HDInsight pointant vers l’emplacement actuel de stockage. La nouvelle instance est créée avec l’ensemble des données existantes.

* Copiez à un autre emplacement le dossier `hbase` dans un conteneur d’objets blob du stockage Azure ou dans une instance Data Lake Storage, puis démarrez un nouveau cluster avec ces données. Pour le stockage Azure, utilisez [AzCopy](../../storage/common/storage-use-azcopy-v10.md), et pour Data Lake Storage, utilisez [AdlCopy](../../data-lake-store/data-lake-store-copy-data-azure-storage-blob.md).

## <a name="export-then-import"></a>Exporter puis importer

Sur le cluster HDInsight source, utilisez l’[utilitaire d’exportation](https://hbase.apache.org/book.html#export) (inclus avec HBase) pour exporter les données d’une table source vers le stockage joint par défaut. Vous pouvez ensuite copier le dossier exporté vers l’emplacement de stockage de destination et exécuter l’[utilitaire d’importation](https://hbase.apache.org/book.html#import) sur le cluster HDInsight de destination.

Pour exporter les données de table, utilisez tout d’abord SSH dans le nœud principal de votre cluster HDInsight source, puis exécutez la commande `hbase` suivante :

```console
hbase org.apache.hadoop.hbase.mapreduce.Export "<tableName>" "/<path>/<to>/<export>"
```

Le répertoire d’exportation ne doit pas déjà exister. Le nom de la table respecte la casse.

Pour importer les données de table, utilisez SSH dans le nœud principal de votre cluster HDInsight de destination, puis exécutez la commande `hbase` suivante :

```console
hbase org.apache.hadoop.hbase.mapreduce.Import "<tableName>" "/<path>/<to>/<export>"
```

La table doit déjà exister.

Spécifiez le chemin d’accès complet d’exportation sur le stockage par défaut ou sur l’une des options de stockage jointes. Par exemple, dans Stockage Azure :

`wasbs://<containername>@<accountname>.blob.core.windows.net/<path>`

Dans Azure Data Lake Storage Gen2, la syntaxe est la suivante :

`abfs://<containername>@<accountname>.dfs.core.windows.net/<path>`

Dans Azure Data Lake Storage Gen1, la syntaxe est la suivante :

`adl://<accountName>.azuredatalakestore.net:443/<path>`

Cette approche procure une granularité au niveau de la table. Vous pouvez également spécifier une plage de données pour les lignes à inclure, ce qui vous permet d’exécuter le processus de manière incrémentielle. Depuis l’ère Unix, chaque date est exprimée en millisecondes.

```console
hbase org.apache.hadoop.hbase.mapreduce.Export "<tableName>" "/<path>/<to>/<export>" <numberOfVersions> <startTimeInMS> <endTimeInMS>
```

Notez que vous devez spécifier le nombre de versions de chaque ligne à exporter. Pour inclure l’ensemble des versions dans la plage de dates, définissez `<numberOfVersions>` sur une valeur supérieure à vos versions maximales possibles de lignes, comme 100000.

## <a name="copy-tables"></a>Copier des tables

L’[utilitaire de copie de table](https://hbase.apache.org/book.html#copy.table) copie les données d’une table source, ligne par ligne, vers une table de destination existante présentant un schéma identique à celui de la source. La table des destination peut être sur le même cluster, ou sur un clustr HBase différent. Les noms de table sont sensibles à la casse.

Pour utiliser l’utilitaire dans un cluster, exécutez SSH dans le nœud principal de votre cluster HDInsight source, puis exécutez cette commande `hbase` :

```console
hbase org.apache.hadoop.hbase.mapreduce.CopyTable --new.name=<destTableName> <srcTableName>
```

Pour valoriser l’utilitaire afin de copier vers une table d’un cluster différent, ajoutez le commutateur `peer` avec l’adresse du cluster de destination :

```console
hbase org.apache.hadoop.hbase.mapreduce.CopyTable --new.name=<destTableName> --peer.adr=<destinationAddress> <srcTableName>
```

L’adresse de destination est composée des trois éléments suivants :

`<destinationAddress> = <ZooKeeperQuorum>:<Port>:<ZnodeParent>`

* `<ZooKeeperQuorum>` est une liste de nœuds Apache ZooKeeper séparés par des virgules. Exemple :

    zk0-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk4-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk3-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net

* `<Port>` sur HDInsight est défini par défaut sur 2181, et `<ZnodeParent>` est défini sur `/hbase-unsecure`, ainsi la valeur `<destinationAddress>` complète est la suivante :

    zk0-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk4-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk3-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net:2181:/hbase-unsecure

Pour découvrir comment récupérer ces valeurs pour votre cluster HDInsight, consultez la section [Collecter manuellement la liste du quorum Apache ZooKeeper](#manually-collect-the-apache-zookeeper-quorum-list) de cet article.

L’utilitaire de copie de table prend également en charge les paramètres spécifiant la plage temporelle des lignes à copier, et spécifiant le sous-ensemble de familles de colonnes d’une table à copier. Pour afficher la liste complète des paramètres pris en charge par l’utilitaire, exécutez-le sans aucun paramètre :

```console
hbase org.apache.hadoop.hbase.mapreduce.CopyTable
```

L’utilitaire analyse l’intégralité du contenu de la table source à copier sur la table de destination. Durant l’exécution de l’utilitaire, les performances de votre cluster HBase peuvent être réduites.

> [!NOTE]  
> Pour automatiser la copie des données entre les tables, consultez le script `hdi_copy_table.sh` dans le référentiel [Azure HBase Utils](https://github.com/Azure/hbase-utils/tree/master/replication) sur GitHub.

### <a name="manually-collect-the-apache-zookeeper-quorum-list"></a>Collecter manuellement la liste du quorum Apache ZooKeeper

Lorsque les deux clusters HDInsight se trouvent dans le même réseau virtuel, tel que décrit auparavant, la résolution de nom d’hôte interne est automatique. Pour exécuter l’utilitaire de copie de table pour des clusters HDInsight de deux réseaux virtuels distincts connectés par une passerelle VPN, vous devez fournir les adresses IP des hôtes des nœuds ZooKeeper du quorum.

Pour acquérir les noms d’hôtes du quorum, exécutez la commande curl suivante :

```console
curl -u admin:<password> -X GET -H "X-Requested-By: ambari" "https://<clusterName>.azurehdinsight.net/api/v1/clusters/<clusterName>/configurations?type=hbase-site&tag=TOPOLOGY_RESOLVED" | grep "hbase.zookeeper.quorum"
```

La commande curl récupère un document JSON comportant les informations de configuration HBase, tandis que la commande greg renvoie uniquement l’entrée « hbase.zookeeper.quorum », par exemple :

```output
"hbase.zookeeper.quorum" : "zk0-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk4-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net,zk3-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net"
```

La valeur des noms d’hôtes du quorum correspond à l’intégralité de la chaîne située à la droite du point virgule.

Pour récupérer les adresses IP de ces hôtes, exécutez la commande curl suivante pour chacun des hôtes de la liste précédente :

```console
curl -u admin:<password> -X GET -H "X-Requested-By: ambari" "https://<clusterName>.azurehdinsight.net/api/v1/clusters/<clusterName>/hosts/<zookeeperHostFullName>" | grep "ip"
```

Dans cette commande curl, `<zookeeperHostFullName>` est le nom DNS complet d’un hôte ZooKeeper, comme dans l’exemple `zk0-hdizc2.54o2oqawzlwevlfxgay2500xtg.dx.internal.cloudapp.net`. La sortie de la commande contient l’adresse IP de l’hôte spécifié, par exemple :

`100    "ip" : "10.0.0.9",`

Une fois que vous avez collecté les adresses IP de l’ensemble des nœuds ZooKeeper de votre quorum, reconstruisez l’adresse de destination :

`<destinationAddress>  = <Host_1_IP>,<Host_2_IP>,<Host_3_IP>:<Port>:<ZnodeParent>`

Dans notre exemple :

`<destinationAddress> = 10.0.0.9,10.0.0.8,10.0.0.12:2181:/hbase-unsecure`

## <a name="snapshots"></a>Instantanés

Les [instantanés](https://hbase.apache.org/book.html#ops.snapshots) vous permettent d’exécuter une sauvegarde à un point dans le temps des données de votre magasin de données HBase. Les instantanés présentent une charge minimale et s’exécutent en quelques secondes, dans la mesure où l’opération associée correspond en réalité à une opération de métadonnées consistant à capturer les noms de tous les fichiers stockés à cet instant. Au moment de la prise d’un instantané, aucune donnée réelle n’est copiée. Les instantanés s’appuient sur la nature immuable des données stockées dans HDFS, où les mises à jour, les suppressions et les insertions sont toutes représentées en tant que nouvelles données. Vous pouvez restaurer (*cloner*) un instantané sur le même cluster, ou exporter un instantané vers un cluster différent.

Pour créer un instantané, exécutez SSH dans le nœud principal de votre cluster HBase, puis démarrez le shell `hbase` :

```console
hbase shell
```

Dans le shell hbase, utiliser la commande d’instantané avec les noms de la table et de cet instantané :

```console
snapshot '<tableName>', '<snapshotName>'
```

Pour restaurer un instantané par le nom au sein du shell `hbase`, commencez par désactiver la table, puis restaurer l’instantané avant d’enfin réactiver la table :

```console
disable '<tableName>'
restore_snapshot '<snapshotName>'
enable '<tableName>'
```

Pour restaurer un instantané sur une nouvelle table, recourez à clone_snapshot :

```console
clone_snapshot '<snapshotName>', '<newTableName>'
```

Pour exporter un instantané vers HDFS afin de l’utiliser dans un autre cluster, créez dans un premier temps l’instantané selon la procédure décrite ci-dessus, puis exécutez l’utilitaire d’exportation d’instantané. Exécutez cet utilitaire depuis la session SSH sur le nœud principal, pas au sein du shell `hbase` :

```console
hbase org.apache.hadoop.hbase.snapshot.ExportSnapshot -snapshot <snapshotName> -copy-to <hdfsHBaseLocation>
```

L’élément `<hdfsHBaseLocation>` peut être l’un des emplacements de stockage accessibles à votre cluster source, et doit pointer vers le dossier hbase utilisé par votre cluster de destination. Par exemple, si vous possédez un compte Stockage Azure secondaire joint à votre cluster source, et que ce compte procure un accès au conteneur utilisé par le stockage par défaut du cluster de destination, vous pouvez utiliser cette commande :

```console
hbase org.apache.hadoop.hbase.snapshot.ExportSnapshot -snapshot 'Snapshot1' -copy-to 'wasbs://secondcluster@myaccount.blob.core.windows.net/hbase'
```

Si vous n’avez pas de compte Stockage Azure secondaire attaché à votre cluster source ou si celui-ci est un cluster local (ou un cluster non HDI), il se peut que vous rencontriez des problèmes d’autorisation lorsque vous tentez d’accéder au compte de stockage de votre cluster HDI. Pour les résoudre, spécifiez la clé de votre compte de stockage en tant que paramètre de ligne de commande, comme indiqué dans l’exemple suivant. Vous pouvez récupérer la clé de votre compte de stockage dans le portail Azure.

```console
hbase org.apache.hadoop.hbase.snapshot.ExportSnapshot -Dfs.azure.account.key.myaccount.blob.core.windows.net=mykey -snapshot 'Snapshot1' -copy-to 'wasbs://secondcluster@myaccount.blob.core.windows.net/hbase'
```

Si votre cluster de destination est un cluster ADLS Gen2, modifiez la commande précédente pour tenir compte des configurations utilisées par ADLS Gen2 :

```console
hbase org.apache.hadoop.hbase.snapshot.ExportSnapshot -Dfs.azure.account.key.<account_name>.dfs.core.windows.net=<key> -Dfs.azure.account.auth.type.<account_name>.dfs.core.windows.net=SharedKey -Dfs.azure.always.use.https.<account_name>.dfs.core.windows.net=false -Dfs.azure.account.keyprovider.<account_name>.dfs.core.windows.net=org.apache.hadoop.fs.azurebfs.services.SimpleKeyProvider -snapshot 'Snapshot1' -copy-to 'abfs://<container>@<account_name>.dfs.core.windows.net/hbase'
```

Une fois l’instantané exporté, exécutez SSH dans le nœud principal du cluster de destination, puis restaurez la capture instantanée à l’aide de la commande `restore_snapshot`, tel que décrit précédemment.

Les instantanés offrent une sauvegarde complète de la table au moment de l’exécution de la commande `snapshot`. Les instantanés ne peuvent aucunement être exécutés de manière incrémentielle par fenêtre temporelle, ni spécifier des sous-ensembles de familles de colonnes à inclure.

## <a name="replication"></a>Réplication

La [réplication HBase](https://hbase.apache.org/book.html#_cluster_replication) transfère automatiquement les transactions d’un cluster source vers un cluster de destination, au moyen d’un mécanisme asynchrone présentant une charge minimale sur le cluster source. Dans HDInsight, vous pouvez configurer la réplication entre les clusters où :

* Les clusters source et de destination se trouvent dans le même réseau virtuel.
* Les clusters source et de destination se trouvent dans des réseaux virtuels différents connectés par une passerelle VPN, mais ils sont hébergés au même emplacement géographique.
* Les clusters source et de destination se trouvent dans des réseaux virtuels différents connectés par une passerelle VPN, et chacun d’entre eux est hébergé à un emplacement géographique propre.

Les étapes générales de configuration de la réplication sont les suivantes :

1. Sur le cluster source, créez les tables et renseignez les données.
2. Sur le cluster de destination, créez des tables de destination vides avec le schéma de la table source.
3. Inscrivez le cluster de destination en tant qu’homologue du cluster source.
4. Activez la réplication sur les tables sources souhaitées.
5. Copiez les données existantes des tables sources sur les tables de destination.
6. La réplication copie automatiquement les nouvelles modifications de données apportées sur les tables sources sur les tables de destination.

Pour activer la réplication sur HDInsight, appliquez une action de script sur votre cluster HDInsight source en cours d’exécution. Pour bénéficier d’une description de l’activation de la réplication dans votre cluster, ou pour tester la réplication sur des exemples de clusters créés dans des réseaux virtuels à l’aide de modèles de gestion des ressources Azure, consultez [Configurer la réplication Apache HBase](apache-hbase-replication.md). Cet article comporte également des instructions relatives à l’activation de la réplication de métadonnées Phoenix.

## <a name="next-steps"></a>Étapes suivantes

* [Configurer la réplication Apache HBase](apache-hbase-replication.md)
* [Utilisation de l’utilitaire d’importation et d’exportation HBase](/archive/blogs/data_otaku/working-with-the-hbase-import-and-export-utility)
