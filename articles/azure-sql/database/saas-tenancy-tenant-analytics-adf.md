---
title: Exécuter des requêtes d’analyse sur des bases de données de locataires
description: Requêtes analytiques interlocataires à l'aide de données extraites d'Azure SQL Database, Azure Synapse Analytics, Azure Data Factory ou Power BI.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 12/18/2018
ms.openlocfilehash: dc47c996748b126841cbeff1ea3f6f18f423951f
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96457637"
---
# <a name="explore-saas-analytics-with-azure-sql-database-azure-synapse-analytics-data-factory-and-power-bi"></a>Explorer des analyses SaaS avec Azure SQL Database, Azure Synapse Analytics, Data Factory et Power BI
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Dans ce didacticiel, vous suivez un scénario d’analyse de bout en bout. Le scénario illustre comment les analyses sur les données des clients peuvent aider les fournisseurs de logiciels à prendre des décisions intelligentes. À l’aide de données extraites de la base de données de chaque client, vous utilisez les analyses pour obtenir des informations sur un comportement de client, y compris son utilisation de l’exemple d’application Wingtip Tickets SaaS. Ce scénario implique trois étapes :

1. **Extrayez des données** à partir de chaque base de données de locataire dans un magasin d’analytique, dans le cas présent, un pool SQL dédié.
2. **Optimisez les données extraites** pour le traitement analytique.
3. Utilisez les outils **d’Aide à la décision** pour en tirer des informations utiles, qui peuvent guider la prise de décision.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
>
> - Créer le magasin d’analytique du client pour le chargement.
> - Utiliser Azure Data Factory (ADF) pour extraire des données de chaque base de données client dans l’entrepôt de données d’analyse.
> - Optimisez les données extraites (réorganisez-les dans un schéma en étoile).
> - Interroger l’entrepôt de données d’analyse.
> - Utilisez Power BI pour la visualisation des données pour mettre en évidence les tendances dans les données client et effectuer des recommandations pour les améliorations.

![architectureOverView](./media/saas-tenancy-tenant-analytics-adf/adf_overview.png)

## <a name="analytics-over-extracted-tenant-data"></a>Analyse sur les données client extraites

Les applications SaaS maintiennent une quantité potentiellement grande de données client dans le cloud. Les données peuvent constituer une source riche d’informations sur le fonctionnement et l’utilisation de votre application et sur le comportement de vos clients. Ces informations peuvent guider le développement des fonctionnalités, les améliorations de convivialité et autres investissements dans les applications et la plateforme.

L’accès aux données pour tous les clients est simple lorsque toutes les données se trouvent dans une seule base de données. Mais l’accès est plus complexe lors d’une distribution à grande échelle sur des milliers de bases de données. Une façon de maîtriser la complexité consiste à extraire les données à une base de données analytique ou un entrepôt de données pour les requêtes.

Ce didacticiel présente un scénario d’analytique de bout en bout pour l’application Wingtip Tickets. Tout d’abord, [Azure Data Factory (ADF)](../../data-factory/introduction.md) est utilisé comme outil d’orchestration pour extraire les ventes de tickets et les données associées de chaque base de données client. Ces données sont chargées dans des tables de mise en lots dans un magasin d’analytique. Le magasin d’analytique peut être une base de données SQL ou un pool SQL dédié. Ce tutoriel utilise [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) comme magasin d’analytique.

Ensuite, les données extraites sont transformées et chargées en un ensemble de tables à [schéma en étoile](https://www.wikipedia.org/wiki/Star_schema). Les tables sont constituées d’une table de faits centrale ainsi que de tables de dimension associées :

- La table de faits centrale dans le schéma en étoile contient les données de ticket.
- Les tables de dimension contiennent des données sur les emplacements, les événements, les clients et les dates d’achat.

Ensemble les tables centrale et de dimension permettent un traitement analytique efficace. Le schéma en étoile utilisé dans ce didacticiel s’affiche dans l’image suivante :

![Diagramme montrant le schéma en étoile qui est utilisé dans ce tutoriel.](./media/saas-tenancy-tenant-analytics-adf/starschematables.JPG)

Enfin, les tables du schéma en étoile sont interrogées. Les résultats de requête sont affichés visuellement à l’aide de Power BI pour mettre en évidence le comportement du client et son utilisation de l’application. Avec ce schéma en étoile, vous exécutez des requêtes qui exposent :

- Qui achète des tickets et à partir de quel emplacement.
- Les modèles et les tendances des ventes de tickets.
- La popularité relative de chaque emplacement.

Ce didacticiel fournit des exemples simples d’informations que vous pouvez recueillir à partir des données de Wingtip Tickets. Comprendre comment chaque salle utilise le service peut permettre au fournisseur de Wingtip Tickets de réfléchir à des plans de service différents ciblant des lieux plus ou moins actifs, par exemple.

## <a name="setup"></a>Programme d’installation

### <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vérifiez que les conditions préalables ci-dessous sont bien satisfaites :

- L’application de base de données Wingtip Tickets SaaS par client est déployée. Pour procéder à un déploiement en moins de cinq minutes, consultez [Déployer et explorer l’application SaaS Wingtip](./saas-dbpertenant-get-started-deploy.md).
- Les scripts de base de données Wingtip Tickets SaaS par client et le [code source](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant/) de l’application sont disponibles au téléchargement sur GitHub. Consultez les instructions de téléchargement. Veillez à *débloquer le fichier zip* avant d’extraire son contenu.
- Power BI Desktop est installé. [Télécharger Power BI Desktop](https://powerbi.microsoft.com/downloads/).
- Le lot de clients supplémentaires a été configuré, consultez le [**Didacticiel de configuration des clients**](./saas-dbpertenant-provision-and-catalog.md).

### <a name="create-data-for-the-demo"></a>Créer des données pour la démonstration

Ce didacticiel explore les analytiques sur les données de ventes de ticket. À cette étape, vous générez des données de ticket pour tous les clients. Lors d’une prochaine étape, ces données sont extraites pour l’analyse. _Assurez-vous de configurer le lot de clients_ (comme décrit précédemment) afin d’avoir suffisamment de données pour exposer une plage de différents modèles d’achat de tickets.

1. Dans PowerShell ISE, ouvrez *…\Learning Modules\Operational Analytics\Tenant Analytics DW\Demo-TenantAnalyticsDW.ps1*, et configurez la valeur suivante :
    - **$DemoScenario** = **1** Acheter des tickets pour des événements dans tous les lieux
2. Appuyez sur **F5** pour exécuter le script et créez un historique d’achat de tickets pour tous les lieux. Avec 20 clients, le script génère des dizaines de milliers de tickets et peut prendre 10 minutes ou plus.

### <a name="deploy-azure-synapse-analytics-data-factory-and-blob-storage"></a>Déployer Azure Synapse Analytics, Data Factory et le stockage Blob

Dans l’application Wingtip Tickets, les données transactionnelles des clients sont distribuées sur de nombreuses bases de données. Azure Data Factory (ADF) est utilisé pour orchestrer l’extraction, le chargement et la transformation (ELT) de ces données dans l’entrepôt de données. Pour charger des données plus efficacement dans Azure Synapse Analytics, ADF extrait les données dans des fichiers d’objets blob intermédiaires, puis utilise [PolyBase](../../synapse-analytics/sql-data-warehouse/design-elt-data-loading.md) pour charger les données dans l’entrepôt de données.

Au cours de cette étape, vous allez déployer les ressources supplémentaires utilisées dans le tutoriel : un pool SQL dédié, appelé _tenantanalytics_, une instance Azure Data Factory appelée _dbtodwload-\<user\>_ et un compte de stockage Azure appelé _wingtipstaging\<user\>_ . Le compte de stockage est utilisé pour stocker temporairement des fichiers de données extraits en tant qu’objets blob avant leur chargement dans l’entrepôt de données. Cette étape déploie également le schéma d’entrepôt de données et définit les pipelines ADF qui orchestrent le processus ELT.

1. Dans PowerShell ISE, ouvrez *…\Learning Modules\Operational Analytics\Tenant Analytics DW\Demo-TenantAnalyticsDW.ps1*, et configurez :
    - **$DemoScenario** = **2** déployer l’entrepôt de données analytiques, le stockage d’objets blob et la fabrique de données du client
1. Appuyez sur **F5** pour exécuter le script de démonstration et déployer les ressources Azure.

Maintenant, examinez les ressources Azure déployées :

#### <a name="tenant-databases-and-analytics-store"></a>Bases de données client et magasin d’analytique

Utilisez [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) pour vous connecter aux serveurs **tenants1-dpt-&lt;utilisateur&gt;** et **catalogue-dpt-&lt;utilisateur&gt;** . Remplacez &lt;utilisateur&gt; par la valeur utilisée lors du déploiement de l’application. Utilisez le nom de connexion = *developer* et le mot de passe = *P\@ssword1*. Consultez le [didacticiel d’introduction](./saas-dbpertenant-wingtip-app-overview.md) pour plus d’informations.

![Se connecter à SQL Database à partir de SSMS](./media/saas-tenancy-tenant-analytics-adf/ssmsSignIn.JPG)

Dans l'Explorateur d'objets :

1. Développez le serveur *tenants1-dpt-&lt;Utilisateur&gt;* .
1. Développez le nœud Bases de données et affichez la liste des bases de données client.
1. Développez le serveur *catalog-dpt-&lt;Utilisateur&gt;* .
1. Vérifiez que vous voyez le magasin d’analytique contenant les objets suivants :
    1. Les tables **raw_Tickets**, **raw_Customers**, **raw_Events** et **raw_Venues** contiennent les données brutes extraites des bases de données client.
    1. Les tables du schéma en étoile sont **fact_Tickets**, **dim_Customers**, **dim_Venues**, **dim_Events** et **dim_Dates** .
    1. La procédure stockée, **sp_transformExtractedData** est utilisée pour transformer les données et les charger dans les tables du schéma en étoile.

![La capture d’écran présente l’Explorateur d’objets avec des tableaux développés pour afficher divers objets de base de données.](./media/saas-tenancy-tenant-analytics-adf/DWtables.JPG)

#### <a name="blob-storage"></a>Stockage d'objets blob

1. Sur le [portail Azure](https://ms.portal.azure.com), accédez au groupe de ressources que vous avez utilisé pour le déploiement de l'application. Vérifiez qu'un compte de stockage appelé **wingtipstaging\<user\>** a été ajouté.

   ![DWtables](./media/saas-tenancy-tenant-analytics-adf/adf-staging-storage.PNG)

1. Cliquez sur le compte de stockage **wingtipstaging\<user\>** pour explorer les objets présents.
1. Cliquez sur la vignette **Objets blob**
1. Cliquez sur le conteneur **configfile**
1. Vérifiez que **configfile** contient un fichier JSON appelé **TableConfig.json**. Ce fichier contient les noms des tables de source et de destination, les noms des colonnes et le nom de la colonne de suivi.

#### <a name="azure-data-factory-adf"></a>Azure Data Factory (ADF)

Sur le [portail Azure](https://ms.portal.azure.com), accédez au groupe de ressources et vérifiez qu'une instance d'Azure Data Factory appelée _dbtodwload-\<user\>_ a été ajoutée.

 ![adf_portal](./media/saas-tenancy-tenant-analytics-adf/adf-data-factory-portal.png)

Cette section traite de la fabrique de données créée.
Suivez les étapes ci-dessous pour lancer la fabrique de données :

1. Sur le portail, cliquez sur la fabrique de données **dbtodwload -\<user\>** .
2. Cliquez sur le titre **Créer et surveiller** pour lancer le concepteur Data Factory dans un onglet séparé.

## <a name="extract-load-and-transform-data"></a>Extraire, charger et transformer des données

Azure Data Factory est utilisé pour orchestrer l’extraction, le chargement et la transformation de données. Dans ce didacticiel, vous extrayez des données à partir des quatre affichages SQL à partir de chacune des bases de données client : **rawTickets**, **rawCustomers**, **rawEvents**, et  **rawVenues**. Ces affichages incluent notamment l'ID du lieu, vous pouvez donc discriminer des données à partir de chaque salle de l'entrepôt de données. Les données sont chargées dans les tables de mise en lots correspondantes dans l’entrepôt de données : **raw_Tickets**, **raw_customers**, **raw_Events** et **raw_Venue**. Une procédure stockée convertit alors les données brutes et remplit les tables du schéma en étoile : **fact_Tickets**, **dim_Customers**, **dim_Venues**, **dim_Events** , et **dim_Dates**.

Dans la section précédente, vous déployez et initialisez les ressources Azure nécessaires, y compris la fabrique de données. La fabrique de données déployée inclut des pipelines, des jeux de données, des services liés, etc., requis pour extraire, charger et transformer les données client. Examinons ces objets de plus près, puis déclenchez le pipeline pour déplacer des données depuis les bases de données client vers l’entrepôt de données.

### <a name="data-factory-pipeline-overview"></a>Vue d’ensemble du pipeline de fabrique de données

Cette section traite des objets créés dans la fabrique de données. L’illustration suivante décrit le flux de travail global du pipeline ADF utilisé dans ce didacticiel. Si vous préférez explorer le pipeline plus tard et voir les résultats d’abord, passez à la section suivante **Déclencher l’exécution du pipeline**.

![adf_overview](./media/saas-tenancy-tenant-analytics-adf/adf-data-factory.PNG)

Dans la page Vue d’ensemble, basculez vers l’onglet **Auteur** dans le volet gauche et observez qu’il y a trois [pipelines](../../data-factory/concepts-pipelines-activities.md) et trois [jeux de données](../../data-factory/concepts-datasets-linked-services.md) créés.
![adf_author](./media/saas-tenancy-tenant-analytics-adf/adf_author_tab.JPG)

Les trois pipelines imbriqués sont : SQLDBToDW, DBCopy et TableCopy.

**Pipeline 1 - SQLDBToDW** recherche les noms des bases de données client stockés dans la base de données de catalogue (nom de la table : [__ShardManagement].[ShardsGlobal]) et pour chaque base de données client, exécutez le pipeline **DBCopy**. À l’achèvement, le schéma fourni de la procédure stockée **sp_TransformExtractedData** est exécuté. Cette procédure stockée transforme les données chargées dans les tables de mise en lots et remplit les tables du schéma en étoile.

**Pipeline 2 - DBCopy** recherche les noms des tables et des colonnes sources à partir d’un fichier de configuration stocké dans le stockage d’objets blob.  Le pipeline **TableCopy** est ensuite exécuté pour chacune des quatre tables : TicketFacts CustomerFacts, EventFacts et VenueFacts. L’activité **[Foreach](../../data-factory/control-flow-for-each-activity.md)** s’exécute en parallèle pour l’ensemble des 20 bases de données. ADF autorise un maximum de 20 itérations de boucle à exécuter en parallèle. Envisagez de créer plusieurs pipelines pour un plus grand nombre de bases de données.

**Pipeline 3 - tableauCopier** utilise les numéros de version des lignes dans SQL Database (_rowversion_) pour identifier les lignes modifiées ou mises à jour. Cette activité recherche la version de la ligne du début et de fin pour extraire des lignes à partir des tables sources. La table **CopyTracker** stockée dans chaque base de données client effectue le suivi de la dernière ligne extraite à partir de chaque table source durant chaque exécution. Les lignes nouvelles ou modifiées sont copiées dans les tables de mise en lots correspondantes dans l’entrepôt de données : **raw_Tickets**, **raw_customers**, **raw_Events** et **raw_Venues**. Enfin, la version de la dernière ligne est enregistrée dans la table **CopyTracker** pour l’utiliser comme version de la ligne initiale lors de la prochaine extraction.

Il existe également trois services liés paramétrables qui lient la fabrique de données aux bases de données SQL sources, au pool SQL dédié cible et au stockage Blob intermédiaire. Dans l’onglet **Auteur**, cliquez sur **Connexions** pour explorer les services liés, comme indiqué dans l’image suivante :

![adf_linkedservices](./media/saas-tenancy-tenant-analytics-adf/linkedservices.JPG)

Correspondant aux trois services liés, il existe trois jeux de données faisant référence aux données que vous utilisez dans les activités de pipeline en tant qu’entrée ou sortie. Explorez chacun des jeux de données pour observer les connexions et les paramètres utilisés. _AzureBlob_ pointe vers le fichier de configuration contenant des tables et des colonnes source et cible, ainsi que la colonne de suivi de chaque source.
  
### <a name="data-warehouse-pattern-overview"></a>Vue d’ensemble du modèle de l’entrepôt de données

Azure Synapse est utilisé comme magasin d’analytique pour effectuer l’agrégation sur les données de locataire. Dans cet exemple, PolyBase est utilisé pour charger des données dans l'entrepôt de données. Les données brutes sont chargées dans des tables de mise en lots qui ont une colonne d’identité pour suivre des lignes transformées dans les tables du schéma en étoile. L’image suivante montre le modèle de chargement : ![Le diagramme illustre le modèle de chargement des tables de base de données.](./media/saas-tenancy-tenant-analytics-adf/loadingpattern.JPG)

Les tables de dimension du type 1 de SCD (Slowly Changing Dimension) sont utilisées dans cet exemple. Chaque dimension possède une clé de substitution définie à l’aide d’une colonne d’identité. Comme meilleure pratique, la table de dimension de date est préremplie afin de gagner du temps. Pour les autres tables de dimension, une instruction CREATE TABLE AS SELECT... (CTAS) est utilisée pour créer une table temporaire contenant les lignes existantes non modifiées et modifiées, avec des clés de substitution. Cette opération s’effectue avec IDENTITY_INSERT=ON. De nouvelles lignes sont ensuite insérées dans la table avec IDENTITY_INSERT=OFF. Pour une restauration facile, la table de dimension existante est renommée et la table temporaire est renommée pour devenir la nouvelle table de dimension. Avant chaque exécution, l’ancienne table de dimension est supprimée.

Les tables de dimension sont chargées avant la table de faits. Cette séquence garantit que toutes les dimensions de référence existent déjà pour chaque fait qui arrive. Comme les faits sont chargés, la clé d’entreprise pour chaque dimension correspondante est mise en correspondance et les clés de substitution correspondantes sont ajoutées à chaque fait.

La dernière étape de la transformation supprime les données de mise en lots prêtes pour la prochaine exécution du pipeline.

### <a name="trigger-the-pipeline-run"></a>Déclencher l’exécution du pipeline

Suivez les étapes ci-dessous pour exécuter le pipeline d’extraction, de chargement et de transformation pour toutes les bases de données client :

1. Dans l’onglet **Auteur** de l’interface utilisateur de ADF, sélectionnez le pipeline **SQLDBToDW** dans le volet gauche.
1. Cliquez sur **Déclencher** puis sur **Déclencher maintenant** depuis le menu déroulant. Cette action exécute le pipeline immédiatement. Dans un scénario de production, vous pouvez définir un emploi du temps pour l’exécution du pipeline visant à actualiser les données selon une planification.
  ![La capture d’écran présente des ressources Factory pour un pipeline nommé SQL DB à DW avec l’option Déclencheur développée et la commande Déclencher maintenant sélectionnée.](./media/saas-tenancy-tenant-analytics-adf/adf_trigger.JPG)
1. Sur la page **Exécution du pipeline**, cliquez sur **Terminer**.

### <a name="monitor-the-pipeline-run"></a>Surveiller l’exécution du pipeline.

1. Dans l’interface utilisateur ADF, basculez vers l’onglet **Surveiller** dans le menu de gauche.
1. Cliquez sur **Actualiser** jusqu'à ce que l’état du pipeline SQLDBToDW soit **Réussi**.
  ![La capture d’écran présente le pipeline SQLD DB à DW avec l’état Réussi.](./media/saas-tenancy-tenant-analytics-adf/adf_monitoring.JPG)
1. Connectez-vous à l’entrepôt de données avec SSMS et interrogez les tables du schéma en étoile pour vérifier que les données ont été chargées dedans.

Une fois le pipeline terminé, la table de faits conserve les données de ventes de tickets pour tous les lieux et les tables de dimension sont remplies avec les lieux, les événements et les clients correspondants.

## <a name="data-exploration"></a>Exploration des données

### <a name="visualize-tenant-data"></a>Visualiser les données client

Les données dans le schéma en étoile fournissent toutes les données de ventes de tickets nécessaires pour votre analyse. La visualisation sous forme graphique permet de voir les tendances dans de grands jeux de données plus facilement. Dans cette section, vous utilisez **Power BI** pour manipuler et visualiser les données client dans l’entrepôt de données.

Utilisez les étapes suivantes pour vous connecter à Power BI et importer les vues que vous avez créées précédemment :

1. Lancez Power BI Desktop.
2. Dans le ruban Accueil, sélectionnez **Obtenir des données**, puis **Plus...** .
3. Dans la fenêtre **Obtenir des données**, sélectionnez **Azure SQL Database**.
4. Dans la fenêtre de connexion à la base de données, entrez le nom de votre serveur (**catalog-dpt-&lt;Utilisateur&gt;.database.windows.net**). Sélectionnez **Importer** pour **Mode de connectivité de données**, puis cliquez sur **OK**.

    ![sign-in-to-power-bi](./media/saas-tenancy-tenant-analytics-adf/powerBISignIn.PNG)

5. Sélectionnez **Base de données** dans le volet de gauche, puis saisissez le nom d’utilisateur = *developer* et le mot de passe = *P\@ssword1*. Cliquez sur **Connecter**.  

    ![database-sign-in](./media/saas-tenancy-tenant-analytics-adf/databaseSignIn.PNG)

6. Dans le volet **Navigateur**, sous la base de données analytique, sélectionnez les tables du schéma en étoile : **fact_Tickets**, **dim_Events**, **dim_Venues**, **dim_Customers** et **dim_Dates**. Sélectionnez ensuite **Charger**.

Félicitations ! Vous avez correctement chargé les données dans Power BI. Maintenant, explorez les visualisations intéressantes pour obtenir des informations sur vos clients. Examinez ensuite comment les analytiques peuvent permettre de fournir des recommandations basées sur certaines données à l’équipe de professionnels de Wingtip Tickets. Les recommandations peuvent aider à optimiser l’expérience client et le modèle d’affaires.

Commencez en analysant les données de ventes de ticket pour afficher la variation de l’utilisation sur les systèmes. Sélectionnez les options affichées dans Power BI pour tracer un graphique à barres du nombre total de tickets vendus par emplacement. (En raison d’une variation aléatoire dans le générateur de tickets, vos résultats peuvent être différents.)

![TotalTicketsByVenues](./media/saas-tenancy-tenant-analytics-adf/TotalTicketsByVenues-DW.PNG)

Le tracé précédent confirme que le nombre d’entrées réalisées par chaque salle varie. Les emplacements qui vendent le plus de tickets plus utilisent votre service d’une manière plus importante que les systèmes qui en vendent moins. Il peut y avoir une opportunité de personnaliser l’allocation des ressources en fonction des besoins de chaque client.

Vous pouvez approfondir l’analyse des données pour voir comment les ventes de tickets varient au fil du temps. Sélectionnez les options affichées dans l’image suivante dans Power BI pour tracer le nombre total de tickets vendus chaque jour pendant une période 60 jours.

![SaleVersusDate](./media/saas-tenancy-tenant-analytics-adf/SaleVersusDate-DW.PNG)

Le graphique précédent montre ce pic de ventes de tickets pour certains emplacements. Ces pics renforcent l’idée que certains emplacements peuvent consommer les ressources système de façon disproportionnée. Jusqu'à présent aucun motif évident ne permet de déterminer quand les pics se produisent.

Ensuite, approfondissez vos recherches pour déterminer la particularité de ces jours de vente de pointe. Quand ces pics se produisent-ils une fois que les tickets sont mis en vente ? Pour tracer les tickets vendus par jour, sélectionnez les options affichées dans l’image suivante au sein de Power BI.

![SaleDayDistribution](./media/saas-tenancy-tenant-analytics-adf/SaleDistributionPerDay-DW.PNG)

Ce graphique montre que certains emplacements vendent un grand nombre de tickets lors du premier jour de vente. Dès que les tickets sont mis en vente dans ces emplacements, il semble y avoir une grosse affluence. Cette croissance de l’activité pour quelques emplacements risque d’influer sur le service des autres clients.

Vous pouvez explorer les données à nouveau pour voir si cette forte affluence est vraie pour tous les événements hébergés par ces emplacements. Dans les graphiques précédents, vous avez observé que la Salle de concert Contoso vendait beaucoup de tickets et que Contoso possédait également un pic de ventes de tickets certains jours. Familiarisez-vous avec les options de Power BI pour tracer les ventes de ticket cumulatives de la Salle de concert Contoso, en mettant l’accent sur les tendances des ventes pour chacun de ses événements. Tous les événements suivent-ils le même modèle de vente ? Essayez de générer un graphique semblable à celui ci-dessous.

![ContosoSales](./media/saas-tenancy-tenant-analytics-adf/EventSaleTrends.PNG)

Ce graphique des ventes cumulatives des tickets au fil du temps pour une salle de concert Contoso pour chaque événement indique qu’une ruée ne se produit pas pour tous les événements. Familiarisez-vous avec les options de filtre pour explorer les tendances de vente pour les autres emplacements.

Les informations sur les modèles de ventes de tickets peuvent aider Wingtip Tickets à optimiser leur modèle d’affaires. Au lieu de facturer tous les locataires à niveau égal, Wingtip peut proposer des niveaux de service avec différentes tailles de calcul. Les plus grands emplacements devant vendre plus de tickets par jour peuvent se voir proposer un niveau supérieur avec un contrat de niveau de service (SLA) plus élevé. Ces emplacements peuvent avoir leurs bases de données placées dans le pool avec des limites de ressources par base de données plus importantes. Chaque niveau de service peut avoir une allocation de vente horaire, avec des frais supplémentaires facturés pour les dépassements. Les plus grands emplacements qui ont des pics de vente périodiques peuvent tirer parti des niveaux supérieurs, et Wingtip Tickets peut commercialiser son service plus efficacement.

Dans le même temps, certains clients Wingtip Tickets se plaignent d’éprouver des difficultés à vendre suffisamment de tickets pour justifier le coût du service. Dans ces aperçus, il y a peut-être une opportunité de dynamiser les ventes de tickets pour les emplacements sous-performants. Des ventes plus élevées augmenteraient la valeur perçue du service. Cliquez avec le bouton droit sur fact_Tickets et sélectionnez **Nouvelle mesure**. Entrez l’expression suivante pour la nouvelle mesure appelée **AverageTicketsSold** :

```sql
AverageTicketsSold = DIVIDE(DIVIDE(COUNTROWS(fact_Tickets),DISTINCT(dim_Venues[VenueCapacity]))*100, COUNTROWS(dim_Events))
```

Sélectionnez les options de visualisation suivantes pour tracer les pourcentages de tickets vendus par emplacement pour déterminer leur réussite relative.

![AvgTicketsByVenues](./media/saas-tenancy-tenant-analytics-adf/AvgTicketsByVenues-DW.PNG)

Le graphique ci-dessus montre que même si la plupart des emplacements vendent plus de 80 % de leurs tickets, certains ont du mal à remplir plus de la moitié des sièges. Familiarisez-vous avec les valeurs pour sélectionner le pourcentage minimal ou maximal de tickets vendus pour chaque emplacement.

## <a name="embedding-analytics-in-your-apps"></a>Incorporation d’analytique dans vos applications

Ce didacticiel est concentré sur des analyses entre clients utilisées pour améliorer la compréhension du fournisseur de logiciels de leurs clients. Les analytiques peuvent également fournir des informations aux _clients_, pour les aider à gérer eux-mêmes leur entreprise de façon plus efficace.

Dans l’exemple Wingtip Tickets, vous avez découvert que les ventes de tickets ont tendance à suivre des modèles prévisibles. Cette information peut être utilisée pour aider à améliorer les ventes de tickets des emplacements en difficulté. Il peut exister une opportunité d’employer des techniques d’apprentissage automatique pour prédire les ventes de tickets pour des événements. Les effets des changements de prix peuvent également être modélisés, pour prédire l’impact des remises. Power BI Embedded peut être intégré à une application de gestion d’événements pour visualiser les ventes prédites, y compris l’impact des remises sur le total des sièges vendus et le chiffre d’affaires sur les événements faisant l’objet de faibles ventes. Avec Power BI Embedded, vous pouvez même intégrer l’application d’une remise aux prix des tickets, au sein de l’expérience de visualisation.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
>
> - Créer le magasin d’analytique du client pour le chargement.
> - Utiliser Azure Data Factory (ADF) pour extraire des données de chaque base de données client dans l’entrepôt de données d’analyse.
> - Optimisez les données extraites (réorganisez-les dans un schéma en étoile).
> - Interroger l’entrepôt de données d’analyse.
> - Utilisez Power BI pour la visualisation des données pour mettre en évidence les tendances dans les données client et effectuer des recommandations pour les améliorations.

Félicitations !

## <a name="additional-resources"></a>Ressources supplémentaires

- Autres [didacticiels reposant sur l’application SaaS Wingtip](./saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials).