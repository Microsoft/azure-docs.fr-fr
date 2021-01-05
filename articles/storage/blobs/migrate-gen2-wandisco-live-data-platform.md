---
title: Data Lake Storage et WANdisco LiveData Platform for Azure (préversion)
description: Migrez les données Hadoop locales vers Azure Data Lake Storage Gen2 en utilisant WANdisco LiveData Platform for Azure.
author: normesta
ms.topic: how-to
ms.author: normesta
ms.reviewer: b-pauls
ms.date: 11/06/2020
ms.service: storage
ms.custom: references_regions
ms.subservice: data-lake-storage-gen2
ms.openlocfilehash: db95f22a17c3776d84f12249693fb23b3d2d94e6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95914437"
---
# <a name="meet-demanding-migration-requirements-with-wandisco-livedata-platform-for-azure-preview"></a>Répondre aux conditions strictes de migration avec WANdisco LiveData Platform for Azure (préversion)

Migrez les données Hadoop locales vers Azure Data Lake Storage Gen2 en utilisant [WANdisco LiveData Platform for Azure](https://docs.wandisco.com/live-data-platform/docs/landing/). Cette plateforme élimine la nécessité d’arrêter les applications, élimine les risques de perte de données et garantit la cohérence des données, même si des opérations se poursuivent en local.  

> [!NOTE]
> WANdisco LiveData Platform for Azure est en préversion publique. Pour la disponibilité dans les différentes régions, consultez [Régions prises en charge](https://docs.wandisco.com/live-data-platform/docs/prereq#supported-regions).

La plateforme se compose de deux services : [LiveData Migrator for Azure](https://www.wandisco.com/products/livedata-migrator-for-azure) pour migrer des données faisant l’objet d’une utilisation active depuis des environnements locaux vers Stockage Azure, et [LiveData Plane for Azure](https://www.wandisco.com/products/livedata-plane-for-azure) qui garantit que toutes les données modifiées ou ingérées sont répliquées de façon cohérente. 

> [!div class="mx-imgBorder"]
> ![Illustration - Vue d’ensemble de Live Data Platform](./media/migrate-gen2-wandisco-live-data-platform/live-data-platform-overview.png)

Vous pouvez gérer les deux services en utilisant le portail Azure et Azure CLI, et les deux suivent le même modèle de facturation mesuré à l’utilisation que tous les autres services Azure. La consommation de LiveData Platform for Azure apparaît sur la même facture Azure mensuelle, et fournit un moyen cohérent et pratique de suivre et de superviser votre utilisation.

Contrairement à une migration des données effectuée _hors connexion_ en [copiant des informations statiques dans Azure Data Box](./data-lake-storage-migrate-on-premises-hdfs-cluster.md) ou en utilisant des outils Hadoop comme [DistCp](https://hadoop.apache.org/docs/current/hadoop-distcp/DistCp.html), vous pouvez conserver un fonctionnement entièrement opérationnel de vos systèmes métier pendant la _migration_ en ligne avec WANdisco LiveData for Azure. Gardez vos environnements Big Data opérationnels même lors du déplacement de leurs données vers Azure.

## <a name="key-features-of-wandisco-livedata-platform-for-azure"></a>Fonctionnalités clés de WANdisco LiveData Platform for Azure

[WANdisco LiveData Platform for Azure](https://docs.wandisco.com/live-data-platform/docs/landing/) utilise un moteur de consensus avec des capacités WAN pour assurer la cohérence des données et la réplication des données à grande échelle, tandis que les applications peuvent continuer à modifier les données en cours de réplication.  

Les fonctionnalités clés de la plateforme sont les suivantes :

- **Cohérence des données** : Relevez les défis d’une migration de grands volumes de données entre différents environnements, en conservant la cohérence de ces données entre les systèmes de stockage tout au long de la migration, même quand ils sont soumis à des modifications de façon continue. Utilisez le moteur de consensus unique avec des capacités WAN directement dans Azure pour assurer la cohérence des données et migrer les données avec des garanties de cohérence alors qu’elles continuent d’être modifiées.

- **Maintenir l’exploitation** : Comme les applications peuvent continuer à créer, modifier, lire et supprimer des données pendant la migration, il n’est pas nécessaire d’interrompre les opérations de l’entreprise ou d’introduire une fenêtre d’indisponibilité simplement pour migrer le Big Data vers Azure. Continuez à faire fonctionner les applications, l’infrastructure d’analytique, les travaux d’ingestion et d’autres traitements.

- **Valider les résultats** : La validation complète du fait que vos données peuvent être utilisées réellement une fois migrées vers Azure nécessite l’exécution de charges de travail d’applications de production sur ces données. Seul un service LiveData offre cette possibilité sans introduire un risque de divergence des données, en conservant la cohérence des données, que les modifications se produisent au niveau de la source ou de la cible de votre migration. Testez et validez le comportement de l’application sans risque ni modification de vos processus et de vos systèmes.

- **Réduire la complexité** : Éliminez la nécessité de créer et de gérer des travaux planifiés pour copier les données en migrant des données en faisant appel à l’automatisation. Utilisez l’intégration étroite à Azure comme plan de contrôle pour gérer et superviser la progression de la migration, avec notamment la réplication sélective des données, les métadonnées Hive, et la sécurité et la confidentialité des données.

- **Efficacité** : Conservez des performances et un débit élevés, et mettez-les facilement à l’échelle pour de gros volumes de données. Avec un contrôle de la consommation de la bande passante, vous pouvez être sûr d’atteindre vos objectifs de migration sans impacter le fonctionnement des autres systèmes.

## <a name="migrate-big-data-faster-without-risk"></a>Migrer le Big Data plus rapidement sans risque

Le premier service de WANdisco LiveData Platform for Azure est [LiveData Migrator for Azure](https://www.wandisco.com/products/livedata-migrator-for-azure) : une solution pour migrer activement des données utilisées depuis des environnements locaux vers Stockage Azure. LiveData Migrator for Azure est provisionné et géré entièrement à partir du portail Azure ou d’Azure CLI, et il fonctionne à côté de votre cluster Hadoop local sans modification de configuration, sans modification de l’application ou sans redémarrages du service pour commencer à migrer les données immédiatement.

> [!div class="mx-imgBorder"]
> ![Architecture de LiveData Migrator for Azure](./media/migrate-gen2-wandisco-live-data-platform/live-data-migrator-architecture.png)

Les migrations du Big Data peuvent être complexes et difficiles. Le déplacement de pétaoctets d’informations sans interrompre les opérations de l’entreprise était impossible à réaliser avec les technologies de copie de données hors ligne. [LiveData Migrator for Azure](https://www.wandisco.com/products/livedata-migrator-for-azure) offre un déploiement simple et peut établir un service LiveData avec la migration et la réplication continues des données pendant que les applications lisent, écrivent et modifient les données en cours de migration.

Pour effectuer une migration, il suffit de suivre les trois étapes suivantes :

1. Provisionnez l’instance LiveData Migrator à partir du portail Azure sur votre cluster Hadoop local. Aucune modification ou temps d’arrêt du cluster n’est nécessaire, et les applications peuvent continuer à fonctionner.

   > [!div class="mx-imgBorder"]
   >![Créer une instance LiveData Migrator](./media/migrate-gen2-wandisco-live-data-platform/create-live-data-migrator.png)

2. Définissez le compte de stockage cible Azure Data Lake Storage Gen2.

   > [!div class="mx-imgBorder"]
   >![Créer une cible LiveData Migrator](./media/migrate-gen2-wandisco-live-data-platform/create-target.png)

3. Définissez l’emplacement des données que vous voulez migrer, par exemple `/user/hive/warehouse`, et démarrez la migration.

   > [!div class="mx-imgBorder"]
   > ![Créer une migration LiveData Migrator](./media/migrate-gen2-wandisco-live-data-platform/create-migration.png)

Supervisez la progression de votre migration via les outils Azure standard, notamment Azure CLI et le portail Azure, et continuez à utiliser pendant ce temps-là votre environnement local. Avant de commencer, passez en revue ces [prérequis](https://docs.wandisco.com/live-data-platform/docs/prereq/).

## <a name="replicate-data-under-active-change"></a>Répliquer des données alors qu’elles font l’objet de modifications

Les migrations à grande échelle de lacs de données locaux vers Azure demandent de tester et de valider les applications. La possibilité de faire cela sans risquer d’introduire des modifications des données qui vont créer plusieurs sources de vérité difficiles à réconcilier est critique pour éliminer les risques et réduire le coût d’un déplacement vers Azure. [LiveData Plane for Azure](https://www.wandisco.com/products/livedata-plane-for-azure) utilise la technologie du moteur de coordination de WANdisco résoudre ces problèmes.

> [!div class="mx-imgBorder"]
> ![Architecture de LiveData Plane for Azure](./media/migrate-gen2-wandisco-live-data-platform/live-data-plane-architecture.png)

Conservez la cohérence de vos données entre les clusters Hadoop locaux et Stockage Azure avec LiveData Plane for Azure après la migration initiale :

1. Provisionnez LiveData Plane for Azure localement et dans Azure, en commençant à partir du portail Azure. Aucune modification des applications n’est nécessaire.
2. Configurez des règles de réplication qui couvrent les emplacements de ces données que vous voulez conserver cohérentes, par exemple : `/user/contoso/sales/region/WA`.
3. Exécutez les applications qui accèdent aux données et les modifient dans l’un ou l’autre emplacement en tant que système de fichiers compatible Hadoop, selon vos besoins.

LiveData Plane for Azure assure la cohérence de vos données sans imposer de surcharge importante sur les performances des opérations ou des applications du cluster. Modifiez ou ingérez des données alors que toutes les modifications sont répliquées de façon cohérente.

## <a name="next-steps"></a>Étapes suivantes

- [LiveData Platform for Azure](https://docs.wandisco.com/live-data-platform/docs/landing/) est utilisé comme n’importe quelle autre ressource Azure et est disponible dès maintenant en préversion. 

- Découvrez les [prérequis](https://docs.wandisco.com/live-data-platform/docs/prereq/), planifiez votre migration et effectuez rapidement une migration à grande échelle avec LiveData Migrator for Azure.

- Essayez LiveData Migrator sans avoir besoin d’un cluster Hadoop local grâce au [Sandbox (bac à sable) HDFS](https://docs.wandisco.com/live-data-platform/docs/create-sandbox-intro/).

## <a name="see-also"></a>Voir aussi

- [LiveData Migrator for Azure sur la Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/wandisco.ldm?tab=Overview)

- [LiveData Plane for Azure sur la Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/wandisco.ldp?tab=Overview)

- [Plans et tarification de LiveData Migrator for Azure](https://azuremarketplace.microsoft.com/marketplace/apps/wandisco.ldm?tab=PlansAndPrice)

- [Plans et tarification de LiveData Plane for Azure](https://azuremarketplace.microsoft.com/marketplace/apps/wandisco.ldp?tab=PlansAndPrice) 

- [Questions fréquentes (FAQ) sur LiveData Platform for Azure](https://docs.wandisco.com/live-data-platform/docs/faq/)

- [Problèmes connus avec LiveData Platform for Azure](https://docs.wandisco.com/live-data-platform/docs/known-issues/)

- [Introduction à Azure Data Lake Storage Gen2](data-lake-storage-introduction.md)