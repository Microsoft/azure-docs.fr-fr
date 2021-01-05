---
title: Déployer une app SaaS de base de données multilocataire partitionnée
description: Déployez et explorez l’application de base de données Wingtip Tickets SaaS multilocataire partitionnée, qui présente les modèles SaaS à l’aide d’Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 10/16/2018
ms.openlocfilehash: 4de7e428bff0feaafdec00b0c0014bbaf6acb917
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92790965"
---
# <a name="deploy-and-explore-a-sharded-multi-tenant-application"></a>Déployer et explorer une application mutualisée sharded
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Ce didacticiel montre comment déployer et explorer un exemple d’application SaaS mutualisée nommée Wingtip Tickets. L’application Wingtip Tickets est conçue pour tirer parti des fonctionnalités Azure SQL Database qui simplifient l’implémentation de scénarios SaaS.

Cette implémentation de l’application Wingtip Tickets utilise un modèle de base de données mutualisée partitionnée. Le partitionnement est effectué par identificateur du locataire. Les données du locataire sont transmises à une base de données spécifique en fonction des valeurs d’identificateur du locataire. 

Ce modèle de base de données vous permet de stocker un ou plusieurs locataires dans chaque partition ou base de données. Vous pouvez optimiser les coûts en partitionnant chaque base de données entre plusieurs locataires. Vous pouvez également l’isolation en faisant en sorte que chaque base de données ne stocke qu’un seul locataire. Votre choix d’optimisation peut s’appliquer de manière indépendante pour chaque locataire spécifique. Effectuez votre choix lors du premier stockage du locataire, vous pourrez toujours changer d’avis ultérieurement. L’application est conçue pour fonctionner correctement dans les deux cas.

## <a name="app-deploys-quickly"></a>Déploiement rapide de l’application

L’application s’exécute dans le cloud Azure et utilise Azure SQL Database. La section relative au déploiement qui suit inclut le bouton bleu **Déployer dans Azure** . Lors de l’activation du bouton est activé, l’application est entièrement déployée vers votre abonnement Azure dans les cinq minutes. Vous avez un accès complet pour utiliser les composants d’application individuels.

L’application est déployée avec des données pour trois exemples de locataires. Les locataires sont stockés ensemble dans une base de données multilocataire.

N’importe quel utilisateur peut télécharger le code source C# et PowerShell pour Wingtip Tickets à partir de [son référentiel GitHub][link-github-wingtip-multitenantdb-55g].

## <a name="learn-in-this-tutorial"></a>Découvrir, dans ce didacticiel, comment

> [!div class="checklist"]
> - Comment déployer l’application SaaS Wingtip Tickets.
> - Obtenir le code source de l’application et les scripts de gestion.
> - Explorer les serveurs et les bases de données qui composent l’application.
> - Identifier comment les locataires sont mappés à leurs données grâce au *catalogue* .
> - Approvisionner un nouveau locataire.
> - Surveiller l’activité d’un locataire dans l’application.

Une série de didacticiels associés, basés sur ce déploiement initial, est disponible. Les didacticiels explorent une gamme de modèles de conception et de gestion de SaaS. Lorsque vous utilisez les didacticiels, vous êtes encouragé à parcourir les scripts fournis pour voir comment les différents modèles SaaS sont implémentés.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vérifiez que les prérequis suivants sont remplis :

- La dernière version d’Azure PowerShell est installée. Pour plus d’informations, consultez [Prise en main d’Azure PowerShell][link-azure-get-started-powershell-41q].

## <a name="deploy-the-wingtip-tickets-app"></a>Déployer l’application Wingtip Tickets

### <a name="plan-the-names"></a>Planifier les noms

Les étapes de cette section vous permettent de fournir une valeur *utilisateur* utilisée pour garantir que les noms de ressources sont globalement uniques et un nom du *groupe de ressources* qui contient toutes les ressources créées par un déploiement de l’application. Pour une personne nommée *Ann Finley* , nous vous suggérons :
- *Utilisateur :* **af1**  *(ses initiales, plus un chiffre. Utilisez une valeur différente (par exemple, af2) si vous déployez l’application une deuxième fois.)*
- *Groupe de ressources :* **wingtip-mt-af1** *(wingtip-mt indique qu’il s’agit de l’application multi-locataires partitionnée. L’ajout de af1 au nom d’utilisateur correspond au nom du groupe de ressources avec les noms des ressources qu’il contient.)*

Choisissez vos noms maintenant et notez-les. 

### <a name="steps"></a>Étapes

1. Cliquez sur le bouton bleu **Déployer dans Azure** suivant.
   - Il ouvre le portail Azure avec le modèle de déploiement Wingtip Tickets SaaS.

     [![Bouton pour Déployer dans Azure.][image-deploy-to-azure-blue-48d]][link-aka-ms-deploywtp-mtapp-52k]

1. Entrez les valeurs de paramètre requises pour le déploiement.

    > [!IMPORTANT]
    > Pour cette démonstration, n’utilisez pas les groupes de ressources, serveurs ou pools préexistants. Sélectionnez plutôt **Créer un groupe de ressources** . Supprimez ce groupe de ressources lorsque vous en avez terminé avec l’application pour interrompre la facturation associée.
    > N’utilisez pas cette application, ni les ressources qu’elle crée, pour la production. Certains aspects de l’authentification et les paramètres de pare-feu serveur sont intentionnellement non sécurisés dans l’application afin de faciliter la démonstration.

    - Pour **Groupe de ressources** , sélectionnez **Création** et indiquez un **Nom** pour le groupe de ressources (sensible à la casse).
        - Sélectionnez un **Emplacement** dans la liste déroulante.
    - Pour **Utilisateur** , nous vous recommandons de choisir un nom d’ **utilisateur** court.

1. **Déployez l’application** .

    - Cliquez pour accepter les conditions générales.
    - Cliquez sur **Achat** .

1. Surveillez l’état du déploiement en cliquant sur **Notifications** , l’icône représentant une cloche à droite de la zone de recherche. Le déploiement de l’application Wingtip dure environ cinq minutes.

   ![déploiement réussi](./media/saas-multitenantdb-get-started-deploy/succeeded.png)

## <a name="download-and-unblock-the-management-scripts"></a>Télécharger et débloquer les scripts de gestion

Lors du déploiement de l’application, téléchargez le code source de l’application et les scripts de gestion.

> [!NOTE]
> Le contenu exécutable (scripts, DLL) peut être bloqué par Windows lors du téléchargement et de l’extraction de fichiers .zip à partir d’une source externe. Lorsque vous extrayez les scripts d’un fichier zip, utilisez les étapes suivantes pour débloquer le fichier .zip avant l’extraction. En débloquant le fichier .zip, vous êtes assuré de l’exécution des scripts.

1. Accédez au [dépôt GitHub WingtipTicketsSaaS-MultiTenantDb](https://github.com/Microsoft/WingtipTicketsSaaS-MultiTenantDb).
2. Cliquez sur **Cloner ou télécharger** .
3. Cliquez sur **Télécharger ZIP** et enregistrez le fichier.
4. Cliquez avec le bouton droit sur le fichier **WingtipTicketsSaaS-MultiTenantDb-master.zip** , puis sélectionnez **Propriétés** .
5. Sous l’onglet **Général** , sélectionnez **Débloquer** , puis cliquez sur **Appliquer** .
6. Cliquez sur **OK** .
7. Procédez à l’extraction des fichiers.

Les scripts se trouvent dans le dossier *...\\WingtipTicketsSaaS-MultiTenantDb-master\\Learning Modules\\* .

## <a name="update-the-configuration-file-for-this-deployment"></a>Mettre à jour le fichier de configuration pour ce déploiement

Avant d’exécuter des scripts, définissez les valeurs *resource group* et *user* dans **UserConfig.psm1** . Pour ces variables, utilisez les mêmes valeurs que vous avez définies pendant le déploiement.

1. Ouvrez ...\\Learning Modules\\*UserConfig.psm1* dans *PowerShell ISE* .
2. Mettez à jour *ResourceGroupName* et *Name* avec les valeurs spécifiques à votre déploiement (lignes 10 et 11 uniquement).
3. Enregistrez les modifications.

Les valeurs définies dans ce fichier sont utilisées par tous les scripts. Il est donc important qu’elles soient exactes. Si vous redéployez l’application, vous devez choisir des valeurs différentes pour l’utilisateur et le groupe de ressources. Mettez ensuite à jour le fichier UserConfig.psm1 avec les nouvelles valeurs.

## <a name="run-the-application"></a>Exécution de l'application

Dans l’application Wingtip, les locataires sont des lieux. Un lieu peut être une salle de concert, un club sportif ou tout autre emplacement qui accueille des événements. Les lieux sont inscrits dans Wingtip en tant que clients, et un identificateur de locataire est généré pour chaque lieu. Chaque lieu répertorie ses événements à venir dans Wingtip pour que le public puisse acheter des tickets.

Chaque lieu bénéficie d’un site web personnalisé pour répertorier ses événements et vendre ses tickets. Chaque application web est indépendante et isolée des autres locataires. En interne dans Azure SQL Database, les données de chaque locataire sont stockées dans une base de données partitionnée multilocataire, par défaut. Toutes les données sont marquées avec l’identificateur du locataire.

Une page web centrale de **concentrateur d’événements** fournit une liste de liens vers les locataires de votre déploiement. Réalisez les étapes suivantes pour vous familiariser avec la page web de **concentrateur d’événements** et une application web individuelle :

1. Ouvrez le **concentrateur d’événements** dans votre navigateur web :
   - http://events.wingtip-mt.&lt;user&gt;.trafficmanager.net &nbsp; *(remplacez &lt; user&gt; par la valeur d’utilisateur de votre déploiement.)*

     ![events hub](./media/saas-multitenantdb-get-started-deploy/events-hub.png)

2. Cliquez sur **Fabrikam Jazz Club** dans le **concentrateur d’événements** .

   ![Événements](./media/saas-multitenantdb-get-started-deploy/fabrikam.png)

### <a name="azure-traffic-manager"></a>Azure Traffic Manager

Pour contrôler la distribution des requêtes entrantes, l’application Wingtip utilise [Azure Traffic Manager](../../traffic-manager/traffic-manager-overview.md). La page des événements de chaque locataire inclut le nom du locataire dans son URL. Chaque URL comprend également la valeur d’utilisateur spécifique. Chaque URL respecte le format indiqué en procédant comme suit :

- http://events.wingtip-mt.&lt;user&gt;.trafficmanager.net/ *fabrikamjazzclub*

1. L’application d’événements analyse le nom du locataire dans l’URL. Le nom du locataire est *fabrikamjazzclub* dans l’exemple d’URL ci-dessus.
2. L’application applique un hachage au nom du locataire pour créer une clé permettant d’accéder à un catalogue utilisant la [gestion des cartes de partitions](elastic-scale-shard-map-management.md).
3. L’application recherche la clé dans le catalogue et obtient l’emplacement correspondant de la base de données du locataire.
4. L’application utilise les informations d’emplacement pour rechercher une base de données qui contient toutes les données du locataire et y accéder.

### <a name="events-hub"></a>Concentrateur d’événements

1. Le **concentrateur d’événements** répertorie tous les locataires inscrits dans le catalogue et les lieux correspondants.
2. Le **concentrateur d’événements** utilise les métadonnées étendues dans le catalogue pour récupérer le nom du locataire associé à chaque mappage pour créer les URL.

Dans un environnement de production, vous créez généralement un enregistrement DNS CNAME pour [pointer un domaine Internet d’entreprise](../../traffic-manager/traffic-manager-point-internet-domain.md) vers le profil Traffic Manager.

## <a name="start-generating-load-on-the-tenant-databases"></a>Démarrer la génération de charge sur les bases de données de locataires

Maintenant que l’application est déployée, nous allons l’exécuter ! Le script PowerShell *Demo-LoadGenerator* démarre une charge de travail qui s’exécute pour chaque locataire. La charge réelle sur de nombreuses applications SaaS est généralement sporadique et imprévisible. Pour simuler ce type de charge, le générateur produit une charge répartie sur tous les locataires. La charge inclut des pics aléatoires sur chaque locataire qui se produisent à des intervalles aléatoires. Plusieurs minutes sont nécessaires avant que le modèle de charge n’émerge, et il est donc préférable de laisser le générateur s’exécuter pendant au moins trois ou quatre minutes avant d’analyser la charge.

1. Dans *PowerShell ISE* , ouvrez le script... \\Learning Modules\\Utilities\\*Demo-LoadGenerator.ps1* .
2. Appuyez sur **F5** pour exécuter le script et démarrer le générateur de charge (laissez les valeurs de paramètre par défaut pour l’instant).

Le script *Demo-LoadGenerator.ps1* ouvre une autre session PowerShell dans laquelle le générateur de charge s’exécute. Le générateur de charge s’exécute dans cette session comme une tâche de premier plan qui appelle des travaux de génération de charge en arrière-plan, un pour chaque locataire.

Après le démarrage de la tâche de premier plan, elle reste dans un état d’appel de travail. La tâche démarre les travaux en arrière-plan supplémentaires pour les nouveaux locataires qui sont approvisionnés par la suite.

La fermeture de la session PowerShell arrête tous les travaux.

Vous pouvez souhaiter redémarrer la session de générateur de charge pour utiliser des valeurs de paramètre différentes. Dans ce cas, fermez la session de génération PowerShell, puis réexécutez *Demo-LoadGenerator.ps1* .

## <a name="provision-a-new-tenant-into-the-sharded-database"></a>Approvisionner un nouveau locataire dans la base de données partitionnée

Le déploiement initial inclut trois exemples de locataires dans la base de données *Tenants1* . Nous allons créer un autre locataire et observer son impact sur l’application déployée. À cette étape, appuyez sur une touche pour créer un locataire :

1. Ouvrez ...\\Learning Modules\\Provision et Catalog\\*Demo-ProvisionTenants.ps1* dans *PowerShell ISE* .
2. Appuyez sur **F5** (et non **F8** )pour exécuter le script (laissez les valeurs par défaut pour l’instant).

   > [!NOTE]
   > Vous devez exécuter les scripts PowerShell uniquement en appuyant sur la touche  **F5** , et non en appuyant sur **F8** pour exécuter une partie sélectionnée du script. Le problème avec **F8** est que la variable *$PSScriptRoot* n’est pas évaluée. Cette variable est requise par de nombreux scripts pour naviguer dans des dossiers, appeler d’autres scripts ou importer des modules.

Le nouveau locataire Red Maple Racing est ajouté à la base de données *Tenants1* et enregistré dans le catalogue. Le site **événements** de vente de tickets du nouveau locataire s’ouvre dans votre navigateur :

![Nouveau locataire](./media/saas-multitenantdb-get-started-deploy/red-maple-racing.png)

Actualisez le **concentrateur d’événements**  : le nouveau locataire apparaît dans la liste.

## <a name="provision-a-new-tenant-in-its-own-database"></a>Approvisionner un nouveau locataire dans sa propre base de données

Le modèle multilocataire partitionné vous permet de choisir s’il faut approvisionner un nouveau locataire dans une base de données qui contient d’autres locataires dans sa propre base de données. Un locataire isolé dans sa propre base de données bénéficie des avantages suivants :

- Les performances de la base de données du locataire peuvent être gérés indépendamment des besoins des autres locataires.
- Si nécessaire, la base de données peut être restaurée à un point antérieur dans le temps, car aucun autre locataire n’est impacté.

Vous pouvez placer les clients d’une version d'évaluation ou les clients en mode économique dans des bases de données multilocataires. Vous pouvez placer chaque client Premium dans sa propre base de données dédiée. Si vous créez un grand nombre de bases de données qui ne contiennent qu’un seul locataire, vous pouvez les gérer collectivement dans un pool élastique afin d’optimiser les coûts de ressource.

Ensuite, nous approvisionnerons un autre locataire, dans sa propre base de données cette fois-ci :

1. Dans ...\\Learning Modules\\Provision and Catalog\\*Demo-ProvisionTenants.ps1* , remplacez *$TenantName* par **Salix Salsa** , *$VenueType* par **dance** et *$Scenario* par **2** .

2. Appuyez sur **F5** pour réexécuter le script.
    - Cet appui sur **F5** configure le nouveau locataire dans une base de données distincte. La base de données et le locataire sont enregistrés dans le catalogue. Le navigateur s’ouvre alors sur la page des événements du locataire.

   ![Page d’événements Salix Salsa](./media/saas-multitenantdb-get-started-deploy/salix-salsa.png)

   - Faites défiler vers le bas de la page. Dans la bannière, vous voyez le nom de la base de données dans laquelle les données du locataire sont stockées.

3. Actualisez le **concentrateur d’événements**  : les deux nouveaux locataires apparaissent maintenant dans la liste.

## <a name="explore-the-servers-and-tenant-databases"></a>Explorer les serveurs et les bases de données de locataires

Examinons maintenant quelques-unes des ressources qui ont été déployées :

1. Dans le [portail Azure](https://portal.azure.com), accédez à la liste des groupes de ressources. Ouvrez le groupe de ressources que vous avez créé lors du déploiement de l’application.

   ![resource group](./media/saas-multitenantdb-get-started-deploy/resource-group.png)

2. Cliquez sur le serveur **catalog-mt&lt;utilisateur&gt;** . Le serveur de catalogue contient deux bases de données nommées *tenantcatalog* et *basetenantdb* . La base de données *basetenantdb* est une base de données de modèle vide. Elle est copiée pour créer une nouvelle base de données de locataires, quelle soit utilisée par plusieurs locataires ou un seul.

   ![catalog server](./media/saas-multitenantdb-get-started-deploy/catalog-server.png)

3. Revenez au groupe de ressources et sélectionnez le serveur *tenants1-mt* contenant les bases de données de locataires.
    - La base de données tenants1 est une base de données multilocataire dans laquelle les trois locataires d’origine, plus le premier locataire que vous avez ajouté, sont stockés. Elle est configurée comme une base de données 50 DTU standard.
    - La base de données **salixsalsa** contient la salle de danse Salix Salsa comme son unique locataire. Elle est configurée comme une base de données d’édition standard avec 50 DTU par défaut.

   ![serveur de locataires](./media/saas-multitenantdb-get-started-deploy/tenants-server.png)

## <a name="monitor-the-performance-of-the-database"></a>Surveiller les performances de la base de données

Si le générateur de charge s’exécute depuis plusieurs minutes, suffisamment de télémétrie est disponible pour rechercher les fonctionnalités de surveillance de base de données intégrées au portail Azure.

1. Accédez au serveur **tenants1-mt&lt;USER&gt;** , puis cliquez sur **tenants1** pour afficher l’utilisation des ressources pour la base de données contenant quatre locataires. Chaque client est soumis à une charge sporadique importante dans le générateur de charge :

   ![surveiller tenants1](./media/saas-multitenantdb-get-started-deploy/monitor-tenants1.png)

   Le graphique d’utilisation de DTU montre clairement comment une base de données peut multilocataire peut supporter une charge de travail imprévisible entre plusieurs locataires. Dans ce cas, le générateur de charge applique une charge sporadique de 30 DTU environ sur chaque locataire. Cette charge équivaut à 60 % d’utilisation d’une base de données de 50 DTU. Des pics supérieurs à 60 % sont le résultat d’une charge appliquée sur plusieurs locataires simultanément.

2. Accédez au serveur **tenants1-mt&lt;utilisateur&gt;** , puis cliquez sur la base de données **salixsalsa** . Vous voyez l’utilisation des ressources sur cette base de données qui contient un seul locataire.

   ![base de données salixsalsa](./media/saas-multitenantdb-get-started-deploy/monitor-salix.png)

Le générateur de charge applique une charge similaire sur chaque locataire, quelle que soit la base de données dans laquelle se trouve chaque locataire. Avec un seul locataire dans la base de données **salixsalsa** , vous pouvez constater que la base de données peut supporter une charge bien supérieure à celle de la base de données en contenant plusieurs. 

### <a name="resource-allocations-vary-by-workload"></a>Allocations de ressources variant selon la charge de travail

Pour fournir de bonnes performances, une base de données multilocataire exige parfois davantage de ressources qu’une base de données à locataire unique, mais pas toujours. La répartition optimale des ressources dépend des caractéristiques de charge de travail pour les locataires de votre système.

Les charges de travail générées par le script de génération de charge sont uniquement fournies à des fins d’exemple.

## <a name="additional-resources"></a>Ressources supplémentaires

- Pour de plus amples informations sur les applications SaaS mutualisées, consultez [Modèles de location de base de données SaaS multi-locataire](saas-tenancy-app-design-patterns.md).

- Pour en savoir plus sur les pools élastiques, voir :

  - [Les pools élastiques vous aident à gérer et à mettre à l'échelle plusieurs bases de données Azure SQL Database](elastic-pool-overview.md)
  - [Scale-out avec Azure SQL Database](elastic-scale-introduction.md)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à effectuer les opérations suivantes :

> [!div class="checklist"]
> - Déployer l’application de base de données multilocataire Wingtip Tickets SaaS.
> - Explorer les serveurs et les bases de données qui composent l’application.
> - Explorer les locataires qui sont mappés à leurs données avec le *catalogue* .
> - Approvisionner les nouveaux locataires dans une base de données multilocataire et une base de données à locataire unique.
> - Afficher l’utilisation du pool pour surveiller l’activité des locataires.
> - Comment supprimer les exemples de ressources pour arrêter la facturation associée.

Essayez maintenant le [didacticiel sur l’approvisionnement et le catalogue](saas-multitenantdb-provision-and-catalog.md).


<!--  Link references.

A [series of related tutorials] is available that build upon this initial deployment.
[link-wtp-overivew-jumpto-saas-tutorials-97j]: saas-multitenantdb-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials

-->

[link-aka-ms-deploywtp-mtapp-52k]: https://aka.ms/deploywtp-mtapp


[link-azure-get-started-powershell-41q]: /powershell/azure/get-started-azureps

[link-github-wingtip-multitenantdb-55g]: https://github.com/Microsoft/WingtipTicketsSaaS-MultiTenantDB/



<!--  Image references.

[image-deploy-to-azure-blue-48d]: https://aka.ms/deploywtp-mtapp "Button for Deploy to Azure."
-->

[image-deploy-to-azure-blue-48d]: media/saas-multitenantdb-get-started-deploy/deploy.png "Bouton pour le déploiement dans Azure."