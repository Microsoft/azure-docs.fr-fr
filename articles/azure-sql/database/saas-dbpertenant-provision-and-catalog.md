---
title: Approvisionner de nouveaux locataires dans une application multilocataire
description: Découvrez comment provisionner et cataloguer de nouveaux locataires dans une application SaaS multilocataire Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 09/24/2018
ms.openlocfilehash: 26add03929551c912b4d7b7cf10741d53333689a
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92780561"
---
# <a name="learn-how-to-provision-new-tenants-and-register-them-in-the-catalog"></a>Découvrez comment approvisionner de nouveaux locataires et les inscrire dans le catalogue
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Dans ce tutoriel, vous allez apprendre à provisionner et à cataloguer des modèles SaaS. Vous allez également voir comment ils sont implémentés dans l’application SaaS Wingtip Tickets, où chaque locataire a sa propre base de données. Vous allez créer et initialiser de nouvelles bases de données de locataire et les inscrire dans le catalogue de locataires de l’application. Le catalogue est une base de données qui gère le mappage entre les nombreux clients des applications SaaS et les données associées. Le catalogue joue un rôle important, car il dirige les demandes d’application et de gestion vers la base de données appropriée.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Provisionner un nouveau locataire
> * Provisionner un lot de locataires supplémentaires


Pour suivre ce didacticiel, vérifiez que les prérequis suivants sont remplis :

* L’application de base de données par locataire SaaS Wingtip Tickets est déployée. Pour procéder à un déploiement en moins de cinq minutes, consultez [Déployer et explorer l’application de base de données par locataire SaaS Wingtip Tickets](./saas-dbpertenant-get-started-deploy.md).
* Azure PowerShell est installé. Pour plus d’informations, consultez [Bien démarrer avec Azure PowerShell](/powershell/azure/get-started-azureps).

## <a name="introduction-to-the-saas-catalog-pattern"></a>Présentation du modèle de catalogue SaaS

Dans une application SaaS multilocataire appuyée par une base de données, il est important de savoir où sont stockées les informations de chaque locataire. Dans le modèle de catalogue SaaS, une base de données de catalogue permet de conserver le mappage entre les locataires et la base de données dans laquelle sont stockées leurs données. Ce modèle s’applique chaque fois que les données de client sont réparties sur plusieurs bases de données.

Chaque client est identifié par une clé dans le catalogue, qui est mappée à l’emplacement de sa base de données. Dans le cas de l’application Wingtip Tickets, la clé est formée à partir du hachage du nom du locataire. Cette méthode permet à l’application de créer la clé à partir du nom de locataire qui est dans l’URL de l’application. Vous pouvez utiliser d’autres schémas de clé de locataire.

Le catalogue permet de modifier le nom ou l’emplacement de la base de données avec un impact minimal sur l’application. Dans un modèle de base de données multilocataire, cette fonctionnalité facilite également le déplacement des locataires entre les différentes bases de données. Le catalogue peut également être utilisé pour indiquer si un locataire ou une base de données est hors connexion pour maintenance ou dans le cadre d’autres opérations. Cette fonctionnalité est décrite dans le [tutoriel sur la restauration d’un locataire unique](saas-dbpertenant-restore-single-tenant.md).

Le catalogue peut aussi stocker d’autres métadonnées de locataire ou de base de données, telles que la version du schéma, le plan de service ou les contrats de niveau de service (SLA) proposés aux locataires. Il peut également stocker d’autres informations sur la gestion des applications, le support client ou DevOps.

Au-delà de l’application SaaS, le catalogue permet d’accéder à des outils de base de données. Dans l’exemple d’application SaaS Wingtip Tickets où chaque locataire dispose de sa propre base de données, le catalogue est utilisé pour permettre les requêtes entre locataires, comme l’explique le [tutoriel sur la création de rapports ad hoc](saas-tenancy-cross-tenant-reporting.md). La gestion des tâches entre différentes bases de données est expliquée dans le tutoriel sur la [gestion des schémas](saas-tenancy-schema-management.md) et le tutoriel sur [l’analyse des données locataires](saas-tenancy-tenant-analytics.md).

Dans les exemples d’applications SaaS Wingtip Tickets, le catalogue est implémenté à l’aide des fonctionnalités de gestion des partitions de la [bibliothèque EDCL (Elastic Database Client Library)](elastic-database-client-library.md). La bibliothèque EDCL est disponible dans Java et dans le .NET Framework. La bibliothèque EDCL permet à une application de créer, gérer et utiliser une carte de partitions reposant sur des bases de données.

Une carte de partitions contient une liste de partitions (bases de données) et le mappage entre les clés (locataires) et les partitions. Les fonctions de la bibliothèque EDCL sont utilisées pendant le provisionnement des locataires pour créer les entrées de la carte des partitions. Elles sont utilisées par les applications au moment de l’exécution, pour se connecter à la base de données appropriée. La bibliothèque EDCL met en cache les informations de connexion pour réduire le trafic vers la base de données de catalogue et booster les performances de l’application.

> [!IMPORTANT]
> Les données de mappage sont accessibles dans la base de données du catalogue. Cependant, *vous ne devez pas les modifier*. Pour modifier les données de mappage, vous devez uniquement utiliser des API Elastic Database Client Library. La manipulation directe des données de mappage risque d’endommager le catalogue et n’est donc pas prise en charge.


## <a name="introduction-to-the-saas-provisioning-pattern"></a>Présentation du modèle d’approvisionnement SaaS

Lors de l’ajout d’un nouveau locataire dans une application SaaS qui utilise un modèle de base de données monolocataire, vous devez provisionner une nouvelle base de données locataire. La base de données doit être créée dans l’emplacement et le niveau de service adaptés. Elle doit également être initialisée avec le schéma et les données de référence appropriés. De plus, elle doit être inscrite dans le catalogue sous la clé de locataire appropriée.

Il existe plusieurs méthodes pour configurer le provisionnement des bases de données. Vous pouvez exécuter des scripts SQL, déployer un fichier bacpac ou copier une base de données de modèle.

La configuration des bases de données doit faire partie de votre stratégie de gestion des schémas. Vous devez faire en sorte que les nouvelles bases de données soient provisionnées à l’aide du schéma le plus récent. Cette condition est expliquée dans le [tutoriel sur la gestion des schémas](saas-tenancy-schema-management.md).

L’application Wingtip Tickets, qui comporte une base de données par locataire, provisionne les nouveaux locataires en copiant la base de données de modèle nommée _basetenantdb_, qui est déployée sur le serveur de catalogue. Le provisionnement peut être intégré à l’application dans le cadre d’un abonnement. Il est également possible d’effectuer un provisionnement hors connexion, à l’aide de scripts. Ce tutoriel décrit le processus de provisionnement à l’aide de PowerShell.

Les scripts de provisionnement copient la base de données _basetenantdb_ pour créer une nouvelle base de données de locataire dans un pool élastique. La base de données de locataire est créée dans le serveur du locataire mappé sur l’alias DNS _newtenant_. Cet alias conserve une référence au serveur utilisé pour provisionner de nouveaux locataires et est mis à jour pour pointer vers un serveur de locataire de récupération dans les didacticiels consacrés à la récupération d’urgence ([DR using georestore](./saas-dbpertenant-dr-geo-restore.md) (La récupération d’urgence à l’aide de la géorestauration), [DR using georeplication](./saas-dbpertenant-dr-geo-replication.md)) (la récupération d’urgence à l’aide de la géoréplication). Ensuite, les scripts initialisent la base de données avec les informations du locataire, et l’inscrivent dans la carte de partitions du catalogue. Les bases de données des locataires portent le nom de leur locataire. Ce schéma de nommage ne constitue pas un élément essentiel du modèle. Le catalogue mappe la clé de locataire dans le nom de la base de données, ce qui permet d’utiliser n’importe quelle convention de nommage.


## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>Obtenir les scripts de l’application de base de données par locataire SaaS Wingtip Tickets

Les scripts et le code source de l’application Wingtip Tickets SaaS sont disponibles dans le référentiel GitHub [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant). Consultez les [conseils généraux](saas-tenancy-wingtip-app-guidance-tips.md) avant de télécharger et de débloquer les scripts Wingtip Tickets SaaS.


## <a name="provision-and-catalog-detailed-walkthrough"></a>Procédure pas à pas détaillée sur l’approvisionnement et l’inscription dans le catalogue

Pour comprendre comment l’application Wingtip Tickets implémente le provisionnement pour les nouveaux locataires, ajoutez un point d’arrêt et suivez le flux de travail pendant le provisionnement.

1. Dans PowerShell ISE, ouvrez \\Learning Modules\\ProvisionAndCatalog\\_Demo-ProvisionAndCatalog.ps1_, puis définissez les paramètres suivants :

   * **$TenantName** = nom du nouveau lieu (par exemple *Bushwillow Blues*).
   * **$VenueType** = un des types de lieux prédéfinis : _blues, classicalmusic, dance, jazz, judo, motorracing, multipurpose, opera, rockmusic, soccer_.
   * **$DemoScenario** = **1**, *Provisionner un seul locataire*.

2. Pour ajouter un point d’arrêt, placez votre curseur n’importe où sur la ligne qui indique *New-Tenant `* . Ensuite, appuyez sur F9.

   ![La capture d’écran présente un script avec New-Tenant en surbrillance pour l’ajout d’un point d’arrêt.](./media/saas-dbpertenant-provision-and-catalog/breakpoint.png)

3. Pour exécuter le script, appuyez sur la touche F5.

4. Une fois que l’exécution du script s’arrête au point d’arrêt, appuyez sur F11 pour effectuer un pas à pas détaillé du code.

   ![Débogage](./media/saas-dbpertenant-provision-and-catalog/debug.png)



Tracez l’exécution du script à l’aide des options du menu **Débogage**. Appuyez sur F10 et F11 pour effectuer un pas à pas principal des fonctions appelées. Pour plus d’informations sur le débogage des scripts PowerShell, consultez [Conseils sur l’utilisation et le débogage de scripts PowerShell](/powershell/scripting/components/ise/how-to-debug-scripts-in-windows-powershell-ise).


Il n’est pas nécessaire de suivre explicitement ce flux de travail, qui explique comment déboguer le script.

* **Importez le module CatalogAndDatabaseManagement.psm1.** Ce module fournit un catalogue et l’abstraction au niveau du locataire pour les fonctions de [gestion des partitions](elastic-scale-shard-map-management.md). Ce module encapsule une bonne partie du modèle de catalogue et que nous vous conseillons d’explorer.
* **Importez le module SubscriptionManagement.psm1.** Ce module contient des fonctions pour la connexion à Azure et la sélection de l’abonnement Azure que vous souhaitez utiliser.
* **Accédez à la configuration détaillée.** Accédez à Get-Configuration (avec F11) et découvrez la façon dont la configuration de l’application est spécifiée. C’est là que sont définis les noms de ressources et les autres valeurs relatives à l’application. Ne modifiez pas ces valeurs tant que vous n’êtes pas suffisamment familiarisé avec les scripts.
* **Obtenez l’objet catalogue.** Accédez à Get-Catalog qui compose et retourne un objet catalogue utilisé dans le script de niveau supérieur. Cette fonction utilise les fonctions de gestion de partitions qui sont importées à partir de **AzureShardManagement.psm1**. L’objet de catalogue est composé des éléments suivants :

   * $catalogServerFullyQualifiedName est construit à l'aide de la ressource standard et de votre nom d'utilisateur : _catalog-\<user\>.database.windows .net_.
   * $catalogDatabaseName est extrait de la configuration : *tenantcatalog*.
   * L’objet $shardMapManager est initialisé à partir de la base de données de catalogue.
   * L’objet $shardMap est initialisé à partir de la carte de partitions _tenantcatalog_ dans la base de données de catalogue. Un objet catalogue est composé, puis retourné. Il est utilisé dans le script de niveau supérieur.
* **Calculez la clé du nouveau locataire**. Une fonction de hachage est utilisée pour créer la clé de locataire à partir du nom du locataire.
* **Vérifiez si la clé de locataire existe déjà**. Une recherche est effectuée pour vérifier la présence de la clé dans le catalogue.
* **La base de données de locataire est approvisionnée avec New-TenantDatabase.** Utilisez F11 pour effectuer un pas à pas détaillé et voir la façon dont la base de données est provisionnée à l’aide d’un [modèle Azure Resource Manager](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

    Le nom de la base de données est construit à partir du nom de locataire, ce qui permet d’indiquer clairement quelle partition appartient à tel locataire. Vous pouvez également utiliser d’autres conventions de nommage pour les bases de données. Un modèle Resource Manager crée une base de données locataire en copiant une base de données modèle (_baseTenantDB_) sur le serveur de catalogue. En guise d’alternative, vous pouvez créer une base de données et l’initialiser en important un fichier bacpac. Vous pouvez également exécuter un script d’initialisation à partir d’un emplacement connu.

    Le modèle Resource Manager est situé dans le dossier ...\Learning Modules\Common\ : *tenantdatabasecopytemplate.json*

* **La base de données du locataire est à nouveau initialisée.** Le nom du lieu (locataire) et le type du lieu sont ajoutés. Vous pouvez aussi y effectuer une autre initialisation.

* **La base de données de locataire est inscrite dans le catalogue.** Elle est inscrite auprès de *Add-TenantDatabaseToCatalog* avec la clé du locataire. Utilisez F11 pour parcourir les informations :

    * La base de données de catalogue est ajoutée à la carte de partitions (liste des bases de données connues).
    * Le mappage qui lie la valeur de clé à la partition est créé.
    * Des métadonnées supplémentaires sur le locataire (nom du lieu) sont ajoutées dans le tableau Locataires du catalogue. Le tableau Locataires ne fait pas partie du schéma ShardManagement et n’est pas installé par la bibliothèque EDCL. Le tableau suivant illustre comment la base de données de catalogue peut être étendue pour prendre en charge des données supplémentaires spécifiques à l’application.


À l’issue du provisionnement, l’exécution retourne au script d’origine *Demo-ProvisionAndCatalog*. Ensuite, la page **Events** (Événements) du nouveau locataire s’ouvre dans le navigateur.

   ![Page Events](./media/saas-dbpertenant-provision-and-catalog/new-tenant.png)


## <a name="provision-a-batch-of-tenants"></a>Approvisionner un lot de locataires

Cet exercice permet d’approvisionner un lot de 17 clients. Il est recommandé de provisionner ce lot de locataires avant de commencer les autres tutoriels relatifs aux applications SaaS Wingtip Tickets comportant une base de données par locataire. Les bases de données que vous pouvez utiliser sont nombreuses.

1. Dans PowerShell ISE, ouvrez ...\\Learning Modules\\ProvisionAndCatalog\\*Demo-ProvisionAndCatalog.ps1*. Remplacez la valeur du paramètre *$DemoScenario* par 3 :

   * **$DemoScenario** = **3**, *Provisionner un lot de locataires*.
2. Pour exécuter le script, appuyez sur la touche F5.

Le script déploie un lot de locataires supplémentaires. Il utilise un [modèle Azure Resource Manager](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md) qui contrôle le lot et délègue ensuite le provisionnement de chaque base de données à un modèle lié. Utiliser des modèles de cette façon permet à Azure Resource Manager de répartir le processus d’approvisionnement pour votre script. Les modèles provisionnent des bases de données en parallèle et, au besoin, gèrent les nouvelles tentatives. Comme le script est idempotent, en cas d’échec ou d’arrêt pour une raison quelconque, exécutez-le à nouveau.

### <a name="verify-the-batch-of-tenants-that-successfully-deployed"></a>Vérifier que le lot de locataires a été correctement déployé

* Dans le [portail Azure](https://portal.azure.com), accédez à la liste des serveurs et ouvrez le serveur *tenants1*. Sélectionnez **Bases de données SQL** et vérifiez que le lot de 17 bases de données supplémentaires figure désormais dans la liste.

   ![Liste des bases de données](./media/saas-dbpertenant-provision-and-catalog/database-list.png)



## <a name="other-provisioning-patterns"></a>Autres modèles d’approvisionnement

Voici d’autres modèles de provisionnement non inclus dans ce tutoriel :

**Pré-approvisionnement des bases de données** : Le modèle de pré-approvisionnement exploite le fait que les bases de données d’un pool élastique n’ajoutent pas de frais supplémentaires. La facturation est liée au pool élastique, et non aux bases de données. Les bases de données inactives ne consomment pas de ressources. En préprovisionnant les bases de données d’un pool et en les allouant en cas de besoin, vous pouvez réduire le délai nécessaire à l’ajout de nouveaux locataires. Le nombre de bases de données pré-approvisionnées peut être ajusté en fonction des besoins pour conserver une mémoire tampon adaptée au taux d’approvisionnement prévu.

**Provisionnement automatique** : Dans le modèle de provisionnement automatique, un service de provisionnement dédié est utilisé pour provisionner automatiquement des serveurs, pools et bases de données en fonction des besoins. Si vous le souhaitez, vous pouvez ajouter le préprovisionnement des bases de données dans les pools élastiques. Si les bases de données sont désactivées et supprimées, les écarts des pools élastiques peuvent être comblés par le service de provisionnement. Un tel service peut être simple ou complexe, comme pour la gestion du provisionnement de plusieurs zones géographiques ou la configuration de la géoréplication pour la récupération d’urgence.

Avec le modèle de provisionnement automatique, une application cliente ou un script envoie une demande de provisionnement à une file d’attente pour qu’elle soit traitée par le service de provisionnement. Il interroge ensuite le service afin de déterminer la complétion. Si le préprovisionnement est utilisé, les demandes sont traitées rapidement. Le service provisionne une base de données de remplacement en arrière-plan.


## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
>
> * Provisionner un nouveau locataire
> * Provisionner un lot de locataires supplémentaires
> * Parcourir les étapes du provisionnement des locataires et de leur inscription dans le catalogue

Essayez le [didacticiel Surveillance des performances](./saas-dbpertenant-performance-monitoring.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* Autres [tutoriels reposant sur les applications SaaS Wingtip Tickets comportant une base de données par locataire](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [Bibliothèque cliente de base de données élastique](elastic-database-client-library.md)
* [Déboguer les scripts dans l’ISE Windows PowerShell](/powershell/scripting/components/ise/how-to-debug-scripts-in-windows-powershell-ise)