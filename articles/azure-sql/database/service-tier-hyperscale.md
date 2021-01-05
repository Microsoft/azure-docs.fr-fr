---
title: Présentation du niveau de service Hyperscale
description: Cet article décrit le niveau de service Hyperscale dans le modèle d’achat vCore dans Azure SQL Database, et explique en quoi il diffère des niveaux de service Usage général et Critique pour l’entreprise.
services: sql-database
ms.service: sql-database
ms.subservice: service
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 10/19/2020
ms.openlocfilehash: 56c3475ae6a03600723e7a12b3f3809f003ce7c4
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96922272"
---
# <a name="hyperscale-service-tier"></a>Niveau de service Hyperscale

Azure SQL Database est basé sur une architecture de moteur de base de données SQL Server. Celle-ci est ajustée pour l’environnement cloud afin de garantir une disponibilité de 99,99 % même en cas de panne d’infrastructure. Trois modèles d’architecture sont utilisés dans Azure SQL Database :

- Usage général/Standard
- Hyperscale
- Critique pour l’entreprise/Premium

Le niveau de service Hyperscale dans Azure SQL Database est le tout nouveau niveau de service du modèle d’achat vCore. Ce niveau de service est un stockage hautement scalable et un niveau de performances de calcul qui tire partie de l’architecture Azure pour effectuer un scale-out du stockage et des ressources de calcul d’une base de données Azure SQL bien au-delà des limites disponibles des niveaux Usage général et Critique pour l’entreprise.

> [!NOTE]
>
> - Pour en savoir plus sur les niveaux de service Usage général et Critique pour l’entreprise du modèle d’achat vCore, consultez les niveaux de service [Usage général](service-tier-general-purpose.md) et [Critique pour l’entreprise](service-tier-business-critical.md). Pour obtenir une comparaison du modèle d’achat vCore avec le modèle d’achat DTU, consultez [Ressources et modèles d’achat Azure SQL Database](purchasing-models.md).
> - Le niveau de service Hyperscale n'est actuellement disponible que pour Azure SQL Database, et non pour Azure SQL Managed Instance.

## <a name="what-are-the-hyperscale-capabilities"></a>Présentation des fonctionnalités Hyperscale

Le niveau de service Hyperscale dans Azure SQL Database fournit les fonctionnalités supplémentaires suivantes :

- Prise en charge d’une taille de base de données pouvant atteindre 100 To
- Sauvegardes de base de données quasi instantanées (basées sur des instantanés de fichiers conservés dans le Stockage Blob Azure), quel que soit leur taille, sans impact des E/S sur les ressources de calcul  
- Restaurations de base de données rapides (basées sur des instantanés de fichiers) en minutes plutôt qu’en heures ou en jours (opération qui ne dépend pas de la taille des données)
- Meilleures performances générales en raison d’un débit de journal plus élevé et de temps de validation de transaction plus rapides, quels que soient les volumes de données
- Effectuer un scale-out rapide : vous pouvez provisionner un ou plusieurs nœuds en lecture seule pour décharger votre charge de travail de lecture et les utiliser comme serveurs de secours
- Scale-up rapide : vous pouvez, en temps constant, augmenter la puissance de vos ressources de calcul pour prendre en charge des charges de travail lourdes en cas de besoin, puis la diminuer à nouveau.

Le niveau de service Hyperscale supprime de nombreuses limites pratiques traditionnellement rencontrées dans les bases de données cloud. Là où la plupart des autres bases de données sont limitées par les ressources disponibles dans un seul nœud, les bases de données du niveau de service Hyperscale n’ont pas de limite. Avec son architecture de stockage flexible, le stockage augmente en fonction des besoins. En fait, les bases de données Hyperscale sont créées sans taille maximale définie. Une base de données Hyperscale augmente en fonction des besoins, et vous êtes facturé uniquement pour la capacité que vous utilisez. Pour les charges de travail de lecture intensives, le niveau de service Hyperscale permet une expansion rapide en provisionnant des réplicas de lecture supplémentaires en fonction des besoins pour décharger les charges de travail de lecture.

Par ailleurs, le temps nécessaire pour créer des sauvegardes de base de données ou pour augmenter ou diminuer la puissance n’est plus lié au volume de données de la base de données. Les bases de données Hyperscale peuvent être sauvegardées presque instantanément. Vous pouvez aussi mettre à l’échelle une base de données de plusieurs dizaines de téraoctets en quelques minutes. Cette fonctionnalité vous évite d’être limité par votre choix de configuration initial.

Pour plus d’informations sur les tailles de calcul pour le niveau de service Hyperscale, consultez [Caractéristiques du niveau de service](service-tiers-vcore.md#service-tiers).

## <a name="who-should-consider-the-hyperscale-service-tier"></a>À qui est destiné le niveau de service Hyperscale

Le niveau de service Hyperscale est destiné à la plupart des charges de travail métier, car il offre une grande flexibilité et des performances élevées avec des ressources de calcul et de stockage scalables de manière indépendante. Avec la possibilité de mettre à l’échelle automatiquement le stockage jusqu’à 100 To, c’est un excellent choix pour les clients qui :

- Ont des bases de données locales volumineuses et souhaitent moderniser leurs applications en passant au cloud.
- Sont déjà dans le cloud et sont limités par les restrictions de taille maximale de base de données des autres niveaux de service (1-4 To).
- Ont des bases de données plus petites mais ont besoin d’une mise à l’échelle de calcul horizontale et verticale rapide, de performances élevées, d’une sauvegarde instantanée et d’une restauration rapide des bases de données.

Le niveau de service Hyperscale prend en charge un large éventail de charges de travail SQL Server, de l’OLTP pur à l’analytique pure, mais il est optimisé principalement pour les charges de travail OLTP et HTAP (traitement analytique et de transaction hybride).

> [!IMPORTANT]
> Les pools élastiques ne prennent pas en charge le niveau de service Hyperscale.

## <a name="hyperscale-pricing-model"></a>Modèle tarifaire Hyperscale

Le niveau de service Hyperscale est disponible uniquement dans le [modèle vCore](service-tiers-vcore.md). Pour s’aligner sur la nouvelle architecture, le modèle tarifaire est légèrement différent des niveaux de service Usage général ou Critique pour l’entreprise :

- **Calcul** :

  Le prix unitaire du calcul Hyperscale est par réplica. Le prix [Azure Hybrid Benefit](https://azure.microsoft.com/pricing/hybrid-benefit/) est automatiquement appliqué aux réplicas avec échelle lecture. Par défaut, nous créons un réplica principal et un réplica en lecture seule par base de données Hyperscale.  Les utilisateurs peuvent ajuster le nombre total de réplicas, y compris de réplicas principaux de 1 à 5.

- **Stockage** :

  Vous n’avez pas besoin de spécifier la taille maximale des données lors de la configuration d’une base de données Hyperscale. Au niveau Hyperscale, le stockage de votre base de données est facturé en fonction de la répartition réelle. Un espace de stockage compris entre 40 Go et 100 To est automatiquement alloué, par incréments de 10 Go. Si nécessaire, plusieurs fichiers de données peuvent croître en même temps. Une base de données Hyperscale est créée avec une taille de départ de 10 Go et elle commence à croître de 10 Go toutes les 10 minutes, jusqu'à ce qu'elle atteigne la taille de 40 Go.

Pour plus d’informations sur les tarifs Hyperscale, consultez [Tarifs Azure SQL Database](https://azure.microsoft.com/pricing/details/sql-database/single/).

## <a name="distributed-functions-architecture"></a>Architecture des fonctions distribuées

Contrairement aux moteurs de base de données traditionnels qui centralisaient toutes les fonctions de gestion de données dans un même emplacement/processus (même les bases de données distribuées en production aujourd'hui ont plusieurs copies d’un moteur de données monolithique), une base de données Hyperscale sépare le moteur de traitement des requêtes, où la sémantique des différents moteurs de données diffère, des composants qui fournissent un stockage à long terme et une durabilité pour les données. De cette façon, la capacité de stockage peut être facilement mise à l’échelle pour répondre aux besoins (la cible initiale est de 100 To). Comme les réplicas en lecture seule partagent les mêmes composants de stockage, aucune copie de données n’est nécessaire pour mettre en place un nouveau réplica accessible en lecture.

Le diagramme suivant illustre les différents types de nœuds dans une base de données Hyperscale :

![architecture](./media/service-tier-hyperscale/hyperscale-architecture.png)

Une base de données Hyperscale contient les différents types de composants suivants :

### <a name="compute"></a>Calcul

Le nœud de calcul est l’emplacement où réside le moteur relationnel. Il traite le langage, les requêtes et les transactions. Toutes les interactions utilisateur avec une base de données Hyperscale se produisent dans ces nœuds de calcul. Les nœuds de calcul ont des caches SSD (étiquetés RBPEX - Extension du pool de mémoires tampons durable dans le diagramme précédent) afin de réduire le nombre d’allers-retours sur le réseau nécessaires pour récupérer une page de données. Il existe un nœud de calcul principal où sont traitées toutes les charges de travail de lecture-écriture et les transactions. Il existe un ou plusieurs nœuds de calcul secondaires qui agissent comme des serveurs de secours pour le basculement, et comme des nœuds de calcul en lecture seule pour décharger les charges de travail de lecture (si cette fonctionnalité est souhaitée).

Le moteur de base de données s’exécutant sur des nœuds de calcul Hyperscale est le même que dans les autres niveaux de service Azure SQL Database. Lorsque les utilisateurs interagissent avec le moteur de base de données sur des nœuds de calcul Hyperscale, la surface d’exposition prise en charge et le comportement du moteur sont les mêmes que dans d’autres niveaux de service, à l’exception des [limitations connues](#known-limitations).

### <a name="page-server"></a>Serveur de pages

Les serveurs de pages sont des systèmes qui représentent un moteur de stockage scale-out.  Chaque serveur de pages est responsable d’un sous-ensemble de pages dans la base de données.  Nominalement, chaque serveur de pages contrôle entre 128 Go et 1 To de données. Aucune donnée n’est partagée sur plusieurs serveurs de pages (en dehors des réplicas de serveur de page qui sont conservés pour la redondance et la disponibilité). Le travail d’un serveur de pages est de servir à la demande des pages de base de données aux nœuds de calcul et d’actualiser les pages à mesure que les transactions mettent à jour les données. Les serveurs de pages sont actualisés en lisant les enregistrements du journal à partir du service de journalisation. Les serveurs de pages entretiennent aussi les caches SSD de couverture pour améliorer les performances. Le stockage à long terme des pages de données s’effectue dans le Stockage Azure pour une meilleure fiabilité.

### <a name="log-service"></a>Service de journal

Le service de journal accepte les enregistrements de journal du réplica de calcul principal, les conserve dans un cache durable et les transfère au reste des réplicas de calcul (afin qu’ils mettent à jour leurs caches) ainsi qu’aux serveurs de pages appropriés afin que les données y soient mises à jour. De cette façon, tous les changements de données dans le réplica de calcul principal sont propagés par le biais du service de journal à tous les réplicas de calcul secondaires et les serveurs de pages. Pour finir, les enregistrements de journal sont poussés vers un stockage à long terme dans Stockage Azure, qui est un dépôt de stockage pratiquement infini. Avec ce mécanisme, vous n’avez plus besoin de tronquer fréquemment le journal. Le service de journal a aussi une mémoire locale et des caches SSD pour accélérer l’accès aux enregistrements de journal.

### <a name="azure-storage"></a>Stockage Azure

Stockage Azure contient tous les fichiers de données d’une base de données. Les serveurs de pages tiennent à jour les fichiers de données dans Stockage Azure. Ce stockage est utilisé pour la sauvegarde ainsi que pour la réplication entre régions Azure. Les sauvegardes sont implémentées à l’aide de captures instantanées de stockage de fichiers de données. Les opérations de restauration à l’aide d’instantanés sont rapides, quelle que soit la taille des données. Les données peuvent être restaurées à n’importe quel point dans le temps au cours de la période de conservation de sauvegarde de la base de données.

## <a name="backup-and-restore"></a>Sauvegarde et restauration

Comme les sauvegardes sont basées sur des instantanés de fichiers, elles sont quasi instantanées. La séparation du stockage et du calcul permet de pousser l’opération de sauvegarde/restauration vers la couche de stockage afin de réduire la charge de traitement sur le réplica de calcul principal. Par conséquent, la sauvegarde de base de données n’influence pas les performances du nœud de calcul principal. De même, la restauration à un instant dans le passé est effectuée en rétablissant les instantanés de fichier et n’a donc pas la taille d’une opération de données. La restauration d’une base de données Hyperscale dans la même région Azure est une opération à temps constant, et même les bases de données de plusieurs téraoctets peuvent être restaurées en quelques minutes au lieu de plusieurs heures ou jours. La création de nouvelles bases de données en restaurant une sauvegarde existante tire également parti de cette fonctionnalité : la création de copies de base de données à des fins de développement ou de test, notamment des bases de données de plusieurs téraoctets, est réalisable en minutes.

Pour la géorestauration de bases de données Hyperscale, consultez [Restauration d’une base de données Hyperscale dans une autre région](#restoring-a-hyperscale-database-to-a-different-region).

## <a name="scale-and-performance-advantages"></a>Avantages de scalabilité et de performances

Avec la possibilité d’ajouter ou de supprimer rapidement des nœuds de calcul en lecture seule, l’architecture Hyperscale apporte des fonctionnalités d’échelle horizontale en lecture significatives et peut aussi libérer le nœud de calcul principal pour qu’il traite davantage de demandes d’écriture. Par ailleurs, les nœuds de calcul peuvent être mis à l’échelle (augmentation ou diminution) rapidement grâce à la fonctionnalité de stockage partagé propre à l’architecture Hyperscale.

## <a name="create-a-hyperscale-database"></a>Créer une base de données Hyperscale

Vous pouvez créer une base de données Hyperscale à l’aide du [portail Azure](https://portal.azure.com), de [T-SQL](/sql/t-sql/statements/create-database-transact-sql), de [PowerShell](/powershell/module/azurerm.sql/new-azurermsqldatabase) ou de l’[interface CLI](/cli/azure/sql/db#az-sql-db-create). Les bases de données Hyperscale sont uniquement disponibles avec le [modèle d'achat vCore](service-tiers-vcore.md).

La commande T-SQL suivante crée une base de données Hyperscale. Vous devez spécifier l’édition et l’objectif du service dans l’instruction `CREATE DATABASE`. Pour obtenir la liste des objectifs de service valides, consultez les [limites de ressources](./resource-limits-vcore-single-databases.md#hyperscale---provisioned-compute---gen4).

```sql
-- Create a Hyperscale Database
CREATE DATABASE [HyperscaleDB1] (EDITION = 'Hyperscale', SERVICE_OBJECTIVE = 'HS_Gen5_4');
GO
```

Cela a pour effet de créer une base de données Hyperscale sur du matériel Gen5 avec quatre cœurs.

## <a name="upgrade-existing-database-to-hyperscale"></a>Mettre à niveau une base de données existante vers le niveau Hyperscale

Vous pouvez déplacer vos bases de données Azure SQL Database existantes vers le niveau Hyperscale à l'aide du [portail Azure](https://portal.azure.com), de [T-SQL](/sql/t-sql/statements/alter-database-transact-sql), de [PowerShell](/powershell/module/azurerm.sql/set-azurermsqldatabase) ou de l'[interface CLI](/cli/azure/sql/db#az-sql-db-update). À ce stade, il s’agit d’une migration à sens unique. Vous ne pouvez pas déplacer des bases de données d’Hyperscale vers un autre niveau de service, sauf en exportant et en important des données. Pour les preuves de concept, nous vous recommandons d’effectuer une copie de vos bases de données de production et de migrer la copie vers Hyperscale. La migration d'une base de données Azure SQL Database existante vers le niveau Hyperscale est une opération de taille de données.

La commande T-SQL suivante déplace une base de données vers le niveau de service Hyperscale. Vous devez spécifier l’édition et l’objectif du service dans l’instruction `ALTER DATABASE`.

```sql
-- Alter a database to make it a Hyperscale Database
ALTER DATABASE [DB2] MODIFY (EDITION = 'Hyperscale', SERVICE_OBJECTIVE = 'HS_Gen5_4');
GO
```

## <a name="connect-to-a-read-scale-replica-of-a-hyperscale-database"></a>Se connecter à un réplica avec échelle lecture d’une base de données Hyperscale

Dans les bases de données Hyperscale, l'argument `ApplicationIntent` de la chaîne de connexion fournie par le client détermine si la connexion est routée vers le réplica en écriture ou vers un réplica secondaire en lecture seule. Si l’option `ApplicationIntent` est définie sur `READONLY` et que la base de données ne dispose pas de réplica secondaire, la connexion est routée vers le réplica principal et la valeur par défaut est le comportement `ReadWrite`.

```cmd
-- Connection string with application intent
Server=tcp:<myserver>.database.windows.net;Database=<mydatabase>;ApplicationIntent=ReadOnly;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;
```

Les réplicas secondaires Hyperscale sont tous identiques ; ils utilisent le même objectif de niveau de service que le réplica principal. Si plusieurs réplicas secondaires sont présents, la charge de travail est répartie sur l’ensemble des réplicas secondaires disponibles. Chaque réplica secondaire est mis à jour de manière indépendante. Ainsi, différents réplicas peuvent avoir une latence de données différente par rapport au réplica principal.

## <a name="database-high-availability-in-hyperscale"></a>Haute disponibilité de la base de données dans Hyperscale

Comme pour tous les autres niveaux de service, Hyperscale garantit la durabilité des données pour les transactions validées, quelle que soit la disponibilité des réplicas de calcul. L’étendue des temps d’arrêt dus au fait que le réplica principal devient indisponible dépend du type de basculement (planifié ou non planifié) et de la présence d’au moins un réplica secondaire. Dans un basculement planifié (par exemple un événement de maintenance), le système crée le nouveau réplica principal avant de lancer un basculement, ou utilise un réplica secondaire existant comme cible de basculement. Dans un basculement non planifié (par exemple une défaillance matérielle sur le réplica principal), le système utilise un réplica secondaire (s’il en existe un) comme cible de basculement, ou crée un nouveau réplica principal à partir du pool de capacité de calcul disponible. Dans ce dernier cas, la durée d’inactivité est plus longue en raison des étapes supplémentaires nécessaires pour créer le réplica principal.

Pour plus d’informations sur les contrats SLA Hyperscale, consultez [SLA pour Azure SQL Database](https://azure.microsoft.com/support/legal/sla/sql-database/).

## <a name="disaster-recovery-for-hyperscale-databases"></a>Récupération d’urgence pour les bases de données Hyperscale

### <a name="restoring-a-hyperscale-database-to-a-different-region"></a>Restauration d’une base de données Hyperscale dans une autre région

Si vous avec besoin de restaurer une base de données Hyperscale dans Azure SQL Database dans une région autre que celle dans laquelle elle est actuellement hébergée, à des fins de récupération d’urgence, d’exploration, de relocalisation ou pour tout autre motif, la méthode principale consiste à opérer une géo-restauration de la base de données. La procédure à suivre est exactement la même que celle utilisée pour restaurer une base de données dans SQL Database dans une autre région :

1. Créez un [serveur](logical-servers.md) dans la région cible si vous n'y disposez pas encore d'un serveur approprié.  Ce serveur doit appartenir au même abonnement que le serveur (source) d’origine.
2. Suivez les instructions de la rubrique [Géo-restauration](./recovery-using-backups.md#geo-restore) sur la page consacrée à la restauration d’une base de données dans Azure SQL Database à partir de sauvegardes automatiques.

> [!NOTE]
> Etant donné que la source et la cible se trouvent dans des régions distinctes, la base de données ne peut pas partager de stockage de captures instantanées avec la base de données source, comme c’est le cas dans le cadre de restaurations non géographiques qui s’opèrent très rapidement. Dans le cas d’une géo-restauration d’une base de données Hyperscale, il s’agit d’une opération tributaire de la taille des données, même si la cible se trouve dans la région associée du stockage géo-répliqué.  Cela signifie que la géo-restauration prend un temps proportionnel à la taille de la base de données restaurée.  Si la cible se trouve dans la région associée, la copie est effectuée au sein d'une région, ce qui est beaucoup plus rapide qu'une copie entre régions, mais il s'agit toujours d'une opération à l'échelle des données.

## <a name="available-regions"></a><a name=regions></a>Régions disponibles

Le niveau Hyperscale Azure SQL Database est disponible dans toutes les régions, mais il est activé par défaut dans les régions suivantes.
Si vous souhaitez créer une base de données Hyperscale dans une région non répertoriée comme prise en charge, vous pouvez envoyer une demande d’intégration via le portail Azure. Pour obtenir des instructions, consultez [Demander des augmentations de quota pour Azure SQL Database](quota-increase-request.md). Quand vous soumettez votre demande, suivez les instructions ci-après :

- Utilisez le type de quota [Accès à une région](quota-increase-request.md#region) SQL Database.
- Dans les détails sous forme de texte, ajoutez la référence SKU de calcul/le nombre total de cœurs, notamment les réplicas lisibles.
- Spécifiez également la taille estimée en To.

Régions prises en charge :
- Australie Est
- Sud-Australie Est
- Centre de l’Australie
- Brésil Sud
- Centre du Canada
- USA Centre
- Chine orientale 2
- Chine Nord 2
- Asie Est
- USA Est
- USA Est 2
- France Centre
- Allemagne Centre-Ouest
- Japon Est
- OuJapon Est
- Centre de la Corée
- Corée du Sud
- Centre-Nord des États-Unis
- Europe Nord
- Norvège Est
- Norvège Ouest
- Afrique du Sud Nord
- États-Unis - partie centrale méridionale
- Asie Sud-Est
- Suisse Ouest
- Sud du Royaume-Uni
- Ouest du Royaume-Uni
- Centre des États-Unis – US DoD
- Est des États-Unis – US DoD
- Us Govt Arizona
- US Gov Texas
- Centre-USA Ouest
- Europe Ouest
- USA Ouest
- USA Ouest 2

## <a name="known-limitations"></a>Limitations connues

Voici les limitations actuelles du niveau de service Hyperscale depuis la disponibilité générale.  Nous travaillons activement à la suppression d’un maximum de ces limitations.

| Problème | Description |
| :---- | :--------- |
| Le volet Gérer les sauvegardes d’un serveur n’affiche pas les bases de données Hyperscale. Elles seront filtrés à partir de la vue.  | Hyperscale dispose d'une méthode distincte pour la gestion des sauvegardes. Par conséquent, les paramètres Conservation à long terme et Conservation des sauvegardes dans le temps ne s'appliquent pas. En conséquence, les bases de données Hyperscale n’apparaissent pas dans le volet Gérer les sauvegardes.<br><br>Pour les bases de données migrées vers Hyperscale à partir d’autres niveaux de service d’Azure SQL Database, les sauvegardes avant migration sont conservées pendant toute la période de [rétention des sauvegardes](automated-backups-overview.md#backup-retention) de la base de données source. Ces sauvegardes de fichiers peuvent servir à [restaurer](recovery-using-backups.md#programmatic-recovery-using-automated-backups) la base de données source à un état antérieur à la migration.|
| Restauration dans le temps | Une base de données non Hyperscale ne peut pas être restaurée en tant que base de données Hyperscale, et une base de données Hyperscale ne peut pas être restaurée en tant que base de données non Hyperscale. Pour une base de données non Hyperscale qui a été migrée vers Hyperscale en changeant son niveau de service, la restauration à un moment donné avant la migration et pendant la période de rétention de la sauvegarde de la base de données est possible [par programmation](recovery-using-backups.md#programmatic-recovery-using-automated-backups). La base de données restaurée sera non Hyperscale. |
| Si une base de données contient un ou plusieurs fichiers de données d’une taille supérieure à 1 To, la migration échoue | Dans certains cas, il peut être possible de contourner ce problème en réduisant la taille des fichiers volumineux à une valeur inférieure à 1 To. Si vous migrez une base de données qui est utilisée pendant le processus de migration, vérifiez qu’aucun fichier ne dépasse 1 To. Utilisez la requête suivante pour déterminer la taille des fichiers de base de données. `SELECT *, name AS file_name, size * 8. / 1024 / 1024 AS file_size_GB FROM sys.database_files WHERE type_desc = 'ROWS'`;|
| Instance managée SQL | L'option Azure SQL Managed Instance n'est actuellement pas prise en charge avec les bases de données Hyperscale. |
| Pools élastiques |  Les pools élastiques ne sont actuellement pas pris en charge avec Hyperscale.|
| La migration vers Hyperscale est actuellement une opération unidirectionnelle | Une fois qu’une base de données est migrée vers Hyperscale, elle ne peut pas être migrée directement vers un niveau de service non Hyperscale. À l’heure actuelle, la seule façon de migrer une base de données d’Hyperscale vers non-Hyperscale consiste à exporter/importer à l’aide d’un fichier bacpac ou d’autres technologies de déplacement de données (copie en bloc, Azure Data Factory, Azure Databricks, SSIS, etc.) L’exportation et l’importation bacpac à partir du portail Azure, à partir de PowerShell à l’aide des cmdlets [New-AzSqlDatabaseExport](/powershell/module/az.sql/new-azsqldatabaseexport) ou [New-AzSqlDatabaseImport](/powershell/module/az.sql/new-azsqldatabaseimport), à partir d’Azure CLI à l’aide des commandes [az sql db export](/cli/azure/sql/db#az-sql-db-export) et [az sql db import](/cli/azure/sql/db#az-sql-db-import), et d’une [API REST](/rest/api/sql/databases%20-%20import%20export), ne sont pas prises en charge. L’exportation et l’importation bacpac pour des bases de données Hyperscale de plus petite taille (jusqu’à 200 Go) est prise en charge à l’aide de SSMS et de [SqlPackage](/sql/tools/sqlpackage) versions 18.4 et ultérieures. Pour des bases de données plus volumineuses, l’exportation et l’importation bacpac peuvent prendre beaucoup de temps et échouer pour différentes raisons.|
| Migration de bases de données avec des objets OLTP en mémoire | Hyperscale prend en charge une partie des objets OLTP en mémoire, notamment les types de tables à mémoire optimisée, les variables de table et les modules compilés en mode natif. Toutefois, lorsqu’un des types d’objets OLTP en mémoire est présent dans la base de données en cours de migration, la migration des niveaux de service Premium et Critique pour l’entreprise vers Hyperscale n’est pas prise en charge. Pour migrer une telle base de données vers Hyperscale, tous les objets OLTP en mémoire et leurs dépendances doivent être supprimés. Une fois la base de données migrée, ces objets peuvent être recréés. Les tables à mémoire optimisée durables et non durables ne sont actuellement pas prises en charge dans Hyperscale et doivent être recréées en tant que tables de disques.|
| Géo-réplication  | Vous ne pouvez pas encore configurer la géo-réplication pour Azure SQL Database Hyperscale. |
| Copie de base de données | La copie de base de données sur Hyperscale est désormais en préversion publique. |
| Fonctionnalités de base de données intelligente | À l'exception de l'option « Forcer le plan », aucune option de réglage automatique n'est encore prise en charge sur Hyperscale : les options peuvent sembler activées, mais aucune recommandation ou action ne sera effectuée. |
| Query Performance Insight | Query Performance Insights n’est actuellement pas pris en charge pour les bases de données Hyperscale. |
| Réduire la base de données | DBCC SHRINKDATABASE ou DBCC SHRINKFILE n’est pas pris en charge actuellement pour les bases de données Hyperscale. |
| Vérification de l’intégrité de la base de données | DBCC CHECKDB n’est pas pris en charge actuellement pour les bases de données Hyperscale. DBCC CHECKFILEGROUP et DBCC CHECKTABLE peuvent être utilisés comme solution de contournement. Pour plus d’informations sur la gestion de l’intégrité des données dans Azure SQL Database, consultez [Intégrité des données dans Azure SQL Database](https://azure.microsoft.com/blog/data-integrity-in-azure-sql-database/). |

## <a name="next-steps"></a>Étapes suivantes

- Pour consultez un forum aux questions sur Hyperscale, consultez [Questions fréquentes (FAQ) sur Hyperscale](service-tier-hyperscale-frequently-asked-questions-faq.md).
- Pour plus d’informations sur les niveaux de service, consultez [Niveaux de service](purchasing-models.md)
- Pour plus d'informations sur les limites au niveau du serveur et de l'abonnement, consultez [Vue d'ensemble des limites de ressources sur un serveur](resource-limits-logical-server.md).
- Pour connaître les limites du modèle d’achat pour une base de données unique, consultez [Limites du modèle d’achat vCore Azure SQL Database pour une base de données unique](resource-limits-vcore-single-databases.md).
- Pour consulter la liste des fonctionnalités et les comparer, consultez [Fonctionnalités SQL communes](features-comparison.md).
