---
title: Analyses entre clients à l’aide de données extraites
description: Obtenez des informations sur les requêtes analytiques entre locataires à l’aide de données extraites de plusieurs bases de données Azure SQL dans une application monolocataire.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 12/18/2018
ms.openlocfilehash: 98896b5b728a729a29f989b3b9a76f29131af8d7
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93305968"
---
# <a name="cross-tenant-analytics-using-extracted-data---single-tenant-app"></a>Analytique entre locataires à l’aide des données extraites – Application monolocataire
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]
 
Dans ce didacticiel, vous suivez un scénario complet d’analyse pour une implémentation monolocataire. Le scénario illustre comment les analyses permettent aux entreprises de prendre des décisions intelligentes. À l’aide de données extraites de la base de données de chaque client, vous utilisez les analyses pour obtenir des informations sur un comportement de client, y compris son utilisation de l’exemple d’application Wingtip Tickets SaaS. Ce scénario implique trois étapes : 

1.  **Extrayez** des données à partir de chaque base de données à **Charger** dans un magasin d’analytique.
2.  **Transformez les données extraites** pour le traitement analytique.
3.  Utilisez les outils **d’Aide à la décision** pour en tirer des informations utiles, qui peuvent guider la prise de décision. 

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> - Créez le magasin analytique client dans lequel extraire les données.
> - Utilisez les travaux élastiques pour extraire des données de chaque base de données client dans le magasin d’analytique.
> - Optimisez les données extraites (réorganisez-les dans un schéma en étoile).
> - Interrogez la base de données analytique.
> - Utilisez Power BI pour la visualisation des données pour mettre en évidence les tendances dans les données client et effectuer des recommandations pour les améliorations.

![Le diagramme montre une vue d’ensemble de l’architecture utilisée pour cet article.](./media/saas-tenancy-tenant-analytics/architectureOverview.png)

## <a name="offline-tenant-analytics-pattern"></a>Modèle analytique client en mode hors connexion

Les applications SaaS mutualisées ont généralement une grande quantité de données client stockées dans le cloud. Les données constituent une source riche d’informations sur le fonctionnement et l’utilisation de votre application et sur le comportement de vos clients. Ces informations peuvent guider le développement des fonctionnalités, les améliorations de convivialité et autres investissements dans l’application et la plateforme.

L’accès aux données pour tous les clients est simple lorsque toutes les données se trouvent dans une seule base de données. Mais l’accès est plus complexe lors d’une distribution à grande échelle sur potentiellement des milliers de bases de données. Une façon de maîtriser la complexité et minimiser l’impact des requêtes analytique sur les données transactionnelles consiste à extraire les données dans un entrepôt de données ou une base de données analytique que vous préparerez à cette fin.

Ce didacticiel présente un scénario complet d’analytique pour l’application Wingtip Tickets. Tout d’abord, les *travaux élastiques* permettent d’extraire les données de chaque base de données client et de les charger dans les tables d’un magasin d’analytique intermédiaire. Le magasin d’analytique peut être une base de données SQL ou un pool SQL dédié. Pour l’extraction de données à grande échelle, [Azure Data Factory](../../data-factory/introduction.md) est recommandé.

Ensuite, les données agrégées sont transformées en un ensemble de tables à [schéma en étoile](https://www.wikipedia.org/wiki/Star_schema). Les tables sont constituées d’une table de faits centrale ainsi que de tables de dimension associées.  Pour Wingtip Tickets :

- La table de faits centrale dans le schéma en étoile contient les données de ticket.
- Les tables de dimension décrivent les emplacements, les événements, les clients et les dates d’achat.

Ensemble les tables de faits centrale et de dimension permettent un traitement analytique efficace. Le schéma en étoile utilisé dans ce didacticiel s’affiche dans l’image suivante :
 
![architectureOverView](./media/saas-tenancy-tenant-analytics/StarSchema.png)

Enfin, le magasin d’analytique est interrogé à l’aide de **Power BI** pour mettre en évidence le comportement du locataire et son utilisation de l’application Wingtip Tickets. Vous exécutez des requêtes qui :
 
- Afficher la popularité relative de chaque emplacement
- Soulignent les profils de vente de tickets pour différents événements
- Afficher la réussite relative des différents systèmes en matière de ventes pour les événements

Comprendre comment chaque client utilise le service pour explorer les options de monétisation du service et d’améliorer le service pour aider les clients à être plus efficaces. Ce didacticiel fournit des exemples simples des types d’analyses que vous pouvez recueillir à partir des données client.

## <a name="setup"></a>Programme d’installation

### <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vérifiez que les conditions préalables ci-dessous sont bien satisfaites :

- L’application de base de données Wingtip Tickets SaaS par client est déployée. Pour procéder à un déploiement en moins de cinq minutes, consultez la page [Déployer et explorer une application SaaS Wingtip](./saas-dbpertenant-get-started-deploy.md)
- Les scripts de base de données Wingtip Tickets SaaS par client et le [code source](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant/) de l’application sont disponibles au téléchargement sur GitHub. Consultez les instructions de téléchargement. Veillez à *débloquer le fichier zip* avant d’extraire son contenu. Consultez les [conseils généraux](saas-tenancy-wingtip-app-guidance-tips.md) avant de télécharger et de débloquer les scripts Wingtip Tickets SaaS.
- Power BI Desktop est installé. [Télécharger Power BI Desktop](https://powerbi.microsoft.com/downloads/)
- Le lot de clients supplémentaires a été configuré, consultez le [**Didacticiel de configuration des clients**](./saas-dbpertenant-provision-and-catalog.md).
- Un compte de travail et la base de données du compte de travail ont été créés. Consultez les étapes appropriées dans le [**Didacticiel de gestion du schéma**](./saas-tenancy-schema-management.md#create-a-job-agent-database-and-new-job-agent).

### <a name="create-data-for-the-demo"></a>Créer des données pour la démonstration

Dans ce didacticiel, l’analyse est effectuée sur les données de vente de tickets. Dans l’étape actuelle, vous générez des données de ticket pour tous les clients.  Ultérieurement, ces données sont extraites pour l’analyse. *Assurez-vous d’avoir configuré le traitement des clients comme décrit précédemment, afin d’avoir une quantité significative de données*. Une quantité suffisante de données peut exposer différents modèles d’achat de tickets.

1. Dans PowerShell ISE, ouvrez *…\Learning Modules\Operational Analytics\Tenant Analytics\Demo-TenantAnalytics.ps1* , et configurez la valeur suivante :
    - **$DemoScenario** = **1** Acheter des tickets pour des événements dans tous les lieux
2. Appuyez sur **F5** pour exécuter le script et créez un historique d’achat de tickets pour chaque événement à chaque emplacement.  Le script s’exécute pendant plusieurs minutes pour générer des dizaines de milliers de tickets.

### <a name="deploy-the-analytics-store"></a>Déployer le magasin analytique
Il existe souvent de nombreuses bases de données transactionnelles qui contiennent toutes les données client. Vous devez agréger les données client à partir de nombreuses bases de données transactionnelles dans le magasin d’analytique. L’agrégation permet d’effectuer des requêtes efficaces sur les données. Dans ce tutoriel, une instance d'Azure SQL Database est utilisée pour stocker les données agrégées.

Dans les étapes suivantes, vous déployez le magasin d’analytique, qui est appelé **tenantanalytics**. Vous déployez également des tables prédéfinies qui sont remplies plus loin dans le didacticiel :
1. Dans PowerShell ISE, open *…\Learning Modules\Operational Analytics\Tenant Analytics\Demo-TenantAnalytics.ps1* 
2. Définissez la variable $DemoScenario dans le script pour correspondre à votre choix de magasin d’analytique :
    - Pour utiliser SQL Database sans columnstore, définissez **$DemoScenario** = **2**
    - Pour utiliser SQL Database avec columnstore, définissez **$DemoScenario** = **3**  
3. Appuyez sur **F5** pour exécuter le script de démonstration (qui appelle le script *Deploy-TenantAnalytics\<XX>.ps1* ), qui crée la base de données d’analyse du locataire. 

Maintenant que vous avez déployé l’application et l’avez remplie de données client intéressantes, utilisez [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) pour connecter les serveurs **tenants1-dpt-&lt;Utilisateur&gt;** et **catalogue-dpt-&lt;Utilisateur&gt;** à l’aide de l’identifiant = *developer* et du mot de passe = *P\@ssword1*. Consultez le [didacticiel d’introduction](./saas-dbpertenant-wingtip-app-overview.md) pour plus d’informations.

![Capture d’écran montrant les informations nécessaires pour se connecter à SQL Server.](./media/saas-tenancy-tenant-analytics/ssmsSignIn.png)

Dans l’Explorateur d’objets, procédez comme suit :

1. Développez le serveur *tenants1-dpt-&lt;Utilisateur&gt;* .
2. Développez le nœud Bases de données et affichez la liste des bases de données client.
3. Développez le serveur *catalog-dpt-&lt;Utilisateur&gt;* .
4. Vérifiez que vous voyez le magasin analytique et la base de données jobaccount.

Consultez les éléments suivants de la base de données dans l’Explorateur d’objets SSMS en développant le nœud du magasin d’analytique :

- Les tables **TicketsRawData** et **EventsRawData** contiennent les données brutes extraites des bases de données client.
- Les tables du schéma en étoile sont **fact_Tickets** , **dim_Customers** , **dim_Venues** , **dim_Events** et **dim_Dates** .
- La procédure stockée est utilisée pour remplir les tables du schéma en étoile à partir des tables de données brutes.

![Capture d’écran des éléments de base de données affichés dans l’Explorateur d’objets SSMS.](./media/saas-tenancy-tenant-analytics/tenantAnalytics.png)

## <a name="data-extraction"></a>Extraction de données 

### <a name="create-target-groups"></a>Créer des groupes cibles 

Avant de continuer, assurez-vous d'avoir déployé le compte de travail et la base de données jobaccount. Dans les étapes suivantes, les travaux élastiques servent à extraire des données à partir de chaque base de données client et à stocker les données dans le magasin d’analytique. Puis la deuxième tâche traite les données et les stocke dans des tables dans le schéma en étoile. Ces deux tâches s’exécutent par rapport à deux différents groupes cibles, à savoir **TenantGroup** et **AnalyticsGroup**. La tâche d’extraction s’exécute sur TenantGroup, qui contient toutes les bases de données client. Le travail de traitement s’exécute sur AnalyticsGroup, qui contient le magasin d’analytique. Créez les groupes de cibles en procédant comme suit :

1. Dans SSMS, connectez-vous à la base de données **jobaccount** , dans catalog-dpt -&lt;Utilisateur&gt;.
2. Dans SSMS, ouvrez *…\Learning Modules\Operational Analytics\Tenant Analytics\ TargetGroups.sql* 
3. Modifiez la variable @User en haut du script, en remplaçant `<User>` par la valeur utilisateur utilisée lors du déploiement de l’application Wingtip SaaS.
4. Appuyez sur **F5** pour exécuter le script qui crée les deux groupes cibles.

### <a name="extract-raw-data-from-all-tenants"></a>Extraire les données brutes de tous les locataires

Des modifications de données significatives peuvent avoir lieu plus fréquemment pour les données de *ticket et de client* que les données que pour *l’événement et l’emplacement*. Par conséquent, envisagez l’extraction des données de ticket et de client séparément et plus fréquemment que pour les données d’événement et d’emplacement. Dans cette section, vous définissez et planifiez deux tâches distinctes :

- Extrayez les données de ticket et de client.
- Extrayez les données d’événement et d’emplacement.

Chaque travail extrait ses données et l’envoie dans le magasin d’analytique. Là-bas, un travail distinct traite les données extraites dans le schéma en étoile analytique.

1. Dans SSMS, connectez-vous à la base de données **jobaccount** , dans le serveur catalog-dpt -&lt;Utilisateur&gt;.
2. Dans SSMS, ouvrez *...\Learning Modules\Operational Analytics\Tenant Analytics\ExtractTickets.sql*.
3. Modifiez @User en haut du script, et remplacez `<User>` par le nom d’utilisateur utilisé lors du déploiement de l’application Wingtip SaaS 
4. Appuyez sur F5 pour exécuter le script qui crée et exécute la tâche qui extrait les données des tickets et des clients à partir de chaque base de données client. La tâche enregistre les données dans le magasin d’analytique.
5. Interrogez la table TicketsRawData dans la base de données tenantanalytics pour vous assurer que la table est remplie avec les informations de ticket de tous les clients.

![La capture d’écran présente la base de données ExtractTickets avec le propriétaire de base de données TicketsRawData sélectionné dans l’Explorateur d’objets.](./media/saas-tenancy-tenant-analytics/ticketExtracts.png)

Répétez les étapes précédentes, mais cette fois remplacez **\ExtractTickets.sql** par **\ExtractVenuesEvents.sql** à l’étape 2.

La tâche en cours d’exécution remplit la table EventsRawData dans le magasin d’analytique avec les nouvelles informations d’événement et d’emplacement de tous les clients. 

## <a name="data-reorganization"></a>Réorganisation des données

### <a name="shred-extracted-data-to-populate-star-schema-tables"></a>Fragmenter les données pour remplir les tables du schéma en étoile

L’étape suivante consiste à fragmenter les données brutes extraites dans un ensemble de tables qui sont optimisées pour les requêtes d’analytique. Un schéma en étoile est utilisé. Une table de faits centrale conserve les enregistrements des ventes de ticket individuelles. Les autres tables sont remplies avec les données associées sur les emplacements, les événements et les clients. Et vous trouverez aussi les tables de dimension de temps. 

Dans cette section du didacticiel, vous définissez et exécutez une tâche qui fusionne les données brutes extraites avec les données dans les tables du schéma en étoile. Une fois la fusion terminée, les données brutes sont supprimées, laissant les tables prêtes à être remplies par la tâche d’extraction de données client suivante.

1. Dans SSMS, connectez-vous à la base de données **jobaccount** , dans catalog-dpt -&lt;Utilisateur&gt;.
2. Dans SSMS, ouvrez *…\Learning Modules\Operational Analytics\Tenant Analytics\ShredRawExtractedData.sql*.
3. Appuyez sur **F5** pour exécuter le script pour définir un travail qui appelle la procédure stockée sp_ShredRawExtractedData dans le magasin d’analytique.
4. Laissez suffisamment de temps pour que le travail s’exécute correctement.
    - Vérifiez la colonne **Lifecycle** de la table jobs.jobs_execution pour l’état du travail. Vérifiez que la tâche a **Réussi** avant de continuer. Une exécution réussie affiche des données similaires au graphique suivant :

![fragmentation de données](./media/saas-tenancy-tenant-analytics/shreddingJob.PNG)

## <a name="data-exploration"></a>Exploration des données

### <a name="visualize-tenant-data"></a>Visualiser les données client

Les données de la table de schéma en étoile fournissent toutes les données de ventes de tickets nécessaires pour votre analyse. Pour voir les tendances dans de grands jeux de données plus facilement, vous devez les visualiser sous forme graphique.  Dans cette section, vous apprenez à utiliser **Power BI** pour manipuler et visualiser les données client que vous avez extraites et organisées.

Utilisez les étapes suivantes pour vous connecter à Power BI et importer les vues que vous avez créées précédemment :

1. Lancez Power BI Desktop.
2. Dans le ruban Accueil, sélectionnez **Obtenir des données** , puis **Plus...** .
3. Dans la fenêtre **Obtenir des données** , sélectionnez Azure SQL Database.
4. Dans la fenêtre de connexion à la base de données, entrez le nom de votre serveur (catalog-dpt-&lt;Utilisateur&gt;.database.windows.net). Sélectionnez **Importer** pour **Mode de connectivité de données** , puis cliquez sur OK. 

    ![signinpowerbi](./media/saas-tenancy-tenant-analytics/powerBISignIn.PNG)

5. Sélectionnez **Base de données** dans le volet de gauche, puis saisissez le nom d’utilisateur = *developer* et le mot de passe = *P\@ssword1*. Cliquez sur **Connecter**.  

    ![La capture d’écran présente la boîte de dialogue de la base de données SQL Server dans laquelle vous pouvez entrer un nom d’utilisateur et un mot de passe.](./media/saas-tenancy-tenant-analytics/databaseSignIn.PNG)

6. Dans le volet **Navigateur** , sous la base de données analytique, sélectionnez les tables du schéma en étoile : fact_Tickets, dim_Events, dim_Venues, dim_Customers et dim_Dates. Sélectionnez ensuite **Charger**. 

Félicitations ! Vous avez correctement chargé les données dans Power BI. Maintenant, vous pouvez commencer l’exploration des visualisations intéressantes pour aider à obtenir des informations sur vos clients. Vous verrez ensuite comment les analyses peuvent vous permettre de fournir des recommandations basées sur les données à l’équipe de professionnels de Wingtip Tickets. Les recommandations peuvent aider à optimiser l’expérience client et le modèle d’affaires.

Vous commencez en analysant les données de ventes de ticket pour afficher la variation de l’utilisation sur les systèmes. Sélectionnez les options suivantes dans Power BI pour tracer un graphique à barres du nombre total de tickets vendus par emplacement. En raison d’une variation aléatoire dans le générateur de tickets, vos résultats peuvent être différents.
 
![La capture d’écran présente une visualisation et des contrôles de Power BI pour la visualisation de données sur le côté droit.](./media/saas-tenancy-tenant-analytics/TotalTicketsByVenues.PNG)

Le tracé précédent confirme que le nombre d’entrées réalisées par chaque salle varie. Les emplacements qui vendent le plus de tickets plus utilisent votre service d’une manière plus importante que les systèmes qui en vendent moins. Il peut y avoir une opportunité de personnaliser l’allocation des ressources en fonction des besoins de chaque client.

Vous pouvez approfondir l’analyse des données pour voir comment les ventes de tickets varient au fil du temps. Sélectionnez les options suivantes dans Power BI pour tracer le nombre total de tickets vendus chaque jour pendant une période 60 jours.
 
![La capture d’écran présente une visualisation de Power BI intitulée Ticket Sale Distribution versus Sale Day (Distribution des ventes de tickets en fonction du jour de la vente).](./media/saas-tenancy-tenant-analytics/SaleVersusDate.PNG)

Le graphique précédent affiche ce pic de ventes de tickets pour certains emplacements. Ces pics renforcent l’idée que certains emplacements peuvent consommer les ressources système de façon disproportionnée. Jusqu'à présent aucun motif évident ne permet de déterminer quand les pics se produisent.

Ensuite, vous souhaitez approfondir vos recherches pour déterminer la particularité de ces jours de vente de pointe. Quand ces pics se produisent-ils une fois que les tickets sont mis en vente ? Pour tracer les tickets vendus par jour, sélectionnez les options suivantes dans Power BI.

![SaleDayDistribution](./media/saas-tenancy-tenant-analytics/SaleDistributionPerDay.PNG)

Le tracé précédent montre que certains emplacements vendent un grand nombre de tickets lors du premier jour de vente. Dès que les tickets sont mis en vente dans ces emplacements, il semble y avoir une grosse affluence. Cette croissance de l’activité pour quelques emplacements risque d’influer sur le service des autres clients.

Vous pouvez explorer les données à nouveau pour voir si cette forte affluence est vraie pour tous les événements hébergés par ces emplacements. Dans les graphiques précédents, vous avez observé que la Salle de concert Contoso vendait beaucoup de tickets et que Contoso possédait également un pic de ventes de tickets certains jours. Familiarisez-vous avec les options de Power BI pour tracer les ventes de ticket cumulatives de la Salle de concert Contoso, en mettant l’accent sur les tendances des ventes pour chacun de ses événements. Tous les événements suivent-ils le même modèle de vente ?

![ContosoSales](./media/saas-tenancy-tenant-analytics/EventSaleTrends.PNG)

Le tracé précédent pour une Salle de concert Contoso montre que la forte affluence ne se produit pas pour tous les événements. Familiarisez-vous avec les options de filtre pour voir les tendances de vente pour les autres systèmes.

Les informations sur les modèles de ventes de tickets peuvent aider Wingtip Tickets à optimiser leur modèle d’affaires. Au lieu de facturer tous les locataires à niveau égal, Wingtip peut proposer des niveaux de service avec différentes tailles de calcul. Les plus grands emplacements devant vendre plus de tickets par jour peuvent se voir proposer un niveau supérieur avec un contrat de niveau de service (SLA) plus élevé. Ces emplacements peuvent avoir leurs bases de données placées dans le pool avec des limites de ressources par base de données plus importantes. Chaque niveau de service peut avoir une allocation de vente horaire, avec des frais supplémentaires facturés pour les dépassements. Les plus grands emplacements qui ont des pics de vente périodiques peuvent tirer parti des niveaux supérieurs, et Wingtip Tickets peut commercialiser son service plus efficacement.

Dans le même temps, certains clients Wingtip Tickets se plaignent d’éprouver des difficultés à vendre suffisamment de tickets pour justifier le coût du service. Dans ces aperçus, il y a peut-être une opportunité de dynamiser les ventes de tickets pour les emplacements sous-performants. Des ventes plus élevées augmenteraient la valeur perçue du service. Cliquez avec le bouton droit sur fact_Tickets et sélectionnez **Nouvelle mesure**. Entrez l’expression suivante pour la nouvelle mesure appelée **AverageTicketsSold** :

```
AverageTicketsSold = AVERAGEX( SUMMARIZE( TableName, TableName[Venue Name] ), CALCULATE( SUM(TableName[Tickets Sold] ) ) )
```

Sélectionnez les options de visualisation suivantes pour tracer les pourcentages de tickets vendus par emplacement pour déterminer leur réussite relative.

![La capture d’écran présente une visualisation de Power BI intitulée Average Tickets Sold By Each Venue (Moyenne des tickets vendus par site).](./media/saas-tenancy-tenant-analytics/AvgTicketsByVenues.PNG)

Le tracé précédent montre que même si la plupart des emplacements vendent plus de 80 % de leurs tickets, certains ont du mal à remplir plus de la moitié des sièges. Familiarisez-vous avec les valeurs pour sélectionner le pourcentage minimal ou maximal de tickets vendus pour chaque emplacement.

Précédemment, vous avez approfondi l’analyse pour découvrir que les ventes de ticket ont tendance à suivre des modèles prévisibles. Cette détection peut permettre à Wingtip Tickets d’aider les emplacements en difficulté en recommandant une tarification dynamique. Cette découverte peut révéler une opportunité d’employer des techniques d’apprentissage automatique pour prédire les ventes de tickets pour chaque événement. Des prédictions pourraient également être faites pour l’impact sur le chiffre d’affaires de l’offre de remises sur les ventes de tickets. Power BI Embedded peut être intégré à une application de gestion des événements. L’intégration peut vous aider à visualiser les ventes prévues et l’effet des différentes remises. L’application peut vous aider à concevoir une remise optimale à appliquer directement à partir de l’affichage analytique.

Vous avez observé les tendances des données client à partir de l’application WingTip. Vous pouvez envisager d’autres façons dont l’application peut indiquer des décisions d’affaires pour les fournisseurs d’applications SaaS. Les fournisseurs peuvent mieux répondre aux besoins de leurs clients. Nous espérons que ce didacticiel vous aura donné les outils nécessaires pour effectuer des analyses sur les données client afin de répondre aux besoins de votre entreprise pour prendre des décisions pilotées par les données.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> - Déployé une base de données analytique client avec des tables de schéma en étoile prédéfinies
> - Utilisé des travaux élastiques pour extraire des données de toutes les bases de données client
> - Fusionné les données extraites dans des tables dans un schéma en étoile, conçu pour l’analytique
> - Interrogé une base de données analytique 
> - Utilisé Power BI pour la visualisation des données afin d’observer les tendances dans les données client 

Félicitations !

## <a name="additional-resources"></a>Ressources supplémentaires

- Autres [didacticiels reposant sur l’application SaaS Wingtip](./saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials).
- [Tâches élastiques](./elastic-jobs-overview.md).
- [Analytique entre locataires à l’aide des données extraites – Application multilocataire](./saas-multitenantdb-tenant-analytics.md)
