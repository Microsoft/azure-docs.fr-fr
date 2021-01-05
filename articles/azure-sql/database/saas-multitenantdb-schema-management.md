---
title: Gérer le schéma dans une application multilocataire
description: Gérer un schéma pour plusieurs locataires dans une application multilocataire qui utilise Azure SQL Database
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
ms.openlocfilehash: d222234cd6ff3d910e6dbc51a394695ce467edce
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011850"
---
# <a name="manage-schema-in-a-saas-application-that-uses-sharded-multi-tenant-databases"></a>Gérer un schéma dans une application SaaS qui utilise des bases de données mutualisées partitionnées
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Ce didacticiel examine les défis liés du maintien d’un parc de bases de données dans une application SaaS (Software as a Service). Les solutions sont présentées pour la ventilation des modifications de schéma dans le parc de bases de données.

Comme n’importe quelle application, l’application SaaS Wingtip Tickets évolue au fil du temps et nécessite l’apport de modifications à la base de données. Les modifications peuvent avoir un impact sur les données de schéma ou de référence ou appliquer des tâches de maintenance des bases de données. Avec une application SaaS qui utilise une base de données par modèle de client, les modifications doivent être coordonnées dans un parc potentiellement massif de bases de données client. En outre, vous devez intégrer ces modifications dans le processus d’approvisionnement de bases de données pour vous assurer qu’elles sont incluses dans les nouvelles bases de données, telles que créées.

#### <a name="two-scenarios"></a>Deux scénarios

Ce didacticiel explore les deux scénarios suivants :
- Déployer des mises à jour des données de référence pour tous les locataires.
- Régénérer un index sur la table qui contient les données de référence.

La fonctionnalité [Tâches élastiques](./elastic-jobs-overview.md) d’Azure SQL Database est utilisée pour exécuter ces opérations sur les bases de données de locataire. Les travaux fonctionnent également sur la base de données de client « modèle ». Dans l’exemple d’application Wingtip Tickets, cette base de données modèle est copiée pour provisionner une nouvelle base de données client.

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un agent de travail.
> * Exécuter une requête T-SQL sur plusieurs bases de données client.
> * Mettre à jour les données de référence dans toutes les bases de données client.
> * Créer un index sur une table dans toutes les bases de données de locataire.

## <a name="prerequisites"></a>Prérequis

- L’application de bases de données mutualisées Wingtip Tickets doit déjà être déployée :
    - Pour obtenir des instructions, consultez le premier didacticiel qui introduit l’application de bases de données mutualisées SaaS Wingtip Tickets :<br />[Déployer et explorer une application multilocataire partitionnée qui utilise Azure SQL Database](./saas-multitenantdb-get-started-deploy.md).
        - Le processus de déploiement dure moins de cinq minutes.
    - Vous devez avoir la version *mutualisée partitionnée* de Wingtip installée. Les versions *Autonome* et *Base de données par client* ne prennent pas en charge ce didacticiel.

- La dernière version de SQL Server Management Studio (SSMS) doit être installée. [Téléchargez et installez SSMS](/sql/ssms/download-sql-server-management-studio-ssms).

- Azure PowerShell doit être installé. Pour plus d’informations, consultez [Prise en main d’Azure PowerShell](/powershell/azure/get-started-azureps).

> [!NOTE]
> Ce tutoriel utilise des fonctionnalités du service Azure SQL Database en préversion limitée ([tâches de base de données élastiques](elastic-database-client-library.md)). Si vous souhaitez réaliser ce didacticiel, envoyez votre ID d’abonnement à *SaaSFeedback\@microsoft.com* avec l’objet Préversion des travaux élastiques. Une fois que vous avez reçu la confirmation que votre abonnement a été activé, [téléchargez et installez les dernières applets de commande pour les travaux en version préliminaire](https://github.com/jaredmoo/azure-powershell/releases). Cette préversion est limitée. Contactez *SaaSFeedback\@microsoft.com* pour toute question ou demande de support associée.

## <a name="introduction-to-saas-schema-management-patterns"></a>Présentation des modèles de gestion de schéma SaaS

Le modèle de base de données partitionnée mutualisée utilisé dans cet exemple permet à une base de données de locataire de contenir un ou plusieurs locataires. Cet exemple présente la possibilité d’utiliser une combinaison de bases de données mutualisées et à locataire unique, créant ainsi un modèle de gestion de locataire *hybride*. La gestion des modifications apportées à ces bases de données peut être compliquée. Les [travaux élastiques](./elastic-jobs-overview.md) facilitent l’administration et la gestion d’un grand nombre de bases de données. Les tâches vous permettent d’exécuter de façon sécurisée et fiable des scripts Transact-SQL en tant que tâches, sur un groupe de bases de données de locataire. Les tâches sont indépendants de la saisie ou de l’interaction utilisateur. Cette méthode peut être utilisée pour déployer les modifications de schéma ou de données de référence commune sur tous les locataires d’une application. Les Tâches élastiques permettent également de conserver une copie du modèle finale de la base de données. Le modèle est utilisé pour créer de nouveaux locataires, afin de s’assurer que le schéma et les données de référence les plus récents sont utilisés.

![Écran](./media/saas-multitenantdb-schema-management/schema-management.png)

## <a name="elastic-jobs-limited-preview"></a>Préversion limitée des travaux élastiques

Il existe une nouvelle version des travaux élastiques qui est désormais une fonctionnalité intégrée d’Azure SQL Database. Cette nouvelle version des travaux élastiques est pour le moment en préversion limitée. Cette préversion limitée prend actuellement en charge l’utilisation de PowerShell pour créer un agent de travail et de T-SQL pour créer et gérer des travaux.
> [!NOTE]
> Ce didacticiel utilise des fonctionnalités du service SQL Database en préversion limitée (travaux de base de données élastiques). Si vous souhaitez réaliser ce didacticiel, envoyez votre ID d’abonnement à SaaSFeedback@microsoft.com avec l’objet Préversion des travaux élastiques. Une fois que vous avez reçu la confirmation que votre abonnement a été activé, téléchargez et installez les dernières cmdlets des travaux en version préliminaire. Cette version préliminaire est limitée. Contactez SaaSFeedback@microsoft.com pour toute question ou demande de prise en charge associées.

## <a name="get-the-wingtip-tickets-saas-multi-tenant-database-application-source-code-and-scripts"></a>Obtenir les scripts et le code source de l’application de base de données multi-locataire SaaS Wingtip Tickets

Les scripts de base de données Wingtip Tickets SaaS mutualisée et le code source de l’application sont disponibles dans le référentiel [WingtipTicketsSaaS-MultiTenantDB](https://github.com/microsoft/WingtipTicketsSaaS-MultiTenantDB) sur GitHub. Consultez les [conseils généraux](saas-tenancy-wingtip-app-guidance-tips.md) avant de télécharger et de débloquer les scripts Wingtip Tickets SaaS.

## <a name="create-a-job-agent-database-and-new-job-agent"></a>Créer une base de données d’agent de travail et un nouvel agent de travail

Ce didacticiel nécessite l’utilisation de PowerShell pour créer la base de données de l’agent de travail et l’agent de travail. Comme la base de données MSDB utilisée par SQL Agent, un agent de travail utilise une base de données d'Azure SQL Database pour stocker les définitions, l'état et l'historique des travaux. Une fois l’agent de travail créé, vous pouvez immédiatement créer et surveiller des travaux.

1. Dans **PowerShell ISE**, ouvrez *...\\Learning Modules\\Schema Management\\Demo-SchemaManagement.ps1*.
2. Appuyez sur **F5** pour exécuter le script.

Le script *Demo-SchemaManagement.ps1* appelle le script *Deploy-SchemaManagement.ps1* pour créer une base de données nommée _jobagent_ sur le serveur de catalogue. Il crée ensuite l’agent de travail, en passant la base de données _jobagent_ comme un paramètre.

## <a name="create-a-job-to-deploy-new-reference-data-to-all-tenants"></a>Créer un travail pour déployer les nouvelles données de référence sur tous les locataires

#### <a name="prepare"></a>Préparation

Chaque base de données de client comprend un ensemble de types de lieux dans la table **VenueTypes**. Chaque type de lieux définit le type d’événements qui peut être hébergé dans un lieu. Ces types de lieux correspondent aux images d’arrière-plan visibles dans l’application des événements client.  Dans cet exercice, vous allez déployer une mise à jour dans toutes les bases de données afin d’ajouter deux types de lieux supplémentaires : *Motorcycle Racing* (Courses de moto) et *Swimming Club* (Club de natation).

Tout d’abord, examinez les types de lieux inclus dans chaque base de données client. Connectez-vous à l’une des bases de données client dans SQL Server Management Studio (SSMS) et vérifiez la table VenueTypes.  Vous pouvez également interroger cette table dans l’éditeur de requêtes du portail Azure, auquel vous avez accès par la page de la base de données.

1. Ouvrez SSMS et connectez-vous au serveur client : *tenants1-dpt-&lt;utilisateur&gt;.database.windows.net*
1. Pour confirmer que *Motorcycle Racing* et *Swimming Club* **ne sont pas** déjà inclus, accédez à la base de données *contosoconcerthall* sur le serveur *tenants1-dpt-&lt;utilisateur&gt;* et interrogez la table *VenueTypes*.



#### <a name="steps"></a>Étapes

Maintenant, vous créez une tâche pour mettre à jour la table **VenueTypes** dans chaque base de données de locataire en ajoutant les deux nouveaux types de lieux.

Pour créer un nouveau travail, utilisez l’ensemble de procédures stockées du système de travaux créé dans la base de données _jobagent_. Les procédures stockées ont été créées lors de la création de l’agent de travail.

1. Dans SSMS, connectez-vous également au serveur de locataire tenants1-mt-&lt;user&gt;.database.windows.net

2. Naviguez jusqu’à la base de données *tenants1*.

3. Interrogez la table *VenueTypes* pour confirmer que *Motorcycle Racing* et *Swimming Club* ne sont pas encore dans la liste des résultats.

4. Connectez-vous au serveur de catalogue *catalog-mt-&lt;user&gt;.database.windows.net*.

5. Connectez-vous à la base de données _jobagent_ dans le serveur de catalogues.

6. Dans SSMS, ouvrez le fichier *...\\Learning Modules\\Schema Management\\DeployReferenceData.sql*.

7. Modifiez l’instruction : set @User = &lt;utilisateur&gt; et remplacer la valeur de l’utilisateur utilisée lors du déploiement de l’application de base de données Wingtip Tickets SaaS mutualisée.

8. Appuyez sur **F5** pour exécuter le script.

#### <a name="observe"></a>Observer

Observez les éléments suivants dans le script *DeployReferenceData.sql* :

- **sp\_add\_target\_group** crée le nom de groupe cible *DemoServerGroup* et ajoute des membres cibles au groupe.

- **sp\_add\_target\_group\_member** ajoute les éléments suivants :
    - Un type de membre cible *serveur*.
        - Il s’agit du serveur *tenants1-mt-&lt;user&gt;* qui contient les bases de données de locataire.
        - Y compris le serveur contient les bases de données client qui existent au moment de l'exécution du travail.
    - Un type de membre cible *base de données* pour la base de données modèle (*basetenantdb*) qui réside sur le serveur *catalog-mt-&lt;utilisateur&gt;* ,
    - Un type de membre cible *base de données* pour inclure la base de données *adhocreporting* utilisée dans un autre didacticiel.

- **sp\_add\_job** crée une tâche appelée *Reference Data Deployment* (Déploiement des données de référence).

- **sp\_add\_jobstep** crée l’étape du travail contenant le texte de la commande T-SQL pour mettre à jour la table de référence, VenueTypes.

- Les autres vues dans le script indiquent l’existence des objets et contrôlent l’exécution du travail. Utilisez ces requêtes pour passer en revue la valeur d’état dans la colonne **cycle de vie** afin de déterminer la fin du travail. La tâche met à jour la base de données de locataire et met à jour les deux bases de données supplémentaires contenant la table de référence.

Dans SSMS, accédez à la base de données de locataire sur le serveur *tenants1-mt -&lt;user&gt;* . Interrogez la table *VenueTypes* pour confirmer que *Motorcycle Racing* et *Swimming Club* sont désormais dans la liste des résultats. Le nombre total de types de lieux doit avoir augmenté par deux unités.

## <a name="create-a-job-to-manage-the-reference-table-index"></a>Créer une tâche pour gérer l’index de la table de référence

Cet exercice crée un travail pour régénérer l’index sur la clé primaire de la table de référence de toutes les bases de données client. Une régénération d’index est une opération d’administration de bases de données qu’un administrateur peut exécuter après le chargement de charge de données volumineuses, afin d’améliorer les performances.

1. Dans SSMS, connectez-vous à la base de données _jobagent_ sur le serveur *catalog-mt-&lt;Utilisateur&gt;.database.windows.net*.

2. Dans SSMS, ouvrez *...\\Learning Modules\\Schema Management\\OnlineReindex.sql*.

3. Appuyez sur **F5** pour exécuter le script.

#### <a name="observe"></a>Observer

Observez les éléments suivants dans le script *OnlineReindex.sql* :

* **sp\_add\_job** crée une tâche nommée *Online Reindex PK\_\_VenueTyp\_\_265E44FD7FD4C885*.

* **sp\_add\_jobstep** crée l’étape du travail contenant le texte de la commande T-SQL pour mettre à jour l’index.

* Les vues restantes dans le script surveillent l’exécution du travail. Utilisez ces requêtes pour passer en revue la valeur d’état dans la colonne **cycle de vie** afin de déterminer le moment où la tâche a été terminée avec succès sur tous les membres du groupe cible.

## <a name="additional-resources"></a>Ressources supplémentaires

<!-- TODO: Additional tutorials that build upon the Wingtip Tickets SaaS Multi-tenant Database application deployment (*Tutorial link to come*)
(saas-multitenantdb-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
-->
* [Gestion des bases de données cloud avec montée en charge](./elastic-jobs-overview.md)

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un agent de travail pour exécuter des travaux T-SQL dans plusieurs bases de données
> * Mettre à jour les données de référence dans toutes les bases de données de locataire
> * Créer un index sur une table dans toutes les bases de données de locataire

Ensuite, consultez le [didacticiel de génération d’états ad hoc](./saas-multitenantdb-adhoc-reporting.md) pour explorer l’exécution de requêtes distribuées dans les bases de données de locataire.