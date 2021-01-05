---
title: Requêtes de génération d’états ad hoc sur plusieurs bases de données
description: Exécutez des requêtes de génération d’états ad hoc sur plusieurs bases de données Azure SQL dans un exemple d’application multilocataire.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 10/30/2018
ms.openlocfilehash: 800592b7a8b263fea2883fdd3e030f78f72647dd
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96459923"
---
# <a name="run-ad-hoc-analytics-queries-across-multiple-databases-azure-sql-database"></a>Exécuter des requêtes d’analytique ad hoc sur plusieurs bases de données (Azure SQL Database)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Dans ce didacticiel, vous allez exécuter des requêtes distribuées sur l’ensemble des bases de données client pour permettre un rapport ad hoc. Ces requêtes peuvent extraire des analyses enfouies dans les données opérationnelles quotidiennes de l’application SaaS Wingtip Tickets. Pour effectuer ces extractions, vous déployez une base de données analytique supplémentaire sur le serveur de catalogue et utilisez une requête élastique pour activer les requêtes distribuées.


Ce didacticiel vous apprend à effectuer les opérations suivantes :

> [!div class="checklist"]
> 
> * Guide de déploiement d’une base de données de création de rapports ad hoc
> * Exécuter des requêtes distribuées sur toutes les bases de données client


Pour suivre ce didacticiel, vérifiez que les prérequis suivants sont remplis :

* L’application de base de données multilocataire SaaS Wingtip Tickets est déployée. Pour procéder à un déploiement en moins de cinq minutes, consultez [Déployer et explorer l’application de base de données multi-locataire SaaS Wingtip Tickets](saas-multitenantdb-get-started-deploy.md)
* Azure PowerShell est installé. Pour plus d’informations, voir [Bien démarrer avec Azure PowerShell](/powershell/azure/get-started-azureps).
* SQL Server Management Studio (SSMS) est installé. Pour télécharger et installer SSMS, consultez la rubrique [Téléchargement de SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms).


## <a name="ad-hoc-reporting-pattern"></a>Modèle de création de rapports ad hoc

![modèle de création de rapports ad hoc](./media/saas-multitenantdb-adhoc-reporting/adhocreportingpattern_shardedmultitenantDB.png)

Les applications SaaS peuvent analyser la vaste quantité de données client stockées de façon centrale dans le cloud. L’analyse offre des informations sur le fonctionnement et l’utilisation de votre application. Ces analyses peuvent guider le développement des fonctionnalités, les améliorations de convivialité et les autres investissements dans vos applications et services.

L’accès à ces données dans une base de données mutualisée est facile, mais pas si simple lors d’une distribution à grande échelle sur des milliers de bases de données. Une approche consiste à utiliser une [requête élastique](elastic-query-overview.md), permettant d’interroger un ensemble distribué de bases de données avec un schéma commun. Ces bases de données peuvent être distribuées sur différents abonnements et groupes de ressources. Toutefois, un même compte de connexion doit avoir accès à l’extraction des données de toutes les bases de données. Les requêtes élastiques utilisent une seule base de données *principale* dans laquelle sont définies des tables externes qui reflètent les tables ou les vues dans les bases de données distribuées (client). Les requêtes envoyées à cette base de données principale sont compilées pour produire un plan de requête distribué, avec des parties de la requête transmises aux bases de données client en fonction des besoins. Les requêtes élastiques utilisent la carte de partitions dans la base de données de catalogue pour déterminer l’emplacement de toutes les bases de données client. La configuration et les requêtes sont simples grâce à l’utilisation de [Transact-SQL](/sql/t-sql/language-reference) standard, et les requêtes ad hoc sont compatibles avec des outils tels que Power BI et Excel.

En distribuant les requêtes sur toutes les bases de données client, les requêtes élastiques permettent d’obtenir immédiatement des informations pour les transformer en données de production actives. Toutefois, étant donné qu’une requête élastique peut extraire des données provenant potentiellement de nombreuses bases de données, la latence de requête peut parfois être supérieure à celle observée pour des requêtes équivalentes soumises à une seule base de données mutualisée. Concevez des requêtes qui permettent de réduire le nombre de données retournées. Une requête élastique est souvent plus adaptée lorsqu’il s’agit d’interroger de petites quantités de données en temps réel. En revanche, ce n’est pas le cas pour la construction de requêtes ou de rapports d’analyse fréquemment utilisés ou complexes. Si les requêtes ne sont pas assez efficaces, consultez le [plan d’exécution](/sql/relational-databases/performance/display-an-actual-execution-plan) pour voir quelle partie de la requête a été repoussée vers la base de données distante. Et évaluez la quantité de données renvoyée. Les requêtes qui requièrent un traitement complexe analytique pourraient être traitées plus efficacement en enregistrant les données client extraites dans une base de données qui est optimisée pour les requêtes analytiques. SQL Database et Azure Synapse Analytics peuvent héberger une telle base de données analytique.

Ce modèle pour analyse est expliqué dans la [didacticiel sur l’analyse des clients](saas-multitenantdb-tenant-analytics.md).

## <a name="get-the-wingtip-tickets-saas-multi-tenant-database-application-source-code-and-scripts"></a>Obtenir les scripts et le code source de l’application de base de données multi-locataire SaaS Wingtip Tickets

Les scripts et le code source de l’application de base de données multilocataire Wingtip Tickets SaaS sont disponibles dans le dépôt GitHub [WingtipTicketsSaaS-MultitenantDB](https://github.com/microsoft/WingtipTicketsSaaS-MultiTenantDB). Consultez les [conseils généraux](saas-tenancy-wingtip-app-guidance-tips.md) avant de télécharger et de débloquer les scripts Wingtip Tickets SaaS.

## <a name="create-ticket-sales-data"></a>Créer des données sur des ventes de tickets

Pour exécuter des requêtes sur un jeu de données plus concret, créez des données de ventes de tickets en exécutant le générateur de tickets.

1. Dans *PowerShell ISE*, ouvrez le script... \\Modules d’apprentissage\\Operational Analytics\\Adhoc Reporting\\*Demo-AdhocReporting.ps1* et définissez les valeurs suivantes :
   * **$DemoScenario** = 1, **Acheter des tickets pour des événements dans tous les lieux**.
2. Appuyez sur **F5** pour exécuter le script et générer des ventes de tickets. Pendant l’exécution du script, poursuivez les étapes de ce didacticiel. Les données de ticket font l’objet d’une requête dans la section *Exécuter des requêtes distribuées ad hoc*. Vous devez donc attendre que le générateur de tickets ait terminé.

## <a name="explore-the-tenant-tables"></a>Explorer les tables client 

Dans l’application de base de données Wingtip Tickets SaaS mutualisée, les clients sont stockés dans un modèle de gestion hybride client - où les données client sont stockées dans une base de données mutualisée ou une base de données à client unique et peuvent être déplacées entre les deux. Lorsque l’ensemble des bases de données client sont interrogées, il est important que la requête élastique puisse traiter les données comme si elles faisaient partie d’une seule base de données logique partitionnée par le client. 

Pour obtenir ce modèle, tous les tableaux client incluent une colonne *VenueId* qui identifie le client auquel les données appartiennent. L’élément *VenueId* est calculé en tant que hachage du nom de lieu, mais toute autre approche peut être utilisée pour introduire une valeur unique pour cette colonne. Cette approche est semblable à la façon dont la clé de client est calculée pour être utilisée dans le catalogue. Les tables contenant *VenueId* sont utilisées par une requête élastique pour paralléliser les requêtes et les transmettre à la bonne base de données client distante. Cela réduit considérablement la quantité de données renvoyées et entraîne une augmentation des performances, en particulier lorsqu’il existe plusieurs clients dont les données sont stockées dans des bases de données à client unique.

## <a name="deploy-the-database-used-for-ad-hoc-distributed-queries"></a>Déployer la base de données utilisée pour les requêtes distribuées ad hoc

Cet exercice déploie la base de données *adhocreporting*. Cette base de données principale contient le schéma utilisé pour interroger toutes les bases de données locataires. La base de données est déployée sur le serveur de catalogue existant, qui est le serveur utilisé pour toutes les bases de données liées à la gestion dans l’exemple d’application.

1. Ouvrez \\Modules d’apprentissage\\Operational Analytics\\Adhoc Reporting\\*Demo-AdhocReporting.ps1* dans *PowerShell ISE* et définissez les valeurs suivantes :
   * **$DemoScenario** = 2, **Déployer la base de données d’analyse ad hoc**.

2. Appuyez sur **F5** pour exécuter le script et créer la base de données *adhocreporting*.

Dans la section suivante, vous allez ajouter un schéma à la base de données, permettant ainsi l’exécution de requêtes distribuées.

## <a name="configure-the-head-database-for-running-distributed-queries"></a>Configurer la base de données 'head' pour l’exécution de requêtes distribuées

Cet exercice ajoute le schéma (la source de données externe et les définitions de la table externe) à la base de données de rapport ad hoc qui permet l’interrogation de toutes les bases de données client.

1. Ouvrez SQL Server Management Studio et connectez-vous à la base de données de rapport ad hoc créée à l’étape précédente. Le nom de la base de données est *adhocreporting*.
2. Ouvrez ...\Modules d’apprentissage\Operational Analytics\Adhoc Reporting\ *Initialize-AdhocReportingDB.sql* dans SSMS.
3. Passez en revue le script SQL et notez les points suivants :

   La requête élastique utilise des informations d’identification de niveau base de données pour accéder à chacune des bases de données client. Ces informations d’identification doivent être disponibles dans toutes les bases de données et doivent normalement bénéficier des droits minimaux nécessaires pour activer ces requêtes ad-hoc.

    ![créer des informations d’identification](./media/saas-multitenantdb-adhoc-reporting/create-credential.png)

   En utilisant la base de données de catalogues comme source de données externe, les requêtes sont distribuées sur toutes les bases de données inscrites dans le catalogue au moment de l’exécution de la requête. Étant donné que les noms de serveur sont différents pour chaque déploiement, ce script d’initialisation obtient l’emplacement de la base de données du catalogue en récupérant le serveur actuel (@@servername) dans lequel le script est exécuté.

    ![créer une source de données externe](./media/saas-multitenantdb-adhoc-reporting/create-external-data-source.png)

   Les tables externes qui font référence à des tables client sont définies avec **DISTRIBUTION = SHARDED(VenueId)** . Cela achemine une requête pour un *VenueId* particulier vers la base de données appropriée et améliore les performances pour de nombreux scénarios comme indiqué dans la section suivante.

    ![créer des tables externes](./media/saas-multitenantdb-adhoc-reporting/external-tables.png)

   La table locale *VenueTypes* qui est créée et alimentée. Cette table de données de référence étant courante dans toutes les bases de données client, elle peut être représentée ici comme une table locale et renseignée avec les données courantes. Pour certaines requêtes, cela peut réduire la quantité de données déplacées entre les bases de données client et la base de données *adhocreporting*.

    ![créer une table](./media/saas-multitenantdb-adhoc-reporting/create-table.png)

   Si vous incluez des tables de référence de cette manière, veillez à mettre à jour le schéma et les données de la table lorsque vous mettez à jour les bases de données client.

4. Appuyez sur **F5** pour exécuter le script et initialiser la base de données *adhocreporting*. 

Vous pouvez à présent exécuter des requêtes distribuées et collecter des informations sur l’ensemble des clients !

## <a name="run-ad-hoc-distributed-queries"></a>Exécuter des requêtes distribuées ad hoc

Maintenant que la base de données *adhocreporting* est configurée, lancez-vous et exécutez des requêtes ad hoc. Intégrez le plan d’exécution pour mieux appréhender le processus de requête. 

Lors de l’inspection du plan d’exécution, passez la souris sur les icônes de plan pour plus d’informations. 

1. Dans *SSMS*, ouvrez ...\\Learning Modules\\Operational Analytics\\Adhoc Reporting\\*Demo-AdhocReportingQueries.sql*.
2. Assurez-vous que vous êtes connecté à la base de données **adhocreporting**.
3. Sélectionnez le menu **Requête** et cliquez sur **Inclure le plan d’exécution réel**
4. Mettez en surbrillance la requête *Quels lieux sont actuellement inscrits ?* , puis appuyez sur **F5**.

   La requête renvoie la liste complète des lieux, montrant à quel point il est facile d’interroger l’ensemble des clients et de renvoyer des données provenant de chacun d’eux.

   Inspectez le plan ; vous constaterez que la totalité des coûts correspond à la requête distante, car nous interrogeons simplement chaque base de données client et sélectionnons les informations relatives aux lieux.

   ![SELECT * FROM dbo.Venues](./media/saas-multitenantdb-adhoc-reporting/query1-plan.png)

5. Sélectionnez la prochaine requête, puis appuyez sur **F5**.

   Cette requête joint les données de bases de données client et la table *VenueTypes* table (elle est locale dans la mesure où elle figure dans la base de données *adhocreporting*).

   Inspectez le plan ; vous constaterez que la majorité des coûts correspond à la requête distante, car nous interrogeons les informations relatives aux lieux de chaque client (dob.Venues), puis effectuons une jointure locale rapide avec la table *VenueTypes* pour afficher le nom convivial.

   ![Joindre des données locales et distantes](./media/saas-multitenantdb-adhoc-reporting/query2-plan.png)

6. Sélectionnez à présent la requête *Quel jour y a-t-il eu le plus de tickets vendus ?* , puis appuyez sur **F5**.

   Cette requête effectue une opération de jointure et d’agrégation un peu plus complexe. Il est important de noter que la plupart du traitement est effectué à distance, et une fois encore, nous récupérons uniquement les lignes dont nous avons besoin, en renvoyant une seule ligne pour le cumul agrégé de ventes de tickets par jour pour chaque lieu.

   ![query](./media/saas-multitenantdb-adhoc-reporting/query3-plan.png)


## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> 
> * Exécuter des requêtes distribuées sur toutes les bases de données client
> * Déployer une base de données de rapport ad hoc et y ajouter un schéma pour exécuter des requêtes distribuées.

Essayez maintenant le [Didacticiel sur l’analyse des clients](saas-multitenantdb-tenant-analytics.md) pour explorer l’extraction de données dans une base de données d’analyse distincte pour un traitement d’analyse plus complexe.

## <a name="additional-resources"></a>Ressources supplémentaires

<!-- ??
* Additional [tutorials that build upon the Wingtip Tickets SaaS Multi-tenant Database application](saas-multitenantdb-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
-->

* [Requête élastique](elastic-query-overview.md)