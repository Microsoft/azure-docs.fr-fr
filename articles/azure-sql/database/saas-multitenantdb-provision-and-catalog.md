---
title: Approvisionner dans une architecture mutualisée SaaS
description: Découvrez comment provisionner et cataloguer de nouveaux locataires dans une application SaaS multilocataire Azure SQL Database.
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
ms.openlocfilehash: eddb0c8339069025f0742e9bcbc371efbef094ee
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92793328"
---
# <a name="provision-and-catalog-new-tenants-in-a-saas-application-using-a-sharded-multi-tenant-azure-sql-database"></a>Approvisionner et inscrire dans un catalogue de nouveaux locataires dans une application SaaS utilisant une base de données Azure SQL Database multilocataire
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Cet article décrit l’approvisionnement et l’inscription de nouveaux locataires dans un modèle de *base de données partitionnée multilocataire* .

Cet article comprend deux parties principales :

- [Présentation du concept](#goto_2_conceptual) d’approvisionnement et d’inscription de nouveaux locataires.

- [Didacticiel](#goto_1_tutorial) qui définit le code de script PowerShell responsable de l’approvisionnement et de l’inscription.
  - Le didacticiel utilise l’application SaaS Wingtip Tickets, adaptée au modèle de base de données partitionnée multilocataire.

<a name="goto_2_conceptual"></a>

## <a name="database-pattern"></a>Modèle de base de données

Cette section, ainsi que quelques autres qui suivent, explique les concepts du modèle de base de données partitionné multilocataire.

Dans ce modèle partitionné multilocataire, les schémas de table à l’intérieur de chaque base de données incluent une clé de locataire dans la clé primaire des tables qui stockent les données du locataire. La clé de locataire permet à chaque base de données de stocker 0, 1 ou plusieurs locataires. L’utilisation des bases de données partitionnées facilite la prise en charge par le système d’applications d’un très grand nombre de locataires. Toutes les données relatives à un locataire sont stockées dans une base de données unique. Les nombreux locataires sont répartis entre plusieurs bases de données partitionnées. Une base de données de catalogues enregistre le mappage de chaque locataire à sa base de données.

#### <a name="isolation-versus-lower-cost"></a>Isolement et réduction des coûts

En disposant d’une base de données pour lui seul, un locataire bénéficie des avantages de l’isolement. Le locataire peut restaurer la base de données à une date antérieure sans être limité par l’impact sur les autres locataires. Il est possible d’ajuster les performances de la base de données pour les optimiser par rapport à l’unique locataire, sans avoir à compromettre son utilisation par d’autres locataires. Le problème est que l’isolement est plus onéreux que le partage d’une base de données avec d’autres locataires.

Lorsqu’un nouveau locataire est configuré, il peut partager une base de données avec d’autres locataires, ou peut être placé dans sa propre base de données. Si vous changez d’avis par la suite, vous pouvez modifier la base de données.

Vous pouvez avoir à la fois des bases de données multilocataires et des bases de données à un seul locataire dans la même application SaaS pour optimiser les coûts ou l’isolement de chaque locataire.

   ![Application de base de données multilocataire partitionnée avec un catalogue de locataires](./media/saas-multitenantdb-provision-and-catalog/MultiTenantCatalog.png)

## <a name="tenant-catalog-pattern"></a>Modèle du catalogue de locataires

Lorsque vous disposez d’au moins deux bases de données qui contiennent chacune au moins un locataire, l’application doit disposer d’un moyen pour savoir quelle base de données inclut le locataire qui l’intéresse. Une base de données de catalogues enregistre ce mappage.

#### <a name="tenant-key"></a>Clé de locataire

Pour chaque locataire, l’application Wingtip peut dériver une clé unique : la clé de locataire. L’application extrait le nom du locataire à partir de l’URL de la page web. L’application hache le nom pour obtenir la clé. L’application utilise la clé pour accéder au catalogue. Le catalogue croise les informations concernant la base de données dans laquelle le locataire est inscrit. L’application utilise les informations de la base de données pour se connecter. Vous pouvez aussi utiliser d’autres schémas de clé de locataire.

Avec un catalogue, vous pouvez changer le nom ou l’emplacement d’une base de données de locataire après le provisionnement sans interrompre l’application. Dans un modèle de base de données multilocataire, le catalogue vous permet de déplacer un locataire entre bases de données.

#### <a name="tenant-metadata-beyond-location"></a>Métadonnées du locataire autres que l’emplacement

Le catalogue peut également indiquer si un locataire est hors connexion pour maintenance ou dans le cadre d’autres opérations. Et le catalogue peut être étendu pour stocker d’autres locataires ou d’autres métadonnées de base de données, par exemple :
- Le niveau de service ou l’édition d’une base de données.
- Version du schéma de la base de données.
- Le nom du locataire et son contrat SLA (contrat de niveau de service).
- Des informations permettant d’activer la gestion des applications, le service clientèle ou les processus devops.

Le catalogue peut aussi servir à activer la création de rapport pour plusieurs locataires, la gestion des schémas et l’extraction de données à des fins d’analytique.

### <a name="elastic-database-client-library"></a>Bibliothèque cliente de base de données élastique

Dans Wingtip, le catalogue est implémenté dans la base de données *tenantcatalog* . La base de données *tenantcatalog* est créée à l’aide des fonctionnalités de gestion des partitions de la [bibliothèque EDCL (Elastic Database Client Library, bibliothèque cliente de bases de données élastiques)](elastic-database-client-library.md). La bibliothèque permet à une application de créer, gérer et utiliser une *carte de partitions* reposant sur des bases de données. Une carte de partitions croise la clé du locataire avec sa partition, c’est-à-dire avec sa base de données partitionnée.

Pendant l’approvisionnement des locataires, les fonctions de la bibliothèque cliente de bases de données élastiques peuvent être utilisées dans des applications ou des scripts PowerShell pour créer les entrées dans la carte de partitions. Ces fonctions peuvent ensuite être utilisées pour se connecter à la base de données appropriée. La bibliothèque cliente de base de données élastiques met en cache les informations de connexion pour réduire le trafic sur la base de données de catalogues et accélérer le processus de connexion.

> [!IMPORTANT]
> Ne modifiez *pas* les données dans la base de données de catalogues via un accès direct ! Les mises à jour directes ne sont pas prises en charge en raison du risque élevé d’altération des données. Au lieu de cela, modifiez les données de mappage uniquement à l’aide des API de la bibliothèque cliente de bases de données élastiques.

## <a name="tenant-provisioning-pattern"></a>Modèle de provisionnement des locataires

#### <a name="checklist"></a>Liste de contrôle

Lorsque vous souhaitez approvisionner un nouveau locataire dans une base de données partagée existante, vous devez vous poser les questions suivantes concernant la base de données partagée :
- A-t-elle suffisamment d’espace pour le nouveau locataire ?
- Contient-elle des tables incluant les données de référence nécessaires pour le nouveau locataire, ou ces données peuvent-elles être ajoutées ?
- Comporte-t-elle la bonne version du schéma de la base pour le nouveau locataire ?
- Est-elle située dans une région géographique appropriée, proche du nouveau locataire ?
- Le niveau de service est-il adéquat pour le nouveau locataire ?

Si vous souhaitez que le nouveau locataire soit isolé dans sa propre base de données, vous pouvez la créer afin qu’elle réponde aux spécifications du locataire.

Une fois l’approvisionnement terminé, vous devez inscrire le locataire dans le catalogue. Enfin, le mappage de locataire peut être ajouté pour référencer la partition appropriée.

#### <a name="template-database"></a>Base de données de modèle

Provisionnez la base de données en exécutant des scripts SQL, en déployant un fichier bacpac ou en copiant une base de données de modèle. Les applications Wingtip copient une base de données de modèle pour créer des bases de données de locataire.

Comme n’importe quelle application, Wingtip est vouée à évoluer. Dans certains cas, Wingtip nécessitera d’apporter des modifications à la base de données. Les modifications peuvent inclure :
- Nouveau schéma ou schéma modifié.
- Nouvelles données de référence ou données de référence modifiées.
- Tâches de maintenance de routine de la base de données pour garantir des performances optimales de l’application.

Avec une application SaaS, ces modifications doivent être déployées de façon coordonnée dans un parc potentiellement immense de bases de données de locataire. Ces modifications doivent être incorporées au processus d’approvisionnement pour apparaître dans les futures bases de données client. Plus de détails sont disponibles dans le [didacticiel sur la gestion des schémas](saas-tenancy-schema-management.md).

#### <a name="scripts"></a>Scripts

Les scripts d’approvisionnement des locataires de ce didacticiel prennent en charge les deux scénarios suivants :
- Approvisionnement d’un locataire dans une base de données existante partagée avec d’autres locataires.
- Approvisionnement d’un locataire dans sa propre base de données.

Les données de locataire sont ensuite initialisées et inscrites dans la carte de partitions du catalogue. Dans l’exemple d’application, les bases de données qui contiennent plusieurs locataires reçoivent un nom générique, tel que *tenants1* ou *tenants2* . Les bases de données contenant un seul locataire prennent le nom du locataire. Les conventions de nommage spécifiques utilisées dans l’exemple ne sont pas un élément essentiel du modèle, car l’utilisation d’un catalogue permet d’attribuer n’importe quel nom à la base de données.

<a name="goto_1_tutorial"></a>

## <a name="tutorial-begins"></a>Début du didacticiel

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Provisionner un locataire dans une base de données multilocataire
> * Provisionner un locataire dans une base de données à un seul locataire
> * Provisionner un groupe de locataires dans des bases de données à la fois multilocataires et à locataire unique
> * Inscrire un mappage base de données/locataire dans un catalogue

#### <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vérifiez que les prérequis suivants sont remplis :

- Azure PowerShell est installé. Pour plus d’informations, voir [Bien démarrer avec Azure PowerShell](/powershell/azure/get-started-azureps).

- L’application de base de données multilocataire SaaS Wingtip Tickets est déployée. Pour procéder à un déploiement en moins de cinq minutes, consultez [Déployer et explorer l’application de base de données multi-locataire SaaS Wingtip Tickets](./saas-multitenantdb-get-started-deploy.md)

- Obtenir les scripts et le code source Wingtip :
    - Les scripts et le code source de l’application de base de données multilocataire Wingtip Tickets SaaS sont disponibles dans le dépôt GitHub [WingtipTicketsSaaS-MultitenantDB](https://github.com/microsoft/WingtipTicketsSaaS-MultiTenantDB).
    - Consultez les [conseils généraux](saas-tenancy-wingtip-app-guidance-tips.md) avant de télécharger et de débloquer les scripts Wingtip.

## <a name="provision-a-tenant-into-a-database-shared-with-other-tenants"></a>Provisionner un locataire dans une base de données *partagée* avec d’autres locataires

Dans cette section figure une liste des principales actions liées à l’approvisionnement effectuées par les scripts PowerShell. Vous utilisez ensuite le débogueur PowerShell ISE pour parcourir les scripts et afficher les actions dans le code.

#### <a name="major-actions-of-provisioning"></a>Principales actions d’approvisionnement

Voici des étapes clés du flux de travail de provisionnement :

- **Calculer la clé du nouveau locataire**  : Une fonction de hachage est utilisée pour créer la clé de locataire à partir du nom du locataire.
- **Vérifier si la clé de locataire existe déjà**  : Le catalogue est examiné pour vérifier que la clé n’a pas déjà été inscrite.
- **Initialiser le locataire dans la base de données de locataire par défaut**  : La base de données de locataire est mise à jour pour ajouter les informations du nouveau locataire.
- **Inscrire le locataire dans le catalogue**  : Le mappage entre la clé du nouveau locataire et la base de données tenants1 existante est ajouté au catalogue.
- **Ajouter le nom du locataire à la table des extensions d'un catalogue**  : Le nom du lieu est ajouté à la table Tenants dans le catalogue.  Voici comment la base de données de catalogues peut être étendue pour prendre en charge des données supplémentaires spécifiques de l’application.
- **Ouvrir la page Événements du nouveau locataire**  : La page des événements *Bushwillow Blues* est ouverte dans le navigateur.

   ![Capture d’écran montrant la page Événements pour un nouveau locataire.](./media/saas-multitenantdb-provision-and-catalog/bushwillow.png)

#### <a name="debugger-steps"></a>Étapes du débogueur

Pour comprendre comment l’application Wingtip implémente le provisionnement d’un nouveau locataire dans une base de données partagée, ajoutez un point d’arrêt et suivez les étapes du flux de travail :

1. Dans *PowerShell ISE* , ouvrez ...\\Learning Modules\\ProvisionTenants\\*Demo-ProvisionTenants.ps1* et définissez les paramètres suivants :
   - **$TenantName** = **Bushwillow Blues** , le nom d’un nouveau lieu.
   - **$VenueType** = **blues** , l’un des types prédéfinis de lieux : blues, classicalmusic, dance, jazz, judo, motorracing, multipurpose, opera, rockmusic, soccer (en minuscules, sans espace).
   - **$DemoScenario** = **1** , pour provisionner un locataire dans une base de données partagée avec d’autres locataires.

2. Ajoutez un point d'arrêt en plaçant votre curseur n'importe où sur la ligne 38, qui indique : *New-Tenant `* , puis appuyez sur **F9** .

   ![Capture d’écran faisant ressortir la ligne qui comporte New Tenant.](./media/saas-multitenantdb-provision-and-catalog/breakpoint.png)

3. Exécutez le script en appuyant sur **F5** .

4. Une fois que l’exécution du script s’arrête au point d’arrêt, appuyez sur **F11** pour effectuer un pas à pas détaillé du code.

   ![La capture d’écran montre l’ISE Windows PowerShell ISE avec le menu Déboguer ouvert et l’option Effectuer un pas à pas détaillé sélectionnée.](./media/saas-multitenantdb-provision-and-catalog/debug.png)

5. Suivez l’exécution du script à l’aide des options du menu **Débogage** , **F10** et **F11** , pour parcourir les fonctions appelées.

Pour plus d’informations sur le débogage des scripts PowerShell, consultez [Conseils sur l’utilisation et le débogage de scripts PowerShell](/powershell/scripting/components/ise/how-to-debug-scripts-in-windows-powershell-ise).

## <a name="provision-a-tenant-in-its-own-database"></a>Provisionner un locataire dans sa *propre* base de données

#### <a name="major-actions-of-provisioning"></a>Principales actions d’approvisionnement

Voici des étapes clés du flux de travail pendant le suivi de script :

- **Calculer la clé du nouveau locataire**  : Une fonction de hachage est utilisée pour créer la clé de locataire à partir du nom du locataire.
- **Vérifier si la clé de locataire existe déjà**  : Le catalogue est examiné pour vérifier que la clé n’a pas déjà été inscrite.
- **Créer une base de données de locataire**  : La base de données est créée en copiant la base de données *basetenantdb* à l’aide d’un modèle Resource Manager.  Le nom de la nouvelle base de données est basé sur le nom du locataire.
- **Ajouter la base de données au catalogue**  : La nouvelle base de données de locataire est inscrite sous forme de partition dans le catalogue.
- **Initialiser le locataire dans la base de données de locataire par défaut**  : La base de données de locataire est mise à jour pour ajouter les informations du nouveau locataire.
- **Inscrire le locataire dans le catalogue**  : Le mappage entre la clé du nouveau locataire et la base de données *sequoiasoccer* existante est ajouté au catalogue.
- **Le nom du locataire est ajouté au catalogue**  : Le nom du lieu est ajouté à la table d’extension Tenants dans le catalogue.
- **Ouvrir la page Événements du nouveau locataire**  : La page des événements *Sequoia Soccer* est ouverte dans le navigateur.

   ![événements](./media/saas-multitenantdb-provision-and-catalog/sequoiasoccer.png)

#### <a name="debugger-steps"></a>Étapes du débogueur

Suivons maintenant le script de création d’un locataire dans sa propre base de données :

1. Toujours dans ...\\Learning Modules\\ProvisionTenants\\*Demo-ProvisionTenants.ps1* , définissez les paramètres suivants :
   - **$TenantName** = **Sequoia Soccer** , le nom d’un nouveau lieu.
   - **$VenueType** = **soccer** , l’un des types de lieux prédéfinis : blues, classicalmusic, dance, jazz, judo, motorracing, multipurpose, opera, rockmusic, soccer (en minuscules, sans espace).
   - **$DemoScenario** = **2** , pour provisionner un locataire dans sa propre base de données.

2. Ajoutez un nouveau point d’arrêt en plaçant votre curseur n’importe où sur la ligne 57 qui indique *&&nbsp;$PSScriptRoot\New-TenantAndDatabase `* , et appuyez sur **F9** .

   ![point d’arrêt](./media/saas-multitenantdb-provision-and-catalog/breakpoint2.png)

3. Exécutez le script en appuyant sur **F5** .

4. Une fois que l’exécution du script s’arrête au point d’arrêt, appuyez sur **F11** pour effectuer un pas à pas détaillé du code.  Utilisez **F10** et **F11** pour parcourir les fonctions et suivre l’exécution.

## <a name="provision-a-batch-of-tenants"></a>Approvisionner un lot de locataires

Cet exercice permet d’approvisionner un lot de 17 clients. Nous vous recommandons de provisionner ce groupe de locataires avant de suivre d’autres didacticiels Wingtip Tickets pour pouvoir utiliser plusieurs bases de données.

1. Dans *PowerShell ISE* , ouvrez...\\Learning Modules\\ProvisionTenants\\*Demo-ProvisionTenants.ps1* et affectez la valeur 4 au paramètre *$DemoScenario* :
   - **$DemoScenario** = **4** , pour provisionner un groupe de locataires dans une base de données partagée.

2. Appuyez sur **F5** pour exécuter le script.

### <a name="verify-the-deployed-set-of-tenants"></a>Vérifier le groupe de locataires déployé

À ce stade, vous avez des locataires déployés dans une base de données partagée et des locataires déployés dans leur propre base de données. Vous pouvez utiliser le portail Azure pour inspecter les bases de données créées. Dans le [portail Azure](https://portal.azure.com), ouvrez le serveur **tenants1-mt-\<USER\>** en parcourant la liste des serveurs SQL Server.  La liste des **bases de données SQL** doit inclure la base de données **tenants1** partagée et les bases de données des locataires qui ont leur propre base de données :

   ![liste de base de données](./media/saas-multitenantdb-provision-and-catalog/Databases.png)

Bien que le portail Azure affiche les bases de données de locataire, vous ne pouvez pas voir les locataires *dans* la base de données partagée. La liste complète des locataires peut être consultée dans la page web du **hub d’événements** de Wingtip et en parcourant le catalogue.

#### <a name="using-wingtip-tickets-events-hub-page"></a>Utilisation de la page Hub d’événements Wingtip Tickets

Ouvrez la page Hub d’événements dans le navigateur (http:events.wingtip-mt.\<USER\>.trafficmanager.net)

#### <a name="using-catalog-database"></a>Utilisation de la base de données de catalogues

La liste complète des locataires et la base de données correspondante à chacun est disponible dans le catalogue. Une vue SQL relie le nom du locataire au nom de la base de données. Cette vue montre clairement l’intérêt d’étendre les métadonnées stockées dans le catalogue.
- La vue SQL est disponible dans la base de données tenantcatalog.
- Le nom du locataire est stocké dans la table Tenants.
- Le nom de la base de données est stocké dans les tables de gestion des partitions.

1. Dans SQL Server Management Studio (SSMS), connectez-vous au serveur de locataires à l’adresse **catalog-mt.\<USER\>.database.windows.net** , avec l’ID de connexion **developer** et le mot de passe **P\@ssword1**

    ![Boîte de dialogue de connexion de SSMS](./media/saas-multitenantdb-provision-and-catalog/SSMSConnection.png)

2. Dans l’Explorateur d’objets SSMS, accédez aux vues de la base de données *tenantcatalog* .

3. Cliquez avec le bouton droit sur la vue *TenantsExtended* et choisissez **Sélectionner les 1 000 premières lignes** . Notez le mappage entre le nom de locataire et la base de données pour les différents locataires.

    ![Vue ExtendedTenants dans SSMS](./media/saas-multitenantdb-provision-and-catalog/extendedtenantsview.png)

## <a name="other-provisioning-patterns"></a>Autres modèles d’approvisionnement

Cette section aborde d’autres modèles de provisionnement intéressants.

#### <a name="pre-provisioning-databases-in-elastic-pools"></a>Préprovisionnement de bases de données dans des pools élastiques

Le modèle de préprovisionnement exploite le fait que, quand vous utiliser des pools élastiques, la facturation s’applique au niveau du pool et non au niveau des bases de données. Par conséquent, des bases de données peuvent être ajoutées à un pool élastique sans qu’elles soient nécessaires, sans entraîner de frais supplémentaires. Ce préprovisionnement réduit considérablement le temps nécessaire à l’approvisionnement d’un locataire dans une base de données. Le nombre de bases de données pré-approvisionnées peut être ajusté en fonction des besoins pour conserver une mémoire tampon adaptée au taux d’approvisionnement prévu.

#### <a name="auto-provisioning"></a>Approvisionnement automatique

Dans le modèle d’approvisionnement automatique, un service d’approvisionnement dédié est utilisé pour provisionner automatiquement des serveurs, pools et bases de données en fonction des besoins. Ce système automatique inclut le préprovisionnement de bases de données dans des pools élastiques. Si les bases de données sont mises hors service et supprimées, les écarts dans les pools élastiques peuvent être comblés par le service d’approvisionnement en fonction des besoins.

Ce type de service automatique peut être simple ou complexe. Par exemple, le système automatique peut gérer l’approvisionnement sur plusieurs zones géographiques et peut configurer la géoréplication pour la récupération d’urgence. Avec le modèle d’approvisionnement automatique, une application cliente ou un script envoie une requête d’approvisionnement à une file d’attente de traitement d’un service d’approvisionnement. Le script interroge ensuite le service pour déterminer l’achèvement de l’opération. Si vous utilisez le préprovisionnement, les demandes sont gérées rapidement, car un service en arrière-plan gère l’approvisionnement d’une base de données de remplacement.

## <a name="additional-resources"></a>Ressources supplémentaires

<!-- - Additional [tutorials that build upon the Wingtip SaaS application](./saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)-->
- [Bibliothèque cliente de base de données élastique](elastic-database-client-library.md)
- [Déboguer les scripts dans l’ISE Windows PowerShell](/powershell/scripting/components/ise/how-to-debug-scripts-in-windows-powershell-ise)


## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Provisionner un nouveau locataire unique dans une base de données multilocataire partagée et dans sa propre base de données
> * Approvisionner un lot de locataires supplémentaires
> * Suivre les étapes de provisionnement des locataires et de leur inscription dans le catalogue

Essayez le [didacticiel Surveillance des performances](./saas-multitenantdb-performance-monitoring.md).