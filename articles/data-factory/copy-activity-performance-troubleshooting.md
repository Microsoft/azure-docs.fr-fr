---
title: Résoudre les problèmes de performances de l’activité de copie
description: Découvrez comment résoudre les problèmes de performances de l’activité de copie dans Azure Data Factory.
services: data-factory
documentationcenter: ''
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 10/12/2020
ms.openlocfilehash: 89f7a4a23f4d1b62fe5a76fbd4625bae8bb3018f
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92634758"
---
# <a name="troubleshoot-copy-activity-performance"></a>Résoudre les problèmes de performances de l’activité de copie

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article décrit comment résoudre les problèmes de performances de l’activité de copie dans Azure Data Factory. 

Après avoir exécuté une activité de copie, vous pouvez collecter les résultats de l’exécution et les statistiques de performances dans l’affichage de [l’analyse de l’activité de copie](copy-activity-monitoring.md). Voici un exemple.

![Détails de l’analyse d’une exécution d’activité de copie](./media/copy-activity-overview/monitor-copy-activity-run-details.png)

## <a name="performance-tuning-tips"></a>Conseils sur le réglage des performances

Dans certains scénarios, lorsque vous exécutez une activité de copie dans Data Factory, vous voyez des **« Conseils sur le réglage des performances »** en haut, comme indiqué dans l’exemple ci-dessus. Les conseils vous indiquent le goulot d’étranglement identifié par ADF pour cette exécution de copie spécifique, ainsi que des suggestions sur la manière d’améliorer le débit de copie. Essayez d’effectuer la modification recommandée, puis réexécutez la copie.

À titre de référence, les conseils de réglage des performances fournissent actuellement des suggestions pour les cas suivants :

| Category              | Conseils sur le réglage des performances                                      |
| --------------------- | ------------------------------------------------------------ |
| Spécifique au magasin de données   | Chargement de données dans **Azure Synpase Analytics (anciennement SQL DW)**  : suggérez l’utilisation de PolyBase ou de l’instruction COPY si elle n’est pas utilisée. |
| &nbsp;                | Copie de données depuis/vers **Azure SQL Database**  : lorsque la DTU est très utilisée, suggérez une mise à niveau vers un niveau supérieur. |
| &nbsp;                | Copie de données depuis/vers **Azure Cosmos DB**  : lorsque la RU est très utilisée, suggérez une mise à niveau vers une RU plus importante. |
|                       | Copie de données à partir d’une **table SAP**  : quand de grandes quantités de données sont copiées, nous suggérons de tirer parti de l’option de partition du connecteur SAP pour activer la charge parallèle et augmenter le nombre maximal de partitions. |
| &nbsp;                | Ingestion de données à partir d’ **Amazon Redshift**  : suggérez l’utilisation de UNLOAD si elle n’est pas utilisée. |
| Limitation du magasin de données | Si plusieurs opérations de lecture/écriture sont limitées par le magasin de données au cours de la copie, suggérez de vérifier et d’augmenter le taux de requêtes autorisées pour le magasin de données ou de réduire la charge de travail simultanée. |
| Runtime d’intégration  | Si vous utilisez un **runtime d’intégration (IR) auto-hébergé** et que l’activité de copie attend longtemps dans la file d’attente avant que le runtime d’intégration ne dispose des ressources nécessaires à son exécution, suggérez une montée en charge ou en puissance de votre IR. |
| &nbsp;                | Si vous utilisez un **Azure Integration Runtime** qui se trouve dans une région non optimale, ce qui entraîne une lecture/écriture lente, suggérez de configurer l’utilisation d’un IR dans une autre région. |
| Tolérance de panne       | Si vous configurez la tolérance de panne et que le fait d’ignorer des lignes incompatibles entraîne un ralentissement des performances, suggérez de vous assurer que les données de la source et du récepteur sont compatibles. |
| copie intermédiaire           | Si la copie intermédiaire est configurée, mais n’est pas utile pour votre paire source-récepteur, suggérez de la supprimer. |
| Reprendre                | Lorsque l’activité de copie est reprise à partir du dernier point d’échec, mais que vous modifiez le paramètre DIU après l’exécution d’origine, notez que le nouveau paramètre DIU n’est pas appliqué. |

## <a name="understand-copy-activity-execution-details"></a>Comprendre les détails de l’exécution de l’activité de copie

Les détails et les durées d’exécution en bas de l’affichage de l’analyse de l’activité de copie décrivent les principales étapes de l’activité de copie (voir l’exemple au début de cet article), ce qui est particulièrement utile pour la résolution des problèmes de performances de copie. Le goulet d’étranglement de votre exécution de copie est celui dont la durée est la plus longue. Reportez-vous au tableau suivant sur la définition de chaque étape et découvrez comment [résoudre les problèmes liés à l’activité de copie sur Azure IR](#troubleshoot-copy-activity-on-azure-ir) et [résoudre les problèmes liés à l’activité de copie sur un IR auto-hébergé](#troubleshoot-copy-activity-on-self-hosted-ir) avec ces informations.

| Étape           | Description                                                  |
| --------------- | ------------------------------------------------------------ |
| File d'attente           | Temps écoulé jusqu’à ce que l’activité de copie commence sur le runtime d’intégration. |
| Script de pré-copie | Temps écoulé entre le début de l’activité de copie sur le runtime d’intégration et la fin de l’exécution du script de pré-copie de l’activité de copie dans la banque de données réceptrice. Appliquez lorsque vous configurez le script de pré-copie pour les récepteurs de base de données : par exemple, lors de l’écriture de données dans Azure SQL Database, effectuez un nettoyage avant de copier les nouvelles données. |
| Transférer        | Temps écoulé entre la fin de l’étape précédente et le transfert par le runtime de toutes les données de la source vers le récepteur. <br/>Notez que les sous-étapes sous le transfert s’exécutent en parallèle et que certaines opérations ne sont pas affichées actuellement, par exemple, l’analyse/la génération du format de fichier.<br><br/>- **Temps jusqu’au premier octet :** Temps écoulé entre la fin de l’étape précédente et l’heure à laquelle le runtime d'intégration reçoit le premier octet du magasin de données source. S’applique aux sources non basées sur des fichiers.<br>- **Liste des sources :** Durée d’énumération des fichiers sources ou des partitions de données. Ces derniers s’appliquent lorsque vous configurez des options de partition pour des sources de base de données, par exemple lorsque vous copiez des données à partir de bases de données comme Oracle/SAP HANA/Teradata/Netezza/etc.<br/>-**Lecture à partir de la source :** Durée de récupération des données dans le magasin de données source.<br/>- **Écriture dans le récepteur :** Durée d’écriture des données dans le magasin de données récepteur. Notez que certains connecteurs n’ont pas cette métrique à l’heure actuelle, notamment Recherche cognitive Azure, Azure Data Explorer, Stockage de tables Azure, Oracle, SQL Server, Common Data Service, Dynamics 365, Dynamics CRM, Salesforce/Salesforce Service Cloud. |

## <a name="troubleshoot-copy-activity-on-azure-ir"></a>Résoudre les problèmes liés à l’activité de copie sur Azure IR

Suivez la [procédure de réglage des performances](copy-activity-performance.md#performance-tuning-steps) pour planifier et effectuer un test de performances pour votre scénario. 

Si les performances de l’activité de copie ne répondent pas à vos attentes et si vous voyez [Conseils sur le réglage des performances](#performance-tuning-tips) affiché dans la vue de l’analyse de copie, appliquez la suggestion, puis réessayez pour résoudre les problèmes liés à l’activité de copie unique exécutée sur Azure Integration Runtime. Dans le cas contraire, [comprenez les détails de l’exécution de l’activité de copie](#understand-copy-activity-execution-details), vérifiez quelle étape a la durée **la plus longue** et appliquez les conseils ci-dessous pour améliorer les performances de copie :

- **Le « script de pré-copie » a connu une longue durée :** cela signifie que le script de pré-copie exécuté sur la base de données du récepteur prend beaucoup de temps à se terminer. Ajustez la logique du script de pré-copie spécifiée pour améliorer les performances. Si vous avez besoin d’aide pour améliorer le script, contactez votre équipe de base de données.

- **Le « transfert – temps jusqu’au premier octet » a connu une longue durée de travail :** cela signifie que votre requête source prend beaucoup de temps pour retourner des données. Vérifiez et optimisez la requête ou le serveur. Si vous avez besoin d’une aide supplémentaire, contactez votre équipe de magasin de données.

- **Le « transfert – liste des sources » a connu une longue durée de travail :** cela signifie que l’énumération des fichiers sources ou des partitions de données de la base de données source est lente.
  - Lorsque vous copiez des données à partir d’une source basée sur des fichiers, si vous utilisez le **filtre de caractères génériques** sur le chemin d’accès au dossier ou le nom de fichier (`wildcardFolderPath` ou `wildcardFileName`), ou si vous utilisez le **filtre de l’heure de dernière modification du fichier** (`modifiedDatetimeStart` ou `modifiedDatetimeEnd`), notez qu’un tel filtre entraînerait une activité de copie répertoriant tous les fichiers sous le dossier spécifié vers le côté client puis appliquerait le filtre. Cette énumération de fichiers peut devenir le goulot d’étranglement, en particulier lorsque seul un petit ensemble de fichiers répond à la règle de filtre.

    - Vérifiez si vous pouvez [copier des fichiers en fonction du chemin d’accès de fichier ou du nom de fichier partitionné DateHeure](tutorial-incremental-copy-partitioned-file-name-copy-data-tool.md). De cette façon, le côté source n’est plus chargé.

    - Vérifiez si vous pouvez utiliser le filtre natif du magasin de données à la place, en particulier «  **prefix**  » pour Amazon S3/Blob Azure/Stockage Fichier Azure et «  **listAfter/listBefore**  » pour ADLS Gen1. Ces filtres sont des filtres côté serveur du magasin de données et offrent de meilleures performances.

    - Envisagez de fractionner un jeu de données volumineux en plusieurs jeux de données plus petits et de laisser les travaux de copie s’exécuter simultanément, chacun s’attaquant à une partie des données. Vous pouvez le faire avec Lookup/GetMetadata + ForEach + Copy. Reportez-vous aux modèles de solution [Copie de fichiers à partir de plusieurs conteneurs](solution-template-copy-files-multiple-containers.md) ou [Migrer des données d’Amazon S3 vers ADLS Gen2](solution-template-migration-s3-azure.md) en guise d’exemple général.

  - Vérifiez si ADF signale une erreur de limitation sur la source ou si votre magasin de données est dans un état d’utilisation élevée. Si c’est le cas, réduisez vos charges de travail sur le magasin de données ou essayez de contacter votre administrateur de magasin de données pour augmenter la valeur de limitation ou la ressource disponible.

  - Utilisez Azure IR dans la région de votre magasin de données source ou près de celle-ci.

- **Le « transfert – lecture à partir de la source » a connu une longue durée de travail :** 

  - adoptez les meilleures pratiques de chargement de données spécifiques au connecteur si elles s’appliquent. Par exemple, lorsque vous copiez des données à partir d’[Amazon Redshift](connector-amazon-redshift.md), configurez l’utilisation de Redshift UNLOAD.

  - Vérifiez si ADF signale une erreur de limitation sur la source ou si votre magasin de données est soumis à une utilisation intensive. Si c’est le cas, réduisez vos charges de travail sur le magasin de données ou essayez de contacter votre administrateur de magasin de données pour augmenter la valeur de limitation ou la ressource disponible.

  - Vérifiez votre modèle de source et de récepteur de copie : 

    - Si votre modèle de copie prend en charge plus de quatre unités d’intégration de données (DIU), reportez-vous à [cette section](copy-activity-performance-features.md#data-integration-units) pour plus de détails, vous pouvez généralement essayer d’augmenter les DIU pour obtenir de meilleures performances. 

    - Dans le cas contraire, envisagez de fractionner un jeu de données volumineux en plusieurs jeux de données plus petits et de laisser les travaux de copie s’exécuter simultanément, chacun s’attaquant à une partie des données. Vous pouvez le faire avec Lookup/GetMetadata + ForEach + Copy. Reportez-vous aux modèles de solution [Copie de fichiers à partir de plusieurs conteneurs](solution-template-copy-files-multiple-containers.md), [Migrer des données d’Amazon S3 vers ADLS Gen2](solution-template-migration-s3-azure.md) ou [Copie en bloc avec une table de contrôles](solution-template-bulk-copy-with-control-table.md) en guise d’exemple général.

  - Utilisez Azure IR dans la région de votre magasin de données source ou près de celle-ci.

- **Le « transfert – écriture dans le récepteur » a connu une longue durée de travail :**

  - Adoptez les meilleures pratiques de chargement de données spécifiques au connecteur si elles s’appliquent. Par exemple, lors de la copie de données dans [Azure Synapse Analytics](connector-azure-sql-data-warehouse.md) (anciennement SQL DW), utilisez PolyBase ou l’instruction COPY. 

  - Vérifiez si ADF signale une erreur de limitation sur le récepteur ou si votre magasin de données est soumis à une utilisation intensive. Si c’est le cas, réduisez vos charges de travail sur le magasin de données ou essayez de contacter votre administrateur de magasin de données pour augmenter la valeur de limitation ou la ressource disponible.

  - Vérifiez votre modèle de source et de récepteur de copie : 

    - Si votre modèle de copie prend en charge plus de quatre unités d’intégration de données (DIU), reportez-vous à [cette section](copy-activity-performance-features.md#data-integration-units) pour plus de détails, vous pouvez généralement essayer d’augmenter les DIU pour obtenir de meilleures performances. 

    - Dans le cas contraire, réglez progressivement les [copies parallèles](copy-activity-performance-features.md). Notez qu’un trop grand nombre de copies parallèles peut également nuire aux performances.

  - Utilisez Azure IR dans la région de votre magasin de données récepteur ou près de celle-ci.

## <a name="troubleshoot-copy-activity-on-self-hosted-ir"></a>Résoudre les problèmes liés à l’activité de copie sur IR auto-hébergé

Suivez la [procédure de réglage des performances](copy-activity-performance.md#performance-tuning-steps) pour planifier et effectuer un test de performances pour votre scénario. 

Si les performances de copie ne répondent pas à vos attentes et si vous voyez [Conseils sur le réglage des performances](#performance-tuning-tips) affiché dans la vue de l’analyse de copie, appliquez la suggestion, puis réessayez pour résoudre les problèmes liés à l’activité de copie unique exécutée sur Azure Integration Runtime. Dans le cas contraire, [comprenez les détails de l’exécution de l’activité de copie](#understand-copy-activity-execution-details), vérifiez quelle étape a la durée **la plus longue** et appliquez les conseils ci-dessous pour améliorer les performances de copie :

- **La « file d’attente » a connu une longue durée :** cela signifie que l’activité de copie attend longtemps dans la file d’attente avant que votre IR auto-hébergé ne dispose des ressources nécessaires à son exécution. Vérifiez la capacité et l’utilisation du runtime d’intégration, puis [montez en puissance ou en charge](create-self-hosted-integration-runtime.md#high-availability-and-scalability) en fonction de votre charge de travail.

- **Le « transfert – temps jusqu’au premier octet » a connu une longue durée de travail :** cela signifie que votre requête source prend beaucoup de temps pour retourner des données. Vérifiez et optimisez la requête ou le serveur. Si vous avez besoin d’une aide supplémentaire, contactez votre équipe de magasin de données.

- **Le « transfert – liste des sources » a connu une longue durée de travail :** cela signifie que l’énumération des fichiers sources ou des partitions de données de la base de données source est lente.

  - Vérifiez si l’ordinateur IR auto-hébergé a une faible latence lors de la connexion au magasin de données source. Si votre source est dans Azure, vous pouvez utiliser [cet outil](http://www.azurespeed.com/Azure/Latency) pour vérifier la latence de l’ordinateur IR auto-hébergé dans la région Azure, sachant que moins la latence est importance, mieux c’est.

  - Lorsque vous copiez des données à partir d’une source basée sur des fichiers, si vous utilisez le **filtre de caractères génériques** sur le chemin d’accès au dossier ou le nom de fichier (`wildcardFolderPath` ou `wildcardFileName`), ou si vous utilisez le **filtre de l’heure de dernière modification du fichier** (`modifiedDatetimeStart` ou `modifiedDatetimeEnd`), notez qu’un tel filtre entraînerait une activité de copie répertoriant tous les fichiers sous le dossier spécifié vers le côté client puis appliquerait le filtre. Cette énumération de fichiers peut devenir le goulot d’étranglement, en particulier lorsque seul un petit ensemble de fichiers répond à la règle de filtre.

    - Vérifiez si vous pouvez [copier des fichiers en fonction du chemin d’accès de fichier ou du nom de fichier partitionné DateHeure](tutorial-incremental-copy-partitioned-file-name-copy-data-tool.md). De cette façon, le côté source n’est plus chargé.

    - Vérifiez si vous pouvez utiliser le filtre natif du magasin de données à la place, en particulier «  **prefix**  » pour Amazon S3/Blob Azure/Stockage Fichier Azure et «  **listAfter/listBefore**  » pour ADLS Gen1. Ces filtres sont des filtres côté serveur du magasin de données et offrent de meilleures performances.

    - Envisagez de fractionner un jeu de données volumineux en plusieurs jeux de données plus petits et de laisser les travaux de copie s’exécuter simultanément, chacun s’attaquant à une partie des données. Vous pouvez le faire avec Lookup/GetMetadata + ForEach + Copy. Reportez-vous aux modèles de solution [Copie de fichiers à partir de plusieurs conteneurs](solution-template-copy-files-multiple-containers.md) ou [Migrer des données d’Amazon S3 vers ADLS Gen2](solution-template-migration-s3-azure.md) en guise d’exemple général.

  - Vérifiez si ADF signale une erreur de limitation sur la source ou si votre magasin de données est dans un état d’utilisation élevée. Si c’est le cas, réduisez vos charges de travail sur le magasin de données ou essayez de contacter votre administrateur de magasin de données pour augmenter la valeur de limitation ou la ressource disponible.

- **Le « transfert – lecture à partir de la source » a connu une longue durée de travail :** 

  - Vérifiez si l’ordinateur IR auto-hébergé a une faible latence lors de la connexion au magasin de données source. Si votre source est dans Azure, vous pouvez utiliser [cet outil](http://www.azurespeed.com/Azure/Latency) pour vérifier la latence de l’ordinateur IR auto-hébergé dans les régions Azure, sachant que moins la latence est importance, mieux c’est.

  - Vérifiez si l’ordinateur IR auto-hébergé a suffisamment de bande passante entrante pour lire et transférer les données efficacement. Si votre magasin de données source est dans Azure, vous pouvez utiliser [cet outil](https://www.azurespeed.com/Azure/Download) pour vérifier la vitesse de téléchargement.

  - Vérifiez la tendance d’utilisation de l’UC et de la mémoire de l’IR auto-hébergé dans Portail Azure -> votre fabrique de données -> page de vue d’ensemble. Envisagez de [monter en puissance ou en charge l’IR](create-self-hosted-integration-runtime.md#high-availability-and-scalability) si l’utilisation de l’UC est élevée ou si la mémoire disponible est faible.

  - Adoptez les meilleures pratiques de chargement de données spécifiques au connecteur si elles s’appliquent. Par exemple :

    - Lorsque vous copiez des données à partir des magasins de données [Oracle](connector-oracle.md#oracle-as-source), [Netezza](connector-netezza.md#netezza-as-source), [Teradata](connector-teradata.md#teradata-as-source), [SAP HANA](connector-sap-hana.md#sap-hana-as-source), [SAP Table](connector-sap-table.md#sap-table-as-source) et [SAP Open Hub](connector-sap-business-warehouse-open-hub.md#sap-bw-open-hub-as-source), activer les options de partition de données pour copier les données en parallèle.

    - Lorsque vous copiez des données à partir de [HDFS](connector-hdfs.md), configurez l’utilisation de DistCp.

    - Lorsque vous copiez des données à partir d’[Amazon Redshift](connector-amazon-redshift.md), configurez l’utilisation de Redshift UNLOAD.

  - Vérifiez si ADF signale une erreur de limitation sur la source ou si votre magasin de données est soumis à une utilisation intensive. Si c’est le cas, réduisez vos charges de travail sur le magasin de données ou essayez de contacter votre administrateur de magasin de données pour augmenter la valeur de limitation ou la ressource disponible.

  - Vérifiez votre modèle de source et de récepteur de copie : 

    - Si vous copiez des données à partir de magasins de données avec option de partition, envisagez de régler progressivement les [copies parallèles](copy-activity-performance-features.md). Notez qu’un trop grand nombre de copies parallèles peut également nuire aux performances.

    - Dans le cas contraire, envisagez de fractionner un jeu de données volumineux en plusieurs jeux de données plus petits et de laisser les travaux de copie s’exécuter simultanément, chacun s’attaquant à une partie des données. Vous pouvez le faire avec Lookup/GetMetadata + ForEach + Copy. Reportez-vous aux modèles de solution [Copie de fichiers à partir de plusieurs conteneurs](solution-template-copy-files-multiple-containers.md), [Migrer des données d’Amazon S3 vers ADLS Gen2](solution-template-migration-s3-azure.md) ou [Copie en bloc avec une table de contrôles](solution-template-bulk-copy-with-control-table.md) en guise d’exemple général.

- **Le « transfert – écriture dans le récepteur » a connu une longue durée de travail :**

  - Adoptez les meilleures pratiques de chargement de données spécifiques au connecteur si elles s’appliquent. Par exemple, lors de la copie de données dans [Azure Synapse Analytics](connector-azure-sql-data-warehouse.md) (anciennement SQL DW), utilisez PolyBase ou l’instruction COPY. 

  - Vérifiez si l’ordinateur IR auto-hébergé a une faible latence lors de la connexion au magasin de données du récepteur. Si votre récepteur est dans Azure, vous pouvez utiliser [cet outil](http://www.azurespeed.com/Azure/Latency) pour vérifier la latence de l’ordinateur IR auto-hébergé dans la région Azure, sachant que moins la latence est importance, mieux c’est.

  - Vérifiez si l’ordinateur IR auto-hébergé a suffisamment de bande passante sortante pour transférer et écrire les données efficacement. Si votre magasin de données récepteur est dans Azure, vous pouvez utiliser [cet outil](https://www.azurespeed.com/Azure/UploadLargeFile) pour vérifier la vitesse de chargement.

  - Vérifiez la tendance d’utilisation de l’UC et de la mémoire de l’IR auto-hébergé dans Portail Azure -> votre fabrique de données -> page de vue d’ensemble. Envisagez de [monter en puissance ou en charge l’IR](create-self-hosted-integration-runtime.md#high-availability-and-scalability) si l’utilisation de l’UC est élevée ou si la mémoire disponible est faible.

  - Vérifiez si ADF signale une erreur de limitation sur le récepteur ou si votre magasin de données est soumis à une utilisation intensive. Si c’est le cas, réduisez vos charges de travail sur le magasin de données ou essayez de contacter votre administrateur de magasin de données pour augmenter la valeur de limitation ou la ressource disponible.

  - Réglez progressivement les [copies parallèles](copy-activity-performance-features.md). Notez qu’un trop grand nombre de copies parallèles peut également nuire aux performances.

## <a name="other-references"></a>Autres références

Voici des références relatives au monitoring et au réglage des performances pour quelques magasins de données pris en charge :

* Stockage Blob Azure : [Objectifs de performance et de scalabilité pour le stockage d’objets blob](../storage/blobs/scalability-targets.md) et [Liste de contrôle des performances et de la scalabilité pour le stockage d’objets blob](../storage/blobs/storage-performance-checklist.md).
* Stockage Table Azure : [Objectifs de performance et de scalabilité pour le stockage Table](../storage/tables/scalability-targets.md) et [Liste de contrôle des performances et de la scalabilité pour le stockage Table](../storage/tables/storage-performance-checklist.md).
* Azure SQL Database : Vous pouvez [surveiller les performances](../azure-sql/database/monitor-tune-overview.md) et vérifier le pourcentage de l’unité de transaction de base de données (DTU).
* Azure Synapse Analytics (anciennement SQL Data Warehouse) : Sa capacité est mesurée en DWU (Data Warehouse Units). Voir [Gérer la puissance de calcul dans Azure Synapse Analytics (présentation)](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-manage-compute-overview.md).
* Azure Cosmos DB : [Niveaux de performances dans Azure Cosmos DB](../cosmos-db/performance-levels.md).
* SQL Server : [Surveiller et régler les performances](/sql/relational-databases/performance/monitor-and-tune-for-performance).
* Serveur de fichiers local : [Réglage des performances des serveurs de fichiers](/previous-versions//dn567661(v=vs.85)).

## <a name="next-steps"></a>Étapes suivantes
Consultez les autres articles relatifs à l’activité de copie :

- [Vue d’ensemble des activités de copie](copy-activity-overview.md)
- [Guide sur les performances et l’extensibilité de l’activité de copie](copy-activity-performance.md)
- [Fonctionnalités d’optimisation des performances de l’activité de copie](copy-activity-performance-features.md)
- [Utiliser Azure Data Factory pour migrer des données de Data Lake ou Data Warehouse vers Azure](data-migration-guidance-overview.md)
- [Migrer des données d’Amazon S3 vers le stockage Azure](data-migration-guidance-s3-azure-storage.md)