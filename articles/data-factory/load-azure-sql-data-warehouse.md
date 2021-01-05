---
title: Charger des données dans Azure Synapse Analytics
description: Utiliser Azure Data Factory pour copier des données dans Azure Synapse Analytics
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 10/30/2020
ms.openlocfilehash: dcf3db33818448116da53d8a01d0c62aca7bc1af
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000077"
---
# <a name="load-data-into-azure-synapse-analytics-by-using-azure-data-factory"></a>Charger des données dans Azure Synapse Analytics à l’aide d’Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

[Azure Synapse Analytics](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) est une base de données de scale-out basée sur le cloud capable de traiter de grands volumes de données relationnelles et non relationnelles. Azure Synapse Analytics repose sur une architecture MPP (massively parallel processing) optimisée pour les charges de travail d’entrepôt de données d’entreprise. Elle offre l’élasticité du cloud avec la flexibilité de mettre à l’échelle le stockage et d’exécuter le calcul indépendamment.

La prise en main d’Azure Synapse Analytics est désormais plus facile lorsque vous utilisez Azure Data Factory. Azure Data Factory est un service informatique d’intégration de données informatique intégralement managé. Vous pouvez utiliser le service pour remplir une instance Azure Synapse Analytics avec les données de votre système existant et gagner du temps lors de la création de vos solutions d’analyse.

Azure Data Factory offre les avantages suivants pour le chargement des données dans Azure Synapse Analytics :

* **Facilité de configuration** : assistant intuitif en 5 étapes. Aucun script nécessaire.
* **Prise en charge étendue du magasin de données** : prise en charge intégrée d’un ensemble complet de magasins de données locaux et dans le cloud. Pour une liste détaillée, consultez le tableau [Banques de données prises en charge](copy-activity-overview.md#supported-data-stores-and-formats).
* **Sécurité et conformité** : les données sont transférées via HTTPS ou ExpressRoute. La présence globale du service garantit que vos données ne quittent jamais les limites géographiques.
* **Performances sans précédent à l’aide de PolyBase** : PolyBase est le moyen le plus efficace de déplacer des données dans Azure Synapse Analytics. Utilisez la fonction blob intermédiaire pour atteindre des vitesses de charge élevées pour tous les types de magasins de données, y compris le stockage Blob Azure et Data Lake Store. (Polybase prend en charge le stockage Blob Azure et Azure Data Lake Store par défaut.) Pour en savoir plus, voir [Performances de l’activité de copie](copy-activity-performance.md).

Cet article explique comment utiliser l’outil de copie de données Data Factory pour _charger des données d’Azure SQL Database dans Azure Synapse Analytics_. Vous pouvez procéder de même pour copier des données à partir d’autres types de banques de données.

> [!NOTE]
> Pour plus d’informations, consultez [Copier des données depuis/vers Azure Synapse Analytics à l’aide d’Azure Data Factory](connector-azure-sql-data-warehouse.md).

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure : Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.
* Azure Synapse Analytics : l'entrepôt de données conserve les données copiées à partir de SQL Database. Si vous ne disposez pas d’Azure Synapse Analytics, consultez les instructions dans [Créer une instance Azure Synapse Analytics](../synapse-analytics/sql-data-warehouse/load-data-from-azure-blob-storage-using-copy.md).
* Azure SQL Database : ce tutoriel copie les données de l’exemple de jeu de données Adventure Works LT dans Azure SQL Database. Vous pouvez créer cet échantillon de base de données dans SQL Database en suivant les instructions données dans [Création d’un échantillon de base de données dans Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md).
* Compte Azure Storage : Stockage Azure est utilisé comme objet blob _intermédiaire_ dans l’opération de copie en bloc. Si vous ne possédez pas de compte de stockage Azure, consultez les instructions dans [Créer un compte de stockage](../storage/common/storage-account-create.md).

## <a name="create-a-data-factory"></a>Créer une fabrique de données

1. Dans le menu de gauche, sélectionnez **Créer une ressource** > **Données + Analytique** > **Data Factory** :

2. Sur la page **Nouvelle fabrique de données**, fournissez les valeurs des éléments suivants :

    * **Name** : Entrez le nom *LoadSQLDWDemo*. Le nom de votre fabrique de données doit être un nom *global unique. Si l’erreur « Le nom de fabrique de données 'LoadSQLDWDemo' n’est pas disponible » apparaît, saisissez un autre nom pour la fabrique de données. Par exemple, utilisez le nom _**votrenom**_**ADFTutorialDataFactory**. Essayez à nouveau de créer la fabrique de données. Pour savoir comment nommer les artefacts Data Factory, voir [Data Factory - Règles d’affectation des noms](naming-rules.md).
    * **Abonnement**: Sélectionnez l’abonnement Azure dans lequel créer la fabrique de données. 
    * **Groupe de ressources** : Sélectionnez un groupe de ressources existant dans la liste déroulante ou sélectionnez l’option **Créer** et entrez le nom d’un groupe de ressources. Pour plus d’informations sur les groupes de ressources, consultez [Utilisation des groupes de ressources pour gérer vos ressources Azure](../azure-resource-manager/management/overview.md).  
    * **Version** : Sélectionnez **V2**.
    * **Emplacement** : Sélectionnez l’emplacement de la fabrique de données. Seuls les emplacements pris en charge sont affichés dans la liste déroulante. Les magasins de données utilisés par la fabrique de données peuvent se trouver dans d’autres emplacements et régions. Ces magasins de données incluent Azure Data Lake Store, Stockage Azure, Azure SQL Database, etc.

3. Sélectionnez **Create** (Créer).
4. Une fois la création terminée, accédez à votre fabrique de données. La page d’accueil **Data Factory** devrait s’afficher comme dans l’image suivante :

   ![Page d’accueil Data Factory](./media/doc-common-process/data-factory-home-page.png)

   Sélectionnez la vignette **Créer et surveiller** pour lancer l’application d’intégration de données dans un onglet séparé.

## <a name="load-data-into-azure-synapse-analytics"></a>Charger des données dans Azure Synapse Analytics

1. Dans la page **Prise en main**, sélectionnez la vignette **Copier les données** pour démarrer l’outil Copier les données.

2. Dans la page **Propriétés**, spécifiez **CopyFromSQLToSQLDW** dans le champ **Nom de tâche**, puis cliquez sur **Suivant**.

    ![Page Propriétés](./media/load-azure-sql-data-warehouse/copy-data-tool-properties-page.png)

3. Dans la page **Banque de données source**, effectuez les étapes suivantes :
    >[!TIP]
    >Dans ce tutoriel, vous utilisez l’*authentification SQL* comme type d’authentification pour votre magasin de données source, mais vous pouvez choisir d’autres méthodes d’authentification prises en charge : *Principal de service* et *Identité managée*, si nécessaire. Pour plus d’informations, reportez-vous aux sections correspondantes de [cet article](./connector-azure-sql-database.md#linked-service-properties).
    >Pour stocker de manière sécurisée des secrets de magasins de données, il est également recommandé d’utiliser un coffre de clés Azure. Pour obtenir des illustrations détaillées, reportez-vous à [cet article](./store-credentials-in-key-vault.md).

    a. Cliquez sur **+ Créer une connexion**.

    b. Sélectionnez **Azure SQL Database** dans la galerie, puis sélectionnez **Continuer**. Vous pouvez taper « SQL » dans la zone de recherche pour filtrer les connecteurs.

    ![Sélectionner Azure SQL DB](./media/load-azure-sql-data-warehouse/select-azure-sql-db-source.png)

    c. Dans la page **Nouveau service lié**, sélectionnez le nom de votre serveur et le nom de votre base de données dans la liste déroulante, puis spécifiez le nom d’utilisateur et le mot de passe. Cliquez sur **Tester la connexion** pour vérifier les paramètres, puis sélectionnez **Créer**.

    ![Configurer Azure SQL DB](./media/load-azure-sql-data-warehouse/configure-azure-sql-db.png)

    d. Sélectionnez le service lié récemment créé comme source, puis cliquez sur **Suivant**.

4. Dans la page **Sélectionner les tables à partir desquelles copier les données ou utiliser une requête personnalisée**, entrez **SalesLT** pour filtrer les tables. Activez la case **(Sélectionner tout)** pour utiliser toutes les tables pour la copie, puis cliquez sur **Suivant**.

    ![Sélectionner les tables source](./media/load-azure-sql-data-warehouse/select-source-tables.png)

5. Sur la page **Appliquer le filtre**, spécifiez vos paramètres ou sélectionnez **Suivant**.

6. Dans la page **Banque de données de destination**, effectuez les étapes suivantes :
    >[!TIP]
    >Dans ce tutoriel, vous utilisez l’*authentification SQL* comme type d’authentification pour votre magasin de données de destination, mais vous pouvez choisir d’autres méthodes d’authentification prises en charge : *Principal de service* et *Identité managée*, si nécessaire. Pour plus d’informations, reportez-vous aux sections correspondantes de [cet article](./connector-azure-sql-data-warehouse.md#linked-service-properties).
    >Pour stocker de manière sécurisée des secrets de magasins de données, il est également recommandé d’utiliser un coffre de clés Azure. Pour obtenir des illustrations détaillées, reportez-vous à [cet article](./store-credentials-in-key-vault.md).

    a. Cliquez sur **+ Créer une connexion** pour ajouter une connexion.

    b. Sélectionnez **Azure Synapse Analytics (anciennement SQL Data Warehouse)** dans la galerie, puis sélectionnez **Continuer**. Vous pouvez taper « SQL » dans la zone de recherche pour filtrer les connecteurs.

    ![Sélectionner Azure Synapse Analytics](./media/load-azure-sql-data-warehouse/select-azure-sql-dw-sink.png)

    c. Dans la page **Nouveau service lié**, sélectionnez le nom de votre serveur et le nom de votre base de données dans la liste déroulante, puis spécifiez le nom d’utilisateur et le mot de passe. Cliquez sur **Tester la connexion** pour vérifier les paramètres, puis sélectionnez **Créer**.

    ![Configurer Azure Synapse Analytics](./media/load-azure-sql-data-warehouse/configure-azure-sql-dw.png)

    d. Sélectionnez le service lié nouvellement créé comme récepteur, puis cliquez sur **Suivant**.

7. Dans la page **Mappage de table**, passez en revue le contenu, puis cliquez sur **Suivant**. Un mappage de table intelligent s’affiche. Les tables source sont mappées sur les tables de destination en fonction des noms de tables. Si une table source n’existe pas dans la destination, Azure Data Factory crée une table de destination par défaut qui porte le même nom. Vous pouvez également mapper une table source sur une table de destination existante.

   ![Page Mappage de table](./media/load-azure-sql-data-warehouse/table-mapping.png)

8. Dans la page **Mappage de colonnes**, passez en revue le contenu, puis cliquez sur **Suivant**. Le mappage de table intelligent repose sur le nom de colonne. Lorsque vous autorisez Data Factory à créer automatiquement les tables, la conversion du type de données peut se produire en cas d’incompatibilités entre les magasins source et de destination. Si la conversion du type de données n’est pas prise en charge entre la colonne de destination source et de destination, un message d’erreur s’affiche en regard de la table correspondante.

    ![Page de mappage de colonnes](./media/load-azure-sql-data-warehouse/schema-mapping.png)

9. Dans la page **Paramètres**, effectuez les étapes suivantes :

    a. Dans **Paramètres de préproduction**, cliquez sur **+ Nouveau** pour créer un stockage de préproduction. Le stockage est utilisé pour les données en préproduction avant leur chargement dans Azure Synapse Analytics avec PolyBase. Une fois la copie terminée, les données temporaires dans Stockage Blob Azure sont nettoyées automatiquement.

    b. Sur la page **Nouveau service lié**, sélectionnez votre compte de stockage, puis choisissez **Créer** pour déployer le service lié.

    c. Dans la section **Paramètres avancés**, décochez l’option **Utiliser le type par défaut**, puis sélectionnez **Suivant**.

    ![Configurer PolyBase](./media/load-azure-sql-data-warehouse/configure-polybase.png)

10. Dans la page **Résumé**, vérifiez les paramètres, puis cliquez sur **Suivant**.

    ![Page de résumé](./media/load-azure-sql-data-warehouse/summary-page.png)

11. Sur la page **Déploiement**, sélectionnez **Surveiller** pour surveiller le pipeline (tâche). 
 
12. Notez que l’onglet **Surveiller** sur la gauche est sélectionné automatiquement. Lorsque l’exécution du pipeline s’effectue correctement, sélectionnez le lien **CopyFromSQLToSQLDW** dans la colonne **NOM DU PIPELINE** pour afficher les détails de l’exécution d'activité ou réexécuter le pipeline.

    [![Superviser les exécutions de pipelines](./media/load-azure-sql-data-warehouse/pipeline-monitoring.png)](./media/load-azure-sql-data-warehouse/pipeline-monitoring.png#lightbox)

13. Pour revenir à l’affichage des exécutions du pipeline, sélectionnez le lien **Toutes les exécutions de pipelines** affiché en haut de la fenêtre. Sélectionnez **Actualiser** pour actualiser la liste.

    ![Surveiller des exécutions d’activités](./media/load-azure-sql-data-warehouse/activity-monitoring.png)

14. Pour surveiller les détails d’exécution de chaque activité de copie, sélectionnez le lien **Détails** (image de lunettes) sous **NOM DE L’ACTIVITÉ** dans la vue des exécutions d’activités. Vous pouvez suivre les informations détaillées comme le volume de données copiées à partir de la source dans le récepteur, le débit des données, les étapes d’exécution avec une durée correspondante et les configurations utilisées.
    ![Détails du suivi de l'exécution des activités en premier lieu](./media/load-azure-sql-data-warehouse/monitor-activity-run-details-1.png)

    ![Détails du suivi de l'exécution des activités en deuxième lieu](./media/load-azure-sql-data-warehouse/monitor-activity-run-details-2.png)

## <a name="next-steps"></a>Étapes suivantes

Lisez l’article suivant pour en savoir plus sur la prise en charge d’Azure Synapse Analytics :

> [!div class="nextstepaction"]
>[Connecteur Azure Synapse Analytics](connector-azure-sql-data-warehouse.md)