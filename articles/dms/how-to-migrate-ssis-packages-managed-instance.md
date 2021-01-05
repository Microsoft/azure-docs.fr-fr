---
title: Migrer des packages SSIS vers SQL Managed Instance
titleSuffix: Azure Database Migration Service
description: Découvrez comment migrer des packages et projets SQL Server Integration Services (SSIS) vers Azure SQL Managed Instance à l’aide d’Azure Database Migration Service ou de l’Assistant Migration de données.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: how-to
ms.date: 02/20/2020
ms.openlocfilehash: 01370092c5e272fe64f4ffdad577b69d3a532810
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012150"
---
# <a name="migrate-sql-server-integration-services-packages-to-an-azure-sql-managed-instance"></a>Migrer les packages SQL Server Integration Services vers une instance managée Azure SQL Managed Instance
Si vous utilisez SQL Server Integration Services (SSIS) et souhaitez migrer vos projets/packages SSIS de la source SSISDB hébergée par SQL Server vers la destination SSISDB hébergée par une instance managée Azure SQL Managed Instance, vous pouvez effectuer cette migration à l’aide d’Azure Database Migration Service.

Si votre version de SSIS est antérieure à 2012, ou si vous utilisez des magasins de packages d’autres types que SSISDB, vous devez convertir vos projets/packages SSIS avant de les migrer, en utilisant l’Assistant Conversion de projet Integration Services disponible dans SSMS. Pour plus d’informations, voir l’article [Conversion de projets en modèle de déploiement de projet](/sql/integration-services/packages/deploy-integration-services-ssis-projects-and-packages?view=sql-server-2017#convert).

> [!NOTE]
> Azure Database Migration Service (DMS) ne prend pas actuellement en charge Azure SQL Database comme destination de migration cible. Pour redéployer des projets/packages SSIS vers Azure SQL Database, consultez l’article [Redéployer des packages SQL Server Integration Services vers Azure SQL Database](./how-to-migrate-ssis-packages.md).

Dans cet article, vous apprendrez comment :
> [!div class="checklist"]
>
> * Évaluer des projets/packages SSIS sources.
> * Migrer des projets/packages SSIS vers Azure.

## <a name="prerequisites"></a>Prérequis

Pour effectuer cette procédure, vous avez besoin de :

* Créer un réseau virtuel Microsoft Azure pour Azure Database Migration Service à l’aide du modèle de déploiement Azure Resource Manager, qui fournit une connectivité de site à site à vos serveurs sources locaux en utilisant [ExpressRoute](../expressroute/expressroute-introduction.md) ou [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). Pour plus d’informations, consultez l’article [Topologies de réseau pour les migrations SQL Managed Instance à l’aide d’Azure Database Migration Service]( https://aka.ms/dmsnetworkformi). Pour plus d’informations sur la création d’un réseau virtuel, consultez la [documentation sur le réseau virtuel](../virtual-network/index.yml), en particulier les articles sur le démarrage rapide, qui fournissent des informations pas à pas.
* Vérifier que les règles du groupe de sécurité réseau de votre réseau virtuel ne bloquent pas les ports suivants pour les communications entrantes vers Azure Database Migration Service : 443, 53, 9354, 445, 12000. Pour plus d’informations sur le filtrage du trafic de groupe de sécurité réseau de réseau virtuel, consultez l’article [Filtrer le trafic avec les groupes de sécurité réseau](../virtual-network/virtual-network-vnet-plan-design-arm.md).
* Configurer votre [Pare-feu Windows pour l’accès au moteur de base de données source](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access?view=sql-server-2017).
* Ouvrir votre Pare-feu Windows et permettre à Azure Database Migration Service d’accéder au serveur SQL Server source, par défaut sur le port TCP 1433.
* Si vous exécutez plusieurs instances nommées de SQL Server avec des ports dynamiques, vous pouvez activer le service SQL Browser et autoriser l’accès au port UDP 1434 à travers vos pare-feu, de sorte qu’Azure Database Migration Service puisse se connecter à une instance nommée sur votre serveur source.
* Si vous utilisez une appliance de pare-feu devant vos bases de données sources, vous devrez peut-être ajouter des règles de pare-feu pour permettre à Azure Database Migration Service d’accéder aux bases de données sources pour la migration, ainsi qu’aux fichiers par le biais du port SMB 445.
* Instance SQL Managed Instance pour héberger SSISDB. Si vous devez en créer une, suivez les indications détaillées dans l’article [Créer une instance Azure SQL Managed Instance](../azure-sql/managed-instance/instance-create-quickstart.md).
* Vous assurer que les comptes de connexion utilisés pour connecter l’instance SQL Server source et l’instance managée cible sont membres du rôle serveur sysadmin.
* Vérifier que SSIS est provisionné dans une instance Azure Data Factory (ADF) contenant Azure-SSIS Integration Runtime avec la SSISDB de destination qui est hébergée par une instance SQL Managed Instance (comme décrit dans l’article [Créer le runtime d’intégration Azure-SSIS IR dans Azure Data Factory](../data-factory/create-azure-ssis-integration-runtime.md)).

## <a name="assess-source-ssis-projectspackages"></a>Évaluer des projets/packages SSIS sources

Bien que l’évaluation de la SSISDB source ne soit pas encore intégrée dans l’Assistant Migration de base de données (DMA), vos projets/packages SSIS seront évalués/validés lors de leur redéploiement vers la SSISDB de destination hébergée sur une instance Azure SQL Managed Instance.

## <a name="register-the-microsoftdatamigration-resource-provider"></a>Inscrire le fournisseur de ressources Microsoft.DataMigration

1. Connectez-vous au portail Azure, sélectionnez **Tous les services**, puis **Abonnements**.

    ![Afficher les abonnements au portail](media/how-to-migrate-ssis-packages-mi/portal-select-subscriptions.png)

2. Sélectionnez l’abonnement dans lequel vous voulez créer l’instance Azure Database Migration Service, puis sélectionnez **Fournisseurs de ressources**.

    ![Afficher les fournisseurs de ressources](media/how-to-migrate-ssis-packages-mi/portal-select-resource-provider.png)

3. Recherchez migration, puis à droite de **Microsoft.DataMigration**, sélectionnez **Inscrire**.

    ![S’inscrire auprès du fournisseur de ressources](media/how-to-migrate-ssis-packages-mi/portal-register-resource-provider.png)

## <a name="create-an-azure-database-migration-service-instance"></a>Créer une instance Azure Database Migration Service

1. Dans le portail Azure, sélectionnez **+ Créer une ressource**, recherchez **Azure Database Migration Service**, puis sélectionnez **Azure Database Migration Service** dans la liste déroulante.

     ![Place de marché Azure](media/how-to-migrate-ssis-packages-mi/portal-marketplace.png)

2. Dans l’écran **Azure Database Migration Service**, sélectionnez **Créer**.

    ![Créer une instance Azure Database Migration Service](media/how-to-migrate-ssis-packages-mi/dms-create1.png)

3. Dans l’écran **Créer un service de migration**, spécifiez un nom pour le service, l’abonnement, et un réseau virtuel nouveau ou existant.

4. Sélectionnez l’emplacement au niveau duquel vous souhaitez créer l’instance DMS.

5. Sélectionnez un réseau virtuel existant ou créez-en un.

    Le réseau virtuel fournit à Azure Database Migration Service un accès au serveur SQL Server source et à l’instance Azure SQL Managed Instance cible.

    Pour plus d’informations sur la création d’un réseau virtuel dans le Portail Azure, consultez l’article [Créer un réseau virtuel à l’aide du Portail Azure](../virtual-network/quick-create-portal.md).

    Pour plus d’informations, consultez l’article [Topologies de réseau pour les migrations d’instances managées d’Azure SQL Database à l’aide du service Azure Database Migration Service](./resource-network-topologies.md).

6. Sélectionnez un niveau tarifaire.

    Pour plus d’informations sur les coûts et les niveaux de tarification, consultez la [page de tarification](https://aka.ms/dms-pricing).

    ![Créer un service DMS](media/how-to-migrate-ssis-packages-mi/dms-create-service2.png)

7. Sélectionnez **Créer** pour créer le service.

## <a name="create-a-migration-project"></a>Créer un projet de migration

Une fois qu’une instance du service a été créée, recherchez-la dans le Portail Azure, ouvrez-la, puis créez un projet de migration.

1. Dans le portail Azure, sélectionnez **Tous les services**, recherchez Azure Database Migration Service, puis sélectionnez **Azure Database Migration Services**.

    ![Rechercher toutes les instances Azure Database Migration Service](media/how-to-migrate-ssis-packages-mi/dms-search.png)

2. Sur l’écran **Azure Database Migration Service**, recherchez le nom de l’instance que vous avez créée, puis sélectionnez-la.

3. Sélectionnez **+ Nouveau projet de migration**.

4. Dans l’écran **Nouveau projet de migration**, spécifiez un nom pour le projet. Dans la zone de texte **Type de serveur source**, sélectionnez **SQL Server**. Dans la zone de texte **Type de serveur cible**, sélectionnez **Azure SQL Managed Instance**. Enfin, dans la zone de texte **Choisir un type d’activité**, sélectionnez **Migration du package SSIS**.

   ![Créer un projet DMS](media/how-to-migrate-ssis-packages-mi/dms-create-project2.png)

5. Sélectionnez **Créer** pour créer le projet.

## <a name="specify-source-details"></a>Spécifier les détails de la source

1. Dans l’écran **Détails de la source de migration**, spécifiez les détails de connexion du serveur SQL Server source.

2. Si vous n’avez pas installé de certificat approuvé sur votre serveur, cochez la case **Faire confiance au certificat de serveur**.

    Quand aucun certificat approuvé n’est installé, SQL Server génère un certificat auto-signé au démarrage de l’instance. Ce certificat permet de chiffrer les informations d’identification des connexions clientes.

    > [!CAUTION]
    > Les connexions TLS chiffrées à l’aide d’un certificat auto-signé n’offrent pas de sécurité renforcée. Elles sont vulnérables aux attaques de l’intercepteur. Ne faites jamais confiance à une connexion TLS utilisant des certificats autosignés dans un environnement de production ou sur des serveurs connectés à Internet.

   ![Détails de la source](media/how-to-migrate-ssis-packages-mi/dms-source-details1.png)

3. Sélectionnez **Enregistrer**.

## <a name="specify-target-details"></a>Spécifier les détails de la cible

1. Dans l’écran **Détails de la cible de migration**, spécifiez les détails de connexion pour la cible.

     ![Détails de la cible](media/how-to-migrate-ssis-packages-mi/dms-target-details2.png)

2. Sélectionnez **Enregistrer**.

## <a name="review-the-migration-summary"></a>Examiner le récapitulatif de la migration

1. Dans l’écran **Récapitulatif de la migration**, spécifiez un nom pour l’activité de migration dans la zone de texte **Nom de l’activité**.

2. Pour l’**option de remplacement des projets et environnements SSIS**, spécifiez s’il faut remplacer ou ignorer les projets et environnements SSIS existants.

    ![Récapitulatif du projet de migration](media/how-to-migrate-ssis-packages-mi/dms-project-summary2.png)

3. Examinez et vérifiez les détails associés au projet de migration.

## <a name="run-the-migration"></a>Exécuter la migration

* Sélectionnez **Exécuter la migration**.

## <a name="next-steps"></a>Étapes suivantes

* Lisez l’aide à la migration du [Guide de migration de bases de données](https://datamigration.microsoft.com/) de Microsoft.