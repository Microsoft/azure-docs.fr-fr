---
title: 'Stockage : Effectuer la migration d’Apache Hadoop en local vers Azure HDInsight'
description: Découvrez les bonnes pratiques concernant le stockage pour la migration des clusters Hadoop locaux vers Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: ashishth
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/10/2019
ms.openlocfilehash: 0594774533f306421f6f3d1260d074bd92b9c919
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92544866"
---
# <a name="migrate-on-premises-apache-hadoop-clusters-to-azure-hdinsight"></a>Effectuer la migration de clusters Apache Hadoop locaux vers Azure HDInsight

Cet article fournit des recommandations pour le stockage de données dans les systèmes Azure HDInsight. Il fait partie d’une série qui propose des bonnes pratiques pour aider à la migration des systèmes Apache Hadoop locaux vers Azure HDInsight.

## <a name="choose-right-storage-system-for-hdinsight-clusters"></a>Choisir le système de stockage approprié pour les clusters HDInsight

La structure des répertoires du système de fichiers Apache Hadoop (HDFS) local peut être recréée dans le stockage Blob Azure ou dans Azure Data Lake Storage. Vous pouvez alors supprimer sans problème les clusters HDInsight utilisés pour le traitement, sans perte de données utilisateur. Les deux services peuvent être utilisés comme système de fichiers par défaut et comme système de fichiers supplémentaire pour un cluster HDInsight. Le cluster HDInsight et le compte de stockage doivent être hébergés dans la même région.

### <a name="azure-storage"></a>Stockage Azure

Les clusters HDInsight peuvent utiliser le conteneur d’objets blob dans Stockage Azure comme système de fichiers par défaut ou comme système de fichiers supplémentaire.  Le compte de stockage de niveau Standard est pris en charge pour une utilisation avec des clusters HDInsight. Le niveau Premier n’est pas pris en charge. Le conteneur d’objets blob par défaut stocke les informations spécifiques de cluster telles que l’historique et les journaux d’activité des travaux.  Le partage d’un conteneur blob en tant que système de fichiers par défaut sur plusieurs clusters n’est pas pris en charge.

Les comptes de stockage définis lors du processus de création et leurs clés respectives sont stockés dans `%HADOOP_HOME%/conf/core-site.xml` sur les nœuds du cluster. Ils sont également accessibles sous la section « Custom core site » dans la configuration HDFS, dans l’interface utilisateur d’Ambari. La clé du compte de stockage est chiffrée par défaut et un script de déchiffrement personnalisé est utilisé pour déchiffrer les clés avant de les passer aux démons Hadoop. Les tâches, notamment Hive, MapReduce, le streaming Hadoop et Pig, véhiculent avec elles une description des comptes de stockage et des métadonnées.

Vous pouvez géorépliquer le stockage Azure. Si la géoréplication permet la récupération à partir d’autres régions géographiques et la redondance des données, un basculement vers un emplacement géorépliqué affecte sérieusement les performances et peut entraîner des frais supplémentaires. Nous vous recommandons de choisir la géoréplication en connaissance de cause et seulement si la valeur des données justifie le coût supplémentaire.

Vous pouvez utiliser un des formats suivants pour accéder aux données stockées dans Stockage Azure :

|Format d’accès aux données |Description |
|---|---|
|`wasb:///`|Accédez au stockage par défaut à l’aide d’une communication non chiffrée.|
|`wasbs:///`|Accédez au stockage par défaut à l’aide d’une communication chiffrée.|
|`wasb://<container-name>@<account-name>.blob.core.windows.net/`|Utilisé pour communiquer avec un compte de stockage autre que celui par défaut. |

[Objectifs d’extensibilité pour les compte de stockage standard](../../storage/common/scalability-targets-standard-account.md) liste les limites actuelles des comptes de stockage Azure. Si les besoins de votre application dépassent cibles de scalabilité d’un seul compte de stockage, vous pouvez concevoir votre application pour qu’elle utilise plusieurs comptes de stockage, puis partitionner vos objets de données entre ces comptes.

[Azure Storage Analytics](../../storage/common/storage-analytics.md) fournit des métriques pour tous les services de stockage et le portail Azure peut être configuré pour la visualisation des métriques collectées via des graphiques. Vous pouvez créer des alertes pour avertir quand des seuils ont été atteints pour les métriques des ressources de stockage.

Stockage Azure offre une fonctionnalité de [suppression réversible pour les objets blob](../../storage/blobs/soft-delete-blob-overview.md), qui vous permet de récupérer plus facilement vos données en cas de modification ou de suppression accidentelles de celles-ci par une application ou par un autre utilisateur du compte de stockage.

Vous pouvez créer des [instantanés d’objets blob](/rest/api/storageservices/creating-a-snapshot-of-a-blob). Un instantané est une version en lecture seule d’un objet blob, qui est capturé à un instant donné, fournissant ainsi une façon de sauvegarder un objet blob. Un instantané peut être lu, copié ou supprimé, mais pas modifié.

> [!Note]
> Pour une version antérieure des distributions Hadoop locales qui n’a pas le certificat « wasbs », il doit être importé dans le magasin d’approbations Java.

Vous pouvez utiliser les méthodes suivantes pour importer des certificats dans le magasin d’approbations Java :

Télécharger le certificat SSL/TLS de l’objet blob Azure dans un fichier

```bash
echo -n | openssl s_client -connect <storage-account>.blob.core.windows.net:443 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > Azure_Storage.cer
```

Importer le fichier ci-dessus dans le magasin d’approbations Java sur tous les nœuds

```bash
keytool -import -trustcacerts -keystore /path/to/jre/lib/security/cacerts -storepass changeit -noprompt -alias blobtrust -file Azure_Storage.cer
```

Vérifier que le certificat ajouté figure dans le magasin d’approbations

```bash
keytool -list -v -keystore /path/to/jre/lib/security/cacerts
```

Pour plus d’informations, consultez les articles suivants :

- [Utiliser Stockage Azure avec des clusters Azure HDInsight](../hdinsight-hadoop-use-blob-storage.md)
- [Objectifs d'extensibilité pour les comptes de stockage standard](../../storage/common/scalability-targets-standard-account.md)
- [Objectifs de performance et d’extensibilité du Stockage Blob](../../storage/blobs/scalability-targets.md)
- [Liste de contrôle des performances et de l’évolutivité de Microsoft Azure Storage](../../storage/blobs/storage-performance-checklist.md)
- [Analyser, diagnostiquer et dépanner Microsoft Azure Storage](../../storage/common/storage-monitoring-diagnosing-troubleshooting.md)
- [Surveiller un compte de stockage dans le portail Azure](../../storage/common/storage-monitor-storage-account.md)

### <a name="azure-data-lake-storage-gen1"></a>Azure Data Lake Storage Gen1

Azure Data Lake Storage Gen1 implémente le modèle de contrôle d’accès de style HDFS et POSIX. Il fournit une intégration étroite à Azure AD pour un contrôle d’accès précis. Il n’existe aucune limite à la taille des données qu’il peut stocker ou à sa capacité à exécuter de l’analytique massivement parallèle.

Pour plus d’informations, consultez les articles suivants :

- [Créer des clusters HDInsight avec Data Lake Storage Gen1 à l’aide du portail Azure](../../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md)
- [Utiliser Data Lake Storage Gen1 avec des clusters Azure HDInsight](../hdinsight-hadoop-use-data-lake-storage-gen1.md)

### <a name="azure-data-lake-storage-gen2"></a>Azure Data Lake Storage Gen2

Azure Data Lake Storage Gen2 est la l’offre de stockage la plus récente. Elle unifie les capacités de base de la première génération d’Azure Data Lake Storage Gen1 avec un point de terminaison de système de fichiers compatible Hadoop, directement intégré dans le stockage Blob Azure. Cette amélioration combine les avantages de la mise à l’échelle et du coût de stockage des objets, avec la fiabilité et les performances généralement associées seulement à des systèmes de fichiers locaux.

Azure Data Lake Storage Gen 2 repose sur le [Stockage Blob Azure](../../storage/blobs/storage-blobs-introduction.md) et vous permet d’interagir avec les données selon les deux paradigmes du système de fichiers et du stockage d’objets. Les fonctionnalités [d’Azure Data Lake Storage Gen1](../../data-lake-store/index.yml), comme la sémantique des systèmes de fichiers, la sécurité au niveau du fichier et la mise à l’échelle, sont combinées à celles du [Stockage Blob Azure](../../storage/blobs/storage-blobs-introduction.md), comme le stockage hiérarchisé économique, la haute disponibilité/reprise après sinistre et le grand écosystème de SDK/d’outils. Dans Data Lake Storage Gen2, toutes les qualités du stockage d’objets sont conservées, avec en plus les avantages d’une interface de système de fichiers optimisée pour les charges de travail analytiques.

Une fonctionnalité fondamentale de Data Lake Storage Gen 2 est l’ajout d’un [espace de noms hiérarchique](../../storage/blobs/data-lake-storage-namespace.md) au service de stockage Blob qui organise les objets et les fichiers en une hiérarchie de répertoires pour optimiser l’accès aux données.  La structure hiérarchique fait que les opérations comme le renommage ou la suppression d’un répertoire sont des opérations atomiques uniques de métadonnées sur le répertoire, au lieu de l’énumération et du traitement de tous les objets qui partagent le préfixe du nom de répertoire.

Dans le passé, l’analytique cloud devait trouver le meilleur compromis entre les performances, la gestion et la sécurité. Les principales fonctionnalités d’Azure Data Lake Storage Gen2 sont les suivantes :

- **Accès compatible Hadoop**  : Azure Data Lake Storage Gen2 vous permet de gérer les données et d'y accéder comme vous le feriez avec un [système de fichiers HDFS (Hadoop Distributed File System)](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html). Le nouveau [pilote ABFS](../../storage/blobs/data-lake-storage-abfs-driver.md) est disponible dans tous les environnements Apache Hadoop inclus dans [Azure HDInsight](../index.yml). Ce pilote vous permet d’accéder aux données stockées dans Data Lake Storage Gen2.

- **Surensemble d’autorisations POSIX**  : Le modèle de sécurité pour Data Lake Gen2 prend entièrement en charge les autorisations ACL et POSIX ainsi que certaines granularités supplémentaires spécifiques de Data Lake Storage Gen2. Les paramètres peuvent être configurés via les outils d’administration ou des infrastructures telles que Hive et Spark.

- **Rentabilité**  : Data Lake Storage Gen2 intègre une capacité de stockage et des transactions économiques. Tout au long du cycle de vie des données, les frais facturés changent de façon à minimiser les coûts via des fonctionnalités intégrées, comme le [cycle de vie du Stockage Blob Azure](../../storage/blobs/storage-lifecycle-management-concepts.md).

- **Fonctionne avec les outils, les frameworks et les applications de stockage Blob**  : Data Lake Storage Gen2 continue de fonctionner avec une large gamme d’outils, de frameworks et d’applications qui existent aujourd’hui pour Stockage Blob.

- **Pilote optimisé**  : Le pilote ABFS (Azure Blob Filesystem) est [optimisé spécifiquement](../../storage/blobs/data-lake-storage-abfs-driver.md) pour l’analytique du Big Data. Les API REST correspondantes sont exposées via le point de terminaison dfs, dfs.core.windows.net.

Vous pouvez utiliser un des formats suivants pour accéder aux données stockées dans Azure Data Lake Storage Gen2 :
- `abfs:///`: Accédez au stockage Data Lake par défaut pour le cluster.
- `abfs://file_system@account_name.dfs.core.windows.net`: Utilisé pour communiquer avec un stockage Data Lake autre que celui par défaut.

Pour plus d’informations, consultez les articles suivants :

- [Présentation d’Azure Data Lake Storage Gen2](../../storage/blobs/data-lake-storage-introduction.md)
- [Pilote Azure Blob FileSystem (ABFS.md)](../../storage/blobs/data-lake-storage-abfs-driver.md)
- [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](../hdinsight-hadoop-use-data-lake-storage-gen2.md)

## <a name="secure-azure-storage-keys-within-on-premises-hadoop-cluster-configuration"></a>Sécuriser les clés de Stockage Azure au sein de la configuration de cluster Hadoop local

Les clés de Stockage Azure qui sont ajoutées aux fichiers de configuration Hadoop établissent la connectivité entre le stockage HDFS local et Stockage Blob Azure. Vous pouvez protéger ces clés en les chiffrant avec l’infrastructure du fournisseur d’informations d’identification Hadoop. Une fois qu’elles sont chiffrées, vous pouvez les stocker et y accéder de façon sécurisée.

**Pour provisionner les informations d’identification :**

```bash
hadoop credential create fs.azure.account.key.account.blob.core.windows.net -value <storage key> -provider jceks://hdfs@headnode.xx.internal.cloudapp.net/path/to/jceks/file
```

**Pour ajouter le chemin du fournisseur ci-dessus au fichier core-site.xml ou à la configuration Ambari sous Custom core-site :**

```xml
<property>
    <name>hadoop.security.credential.provider.path</name>
        <value>
        jceks://hdfs@headnode.xx.internal.cloudapp.net/path/to/jceks
        </value>
    <description>Path to interrogate for protected credentials.</description>
</property>
```

> [!Note]
> La propriété de chemin du fournisseur peut également être ajoutée à la ligne de commande distcp, au lieu de stocker la clé au niveau du cluster dans le fichier core-site.xml, comme suit :

```bash
hadoop distcp -D hadoop.security.credential.provider.path=jceks://hdfs@headnode.xx.internal.cloudapp.net/path/to/jceks /user/user1/ wasb:<//yourcontainer@youraccount.blob.core.windows.net/>user1
```

## <a name="restrict-azure-storage-data-access-using-sas"></a>Restreindre l’accès aux données de Stockage Azure à l’aide de SAP

Par défaut, HDInsight dispose d’un accès total aux données dans les comptes Stockage Azure associés au cluster. Vous pouvez utiliser des signatures d’accès partagé sur le conteneur d’objets blob pour restreindre l’accès aux données, comme fournir aux utilisateurs un accès en lecture seule aux données.

### <a name="using-the-sas-token-created-with-python"></a>Utilisation du jeton de signature d’accès partagé créé avec python

1. Ouvrez le fichier [SASToken.py](https://github.com/Azure-Samples/hdinsight-dotnet-python-azure-storage-shared-access-signature/blob/master/Python/SASToken.py) et modifiez les valeurs suivantes :

    |Propriété de jeton|Description|
    |---|---|
    |policy_name|nom à utiliser pour la stratégie stockée à créer.|
    |storage_account_name|nom de votre compte de stockage.|
    |storage_account_key|Clé du compte de stockage.|
    |storage_container_name|conteneur du compte de stockage auquel vous souhaitez restreindre l’accès.|
    |example_file_path|chemin d’un fichier qui est chargé dans le conteneur.|

2. Le fichier SASToken.py est fourni avec les autorisations `ContainerPermissions.READ + ContainerPermissions.LIST` et peut être ajusté en fonction du cas d’usage.

3. Exécutez le script comme suit : `python SASToken.py`

4. Quand il se termine, le script affiche le jeton de signature d’accès partagé, qui est similaire au texte suivant : `sr=c&si=policyname&sig=dOAi8CXuz5Fm15EjRUu5dHlOzYNtcK3Afp1xqxniEps%3D&sv=2014-02-14`

5. Pour limiter l’accès à un conteneur avec une signature d’accès partagé, ajoutez une entrée personnalisée à la configuration de core-site pour le cluster sous Ambari > HDFS > Configs > Advanced > Custom core-site > Add property.

6. Utilisez les valeurs suivantes pour les champs **Clé** et **Valeur** :

    **Clé**  : `fs.azure.sas.YOURCONTAINER.YOURACCOUNT.blob.core.windows.net` **Valeur**  : Clé SAP retournée l’application Python de l’étape 4 ci-dessus.

7. Cliquez sur le bouton **Ajouter** pour enregistrer cette clé et cette valeur, puis cliquez sur le bouton **Enregistrer** pour enregistrer les modifications de configuration. Lorsque vous y êtes invité, ajoutez une description de la modification (« Ajout d’accès de stockage SAP », par exemple), puis cliquez sur **Enregistrer** .

8. Dans l’interface utilisateur web Ambari, sélectionnez HDFS dans la liste sur la gauche, puis sélectionnez **Restart All Affected** (Redémarrer tous les éléments affectés) dans la liste déroulante Actions de service située à droite. Lorsque vous y êtes invité, sélectionnez **Confirm Restart All** (Confirmer le redémarrage).

9. Répétez ce processus pour les entrées MapReduce2 et YARN.

Trois points sont importants à retenir pour l’utilisation de jetons de signature d’accès partagé dans Azure :

1. Quand des jetons de signature d’accès partagé sont créés avec les autorisations « READ + LIST » (LECTURE + LISTE), les utilisateurs qui accèdent au conteneur d’objets blob avec ce jeton de signature d’accès partagé ne peuvent pas « écrire et supprimer » des données. Les utilisateurs qui accèdent au conteneur d’objets blob avec ce jeton de signature d’accès partagé, et qui essayent d’effectuer une opération d’écriture ou de suppression, reçoivent un message indiquant `"This request is not authorized to perform this operation"`.

2. Quand les jetons de signature d’accès partagé sont générés avec des autorisations `READ + LIST + WRITE` (pour interdire seulement `DELETE`), des commandes comme `hadoop fs -put` écrivent d’abord dans un fichier `\_COPYING\_`, puis essayent de renommer le fichier. Cette opération HDFS est mappée à une opération `copy+delete` pour WASB. Dans la mesure où l’autorisation `DELETE` n’a pas été accordée, le « put » échoue. L’opération `\_COPYING\_` est une fonctionnalité de Hadoop destinée à fournir un contrôle des accès concurrentiels. Il n’existe actuellement aucun moyen d’interdire simplement l’opération « DELETE » (SUPPRESSION) sans affecter également les opérations « WRITE » (ÉCRITURE).

3. Malheureusement, le fournisseur d’informations d’identification Hadoop et le fournisseur de clé de déchiffrement (ShellDecryptionKeyProvider) ne fonctionnent actuellement pas avec les jetons de signature d’accès partagé, et une protection contre la visibilité est donc actuellement impossible.

Pour plus d’informations, consultez [Utiliser des signatures d’accès partagé de Stockage Azure pour restreindre l’accès aux données dans HDInsight](../hdinsight-storage-sharedaccesssignature-permissions.md).

## <a name="use-data-encryption-and-replication"></a>Utiliser le chiffrement et la réplication des données

Toutes les données écrites dans le stockage Azure sont automatiquement chiffrées à l’aide du [Storage Service Encryption (SSE)](../../storage/common/storage-service-encryption.md). Les données placées dans le compte de Stockage Azure sont toujours répliquées de façon à garantir une haute disponibilité.  Quand vous créez un compte de stockage, vous pouvez choisir une des options de réplication suivantes :

- [Stockage localement redondant (LRS)](../../storage/common/storage-redundancy.md#locally-redundant-storage)
- [Stockage redondant interzone (ZRS)](../../storage/common/storage-redundancy.md#zone-redundant-storage)
- [Stockage géo-redondant (GRS)](../../storage/common/storage-redundancy.md#geo-redundant-storage)
- [Stockage géo-redondant avec accès en lecture (RA-GRS)](../../storage/common/storage-redundancy.md)

Le stockage Azure fournit un stockage localement redondant (LRS), mais il est également recommandé de copier les données critiques vers un autre compte de stockage Azure d’une autre région, selon une périodicité adaptée aux besoins du plan de reprise d’activité. Il existe différents moyens de copier des données, notamment [ADLCopy](../../data-lake-store/data-lake-store-copy-data-azure-storage-blob.md), [DistCp](https://hadoop.apache.org/docs/current/hadoop-distcp/DistCp.html), [Azure PowerShell](../../data-lake-store/data-lake-store-get-started-powershell.md) ou [Azure Data Factory](../../data-factory/connector-azure-data-lake-store.md).  Il est également recommandé d’appliquer des stratégies d’accès pour le compte de stockage Azure, afin d’éviter toute suppression accidentelle.

Pour plus d’informations, consultez les articles suivants :

- [Réplication Azure Storage](../../storage/common/storage-redundancy.md)
- [Recommandations en matière de reprise d’activité pour Azure Data Lake Storage Gen1](../../data-lake-store/data-lake-store-disaster-recovery-guidance.md)

## <a name="attach-additional-azure-storage-accounts-to-cluster"></a>Attacher des comptes de Stockage Azure supplémentaires au cluster

Lors du processus de création de HDInsight, un compte de stockage Azure, Azure Data Lake Storage Gen1 ou Azure Data Lake Storage Gen2 est choisi comme système de fichiers par défaut. Lors du processus de création ou à l’issue de la création d’un cluster, à ce compte de stockage par défaut, vous pouvez ajouter d’autres comptes provenant du même abonnement Azure ou d’autres abonnements Azure.

Vous pouvez ajouter un compte de stockage supplémentaire de l’une des façons suivantes :
- Ambari > HDFS > Configs > Advanced > Custom core-site > Add the storage > Account Name and key > Restarting the services
- Avec une [action de script](../hdinsight-hadoop-add-storage.md) en passant le nom et la clé du compte de stockage

> [!Note]
> Dans les cas d’utilisation valides, les limites sur le stockage Azure peuvent être augmentées via une demande adressée au [support technique d’Azure](https://azure.microsoft.com/support/faq/).

Pour plus d’informations, consultez [Ajouter des comptes de stockage supplémentaires à HDInsight](../hdinsight-hadoop-add-storage.md).

## <a name="next-steps"></a>Étapes suivantes

Lisez l’article suivant de cette série : [Bonnes pratiques concernant la migration de données d’une infrastructure locale vers Azure HDInsight Hadoop](apache-hadoop-on-premises-migration-best-practices-data-migration.md).
