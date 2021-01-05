---
title: Bonnes pratiques pour Azure SQL Data Sync
description: Découvrez les bonnes pratiques pour configurer et exécuter Azure SQL Data Sync.
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 12/20/2018
ms.openlocfilehash: 59e28e4a3d630aac0954802e8777058c00261006
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92791441"
---
# <a name="best-practices-for-azure-sql-data-sync"></a>Bonnes pratiques pour Azure SQL Data Sync 

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Cet article décrit les bonnes pratiques relatives à Azure SQL Data Sync.

Pour obtenir une vue d’ensemble de SQL Data Sync, consultez [Synchroniser des données entre plusieurs bases de données locales et cloud avec Azure SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md).

> [!IMPORTANT]
> Azure SQL Data Sync **ne prend pas** en charge Azure SQL Managed Instance.

## <a name="security-and-reliability"></a><a name="security-and-reliability"></a> Sécurité et fiabilité

### <a name="client-agent"></a>Agent client

-   Installez l’agent client à l’aide du compte d'utilisateur doté des moindres privilèges ayant accès aux services réseau.  
-   Installez l’agent client sur un ordinateur qui n’est pas l’ordinateur SQL Server.  
-   N’inscrivez pas une base de données sur site auprès de plusieurs agents,    
    -   même si vous synchronisez différentes tables pour différents groupes de synchronisation.  
    -   L’inscription d’une base de données sur site auprès de plusieurs agents clients risque de poser des problèmes lors de la suppression de l’un des groupes de synchronisation.

### <a name="database-accounts-with-least-required-privileges"></a>Comptes de base de données avec le moins de privilèges nécessaires

-   **Pour la configuration de la synchronisation** . Créer/Modifier la table ; Modifier la base de données ; Créer une procédure ; Sélectionner / Modifier le schéma ; Créer un type défini par l’utilisateur.

-   **Pour la synchronisation continue** . Sélectionner / Insérer / Mettre à jour / Supprimer sur les tables sélectionnées pour la synchronisation et sur les métadonnées de synchronisation et les tables de suivi ; autorisation Exécuter sur les procédures stockées créées par le service ; autorisation Exécuter sur les types de tables définis par l’utilisateur.

-   **Pour annuler le provisionnement** . Modifier sur les tables qui font partie de la synchronisation ; Sélectionner / Supprimer sur les tables de métadonnées de synchronisation, Contrôler sur les tables de suivi de synchronisation, les procédures stockées et les types définis par l’utilisateur.

Azure SQL Database ne prend en charge qu'un seul ensemble d'informations d'identification. Pour accomplir ces tâches avec cette contrainte, prenez en compte les options suivantes :

-   Changez les informations d’identification pour différentes phases (par exemple, *credentials1* pour la configuration et *credentials2* pour les opérations continues).  
-   Modifiez l’autorisation des informations d’identification (autrement dit, modifiez l’autorisation après avoir configuré la synchronisation).

## <a name="setup"></a>Programme d’installation

### <a name="database-considerations-and-constraints"></a><a name="database-considerations-and-constraints"></a> Considérations et contraintes relatives aux bases de données

#### <a name="database-size"></a>Taille de la base de données

Quand vous créez une base de données, définissez la taille maximale afin qu’elle soit toujours supérieure à la base de données que vous déployez. Si vous n’affectez pas à la taille maximale une valeur supérieure à la base de données déployée, la synchronisation échoue. Bien que SQL Data Sync n’offre pas de croissance automatique, vous pouvez exécuter la commande `ALTER DATABASE` pour augmenter la taille de la base de données après sa création. Veillez à rester dans les limites de taille de la base de données.

> [!IMPORTANT]
> SQL Data Sync stocke des métadonnées supplémentaires avec chaque base de données. Veillez à prendre en compte ces métadonnées quand vous calculez l’espace nécessaire. La quantité de surcharge ajoutée est régie par la largeur des tables (par exemple, des tables étroites nécessitent davantage de surcharge) et la quantité de trafic.

### <a name="table-considerations-and-constraints"></a><a name="table-considerations-and-constraints"></a> Considérations et contraintes relatives aux tables

#### <a name="selecting-tables"></a>Sélection de tables

Vous n'avez pas besoin d'inclure toutes les tables figurant dans une base de données d'un groupe de synchronisation. Les tables que vous incluez dans un groupe de synchronisation affectent l'efficacité et les coûts. Incluez les tables (et les tables dont elles dépendent) dans un groupe de synchronisation uniquement si les besoins de l'entreprise l'exigent.

#### <a name="primary-keys"></a>Clés primaires

Chaque table dans un groupe de synchronisation doit avoir une clé primaire. SQL Data Sync ne peut pas synchroniser une table qui n’a pas de clé primaire.

Avant d’utiliser SQL Data Sync en production, testez les performances de synchronisation initiales et actuelles.

#### <a name="empty-tables-provide-the-best-performance"></a>Les tables vides optimisent les performances

Les tables vides offrent les meilleures performances au moment de l’initialisation. Si la table cible est vide, Data Sync utilise l’insertion en bloc pour charger les données. Dans le cas contraire, Data Sync effectue une comparaison et une insertion ligne par ligne afin de rechercher les conflits. Toutefois, si les performances ne sont pas un critère important, vous pouvez configurer la synchronisation entre des tables qui contiennent déjà des données.

### <a name="provisioning-destination-databases"></a><a name="provisioning-destination-databases"></a> Provisionnement des bases de données de destination

SQL Data Sync fournit un provisionnement automatique de base des bases de données.

Cette section présente les limitations du provisionnement dans SQL Data Sync.

#### <a name="autoprovisioning-limitations"></a>Limitations du provisionnement automatique

Voici les limitations du provisionnement automatique de SQL Data Sync :

-   Sélectionnez uniquement les colonnes créées dans la table de destination. Les colonnes qui ne font pas partie du groupe de synchronisation ne sont pas provisionnées dans les tables de destination.
-   Des index sont créés uniquement pour les colonnes sélectionnées. Si l’index de la table source a des colonnes qui ne font pas partie du groupe de synchronisation, ces index ne sont pas provisionnés dans les tables de destination.  
-   Les index sur des colonnes de type XML ne sont pas provisionnés.  
-   Les contraintes CHECK ne sont pas provisionnées.  
-   Les déclencheurs existants sur les tables sources ne sont pas provisionnés.  
-   Les vues et procédures stockées ne sont pas créées sur la base de données de destination.
-   Les actions ON UPDATE CASCADE and ON DELETE CASCADE sur les contraintes de clé étrangère ne sont pas recréées dans les tables de destination.
-   Si vous avez des colonnes de type décimal ou numérique avec une précision supérieure à 28, SQL Data Sync peut être l’objet d’un dépassement de capacité en matière de conversion au cours de la synchronisation. Nous vous recommandons de limiter la précision des colonnes de type décimal ou numérique à 28 ou moins.

#### <a name="recommendations"></a>Recommandations

-   Utilisez la fonctionnalité de provisionnement automatique de SQL Data Sync uniquement quand vous testez le service.  
-   Pour la production, provisionnez le schéma de base de données.

### <a name="where-to-locate-the-hub-database"></a><a name="locate-hub"></a> Emplacement de la base de données Hub

#### <a name="enterprise-to-cloud-scenario"></a>Scénario entreprise-à-cloud

Pour réduire la latence, conservez la base de données Hub proche de la plus grande concentration du trafic de base de données du groupe de synchronisation.

#### <a name="cloud-to-cloud-scenario"></a>Scénario cloud-à-cloud

-   Quand toutes les bases de données d’un groupe de synchronisation sont dans un même centre de données, le hub doit se trouver dans le même centre de données. Cette configuration réduit la latence et le coût de transfert de données entre les centres de données.
-   Quand les bases de données d’un groupe de synchronisation se trouvent dans plusieurs centres de données, le hub doit se trouver dans le même centre de données que la majorité des bases de données et du trafic de base de données.

#### <a name="mixed-scenarios"></a>Scénarios mixtes

Appliquez les instructions précédentes aux configurations de groupe de synchronisation complexes, notamment celles qui combinent des scénarios entreprise-à-cloud et cloud-à-cloud.

## <a name="sync"></a>Synchronisation

### <a name="avoid-slow-and-costly-initial-sync"></a><a name="avoid-a-slow-and-costly-initial-synchronization"></a> Éviter une synchronisation initiale lente et coûteuse

Cette section aborde la synchronisation initiale d'un groupe de synchronisation. Découvrez comment éviter qu'une synchronisation initiale prenne plus de temps et soit plus coûteuse que nécessaire.

#### <a name="how-initial-sync-works"></a>Fonctionnement de la synchronisation initiale

Quand vous créez un groupe de synchronisation, commencez avec des données d’une seule base de données. Si vous avez des données dans plusieurs bases de données, SQL Data Sync traite chaque ligne comme un conflit à résoudre. Cette résolution de conflit entraîne une lente synchronisation initiale. Si vous avez des données dans plusieurs bases de données, la synchronisation initiale peut prendre plusieurs jours voire plusieurs mois, en fonction de la taille de la base de données.

Si les bases de données se trouvent dans des centres de données différents, chaque ligne doit se déplacer entre ces différents centres de données. Cela augmente le coût d'une synchronisation initiale.

#### <a name="recommendation"></a>Recommandation

Si possible, commencez avec les données d’une seule des bases de données du groupe de synchronisation.

### <a name="design-to-avoid-sync-loops"></a><a name="design-to-avoid-synchronization-loops"></a> Conception permettant d'éviter des boucles de synchronisation

Une boucle de synchronisation se produit lorsqu'un groupe de synchronisation contient des références circulaires. Dans ce scénario, chaque modification dans une base de données est répliquée sans fin et de façon circulaire dans toutes les bases de données du groupe de synchronisation.   

Veillez à éviter les boucles de synchronisation, car elles entraînent une dégradation des performances et peuvent considérablement augmenter les coûts.

### <a name="changes-that-fail-to-propagate"></a><a name="handling-changes-that-fail-to-propagate"></a> Modifications dont la propagation échoue

#### <a name="reasons-that-changes-fail-to-propagate"></a>Raisons pour lesquelles la propagation des modifications échoue

La propagation des modifications peut échouer pour l'une des raisons suivantes :

-   Incompatibilité de schéma/type de données
-   Insertion de valeurs null dans des colonnes non nullables
-   Violation de contraintes de clé étrangère

#### <a name="what-happens-when-changes-fail-to-propagate"></a>Que se passe-t-il quand la propagation des modifications échoue ?

-   Le groupe de synchronisation affiche un état d’ **avertissement** .
-   Les détails sont affichés dans la visionneuse du journal de l’interface utilisateur du portail.
-   Si le problème n’est pas résolu pendant 45 jours, la base de données devient obsolète.

> [!NOTE]
> Ces modifications ne sont jamais propagées. Dans ce scénario, la seule façon de récupérer consiste à recréer le groupe de synchronisation.

#### <a name="recommendation"></a>Recommandation

Surveillez l’intégrité du groupe de synchronisation et de la base de données régulièrement par le biais de l’interface du portail et du journal.


## <a name="maintenance"></a>Maintenance

### <a name="avoid-out-of-date-databases-and-sync-groups"></a><a name="avoid-out-of-date-databases-and-sync-groups"></a> Éviter les bases de données et les groupes de synchronisation obsolètes

Un groupe de synchronisation ou une base de données d’un groupe de synchronisation peut devenir obsolète. Quand l’état d’un groupe de synchronisation est **obsolète** , il cesse de fonctionner. Quand l’état d’une base de données est **obsolète** , il y a un risque de perte de données. Il est préférable d'éviter ce scénario plutôt que de tenter de le récupérer.

#### <a name="avoid-out-of-date-databases"></a>Éviter les bases de données obsolètes

Une base de données bascule à l’état **obsolète** quand elle est hors connexion depuis 45 jours ou plus. Pour éviter qu’une base de données ne bascule à l’état **obsolète** , veillez à ce qu’aucune des bases de données ne soit hors connexion pendant 45 jours ou plus.

#### <a name="avoid-out-of-date-sync-groups"></a>Éviter les groupes de synchronisation obsolètes

Un groupe de synchronisation bascule à l’état **obsolète** en cas d’échec de propagation d’une modification dans le groupe de synchronisation vers le reste du groupe de synchronisation pendant 45 jours ou plus. Pour éviter qu’un groupe de synchronisation ne bascule à l’état **obsolète** , vérifiez régulièrement le journal d’historique du groupe de synchronisation. Assurez-vous que tous les conflits sont résolus et que les modifications sont propagées dans les bases de données du groupe de synchronisation.

Un groupe de synchronisation peut échouer à appliquer une modification pour l'une des raisons suivantes :

-   Incompatibilité de schéma entre des tables
-   Incompatibilité de données entre des tables
-   Insertion d’une ligne avec une valeur null dans une colonne qui n’autorise pas les valeurs null
-   Mise à jour d’une ligne avec une valeur qui enfreint une contrainte de clé étrangère

Pour empêcher qu’un groupe de synchronisation ne devienne obsolète :

-   Mettez à jour le schéma pour autoriser les valeurs contenues dans les lignes ayant provoqué l’échec.
-   Mettez à jour les valeurs de clé étrangères pour inclure les valeurs contenues dans les lignes ayant provoqué l’échec.
-   Mettez à jour les valeurs de données dans la ligne ayant provoqué l’échec pour qu’elles soient compatibles avec le schéma ou les clés étrangères dans la base de données cible.

### <a name="avoid-deprovisioning-issues"></a><a name="avoid-deprovisioning-issues"></a> Éviter les problèmes d’annulation du provisionnement

Dans certaines circonstances, la désinscription d’une base de données auprès d’un agent client peut entraîner l’échec de la synchronisation.

#### <a name="scenario"></a>Scénario

1. Le groupe de synchronisation A a été créé avec une instance de SQL Database et une base de données SQL Server, qui est associée à l’agent local 1.
2. La même base de données locale est inscrite auprès de l’agent local 2 (cet agent n’est associé à aucun groupe de synchronisation).
3. Si vous annulez l’inscription de la base de données sur site auprès de l’agent local 2, cela supprime les tables de suivi et métadonnées pour le groupe de synchronisation A pour la base de données sur site.
4. Les opérations du groupe de synchronisation A échouent avec cette erreur : « L’opération en cours a échoué, car la base de données n’est pas provisionnée pour la synchronisation ou vous n’avez pas les autorisations nécessaires sur les tables de configuration de synchronisation. »

#### <a name="solution"></a>Solution

Pour éviter ce scénario, n'inscrivez pas une base de données auprès de plusieurs agents.

Pour résoudre ce problème :

1. Supprimez la base de données de chaque groupe de synchronisation auquel elle appartient.  
2. Rajoutez la base de données dans chaque groupe de synchronisation où vous venez de la retirer.  
3. Déployez chaque groupe de synchronisation concerné (cette action provisionne la base de données).  

### <a name="modifying-a-sync-group"></a><a name="modifying-your-sync-group"></a> Modification d’un groupe de synchronisation

N’essayez pas de supprimer une base de données d’un groupe de synchronisation et de modifier ensuite le groupe de synchronisation sans avoir déployé au préalable l’une des modifications.

Supprimez d'abord une base de données d’un groupe de synchronisation. Ensuite, déployez la modification et attendez que l’annulation du provisionnement soit terminée. Une fois l'annulation du provisionnement terminée, vous pouvez modifier le groupe de synchronisation et déployer les modifications.

Si vous tentez de supprimer une base de données et de modifier un groupe de synchronisation sans avoir déployé au préalable l’une des modifications, l’une ou l’autre opération échoue. L’interface du portail peut basculer dans un état incohérent. Dans ce cas, actualisez la page pour restaurer l’état correct.

### <a name="avoid-schema-refresh-timeout"></a>Éviter le délai d’expiration de l’actualisation du schéma

Si vous avez un schéma complexe à synchroniser, vous risquez de rencontrer une erreur du type « l’opération a expiré » pendant l’actualisation du schéma si la base de données de métadonnées de synchronisation a une référence SKU inférieure (exemple : De base). 

#### <a name="solution"></a>Solution

Pour atténuer ce problème, procédez au scale-up de votre base de données de métadonnées de synchronisation de façon à obtenir une référence SKU supérieure, par exemple S3. 

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur SQL Data Sync, consultez :

-   Vue d’ensemble - [Synchroniser des données entre plusieurs bases de données cloud et locales avec Azure SQL Data Sync](sql-data-sync-data-sql-server-sql-database.md)
-   Configurer SQL Data Sync
    - Sur le portail - [Tutoriel : Configurer SQL Data Sync pour synchroniser les données entre Azure SQL Database et SQL Server](sql-data-sync-sql-server-configure.md)
    - Avec PowerShell
        -  [Utiliser PowerShell pour synchroniser des données entre plusieurs bases de données dans Azure SQL Database](scripts/sql-data-sync-sync-data-between-sql-databases.md)
        -  [Utiliser PowerShell pour synchroniser des données entre une base de données de SQL Database et une base de données d’une instance SQL Server](scripts/sql-data-sync-sync-data-between-azure-onprem.md)
-   Agent de synchronisation des données - [Agent de synchronisation des données pour Azure SQL Data Sync](sql-data-sync-agent-overview.md)
-   Supervision – [Superviser SQL Data Sync avec les journaux d’activité Azure Monitor](./monitor-tune-overview.md)
-   Résolution des problèmes - [Résoudre les problèmes liés à Azure SQL Data Sync](sql-data-sync-troubleshoot.md)
-   Mettre à jour le schéma de synchronisation
    -   Avec Transact-SQL - [Automatiser la réplication des modifications de schéma dans Azure SQL Data Sync](sql-data-sync-update-sync-schema.md)
    -   Avec PowerShell - [Utiliser PowerShell pour mettre à jour le schéma de synchronisation dans un groupe de synchronisation existant](scripts/update-sync-schema-in-sync-group.md)

Pour plus d’informations sur SQL Database, consultez :

-   [Vue d’ensemble de SQL Database](sql-database-paas-overview.md)
-   [Gestion du cycle de vie des bases de données](/previous-versions/sql/sql-server-guides/jj907294(v=sql.110))