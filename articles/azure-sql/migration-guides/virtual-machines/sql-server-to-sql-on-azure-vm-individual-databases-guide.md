---
title: SQL Server vers SQL Server sur les machines virtuelles Azure (Guide de migration)
description: Suivez ce guide pour migrer vos bases de données SQL Server individuelles vers SQL Server sur les machines virtuelles Azure.
ms.custom: ''
ms.service: virtual-machines-sql
ms.subservice: ''
ms.devlang: ''
ms.topic: how-to
author: markjones-msft
ms.author: markjon
ms.reviewer: mathoma
ms.date: 11/06/2020
ms.openlocfilehash: 3b0fdccd3eaf6e6bd94b595107022f738bdd8382
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96325913"
---
# <a name="migration-guide-sql-server-to-sql-server-on-azure-vms"></a>Guide de migration : SQL Server vers SQL Server sur les machines virtuelles Azure 
[!INCLUDE[appliesto--sqlmi](../../includes/appliesto-sqlvm.md)]

Ce guide de migration vous apprend à **découvrir**, **évaluer** et **migrer** vos bases de données utilisateur de SQL Server vers une instance de SQL Server sur les machines virtuelles Azure à l’aide de la fonctionnalité de sauvegarde et restauration, de la fonctionnalité de copie des journaux de transaction et de l’outil [DMA (Assistant Migration de données)](/sql/dma/dma-overview) à des fins d’évaluation. 

Vous pouvez migrer SQL Server s’exécutant en local ou sur :

- SQL Server sur les machines virtuelles  
- Amazon Web Services (AWS) EC2 
- Amazon Relational Database Service (AWS RDS) 
- Compute Engine (Google Cloud Platform - GCP)

Pour plus d’informations sur les autres stratégies de migration, consultez la [vue d’ensemble de la migration de machine virtuelle SQL Server](sql-server-to-sql-on-azure-vm-migration-overview.md).

:::image type="content" source="media/sql-server-to-sql-on-azure-vm-migration-overview/migration-process-flow-small.png" alt-text="Flux du processus de migration":::

## <a name="prerequisites"></a>Prérequis

La migration vers SQL Server sur les machines virtuelles Azure nécessite les éléments suivants : 

- [Assistant Migration de données Microsoft (DMA)](https://www.microsoft.com/download/details.aspx?id=53595)
- [Projet Azure Migrate](../../../migrate/create-manage-projects.md)
- [Instance cible préparée de SQL Server sur une machine virtuelle Azure](../../virtual-machines/windows/create-sql-vm-portal.md) dont la version est identique ou supérieure à celle du serveur SQL Server source
- [Connectivité entre Azure et l’environnement local](/azure/architecture/reference-architectures/hybrid-networking)
- [Choix d’une stratégie de migration appropriée](sql-server-to-sql-on-azure-vm-migration-overview.md#migrate)

## <a name="pre-migration"></a>Prémigration

Avant de commencer votre migration, découvrez la topologie de votre environnement SQL et évaluez la faisabilité de la migration prévue. 

### <a name="discover"></a>Découvrez

Azure Migrate évalue la pertinence de la migration des ordinateurs locaux, effectue un dimensionnement en fonction des performances et fournit des estimations de coût pour une exécution locale. Pour planifier la migration, utilisez Azure Migrate afin d’[identifier les sources de données existantes ainsi que les détails relatifs aux fonctionnalités](../../../migrate/concepts-assessment-calculation.md) utilisées par vos instances de SQL Server. Ce processus implique une analyse du réseau pour identifier toutes les instances de SQL Server de votre organisation ainsi que la version et les fonctionnalités utilisées. 

> [!IMPORTANT]
> Quand vous choisissez une machine virtuelle Azure cible pour votre instance de SQL Server, veillez à prendre en compte les [Recommandations de performances pour SQL Server sur les machines virtuelles Azure](../../virtual-machines/windows/performance-guidelines-best-practices.md).

Si vous souhaitez accéder à des outils de découverte supplémentaires, consultez [Services et outils disponibles pour les scénarios de migration de données](../../../dms/dms-tools-matrix.md#business-justification-phase).


### <a name="assess"></a>Évaluer

Une fois que vous avez découvert toutes les sources de données, utilisez l’outil [DMA (Assistant Migration de données)](/sql/dma/dma-overview) pour évaluer les instances de SQL Server locales qui migrent vers une instance de SQL Server sur une machine virtuelle Azure afin de déterminer les différences entre les instances source et cible. 


> [!NOTE]
> Si vous _ne mettez pas_ à niveau la version de SQL Server, ignorez cette étape et passez à la section [Migrer](#migrate). 


#### <a name="assess-user-databases"></a>Évaluer les bases de données utilisateur 

L’outil DMA (Assistant Migration de données) vous aide à effectuer la migration vers une plateforme de données moderne en détectant les problèmes de compatibilité qui peuvent impacter les fonctionnalités de base de données dans la nouvelle version de SQL Server. L’outil DMA recommande des améliorations pour la fiabilité et les performances de votre environnement cible. De plus, il vous permet de déplacer votre schéma, vos données et vos objets de connexion de votre serveur source vers votre serveur cible.

Pour en savoir plus, consultez les informations relatives à l’[évaluation](/sql/dma/dma-migrateonpremsql). 


> [!IMPORTANT]
>Selon le type d’évaluation, les autorisations nécessaires sur le serveur SQL Server source peuvent être différentes. 
   > - D’après les recommandations de l’outil Advisor sur la **parité des fonctionnalités**, les informations d’identification fournies pour se connecter à la base de données SQL Server source doivent être celles d’un membre du rôle serveur *sysadmin*.
   > - D’après les recommandations de l’outil Advisor sur les problèmes de compatibilité, les informations d’identification fournies doivent être celles d’un rôle disposant au moins des autorisations `CONNECT SQL`, `VIEW SERVER STATE` et `VIEW ANY DEFINITION`.
   > - L’outil DMA met en évidence les autorisations nécessaires d’après l’outil Advisor choisi avant d’exécuter l’évaluation.


#### <a name="assess-applications"></a>Évaluer les applications

En règle générale, la couche Application accède aux bases de données utilisateur pour conserver et modifier les données.  L’outil DMA peut évaluer la couche d’accès aux données d’une application de deux manières : 

- À l’aide des [événements étendus](/sql/relational-databases/extended-events/extended-events) capturés ou des [traces SQL Server Profiler ](/sql/tools/sql-server-profiler/create-a-trace-sql-server-profiler) de vos bases de données utilisateur. Vous pouvez également vous servir de l’[Assistant Expérimentation de base de données](/sql/dea/database-experimentation-assistant-capture-trace) afin de créer un journal des traces utilisable pour les tests A/B.

- À l’aide de [Data Access Migration Toolkit (préversion)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit), également appelé DAMT, qui permet de découvrir et d’évaluer les requêtes SQL contenues dans le code, et de migrer le code source de l’application d’une plateforme de base de données vers une autre. Cet outil prend en charge un grand nombre de types de fichier populaire, notamment les fichiers C#, Java, XML et en texte brut. Pour accéder au guide indiquant comment effectuer une évaluation DMAT, consultez le blog relatif à l’[utilisation de DMAT](https://techcommunity.microsoft.com/t5/microsoft-data-migration/using-data-migration-assistant-to-assess-an-application-s-data/ba-p/990430).

Utilisez l’outil DMA pour [importer](/sql/dma/dma-assesssqlonprem#add-databases-and-extended-events-trace-to-assess) des fichiers de traces capturées ou des fichiers DAMT durant l’évaluation des bases de données utilisateur. 


#### <a name="scale-assessments"></a>Évaluations mises à l’échelle

Si vous disposez de plusieurs serveurs qui nécessitent une évaluation à l’aide de l’outil DMA, vous pouvez automatiser le processus via l’[interface de ligne de commande](/sql/dma/dma-commandline). À l’aide de l’interface, vous pouvez préparer des commandes d’évaluation à l’avance pour chaque instance de SQL Server comprise dans l’étendue de la migration. 

Pour les rapports récapitulatifs portant sur de grands domaines, les évaluations de l’outil DMA (Assistant Migration de données) peuvent désormais être [centralisées dans Azure Migrate](/sql/dma/dma-assess-sql-data-estate-to-sqldb).

#### <a name="refactor-databases-with-dma"></a>Refactoriser les bases de données avec l’outil DMA

En fonction des résultats de l’évaluation effectuée avec l’outil DMA, vous pouvez disposer d’une série de recommandations qui vous permettent de vérifier que vos bases de données utilisateur fonctionneront correctement après la migration. L’outil DMA fournit des détails sur les objets impactés ainsi que des ressources sur la résolution de chaque problème. Il est recommandé de résoudre tous les changements cassants et tous les changements de comportement avant la migration de production.

Pour les fonctionnalités dépréciées, vous pouvez choisir d’exécuter vos bases de données utilisateur dans leur mode de [compatibilité](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level) d’origine, si vous souhaitez éviter d’effectuer ces changements afin d’accélérer la migration. Toutefois, cela empêchera la [mise à niveau de la compatibilité de votre base de données](/sql/database-engine/install-windows/compatibility-certification#compatibility-levels-and-database-engine-upgrades) tant que les éléments dépréciés n’auront pas été résolus.

Il est fortement recommandé que tous les correctifs DMA fassent l’objet d’un script et soient appliqués à la base de données SQL Server cible durant la [postmigration](#post-migration).

> [!CAUTION]
> Toutes les versions de SQL Server ne prennent pas en charge l’ensemble des modes de compatibilité. Vérifiez que votre [version cible de SQL Server](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level) prend en charge la compatibilité de base de données choisie. Par exemple, SQL Server 2019 ne prend pas en charge les bases de données ayant le niveau de compatibilité 90 (qui correspond à SQL Server 2005). Ces bases de données nécessitent au moins une mise à niveau vers le niveau de compatibilité 100.
>

## <a name="migrate"></a>Migrer

Une fois que vous avez effectué les étapes de prémigration, vous êtes prêt à migrer les bases de données utilisateur et les composants. Migrez les bases de données à l’aide de votre [méthode de migration](sql-server-to-sql-on-azure-vm-migration-overview.md#migrate) par défaut.  

Vous trouverez ci-dessous les étapes à suivre pour effectuer une migration à l’aide de la fonctionnalité de sauvegarde et restauration, ou une migration avec temps d’arrêt minimal à l’aide de la fonctionnalité de sauvegarde et restauration et de la copie des journaux de transaction. 

### <a name="backup-and-restore"></a>Sauvegarde et restauration

Pour effectuer une migration standard à l’aide de la fonctionnalité de sauvegarde et restauration, suivez les étapes ci-dessous : 

1. Configurez la connectivité à l’instance cible de SQL Server sur une machine virtuelle Azure en fonction de vos impératifs. Consultez [Se connecter à une machine virtuelle SQL Server sur Azure (Resource Manager)](../../virtual-machines/windows/ways-to-connect-to-sql.md).
1. Suspendez/arrêtez toutes les applications qui utilisent des bases de données destinées à être migrées. 
1. Vérifiez que les bases de données utilisateur sont inactives à l’aide du [mode mono-utilisateur](/sql/relational-databases/databases/set-a-database-to-single-user-mode). 
1. Effectuez une sauvegarde complète de la base de données vers un emplacement sur site.
1. Copiez vos fichiers de sauvegarde locaux sur votre machine virtuelle à l’aide du Bureau à distance, d’[Azure Data Explorer](/azure/data-explorer/data-explorer-overview) ou de l’[utilitaire en ligne de commande AZCopy](../../../storage/common/storage-use-azcopy-v10.md) (espace recommandé pour les sauvegardes > à 2 To).
1. Restaurez les sauvegardes complètes de base de données sur l’instance de SQL Server sur une machine virtuelle Azure.

### <a name="log-shipping--minimize-downtime"></a>Copie des journaux de transaction (réduire le temps d’arrêt)

Pour effectuer une migration avec un temps d’arrêt minimal à l’aide de la fonctionnalité de sauvegarde et restauration, et de la copie des journaux de transaction, suivez les étapes ci-dessous : 

1. Configurez la connectivité à instance cible de SQL Server sur une machine virtuelle Azure en fonction de vos impératifs. Consultez [Se connecter à une machine virtuelle SQL Server sur Azure (Resource Manager)](../../virtual-machines/windows/ways-to-connect-to-sql.md).
1. Vérifiez que les bases de données utilisateur locales à migrer sont en mode de récupération complète ou en mode de récupération utilisant les journaux de transactions.
1. Effectuez une sauvegarde complète de base de données vers un emplacement local, puis modifiez tous les travaux existants de sauvegarde complète de base de données afin d’utiliser le mot clé [COPY_ONLY](/sql/relational-databases/backup-restore/copy-only-backups-sql-server) pour conserver la séquence de journaux de transactions consécutifs.
1. Copiez vos fichiers de sauvegarde locaux sur votre machine virtuelle à l’aide du Bureau à distance, d’[Azure Data Explorer](/azure/data-explorer/data-explorer-overview) ou de l’[utilitaire en ligne de commande AZCopy](../../../storage/common/storage-use-azcopy-v10.md) (espace recommandé pour les sauvegardes > à 1 To).
1. Restaurez les sauvegardes complètes de base de données sur l’instance de SQL Server sur une machine virtuelle Azure.
1. Configurez la [copie des journaux de transaction](/sql/database-engine/log-shipping/configure-log-shipping-sql-server) entre la base de données locale et l’instance cible de SQL Server sur une machine virtuelle Azure. Veillez à ne pas réinitialiser les bases de données, car cette opération a déjà été effectuée au cours des étapes précédentes.
1. Effectuez le **basculement** vers le serveur cible. 
   1. Suspendez/Arrêtez les applications qui utilisent des bases de données à migrer. 
   1. Vérifiez que les bases de données utilisateur sont inactives à l’aide du [mode mono-utilisateur](/sql/relational-databases/databases/set-a-database-to-single-user-mode). 
   1. Une fois que vous êtes prêt, effectuez un [basculement contrôlé](/sql/database-engine/log-shipping/fail-over-to-a-log-shipping-secondary-sql-server) via la copie des journaux de transaction des bases de données locales vers l’instance cible de SQL Server sur une machine virtuelle Azure.



### <a name="migrating-objects-outside-user-databases"></a>Migration d’objets en dehors des bases de données utilisateur

Des objets SQL Server supplémentaires sont parfois nécessaires pour garantir un fonctionnement fluide de vos bases de données utilisateur après la migration. 

Le tableau ci-dessous fournit une liste des composants de référence et des méthodes de migration recommandées à appliquer avant ou après la migration de vos bases de données utilisateur : 


| **Fonctionnalité** | **Composant** | **Méthodes de migration** |
| --- | --- | --- |
| **Bases de données** | Modèle  | Script avec SQL Server Management Studio |
|| TempDB | Pour des performances optimales, prévoyez le déplacement de la base de données tempdb sur un [disque temporaire de machine virtuelle Azure (SSD)](../../virtual-machines/windows/performance-guidelines-best-practices.md#temporary-disk). Veillez à choisir une machine virtuelle disposant d’un disque SSD local dont la taille est suffisante pour accueillir votre base de données tempdb. |
|| Bases de données utilisateur avec Filestream |  Utilisez les méthodes de [sauvegarde et de restauration](../../virtual-machines/windows/migrate-to-vm-from-sql-server.md#back-up-and-restore) pour la migration. L’outil DMA ne prend pas en charge les bases de données avec Filestream. |
| **Sécurité** | Connexions SQL Server et Windows | Utilisez l’outil DMA pour [migrer les connexions utilisateur](/sql/dma/dma-migrateserverlogins). |
|| Rôles SQL Server | Script avec SQL Server Management Studio |
|| Fournisseurs de chiffrement | Recommandez la [conversion pour utiliser le service Azure Key Vault](../../virtual-machines/windows/azure-key-vault-integration-configure.md). Cette procédure utilise le [fournisseur de ressources de machines virtuelles SQL](../../virtual-machines/windows/sql-agent-extension-manually-register-single-vm.md). |
| **Objets de serveur** | Unités de sauvegarde | Remplacez-les par une sauvegarde de base de données à l’aide du [service Sauvegarde Azure](../../../backup/backup-sql-server-database-azure-vms.md), ou écrivez les sauvegardes dans le [Stockage Azure](../../virtual-machines/windows/azure-storage-sql-server-backup-restore-use.md) (SQL Server 2012 SP1 CU2 +). Cette procédure utilise le [fournisseur de ressources de machines virtuelles SQL](../../virtual-machines/windows/sql-agent-extension-manually-register-single-vm.md).|
|| Serveurs liés | Script avec SQL Server Management Studio. |
|| Déclencheurs de serveur | Script avec SQL Server Management Studio. |
| **Réplication** | Publications locales | Script avec SQL Server Management Studio. |
|| Abonnés locaux | Script avec SQL Server Management Studio. |
| **Polybase** | PolyBase | Script avec SQL Server Management Studio. |
| **Gestion** | Messagerie de base de données | Script avec SQL Server Management Studio. |
| **SQL Server Agent** | travaux | Script avec SQL Server Management Studio. |
|| Alertes | Script avec SQL Server Management Studio. |
|| Opérateurs | Script avec SQL Server Management Studio. |
|| Proxies | Script avec SQL Server Management Studio. |
| **Système d’exploitation** | Fichiers, partages de fichiers | Notez tous les fichiers ou partages de fichiers supplémentaires utilisés par vos serveurs SQL Server et répliqués sur la cible de machine virtuelle Azure. |


## <a name="post-migration"></a>Postmigration

Une fois que vous avez réussi la phase de migration, vous devez effectuer toute une série de tâches postmigration pour vérifier que tout fonctionne de manière aussi fluide et efficace que possible.

### <a name="remediate-applications"></a>Corriger les applications

Une fois les données migrées vers l’environnement cible, toutes les applications qui consommaient la source doivent commencer à consommer la cible. Dans certains cas, cela vous oblige à apporter des changements aux applications.

Appliquez les correctifs recommandés par l’Assistant Migration de données aux bases de données utilisateur. Nous vous recommandons de créer des scripts de ces correctifs pour garantir la cohérence et permettre l’automatisation.

### <a name="perform-tests"></a>Effectuer des tests

L’approche de test d’une migration de base de données comprend les activités suivantes :

1. **Développer des tests de validation.**  Pour tester des migrations de bases de données, vous devez utiliser des requêtes SQL. Créez des requêtes de validation à exécuter sur les bases de données source et cible. Vos requêtes de validation doivent couvrir l’étendue que vous avez définie.
2. **Configurer un environnement de test.**  l’environnement de test doit contenir une copie de la base de données source et de la base de données cible. Veillez à isoler l’environnement de test.
3. **Exécuter des tests de validation.**  exécutez les tests de validation sur la source et sur la cible, puis analysez les résultats.
4. **Exécuter des tests de performances.**  Exécutez un test de performances sur la source et sur la cible, puis analysez et comparez les résultats.

> [!TIP]
> Utilisez l’[Assistant Expérimentation de base de données](/sql/dea/database-experimentation-assistant-overview) pour faciliter l’évaluation des performances de l’instance cible de SQL Server.
>

### <a name="optimize"></a>Optimiser

La phase postmigration est cruciale pour résoudre les problèmes de justesse et d’exhaustivité des données ainsi que pour gérer les problèmes de performances potentiels liés à la charge de travail.

Pour plus d’informations sur ces problèmes et sur les étapes spécifiques à entreprendre pour les atténuer, consultez les ressources suivantes :

- [Guide de validation et d’optimisation postmigration.](/sql/relational-databases/post-migration-validation-and-optimization-guide)
- [Paramétrage des performances de SQL Server sur les machines virtuelles Azure](../../virtual-machines/windows/performance-guidelines-best-practices.md).
- [Centre d’optimisation des coûts Azure](https://azure.microsoft.com/overview/cost-optimization/).

## <a name="next-steps"></a>Étapes suivantes

- Pour vérifier la disponibilité des services applicables à SQL Server, accédez au [Centre de l’infrastructure mondiale Azure](https://azure.microsoft.com/global-infrastructure/services/?regions=all&amp;products=synapse-analytics,virtual-machines,sql-database)

- Si vous souhaitez obtenir une matrice des services et outils Microsoft et tiers qui peuvent vous aider dans différents scénarios de migration de données et de base de données ainsi que pour des tâches spécialisées, consultez l’article [Services et outils disponibles pour les scénarios de migration de données](../../../dms/dms-tools-matrix.md).

- Pour en savoir plus sur Azure SQL, consultez :
   - [Options de déploiement](../../azure-sql-iaas-vs-paas-what-is-overview.md)
   - [SQL Server sur machines virtuelles Azure](../../virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md)
   - [Outil de calcul du coût total de possession Azure](https://azure.microsoft.com/pricing/tco/calculator/) 


- Pour en savoir plus sur le framework et le cycle d’adoption des migrations cloud, consultez
   -  [Cloud Adoption Framework pour Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/contoso-migration-scale)
   -  [Bonnes pratiques pour l’évaluation des coûts et du dimensionnement des charges de travail migrées vers Azure](/azure/cloud-adoption-framework/migrate/azure-best-practices/migrate-best-practices-costs) 

- Pour plus d’informations sur la gestion des licences, consultez
   - [BYOL (apportez votre propre licence) avec Azure Hybrid Benefit](../../virtual-machines/windows/licensing-model-azure-hybrid-benefit-ahb-change.md)
   - [Bénéficier d’un support étendu gratuit pour SQL Server 2008 et SQL Server 2008 R2](../../virtual-machines/windows/sql-server-2008-extend-end-of-support.md)


- Pour évaluer la couche d’accès aux applications, consultez [Data Access Migration Toolkit (préversion)](https://marketplace.visualstudio.com/items?itemName=ms-databasemigration.data-access-migration-toolkit)
- Pour plus d’informations sur l’exécution des tests A/B de la couche d’accès aux données, consultez [Assistant Expérimentation de base de données](/sql/dea/database-experimentation-assistant-overview).