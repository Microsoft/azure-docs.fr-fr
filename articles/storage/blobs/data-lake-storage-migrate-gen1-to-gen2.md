---
title: Migrer Azure Data Lake Storage de Gen1 vers Gen2
description: Migrer Azure Data Lake Storage de Gen1 vers Gen2, qui repose sur Stockage Blob Azure et fournit un ensemble de capacités dédiées à l’analytique du Big Data.
author: normesta
ms.topic: how-to
ms.author: normesta
ms.date: 03/11/2020
ms.service: storage
ms.reviewer: rukmani-msft
ms.subservice: data-lake-storage-gen2
ms.openlocfilehash: aeb2c58504d1f058a3b887e02a7b7406c09db5b6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95913145"
---
# <a name="migrate-azure-data-lake-storage-from-gen1-to-gen2"></a>Migrer Azure Data Lake Storage de Gen1 vers Gen2

Vous pouvez effectuer la migration de vos données, de vos charges de travail et de vos applications, de Data Lake Storage Gen1 vers Data Lake Storage Gen2.

Azure Data Lake Storage Gen2 est basé sur le [Stockage Blob Azure](storage-blobs-introduction.md). Il fournit un ensemble de fonctionnalités dédiées à l’analytique du Big Data. [Data Lake Storage Gen2](https://azure.microsoft.com/services/storage/data-lake-storage/) contient les fonctionnalités d’[Azure Data Lake Storage Gen1](../../data-lake-store/index.yml), comme la sémantique des systèmes de fichiers, la sécurité au niveau des fichiers et des répertoires, et la mise à l’échelle, ainsi que les fonctionnalités du [Stockage Blob Azure](storage-blobs-introduction.md) comme le stockage hiérarchisé à faible coût, la haute disponibilité et la reprise d’activité.

> [!NOTE]
> Pour faciliter la lecture de cet article, le terme *Gen1* est utilisé pour faire référence à Azure Data Lake Storage Gen1 tandis que le terme *Gen2* est utilisé pour faire référence à Azure Data Lake Storage Gen2.

## <a name="recommended-approach"></a>Approche recommandée

Pour effectuer la migration vers Gen2, nous vous recommandons l’approche suivante.

:heavy_check_mark: Étape 1 : Évaluer la préparation

:heavy_check_mark: Étape 2 : Préparer la migration

:heavy_check_mark: Étape 3 : Effectuer la migration des données, des charges de travail et des applications

:heavy_check_mark: Étape 4 : Basculer de Gen1 vers Gen2

> [!NOTE]
> Gen1 et Gen2 sont des services différents. Aucune mise à niveau sur place ni aucun effort de migration intentionnel ne sont nécessaires. 

### <a name="step-1-assess-readiness"></a>Étape 1 : Évaluer la préparation

1. Découvrez l’[offre Data Lake Storage Gen2](https://azure.microsoft.com/services/storage/data-lake-storage/), ses avantages, son coût et son architecture générale. 

2. [Comparez les fonctionnalités](#gen1-gen2-feature-comparison) de Gen1 à celles de Gen2. 

3. Consultez la liste des [problèmes connus](data-lake-storage-known-issues.md) pour voir si certaines fonctionnalités sont manquantes.

4. Gen2 prend en charge les fonctionnalités du stockage Blob, telles que la [journalisation des diagnostics](../common/storage-analytics-logging.md), les [niveaux d’accès](storage-blob-storage-tiers.md) et les [stratégies de gestion du cycle de vie du stockage Blob](storage-lifecycle-management-concepts.md). Si ces fonctionnalités vous intéressent, consultez leur [niveau actuel de prise en charge](./data-lake-storage-supported-blob-storage-features.md).

5. Consultez l’état actuel de la [prise en charge de l’écosystème Azure](./data-lake-storage-multi-protocol-access.md) pour voir si Gen2 prend en charge tous les services dont dépendent vos solutions.

### <a name="step-2-prepare-to-migrate"></a>Étape 2 : Préparer la migration

1. Identifiez les jeux de données qui doivent faire l’objet d’une migration.

   Profitez-en pour nettoyer les jeux de données que vous n’utilisez plus. Si vous ne souhaitez pas effectuer la migration de toutes vos données à la fois, profitez-en pour identifier les groupes logiques de données qui peuvent faire l’objet d’une migration en plusieurs phases.
   
2. Déterminez l’impact qu’aura une migration sur votre entreprise.

   Par exemple, déterminez si vous pouvez vous permettre des temps d’arrêt pendant la migration. Ces considérations peuvent vous aider à identifier un modèle de migration approprié et à choisir les outils les mieux adaptés.

3. Créez un plan de migration. 

   Nous vous recommandons d’utiliser [ces modèles de migration](#migration-patterns). Vous pouvez choisir l’un de ces modèles, les associer ou concevoir votre propre modèle personnalisé.

### <a name="step-3-migrate-data-workloads-and-applications"></a>Étape 3 : Migrer les données, les charges de travail et les applications

Effectuez la migration des données, des charges de travail et des applications à l’aide du modèle de votre choix. Nous vous recommandons de valider les scénarios de manière incrémentielle.

1. [Créez un compte de stockage](../common/storage-account-create.md) et activez la fonctionnalité d’espace de noms hiérarchique. 

2. Migrez vos données. 

3. Configurez les [services de vos charges de travail](./data-lake-storage-supported-azure-services.md) pour qu’ils pointent vers votre point de terminaison Gen2. 
   
4. Mettez à jour les applications pour qu’elles utilisent les API Gen2. Consultez les guides concernant [.NET](data-lake-storage-directory-file-acl-dotnet.md), [Java](data-lake-storage-directory-file-acl-java.md), [Python](data-lake-storage-directory-file-acl-python.md), [JavaScript](data-lake-storage-directory-file-acl-javascript.md) et [REST](/rest/api/storageservices/data-lake-storage-gen2). 
   
5. Mettez à jour les scripts pour qu’ils utilisent les [applets de commande PowerShell](data-lake-storage-directory-file-acl-powershell.md) Data Lake Storage Gen2 et les [commandes Azure CLI](data-lake-storage-directory-file-acl-cli.md).
   
6. Recherchez les références URI contenant la chaîne `adl://` dans les fichiers de code, les notebooks Databricks, les fichiers HQL Apache Hive ou tout autre fichier utilisé dans le cadre de vos charges de travail. Remplacez ces références par l’[URI formaté Gen2](data-lake-storage-introduction-abfs-uri.md) de votre nouveau compte de stockage. Par exemple, l’URI Gen1 `adl://mydatalakestore.azuredatalakestore.net/mydirectory/myfile` peut devenir `abfss://myfilesystem@mydatalakestore.dfs.core.windows.net/mydirectory/myfile`. 

7. Configurez la sécurité sur votre compte pour inclure les [rôles RBAC Azure](../common/storage-auth-aad-rbac-portal.md), la [sécurité au niveau des fichiers et des dossiers](data-lake-storage-access-control.md), ainsi que les [pare-feu et réseaux virtuels dans Stockage Azure](../common/storage-network-security.md).

### <a name="step-4-cutover-from-gen1-to-gen2"></a>Étape 4 : Basculer de Gen1 vers Gen2

Une fois que vous êtes certain que vos applications et vos charges de travail sont stables sur Gen2, vous pouvez commencer à utiliser Gen2 pour vos scénarios métier. Désactivez les pipelines restants qui s’exécutent sur Gen1, puis désactivez votre compte Gen1. 

<a id="gen1-gen2-feature-comparison"></a>

## <a name="gen1-vs-gen2-capabilities"></a>Comparaison entre les fonctionnalités de Gen1 et de Gen2

Ce tableau compare les fonctionnalités de Gen1 à celles de Gen2.

|Domaine |Première génération   |Deuxième génération |
|---|---|---|
|Organisation des données|[Espace de noms hiérarchique](data-lake-storage-namespace.md)<br>Prise en charge des fichiers et des dossiers|[Espace de noms hiérarchique](data-lake-storage-namespace.md)<br>Prise en charge des conteneurs, des fichiers et des dossiers |
|Géo-redondance| [LRS](../common/storage-redundancy.md#locally-redundant-storage)| [LRS](../common/storage-redundancy.md#locally-redundant-storage), [ZRS](../common/storage-redundancy.md#zone-redundant-storage), [GRS](../common/storage-redundancy.md#geo-redundant-storage), [RA-GRS](../common/storage-redundancy.md#read-access-to-data-in-the-secondary-region) |
|Authentification|[Identité managée AAD](../../active-directory/managed-identities-azure-resources/overview.md)<br>[Principaux de service](../../active-directory/develop/app-objects-and-service-principals.md)|[Identité managée AAD](../../active-directory/managed-identities-azure-resources/overview.md)<br>[Principaux de service](../../active-directory/develop/app-objects-and-service-principals.md)<br>[Clé d’accès partagé](/rest/api/storageservices/authorize-with-shared-key)|
|Autorisation|Gestion : [Azure RBAC](../../role-based-access-control/overview.md)<br>Données - [Listes ACL](data-lake-storage-access-control.md)|Gestion : [Azure RBAC](../../role-based-access-control/overview.md)<br>Données : [Listes ACL](data-lake-storage-access-control.md), [Azure RBAC](../../role-based-access-control/overview.md) |
|Chiffrement - Données au repos|Côté serveur - Avec des clés [managées par Microsoft](../common/storage-service-encryption.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) ou [gérées par le client](../common/customer-managed-keys-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|Côté serveur - Avec des clés [managées par Microsoft](../common/storage-service-encryption.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) ou [gérées par le client](../common/customer-managed-keys-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)|
|Prise en charge du réseau virtuel|[Intégration au réseau virtuel](../../data-lake-store/data-lake-store-network-security.md)|[Points de terminaison de service](../common/storage-network-security.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json), [points de terminaison privés](../common/storage-private-endpoints.md)|
|Expérience développeur|[REST](../../data-lake-store/data-lake-store-data-operations-rest-api.md), [.NET](../../data-lake-store/data-lake-store-data-operations-net-sdk.md), [Java](../../data-lake-store/data-lake-store-get-started-java-sdk.md), [Python](../../data-lake-store/data-lake-store-data-operations-python.md), [PowerShell](../../data-lake-store/data-lake-store-get-started-powershell.md), [Azure CLI](../../data-lake-store/data-lake-store-get-started-cli-2.0.md)|Disponibilité générale - [REST](/rest/api/storageservices/data-lake-storage-gen2), [.NET](data-lake-storage-directory-file-acl-dotnet.md), [Java](data-lake-storage-directory-file-acl-java.md), [Python](data-lake-storage-directory-file-acl-python.md)<br>Préversion publique - [JavaScript](data-lake-storage-directory-file-acl-javascript.md), [PowerShell](data-lake-storage-directory-file-acl-powershell.md), [Azure CLI](data-lake-storage-directory-file-acl-cli.md)|
|Journaux d’activité de ressources|Journaux classiques<br>[Intégré à Azure Monitor](../../data-lake-store/data-lake-store-diagnostic-logs.md)|[Journaux classiques](../common/storage-analytics-logging.md) - Disponibilité générale<br>Intégration à Azure Monitor - Date à confirmer|
|Écosystème|[HDInsight (3.6)](../../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md), [Azure Databricks (3.1 et versions ultérieures)](https://docs.databricks.com/data/data-sources/azure/azure-datalake.html), [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-load-from-azure-data-lake-store.md), [ADF](../../data-factory/load-azure-data-lake-store.md)|[HDInsight (3.6, 4.0)](../../hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2.md), [Azure Databricks (5.1 et versions ultérieures)](/azure/databricks/data/data-sources/azure/azure-datalake-gen2), [Azure Synapse Analytics](../../azure-sql/database/vnet-service-endpoint-rule-overview.md), [ADF](../../data-factory/load-azure-data-lake-storage-gen2.md)|

<a id="migration-patterns"></a>

## <a name="gen1-to-gen2-patterns"></a>Modèles Gen1 à Gen2

Choisissez un modèle de migration, puis modifiez le modèle en fonction des besoins.

|||
|---|---|
|**Opération lift-and-shift**|Modèle le plus simple. Idéal si vos pipelines de données permettent des temps d’arrêt.|
|**Copie incrémentielle**|Semblable au *lift-and-shift*, mais avec un temps d’arrêt moindre. Idéal pour les grandes quantités de données dont la copie prend plus de temps.|
|**Double pipeline**|Idéal pour les pipelines qui ne peuvent pas se permettre de temps d’arrêt.|
|**Synchronisation bidirectionnelle**|Semblable au *double pipeline*, mais avec une approche plus progressive, adaptée aux pipelines complexes.|

Examinons de plus près chacun de ces modèles.
 
### <a name="lift-and-shift-pattern"></a>Modèle lift-and-shift

Il s’agit du modèle le plus simple. 

1. Arrêtez toutes les écritures dans Gen1.

2. Déplacez les données de Gen1 vers Gen2. Nous vous recommandons [Azure Data Factory](../../data-factory/connector-azure-data-lake-storage.md). Les listes de contrôle d’accès sont copiées avec les données.

3. Faites pointer les opérations d’ingestion et les charges de travail vers Gen2.

4. Désactivez Gen1.

Consultez notre exemple de code pour le modèle lift-and-shift dans notre [exemple de migration lift-and-shift](https://github.com/rukmani-msft/adlsgen1togen2migrationsamples/blob/master/src/Lift%20and%20Shift/README.md).

> [!div class="mx-imgBorder"]
> ![Modèle lift-and-shift](./media/data-lake-storage-migrate-gen1-to-gen2/lift-and-shift.png)

#### <a name="considerations-for-using-the-lift-and-shift-pattern"></a>Considérations relatives à l’utilisation du modèle lift-and-shift

:heavy_check_mark: Effectuez le basculement de Gen1 vers Gen2 pour toutes les charges de travail en même temps.

:heavy_check_mark: Prévoyez un temps d’arrêt pendant la migration et la période de basculement.

:heavy_check_mark: Idéal pour les pipelines qui permettent des temps d’arrêt et dans lesquels toutes les applications peuvent être mises à niveau en même temps.

### <a name="incremental-copy-pattern"></a>Modèle de copie incrémentielle

1. Commencez par déplacer les données de Gen1 vers Gen2. Nous vous recommandons [Azure Data Factory](../../data-factory/connector-azure-data-lake-storage.md). Les listes de contrôle d’accès sont copiées avec les données.

2. Copiez les nouvelles données de façon incrémentielle à partir de Gen1.

3. Une fois que toutes les données ont été copiées, arrêtez toutes les écritures dans Gen1 et faites pointer les charges de travail vers Gen2.

4. Désactivez Gen1.

Consultez notre exemple de code pour le modèle de copie incrémentielle dans notre [exemple de migration de copie incrémentielle](https://github.com/rukmani-msft/adlsgen1togen2migrationsamples/blob/master/src/Incremental/README.md).


> [!div class="mx-imgBorder"]
> ![Modèle de copie incrémentielle](./media/data-lake-storage-migrate-gen1-to-gen2/incremental-copy.png)

#### <a name="considerations-for-using-the-incremental-copy-pattern"></a>Considérations relatives à l’utilisation du modèle de copie incrémentielle :

:heavy_check_mark: Effectuez le basculement de Gen1 vers Gen2 pour toutes les charges de travail en même temps.

:heavy_check_mark: Prévoyez un temps d’arrêt uniquement pendant la période de basculement.

:heavy_check_mark: Idéal pour les pipelines dans lesquels toutes les applications sont mises à niveau en même temps, mais dans lesquels la copie des données nécessite plus de temps.

### <a name="dual-pipeline-pattern"></a>Modèle de pipeline double

1. Déplacez les données de Gen1 vers Gen2. Nous vous recommandons [Azure Data Factory](../../data-factory/connector-azure-data-lake-storage.md). Les listes de contrôle d’accès sont copiées avec les données.

2. Ingérez les nouvelles données à la fois dans Gen1 et dans Gen2.

3. Faites pointer les charges de travail vers Gen2.

4. Arrêtez toutes les écritures dans Gen1, puis désactivez Gen1.

Consultez notre exemple de code pour le modèle de pipeline double dans notre [exemple de migration de pipeline double](https://github.com/rukmani-msft/adlsgen1togen2migrationsamples/blob/master/src/Dual%20pipeline/README.md).

> [!div class="mx-imgBorder"]
> ![Modèle de pipeline double](./media/data-lake-storage-migrate-gen1-to-gen2/dual-pipeline.png)

#### <a name="considerations-for-using-the-dual-pipeline-pattern"></a>Considérations relatives à l’utilisation du modèle de pipeline double :

:heavy_check_mark: Les pipelines Gen1 et Gen2 s’exécutent côte à côte.

:heavy_check_mark: Ne permet aucun temps d’arrêt.

:heavy_check_mark: Idéal lorsque vos charges de travail et vos applications ne peuvent pas se permettre de temps d’arrêt, et lorsque vous pouvez ingérer les données dans les deux comptes de stockage.

### <a name="bi-directional-sync-pattern"></a>Modèle de synchronisation bidirectionnelle

1. Configurez la réplication bidirectionnelle entre Gen1 et Gen2. Nous recommandons [WanDisco](https://docs.wandisco.com/bigdata/wdfusion/adls/). Il offre une fonctionnalité de réparation pour les données existantes.

3. Lorsque tous les déplacements de données ont été effectués, arrêtez toutes les écritures dans Gen1, puis désactivez la réplication bidirectionnelle.

4. Désactivez Gen1.

Consultez notre exemple de code pour le modèle de synchronisation bidirectionnelle dans notre [exemple de migration de synchronisation bidirectionnelle](https://github.com/rukmani-msft/adlsgen1togen2migrationsamples/blob/master/src/Bi-directional/README.md).

> [!div class="mx-imgBorder"]
> ![Modèle de synchronisation bidirectionnelle](./media/data-lake-storage-migrate-gen1-to-gen2/bidirectional-sync.png)

#### <a name="considerations-for-using-the-bi-directional-sync-pattern"></a>Considérations relatives à l’utilisation du modèle de synchronisation bidirectionnelle :

:heavy_check_mark: Idéal pour les scénarios complexes qui impliquent un grand nombre de pipelines et de dépendances, et pour lesquels une approche progressive est sans doute plus adaptée.  

:heavy_check_mark: L’effort de migration est important, mais il offre une prise en charge côte à côte de Gen1 et de Gen2.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez les différentes étapes de configuration de la sécurité pour un compte de stockage. Voir [Guide de sécurité du Stockage Azure](./security-recommendations.md).
- Optimisez les performances de votre instance de Data Lake Store. Voir [Optimiser Azure Data Lake Storage Gen2 pour les performances](data-lake-storage-performance-tuning-guidance.md).
- Découvrez les bonnes pratiques de gestion pour Data Lake Store. Voir [Bonnes pratiques concernant l’utilisation d’Azure Data Lake Storage Gen2](data-lake-storage-best-practices.md)