---
title: Tutoriel SaaS multilocataire
description: Approvisionner et cataloguer de nouveaux clients à l’aide du modèle d’application autonome
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 09/24/2018
ms.openlocfilehash: 2343800f8801105ca75f285972b441ecb027d1a0
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92793243"
---
# <a name="provision-and-catalog-new-tenants-using-the--application-per-tenant-saas-pattern"></a>Approvisionner et cataloguer de nouveaux clients à l’aide du modèle SaaS d’application par client
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Cet article décrit l’approvisionnement et le catalogage de nouveaux clients à l’aide du modèle SaaS d’application autonome par client.
Cet article comprend deux parties principales :
* Présentation du concept d’approvisionnement et de catalogage de nouveaux clients
* Didacticiel mettant en évidence des exemples de code PowerShell effectuant l’approvisionnement et le catalogage
    * Le didacticiel utilise l’exemple d’application SaaS Wingtip Tickets, adaptée à l’application autonome par modèle de client.

## <a name="standalone-application-per-tenant-pattern"></a>Modèle d’application autonome par client

L’application autonome par modèle de client est l’un des modèles pour les applications SaaS mutualisées.  Dans ce modèle, une application autonome est approvisionnée pour chaque client. L'application comprend des composants de niveau application et une instance d'Azure SQL Database.  Chaque application cliente peut être déployée dans l’abonnement du fournisseur.  Azure offre également un [programme d’applications managées](../../azure-resource-manager/managed-applications/overview.md) dans le cadre duquel une application peut être déployée dans l’abonnement d’un client et gérée par le fournisseur pour le compte du client.

   ![modèle d’application-par-client](./media/saas-standaloneapp-provision-and-catalog/standalone-app-pattern.png)

Lors du déploiement d’une application pour un client, l’application et la base de données sont approvisionnées dans un groupe de ressources créé pour le client.  L’utilisation de groupes de ressources distincts isole les ressources d’application de chaque client et permet de gérer celles-ci de façon indépendante. Dans chaque groupe de ressources, chaque instance d’application est configurée pour accéder directement à la base de données qui lui correspond.  Ce modèle de connexion contraste avec d’autres modèles qui utilisent un catalogue pour répartir les connexions entre l’application et la base de données.  Et comme il n’y a aucun partage de ressources, chaque base de données client doit être approvisionnée avec suffisamment de ressources pour gérer sa charge de pointe. Ce modèle tend à être utilisé pour des applications SaaS avec moins de clients, où l’accent est mis davantage sur l’isolation du client que sur les coûts des ressources.

## <a name="using-a-tenant-catalog-with-the-application-per-tenant-pattern"></a>Utilisation d’un catalogue de clients avec le modèle d’application par client

Si l’application et la base de données de chaque client sont totalement isolées, différents scénarios de gestion et d’analyse peuvent fonctionner sur les divers clients.  Par exemple, l’application d’un changement de schéma pour une nouvelle version de l’application nécessite des modifications du schéma de chaque base de données client. Les scénarios de rapport et d’analyse peuvent également nécessiter un accès à toutes les bases de données client, indépendamment de l’endroit où elles sont déployées.

   ![Diagramme montrant comment utiliser un catalogue de locataire avec le modèle d’application par locataire](./media/saas-standaloneapp-provision-and-catalog/standalone-app-pattern-with-catalog.png)

Le catalogue de clients contient un mappage entre un identificateur de client et une base de données client, ce qui permet qu’un identificateur soit résolu en un serveur et un nom de base de données.  Dans l’application SaaS Wingtip, l’identificateur du client est calculé comme un hachage du nom de client, bien que d’autres schémas pourraient être utilisés.  Si les applications autonomes n’ont pas besoin du catalogue pour gérer les connexions, le catalogue peut être utilisé pour définir l’étendue d’autres actions sur un ensemble de bases de données client. Par exemple, une demande élastique peut utiliser le catalogue pour déterminer l’ensemble des bases de données sur lesquelles les requêtes sont distribuées pour la création de rapports inter-client.

## <a name="elastic-database-client-library"></a>Bibliothèque cliente de base de données élastique

Dans l’exemple d’application SaaS Wingtip, le catalogue est implémenté par les fonctionnalités de gestion de partitionnement de base de données de la [bibliothèque cliente de bases de données élastiques](elastic-database-client-library.md).  La bibliothèque permet à une application de créer, gérer et utiliser une carte de partitions stockée dans une base de données. Dans l’exemple Wingtip Tickets, le catalogue est stocké dans la base de données du *catalogue de clients* .  La partition mappe une clé de client à la partition (base de données) dans laquelle sont stockées les données de ce client.  Les fonctions de bibliothèque cliente de bases de données élastiques gèrent une *carte de partitions globale* stockée dans des tables d’une base de données de *catalogue du client* et une *carte de partitions locale* stockée dans chaque partition.

Les fonctions de bibliothèque cliente de bases de données élastiques peuvent être appelées à partir d’applications ou de scripts PowerShell pour créer et gérer les entrées dans la carte de partitions. D’autres fonctions de la bibliothèque cliente de bases de données élastiques peuvent être utilisées pour récupérer l’ensemble de partitions ou se connecter à la base de données correcte pour une clé de client donnée.

> [!IMPORTANT]
> Ne modifiez pas les données directement dans la base de données du catalogue ou la carte de partitions locale dans les bases de données client. Les mises à jour directes ne sont pas prises en charge en raison du risque élevé d’altération des données. Au lieu de cela, modifiez les données de mappage uniquement à l’aide des API de la bibliothèque cliente de bases de données élastiques.

## <a name="tenant-provisioning"></a>Approvisionnement du client

Chaque client a besoin d’un nouveau groupe de ressources Azure qui doit être créé avant que des ressources puissent y être approvisionnées. Une fois que le groupe de ressources existe, un modèle de gestion des ressources Azure peut être utilisé pour déployer les composants et la base de données de l’application, puis configurez la connexion de base de données. Pour initialiser le schéma de base de données, le modèle peut importer un fichier bacpac.  Il est également possible de créer la base de données en tant que copie d’une base de données « modèle ».  La base de données est ensuite mise à jour avec les données d’emplacement initiales, et inscrite dans le catalogue.

## <a name="tutorial"></a>Didacticiel

Ce didacticiel vous montre comment effectuer les opérations suivantes :

* Approvisionner un catalogue
* Inscrire les exemples de bases de données client déployées précédemment dans le catalogue
* Configurer un client supplémentaire et l’inscrire dans le catalogue

Un modèle Azure Resource Manager est utilisé pour déployer et configurer l’application, créer la base de données client, puis importer un fichier bacpac pour l’initialiser. La demande d’importation peut être mise en file d’attente pendant plusieurs minutes avant d’être déclenchée.

À la fin de ce didacticiel, vous avez un ensemble d’applications clientes autonomes, avec chaque base de données inscrite dans le catalogue.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vérifiez que les prérequis suivants sont remplis :

* Azure PowerShell est installé. Pour plus d’informations, voir [Bien démarrer avec Azure PowerShell](/powershell/azure/get-started-azureps).
* Les trois exemples d’applications clientes sont déployés. Pour déployer ces applications en moins de cinq minutes, voir [Déployer et explorer une application à client unique autonome qui utilise Azure SQL Database](./saas-standaloneapp-get-started-deploy.md).

## <a name="provision-the-catalog"></a>Approvisionner le catalogue

Cette tâche montre comment configurer le catalogue utilisé pour inscrire toutes les bases de données client. Vous allez effectuer les étapes suivantes :

* **Configurer la base de données du catalogue** à l’aide d’un modèle de gestion des ressources Azure. La base de données est initialisée en important un fichier bacpac.
* **Inscrire les exemples d’applications clientes** que vous avez déployés précédemment.  Chaque client est inscrit à l’aide d’une clé créée à partir d’un hachage du nom du client.  Le nom du client est également stocké dans une table des extensions à l’intérieur du catalogue.

1. Dans PowerShell ISE, ouvrez *...\Learning Modules\UserConfig.psm* , puis remplacez la valeur **\<user\>** par celle que vous avez utilisée lors du déploiement des trois exemples d'applications.  **Enregistrez le fichier** .
1. Dans PowerShell ISE, ouvrez *...\Learning Modules\ProvisionTenants\Demo-ProvisionAndCatalog.ps1* et définissez **$Scenario = 1** . Déployez le catalogue de clients et inscrivez les clients prédéfinis.

1. Ajoutez un point d’arrêt en plaçant votre curseur n’importe où sur la ligne contenant `& $PSScriptRoot\New-Catalog.ps1`, puis appuyez sur **F9** .

    ![définition d’un point d’arrêt pour le suivi](./media/saas-standaloneapp-provision-and-catalog/breakpoint.png)

1. Exécutez le script en appuyant sur **F5** .
1.  Quand l’exécution du script stoppe au point d’arrêt, appuyez sur **F11** pour effectuer un pas à pas détaillé du script New-Catalog.ps1.
1.  Suivez l’exécution du script à l’aide des options du menu Débogage, F10 et F11, pour parcourir les fonctions appelées.
    *   Pour plus d’informations sur le débogage des scripts PowerShell, consultez [Conseils sur l’utilisation et le débogage de scripts PowerShell](/powershell/scripting/components/ise/how-to-debug-scripts-in-windows-powershell-ise).

Une fois l’exécution du script terminée, le catalogue existe et tous les exemples de client sont inscrits.

Examinez à présent les ressources que vous avez créées.

1. Ouvrez le [portail Azure](https://portal.azure.com/) et parcourez les groupes de ressources.  Ouvrez le groupe de ressources **wingtip-sa-catalog-\<user\>** et notez le serveur de catalogue ainsi que la base de données.
1. Ouvrez la base de données dans le portail, puis sélectionnez l’ *explorateur de données* à partir du menu de gauche.  Cliquez sur la commande de connexion, puis entrez le mot de passe = **P\@ssword1** .


1. Explorez le schéma de la base de données *tenantcatalog* .
   * Les objets dans le schéma `__ShardManagement` sont fournis par la bibliothèque cliente de base de données élastique.
   * Le tableau `Tenants` et la vue `TenantsExtended` sont des extensions ajoutées à l’exemple, qui montrent comment étendre le catalogue afin de fournir une valeur supplémentaire.
1. Exécutez la requête `SELECT * FROM dbo.TenantsExtended`.

   ![explorateur de données](./media/saas-standaloneapp-provision-and-catalog/data-explorer-tenantsextended.png)

    En guise d’alternative à l’utilisation de l’Explorateur de données, vous pouvez vous connecter à la base de données à partir de SQL Server Management Studio. Pour ce faire, connectez-vous au serveur wingtip-


    Notez que vous ne devez pas modifier les données directement dans le catalogue, mais toujours utiliser les API de gestion des partitions.

## <a name="provision-a-new-tenant-application"></a>Configurer une nouvelle application cliente

Dans le cadre de cette tâche, vous allez apprendre à approvisionner une application cliente. Vous allez effectuer les étapes suivantes :

* **Créez un groupe de ressources** pour le client.
* **Configurez l’application et la base de données** dans le nouveau groupe de ressources à l’aide d’un modèle de gestion de ressources Azure.  Cette action inclut l’initialisation de la base de données avec un schéma commun et des données de référence en important un fichier bacpac.
* **Initialisez la base de données avec les informations du client de base** . Cette action inclut la spécification du type de lieu, qui détermine la photographie utilisée comme arrière-plan sur le site web des événements.
* **Inscrivez la base de données dans la base de données du catalogue** .

1. Dans PowerShell ISE, ouvrez *...\Learning Modules\ProvisionTenants\Demo-ProvisionAndCatalog.ps1* et définissez **$Scenario = 2** . Déployer le catalogue de clients et inscrire les clients prédéfinis

1. Ajoutez un point d’arrêt dans le script en plaçant votre curseur n’importe où sur la ligne 49 contenant `& $PSScriptRoot\New-TenantApp.ps1`, puis appuyez sur **F9** .
1. Exécutez le script en appuyant sur **F5** .
1.  Quand l’exécution du script stoppe au point d’arrêt, appuyez sur **F11** pour effectuer un pas à pas détaillé du script New-Catalog.ps1.
1.  Suivez l’exécution du script à l’aide des options du menu Débogage, F10 et F11, pour parcourir les fonctions appelées.

Une fois le client approvisionné, site web d’événements du nouveau client s’ouvre.

   ![course de l’érable rouge](./media/saas-standaloneapp-provision-and-catalog/redmapleracing.png)

Vous pouvez ensuite inspecter les ressources créées dans le portail Azure.

   ![ressources de course de l’érable rouge](./media/saas-standaloneapp-provision-and-catalog/redmapleracing-resources.png)


## <a name="to-stop-billing-delete-resource-groups"></a>Pour arrêter la facturation, supprimer des groupes de ressources

Lorsque vous avez terminé l’exploration de l’exemple, supprimez tous les groupes de ressources que vous avez créés pour arrêter la facturation associée à ceux-ci.

## <a name="additional-resources"></a>Ressources supplémentaires

- Pour en savoir plus sur les applications de base de données SaaS mutualisées, voir [Concevoir des modèles pour des applications SaaS mutualisées](saas-tenancy-app-design-patterns.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Déployer l’application autonome SaaS Wingtip Tickets.
> * Explorer les serveurs et les bases de données qui composent l’application.
> * Comment supprimer les exemples de ressources pour arrêter la facturation associée.

Vous pouvez explorer la manière dont le catalogue est utilisé pour prendre en charge différents scénarios inter-clients à l’aide de la version base de données par client de [l’application SaaS de Tickets Wingtip](./saas-dbpertenant-wingtip-app-overview.md).