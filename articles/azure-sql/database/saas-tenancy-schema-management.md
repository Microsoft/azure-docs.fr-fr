---
title: Gérer le schéma dans une application à un seul locataire
description: Découvrez comment gérer un schéma pour plusieurs locataires dans une application monolocataire qui utilise Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 09/19/2018
ms.openlocfilehash: e4328be0aade0658dedb034dbbb6980b810f771a
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92793192"
---
# <a name="manage-schema-in-a-saas-application-using-the-database-per-tenant-pattern-with-azure-sql-database"></a>Gérer le schéma dans une application SaaS à l’aide du modèle de base de données par locataire avec Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]
 
Etant donné qu’une application de base de données évolue, des modifications doivent inévitablement être effectuées sur le schéma de base de données ou les données de référence.  Des tâches de maintenance de la base de données sont aussi régulièrement nécessaires. La gestion d’une application qui utilise le modèle de base de données par locataire requiert que vous appliquiez ces modifications ou tâches de maintenance sur l’ensemble d’un parc de bases de données de locataire.

Ce didacticiel explore deux scénarios : le déploiement de mises à jour des données de référence pour tous les locataires et la reconstruction d’un index sur la table contenant les données de référence. La fonctionnalité [Travaux élastiques](./elastic-jobs-overview.md) est utilisée pour exécuter ces actions sur toutes les bases de données de locataire et sur la base de données modèle utilisée pour créer des bases de données de locataire.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> 
> * Créer un agent de travail
> * Entraîner l’exécution de travaux T-SQL sur toutes les bases de données de locataire
> * Mettre à jour les données de référence dans toutes les bases de données de locataire
> * Créer un index sur une table dans toutes les bases de données de locataire


Pour suivre ce didacticiel, vérifiez que les conditions préalables ci-dessous sont bien satisfaites :

* L’application de base de données Wingtip Tickets SaaS par client est déployée. Pour procéder à un déploiement en moins de cinq minutes, consultez [Déployer et explorer l’application de base de données par locataire SaaS Wingtip Tickets](./saas-dbpertenant-get-started-deploy.md)
* Azure PowerShell est installé. Pour plus d’informations, voir [Bien démarrer avec Azure PowerShell](/powershell/azure/get-started-azureps).
* La dernière version de SQL Server Management Studio (SSMS) est installée. [Télécharger et installer SSMS](/sql/ssms/download-sql-server-management-studio-ssms)


## <a name="introduction-to-saas-schema-management-patterns"></a>Présentation des modèles de gestion de schéma SaaS

Le modèle de base de données par locataire isole efficacement les données de locataire, mais augmente le nombre de bases de données à gérer et à entretenir. Les [travaux élastiques](./elastic-jobs-overview.md) facilitent l’administration et la gestion de plusieurs bases de données. Les travaux vous permettent d’exécuter de façon sécurisée et fiable des tâches (scripts T-SQL), sur un groupe de bases de données. Les travaux peuvent déployer les modifications de schéma et de données de référence communes sur toutes les bases de données de locataire d’une application. Les travaux élastiques permettent également de maintenir à jour un *modèle* de la base de données utilisée pour créer de nouveaux locataires afin de s’assurer qu’elle contient en permanence le schéma et les données de référence les plus récents.

![Écran](./media/saas-tenancy-schema-management/schema-management-dpt.png)


## <a name="elastic-jobs-public-preview"></a>Préversion publique des tâches élastiques

Il existe une nouvelle version des travaux élastiques qui est désormais une fonctionnalité intégrée d’Azure SQL Database. Cette nouvelle version des tâches élastiques est pour le moment en préversion publique. Cette préversion publique prend actuellement en charge l’utilisation de PowerShell pour créer un agent de travail et de T-SQL pour créer et gérer des tâches.
Consultez l’article [Tâches de base de données élastique](./elastic-jobs-overview.md) pour plus d’informations.

## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>Obtenir les scripts de l’application de base de données par locataire SaaS Wingtip Tickets

Le code source de l’application et les scripts de gestion sont disponibles dans le référentiel GitHub [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant). Consultez les [conseils généraux](saas-tenancy-wingtip-app-guidance-tips.md) avant de télécharger et de débloquer les scripts Wingtip Tickets SaaS.

## <a name="create-a-job-agent-database-and-new-job-agent"></a>Créer une base de données d’agent de travail et un nouvel agent de travail

Ce didacticiel nécessite l’utilisation de PowerShell pour créer un agent de travail et la base de données d’agent de travail correspondante. La base de données d’agent de travail conserve les définitions des travaux, l’état du travail et l’historique. Une fois l’agent de travail et sa base de données créés, vous pouvez immédiatement créer et surveiller des travaux.

1. **Dans PowerShell ISE** , ouvrez …\\Learning Modules\\Schema Management\\*Demo-SchemaManagement.ps1* .
1. Appuyez sur **F5** pour exécuter le script.

Le script *Demo-SchemaManagement.ps1* appelle le script *Deploy-SchemaManagement.ps1* pour créer une base de données nommée *osagent* sur le serveur de catalogue. Il crée ensuite l’agent de travail, à l’aide de la base de données en tant que paramètre.

## <a name="create-a-job-to-deploy-new-reference-data-to-all-tenants"></a>Créer un travail pour déployer les nouvelles données de référence sur tous les locataires

Dans l’application Wingtip Tickets, chaque base de données de locataire inclut un ensemble de types de lieux pris en charge. Chaque lieu est d’un type spécifique, qui définit le type des événements qui peuvent être hébergés et détermine l’image d’arrière-plan utilisée dans l’application. Pour que l’application prenne en charge de nouveaux types d’événements, ces données de référence doivent être mises à jour et de nouveaux types de lieux doivent être ajoutés.  Dans cet exercice, vous allez déployer une mise à jour dans toutes les bases de données du locataire afin d’ajouter deux types de lieux supplémentaires : *Motorcycle Racing* (Courses de moto) et *Swimming Club* (Club de natation).

Tout d’abord, examinez les types de lieux inclus dans chaque base de données client. Connectez-vous à l’une des bases de données client dans SQL Server Management Studio (SSMS) et vérifiez la table VenueTypes.  Vous pouvez également interroger cette table dans l’éditeur de requêtes du portail Azure, auquel vous avez accès par la page de la base de données. 

1. Ouvrez SSMS et connectez-vous au serveur client : *tenants1-dpt-&lt;utilisateur&gt;.database.windows.net*
1. Pour confirmer que *Motorcycle Racing* et *Swimming Club* **ne sont pas** déjà inclus, accédez à la base de données _contosoconcerthall_ sur le serveur *tenants1-dpt-&lt;utilisateur&gt;* et interrogez la table *VenueTypes* .

Maintenant, nous allons créer un travail pour mettre à jour la table *VenueTypes* dans toutes les bases de données de locataire pour ajouter les nouveaux types de lieux.

Pour créer un travail, vous utilisez un ensemble de procédures stockées système dédiées aux travaux créées dans la base de données _jobagent_ lors de la création de l’agent de travail.

1. Dans SSMS, connectez-vous au serveur de catalogue : *catalog-dpt-&lt;user&gt;.database.windows.net* 
1. Dans SSMS, ouvrez le fichier ...\\Learning Modules\\Schema Management\\DeployReferenceData.sql
1. Modifiez l’instruction : SET @wtpUser = &lt;utilisateur&gt; et remplacer la valeur de l’utilisateur utilisée lors du déploiement de l’application Wingtip Tickets SaaS Database Per Tenant
1. Assurez-vous que vous êtes connecté à la base de données _jobagent_ , puis appuyez sur **F5** pour exécuter le script

Observez les éléments suivants dans le script *DeployReferenceData.sql* :
* **sp\_add\_target\_group** crée le nom de groupe cible DemoServerGroup.
* **sp\_add\_target\_group\_member** est utilisé pour définir l’ensemble des bases de données cibles.  Premièrement, le serveur _tenants1-dpt-&lt;user&gt;_ est ajouté.  Avec l’ajout du serveur en tant que cible, les bases de données du serveur sont incluses dans le travail au moment de l’exécution de celui-ci. Puis, la base de données _basetenantdb_ et la base de données *adhocreporting* (utilisée dans un autre didacticiel) sont ajoutées en tant que cibles.
* **sp\_add\_job** crée un travail appelé _Reference Data Deployment_ (Déploiement des données de référence).
* **sp\_add\_jobstep** crée l’étape du travail contenant le texte de la commande T-SQL pour mettre à jour la table de référence, VenueTypes.
* Les autres vues dans le script indiquent l’existence des objets et contrôlent l’exécution du travail. Utilisez ces requêtes pour passer en revue la valeur d’état dans la colonne **cycle de vie** afin de déterminer le moment où le travail a été terminé sur toutes les bases de données cibles.

Une fois le script terminé, vous pouvez vérifier que les données de référence ont été mises à jour.  Dans SSMS, accédez à la base de données *contosoconcerthall* sur le serveur *tenants1-dpt-&lt;user&gt;* et interrogez la table *VenueTypes* .  Vérifiez que *Motorcycle Racing* et *Swimming Club* **sont** désormais présents.


## <a name="create-a-job-to-manage-the-reference-table-index"></a>Créer une tâche pour gérer l’index de la table de référence

Cet exercice utilise un travail pour reconstruire l’index sur la clé primaire de la table de référence.  Il s’agit d’une opération de maintenance de base de données traditionnelle qui peut être effectuée après le chargement de grandes quantités de données.

Créez un travail en utilisant les mêmes procédures stockées « système » dédiées aux travaux.

1. Ouvrez SSMS et connectez-vous au serveur _catalog-dpt-&lt;user&gt;.database.windows.net_
1. Ouvrez le fichier _…\\Learning Modules\\Schema Management\\OnlineReindex.sql_
1. Si vous n’êtes pas déjà connecté, cliquez avec le bouton droit, sélectionnez Connexion et connectez-vous au serveur _catalog-dpt-&lt;user&gt;.database.windows.net_
1. Assurez-vous que vous êtes connecté à la base de données _jobagent_ , puis appuyez sur **F5** pour exécuter le script

Observez les éléments suivants dans le script _OnlineReindex.sql_ :
* sp **sp\_add\_job** crée un travail appelé « Online Reindex PK\_\_VenueTyp\_\_265E44FD7FD4C885 »
* **sp\_add\_jobstep** crée l’étape du travail contenant le texte de la commande T-SQL pour mettre à jour l’index
* Les vues restantes dans le script surveillent l’exécution du travail. Utilisez ces requêtes pour passer en revue la valeur d’état dans la colonne **cycle de vie** afin de déterminer le moment où la tâche a été terminée avec succès sur tous les membres du groupe cible.



## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> 
> * Créer un travail d’agent à exécuter sur des travaux T-SQL dans plusieurs bases de données
> * Mettre à jour les données de référence dans toutes les bases de données de locataire
> * Créer un index sur une table dans toutes les bases de données de locataire

Ensuite, consultez le [didacticiel de génération d’états ad hoc](./saas-tenancy-cross-tenant-reporting.md) pour explorer l’exécution de requêtes distribuées dans les bases de données de locataire.


## <a name="additional-resources"></a>Ressources supplémentaires

* [Autres didacticiels reposant sur le déploiement de l’application de base de données Wingtip Tickets SaaS par client](./saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [Gestion des bases de données cloud avec montée en charge](./elastic-jobs-overview.md)