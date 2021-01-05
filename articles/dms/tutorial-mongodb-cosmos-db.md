---
title: 'Tutoriel : Migrer MongoDB hors connexion vers l’API Azure Cosmos DB pour MongoDB'
titleSuffix: Azure Database Migration Service
description: Découvrez comment effectuer une migration hors connexion d’une instance locale de MongoDB vers l’API MongoDB Azure Cosmos DB à l’aide d’Azure Database Migration Service.
services: dms
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: tutorial
ms.date: 01/08/2020
ms.openlocfilehash: 4f3b201d35781d6d33eead0b0a21d38fbb897097
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94966817"
---
# <a name="tutorial-migrate-mongodb-to-azure-cosmos-dbs-api-for-mongodb-offline-using-dms"></a>Tutoriel : migrer MongoDB vers l’API Azure Cosmos DB pour MongoDB hors connexion à l’aide de DMS

Vous pouvez utiliser Azure Database Migration Service pour effectuer une migration hors connexion (ponctuelle) de bases de données à partir d’une instance locale ou cloud de MongoDB vers l’API MongoDB Azure Cosmos DB.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
>
> * Créer une instance Azure Database Migration Service.
> * Créer un projet de migration en utilisant Azure Database Migration Service.
> * Exécuter la migration.
> * Surveiller la migration.

Dans ce tutoriel, vous allez effectuer la migration d’un jeu de données MongoDB entre une machine virtuelle Azure et l’API Azure Cosmos DB pour MongoDB, à l’aide d’Azure Database Migration Service. Si vous n'avez pas encore configuré d'instance source de MongoDB, consultez l'article [Installer et configurer MongoDB sur une machine virtuelle Windows dans Azure](../virtual-machines/windows/install-mongodb.md).

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous devez effectuer les opérations suivantes :

* Les étapes[Exécuter la prémigration](../cosmos-db/mongodb-pre-migration.md), comme l’estimation du débit, le choix d’une clé de partition et la stratégie d’indexation.
* [Créez un compte d’API Azure Cosmos DB pour MongoDB](https://ms.portal.azure.com/#create/Microsoft.DocumentDB).
* Créez un réseau virtuel Microsoft Azure pour Azure Database Migration Service à l’aide du modèle de déploiement Azure Resource Manager, qui fournit une connectivité site à site à vos serveurs sources locaux via [ExpressRoute](../expressroute/expressroute-introduction.md) ou un [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Pour plus d’informations sur la création d’un réseau virtuel, consultez la [documentation sur le réseau virtuel](../virtual-network/index.yml), en particulier les articles sur le démarrage rapide, qui fournissent des informations pas à pas.

    > [!NOTE]
    > Pendant la configuration du réseau virtuel, si vous utilisez ExpressRoute avec le peering réseau à Microsoft, ajoutez ces [points de terminaison](../virtual-network/virtual-network-service-endpoints-overview.md) au sous-réseau où doit être provisionné le service :
    >
    > * Point de terminaison de base de données cible (un point de terminaison SQL ou Cosmos DB, par exemple)
    > * Point de terminaison de stockage
    > * Point de terminaison Service Bus
    >
    > Cette configuration est nécessaire car Azure Database Migration Service ne dispose pas d’une connectivité Internet.

* Vérifiez que les règles de groupe de sécurité de votre réseau virtuel ne bloquent pas les ports de communication suivants : 53, 443, 445, 9354 et 10000-20000. Pour plus d’informations sur le filtrage du trafic de groupe de sécurité réseau de réseau virtuel, consultez l’article [Filtrer le trafic avec les groupes de sécurité réseau](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Ouvrez votre pare-feu Windows pour permettre à Azure Database Migration Service d’accéder au serveur MongoDB source via le port TCP 27017 (par défaut).
* Lorsque vous utilisez une appliance de pare-feu devant vos bases de données sources, vous devrez peut-être ajouter des règles de pare-feu pour permettre à Azure Database Migration Service d’accéder aux bases de données sources pour la migration.

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Inscrire le fournisseur de ressources Microsoft.DataMigration

1. Connectez-vous au portail Azure, sélectionnez **Tous les services**, puis **Abonnements**.

   ![Afficher les abonnements au portail](media/tutorial-mongodb-to-cosmosdb/portal-select-subscription1.png)

2. Sélectionnez l’abonnement dans lequel vous souhaitez créer l’instance Azure Database Migration Service, puis sélectionnez **Fournisseurs de ressources**.

    ![Afficher les fournisseurs de ressources](media/tutorial-mongodb-to-cosmosdb/portal-select-resource-provider.png)

3. Recherchez migration, puis à droite de **Microsoft.DataMigration**, sélectionnez **Inscrire**.

    ![S’inscrire auprès du fournisseur de ressources](media/tutorial-mongodb-to-cosmosdb/portal-register-resource-provider.png)    

## <a name="create-an-instance"></a>Créer une instance

1. Dans le portail Azure, sélectionnez **+ Créer une ressource**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Service** dans la liste déroulante.

    ![Place de marché Azure](media/tutorial-mongodb-to-cosmosdb/portal-marketplace.png)

2. Dans l’écran **Azure Database Migration Service**, sélectionnez **Créer**.

    ![Créer une instance Azure Database Migration Service](media/tutorial-mongodb-to-cosmosdb/dms-create1.png)
  
3. Dans l’écran **Créer un service de migration**, spécifiez un nom pour le service, l’abonnement, et un réseau virtuel nouveau ou existant.

4. Sélectionnez l’emplacement dans lequel vous souhaitez créer l’instance Azure Database Migration Service. 

5. Sélectionnez un réseau virtuel existant ou créez-en un.

    Le réseau virtuel fournit à Azure Database Migration Service un accès à l’instance MongoDB source et au compte Azure Cosmos DB cible.

    Pour plus d’informations sur la création d’un réseau virtuel dans le portail Azure, consultez l’article [Créer un réseau virtuel au moyen du portail Azure](../virtual-network/quick-create-portal.md).

6. Sélectionnez un niveau tarifaire.

    Pour plus d’informations sur les coûts et les niveaux de tarification, consultez la [page de tarification](https://aka.ms/dms-pricing).

    ![Configurer des paramètres d’instance Azure Database Migration Service](media/tutorial-mongodb-to-cosmosdb/dms-settings2.png)

7. Sélectionnez **Créer** pour créer le service.

## <a name="create-a-migration-project"></a>Créer un projet de migration

Une fois le service créé, recherchez-le dans le portail Azure, ouvrez-le, puis créez un projet de migration.

1. Dans le portail Azure, sélectionnez **Tous les services**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Services**.

      ![Localiser toutes les instances Azure Database Migration Service](media/tutorial-mongodb-to-cosmosdb/dms-search.png)

2. Dans l’écran **Azure Database Migration Services**, recherchez le nom de l’instance Azure Database Migration Service que vous avez créée, puis sélectionnez l’instance.

3. Sélectionnez **+ Nouveau projet de migration**.

4. Sur l'écran **Nouveau projet de migration**, spécifiez un nom pour le projet. Dans la zone de texte **Type de serveur source**, sélectionnez **MongoDB**. Dans la zone de texte **Type de serveur cible**, sélectionnez **CosmosDB (API MongoDB)** . Enfin, dans la zone de texte **Choisir un type d’activité**, sélectionnez **Migration de données hors connexion**. 

    ![Créer un projet Azure Database Migration Service](media/tutorial-mongodb-to-cosmosdb/dms-create-project.png)

5. Sélectionnez **Créer et exécuter une activité** pour créer le projet et exécuter l’activité de migration.

## <a name="specify-source-details"></a>Spécifier les détails de la source

1. Sur l'écran **Détails de la source**, spécifiez les détails de connexion du serveur MongoDB source.

   > [!IMPORTANT]
   > Azure Database Migration Service ne prend pas en charge Azure Cosmos DB comme source.

    Il existe trois modes de connexion à une source :
   * **Mode standard**, qui accepte un nom de domaine complet ou une adresse IP, un numéro de port et des informations d’identification de connexion.
   * **Mode de chaîne de connexion**, qui accepte une chaîne de connexion MongoDB comme décrit dans l’article sur le [format d’URI de la chaîne de connexion](https://docs.mongodb.com/manual/reference/connection-string/).
   * **Mode de données du stockage Azure**, qui accepte une URL SAP du conteneur d’objets blob. Sélectionnez **L’objet blob contient des images mémoire BSON** si le conteneur d’objets blob contient des images mémoire BSON produites par l’[outil bsondump](https://docs.mongodb.com/manual/reference/program/bsondump/) MongoDB, et désélectionnez-le si le conteneur contient des fichiers JSON.

     Si vous sélectionnez cette option, vérifiez que la chaîne de connexion de compte de stockage s’affiche au format suivant :

     ```
     https://blobnameurl/container?SASKEY
     ```

     Cette chaîne de connexion SAP du conteneur d’objets blob se trouve dans l’Explorateur Stockage Azure. La création de la signature d’accès partagé pour le conteneur concerné vous fournira l’URL au format requis ci-dessus.
     
     De plus, en fonction des informations d’images mémoire de type qui se trouvent dans le stockage Azure, gardez à l’esprit les informations suivantes.

     * Pour les images mémoire BSON, les données dans le conteneur d’objets blob doivent être au format bsondump, de sorte que les fichiers de données sont placés dans des dossiers nommés d’après les bases de données conteneur au format collection.bson. Les fichiers de métadonnées (le cas échéant) doivent être nommés en utilisant le format *collection*.metadata.json.

     * Pour les images mémoire JSON, les fichiers présents dans le conteneur d’objets blob doivent être placés dans des dossiers nommés d’après les bases de données conteneur. Dans chaque dossier de base de données, les fichiers de données doivent être placés dans un sous-dossier appelé « data » et nommé en utilisant le format *collection*.json. Les fichiers de métadonnées (le cas échéant) doivent être placés dans un sous-dossier appelé « metadata » et nommé en utilisant le même format, *collection*.json. Les fichiers de métadonnées doivent être au même format que celui produit par l’outil de bsondump MongoDB.

    > [!IMPORTANT]
    > Il est déconseillé d’utiliser un certificat autosigné sur le serveur mongo. Toutefois, si un tel certificat est utilisé, connectez-vous au serveur en utilisant le **mode chaîne de connexion** et vérifiez que votre chaîne de connexion a “”
    >
    >```
    >&sslVerifyCertificate=false
    >```

   Vous pouvez également utiliser l’adresse IP quand la résolution de noms DNS est impossible.

   ![Spécifier les détails de la source](media/tutorial-mongodb-to-cosmosdb/dms-specify-source.png)

2. Sélectionnez **Enregistrer**.

## <a name="specify-target-details"></a>Spécifier les détails de la cible

1. Sur l’écran **Détails de la cible de migration**, spécifiez les détails de connexion du compte Azure Cosmos DB cible, à savoir le compte d’API Azure Cosmos DB pour MongoDB préprovisionné vers lequel vous migrez vos données MongoDB.

    ![Spécifier les détails de la cible](media/tutorial-mongodb-to-cosmosdb/dms-specify-target.png)

2. Sélectionnez **Enregistrer**.

## <a name="map-to-target-databases"></a>Mapper aux bases de données cibles

1. Dans l’écran **Mapper aux bases de données cibles**, mappez les bases de données source et cible pour la migration.

    Si la base de données cible porte le même nom que la base de données source, Azure Database Migration Service sélectionne la base de données cible par défaut.

    Si la chaîne **Create** s’affiche en regard du nom de la base de données, cela indique qu’Azure Database Migration Service n’a pas trouvé la base de données cible et qu’il va créer la base de données pour vous.

    À ce stade de la migration, vous pouvez [configurer le débit](../cosmos-db/set-throughput.md). Dans Cosmos DB, vous pouvez configurer le débit au niveau de la base de données ou individuellement pour chaque collection. Le débit est mesuré en [unités de requête](../cosmos-db/request-units.md) (RU). Apprenez-en davantage sur les [tarifs d'Azure Cosmos DB](https://azure.microsoft.com/pricing/details/cosmos-db/).

    ![Mapper aux bases de données cibles](media/tutorial-mongodb-to-cosmosdb/dms-map-target-databases.png)

2. Sélectionnez **Enregistrer**.
3. Sur l'écran **Paramètre de collection**, développez la liste des collections, puis consultez la liste des collections à migrer.

    Azure Database Migration Service sélectionne automatiquement toutes les collections qui se trouvent sur l’instance source de MongoDB et qui ne se trouvent pas dans le compte Azure Cosmos DB cible. Si vous souhaitez migrer à nouveau les collections qui contiennent déjà des données, vous devez sélectionner explicitement ces collections sur ce panneau.

    Vous pouvez spécifier le nombre de RU à utiliser par les collections. Azure Database Migration Service suggère des valeurs par défaut intelligentes en fonction de la taille de la collection.

    > [!NOTE]
    > Effectuez la migration et la collection de la de base de données en parallèle, en utilisant si nécessaire plusieurs instances d’Azure Database Migration Service, afin d’accélérer l’exécution.

    Vous pouvez également spécifier une clé de partition afin de tirer parti du [partitionnement dans Azure Cosmos DB](../cosmos-db/partitioning-overview.md) pour une extensibilité optimale. Veillez à consulter les [meilleures pratiques de sélection d'une clé de partition](../cosmos-db/partitioning-overview.md#choose-partitionkey).

    ![Sélectionner des tables de collections](media/tutorial-mongodb-to-cosmosdb/dms-collection-setting.png)

4. Sélectionnez **Enregistrer**.

5. Dans l’écran **Récapitulatif de la migration**, spécifiez un nom pour l’activité de migration dans la zone de texte **Nom de l’activité**.

    ![Résumé de la migration](media/tutorial-mongodb-to-cosmosdb/dms-migration-summary.png)

## <a name="run-the-migration"></a>Exécuter la migration

* Sélectionnez **Exécuter la migration**.

    La fenêtre d'activité de migration s'affiche. L'**État** de l'activité est **Non démarré**.

    ![État de l'activité](media/tutorial-mongodb-to-cosmosdb/dms-activity-status.png)

## <a name="monitor-the-migration"></a>Surveiller la migration

* Dans l’écran d’activité de migration, sélectionnez **Actualiser** pour mettre à jour l’affichage jusqu’à voir **l’état** de la migration s’afficher comme **Completed** (Terminé).

   > [!NOTE]
   > Vous pouvez sélectionner l'activité pour obtenir des détails sur les métriques de migration au niveau des bases de données et des collections.

    ![État de l'activité - Terminé](media/tutorial-mongodb-to-cosmosdb/dms-activity-completed.png)

## <a name="verify-data-in-cosmos-db"></a>Vérifier les données dans Cosmos DB

* Au terme de la migration, vous pouvez consulter votre compte Azure Cosmos DB pour vérifier que toutes les collections ont bien été migrées.

    ![Capture d’écran montrant où consulter votre compte Azure Cosmos DB pour vérifier que toutes les collections ont bien été migrées](media/tutorial-mongodb-to-cosmosdb/dms-cosmosdb-data-explorer.png)

## <a name="post-migration-optimization"></a>Optimisation de la post-migration

Après avoir migré les données stockées dans la base de données MongoDB vers l’API Azure Cosmos DB pour MongoDB, vous pouvez vous connecter à Azure Cosmos DB et gérer les données. Vous pouvez également effectuer d’autres étapes d’optimisation de la post-migration, comme l’optimisation de la stratégie d’indexation, la mise à jour du niveau de cohérence par défaut ou la configuration de la distribution mondiale pour votre compte Azure Cosmos DB. Pour plus d’informations, consultez l’article [Optimisation de la post-migration](../cosmos-db/mongodb-post-migration.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Informations sur le service Cosmos DB](https://azure.microsoft.com/services/cosmos-db/)

## <a name="next-steps"></a>Étapes suivantes

* Pour accéder à d'autres scénarios, lisez les instructions de migration du [Guide de migration de bases de données](https://datamigration.microsoft.com/) de Microsoft.