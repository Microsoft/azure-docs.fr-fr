---
title: Récupération d’urgence pour les applications SaaS avec géo-réplication
description: Apprendre à utiliser des géoréplicas Azure SQL Database pour récupérer une application SaaS multilocataire en cas de panne
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/25/2019
ms.openlocfilehash: d2c1ddd1e0b5a080050e1ffeb28eded98dbfea3f
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97652107"
---
# <a name="disaster-recovery-for-a-multi-tenant-saas-application-using-database-geo-replication"></a>Récupération d’urgence d’une application SaaS multi-locataire à l’aide de la géoréplication de bases de données
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Dans ce didacticiel, vous examinez un scénario complet de récupération d’urgence pour une application SaaS multi-locataire implémentée à l’aide du modèle de base de données par locataire. Afin de protéger l’application contre les pannes, vous utilisez la [_géoréplication_](active-geo-replication-overview.md) pour créer des réplicas de bases de données de catalogue et de locataire dans une autre région de récupération. En cas de panne, vous basculez rapidement vers ces réplicas pour reprendre les opérations normales. Lors du basculement, les bases de données dans la région d’origine deviennent des réplicas secondaires des bases de données dans la région de récupération. Après leur retour en ligne, les réplicas reprennent automatiquement l’état des bases de données dans la région de récupération. Une fois la panne résolue, vous restaurez automatiquement les bases de données dans la région de production d’origine.

Ce didacticiel explore les flux de travail de basculement et de restauration automatique. Vous découvrirez comment effectuer les actions suivantes :
> [!div class="checklist"]
> 
> * Synchroniser les informations de configuration des bases de données et du pool élastique dans le catalogue du locataire
> * Configurer un environnement de récupération dans une autre région, comprenant une application, des serveurs et des pools
> * Utiliser la _géoréplication_ pour répliquer les bases de données de catalogue et de locataire dans la région de récupération
> * Basculer l’application et les bases de données de catalogue et de locataire vers la région de récupération 
> * Ensuite, restaurer l’application ainsi que les bases de données de catalogue et de locataire dans la région d’origine, une fois la panne résolue
> * Mettre à jour le catalogue, car chaque base de données du client est basculée pour assurer le suivi de l’emplacement principal de la base de données de chaque locataire
> * Vérifier que l’application et la base de données de locataire primaire sont toujours colocalisées dans la même région Azure pour réduire la latence  


Avant de démarrer ce didacticiel, vérifiez que les conditions préalables sont remplies :
* L’application de base de données SaaS Wingtip Tickets par locataire est déployée. Pour procéder à un déploiement en moins de cinq minutes, consultez [Déployer et explorer l’application de base de données par locataire SaaS Wingtip Tickets](saas-dbpertenant-get-started-deploy.md)  
* Azure PowerShell est installé. Pour plus d’informations, voir [Bien démarrer avec Azure PowerShell](/powershell/azure/get-started-azureps).

## <a name="introduction-to-the-geo-replication-recovery-pattern"></a>Présentation du modèle de récupération par géoréplication

![Architecture de récupération](./media/saas-dbpertenant-dr-geo-replication/recovery-architecture.png)

La récupération d’urgence est un élément important à prendre en compte pour de nombreuses applications, que ce soit pour des raisons de conformité ou de continuité de l’activité. En cas d’interruption prolongée du service, un plan de récupération d’urgence bien préparé peut minimiser la perturbation de l’activité. La géoréplication fournit les RPO et RTO les plus faibles en maintenant les réplicas de base de données dans une région de récupération qui autorise un basculement rapidement.

Un plan de récupération d’urgence basé sur la géoréplication comprend trois parties :
* Configuration - création et maintenance de l’environnement de récupération
* Récupération - basculement de l’application et des bases de données vers l’environnement de récupération en cas de panne
* Rapatriement - basculement de l’application et des bases de données vers la région d’origine une fois l’application résolue 

Toutes les parties doivent être examinées avec précaution, surtout en cas de fonctionnement maximum. Globalement, le plan doit atteindre plusieurs objectifs :

* Programme d’installation
    * Créer un environnement miroir dans la région de récupération et en assurer la maintenance. La création de pools élastiques et la réplication de bases de données dans cet environnement de récupération assurent une capacité de réserve dans la région de récupération. La maintenance de cet environnement comprend la réplication des nouvelles bases de données de locataire, au fur et à mesure de leur approvisionnement.  
* Récupération
    * Lorsque vous utilisez un environnement de récupération dont la taille a été diminuée pour réduire les coûts quotidiens, les pools et les bases de données doivent être mis à l’échelle pour atteindre une capacité opérationnelle complète dans la région de récupération.
     * Activer l’approvisionnement des nouveaux locataires dans la région de récupération dès que possible  
     * Être optimisé pour restaurer les locataires dans leur ordre de priorité
     * Être optimisé pour mettre les locataires en ligne dès que possible en effectuant des tâches en parallèle le cas échéant
     * Être résilient à l’échec, redémarrable et idempotent
     * Pouvoir annuler le processus de restauration pendant son exécution si la région d’origine est de retour en ligne.
* Rapatriement 
     * Basculer les bases de données à partir de la région de récupération vers les réplicas dans la région d’origine avec un impact minimal sur les locataires : aucune perte de données et durée minimale hors ligne par client.

Dans ce tutoriel, ces tâches sont effectuées à l’aide des fonctionnalités Azure SQL Database et de la plateforme Azure :

* [Modèles Azure Resource Manager](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md) pour réserver toute la capacité nécessaire aussi rapidement que possible. Les modèles Azure Resource Manager sont utilisés pour approvisionner une image miroir des serveurs de production et des pools élastiques dans la région de récupération.
* [Géoréplication](active-geo-replication-overview.md) pour créer des copies secondaires en lecture seule et répliquées de façon asynchrone pour toutes les bases de données. Pendant la panne, vous basculez vers les réplicas dans la région de récupération.  Une fois la panne résolue, vous restaurez automatiquement les bases de données dans la région d’origine sans aucune perte de données.
* Opérations de basculement [asynchrones](../../azure-resource-manager/management/async-operations.md) transmises dans l’ordre de priorité des locataires, pour minimiser le temps de basculement d’un grand nombre de bases de données.
* [Fonctionnalités de récupération de gestion de partition](elastic-database-recovery-manager.md) pour modifier les entrées de base de données dans le catalogue pendant la récupération et le rapatriement. Ces fonctionnalités permettent à l’application de se connecter aux bases de données de locataire, quel que soit l’emplacement, sans reconfigurer l’application.
* [Alias DNS de serveur SQL](./dns-alias-overview.md), pour activer l’approvisionnement transparent des nouveaux locataires, quelle que soit la région dans laquelle l’application fonctionne. Les alias DNS permettent également au processus de synchronisation des catalogues de se connecter au catalogue actif, quel que soit son emplacement.

## <a name="get-the-disaster-recovery-scripts"></a>Obtenir des scripts de récupération d’urgence 

> [!IMPORTANT]
> Comme tous les scripts de gestion Wingtip Tickets, les scripts de récupération d’urgence sont fournis à titre d’exemple et n’ont pas vocation à être utilisés en production. 

Les scripts de récupération utilisés dans ce didacticiel et le code source de l’application Wingtip sont disponibles dans le [référentiel GitHub sur la base de données SaaS Wingtip Tickets par locataire](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant/). Consultez les [conseils généraux](saas-tenancy-wingtip-app-guidance-tips.md) pour télécharger et débloquer les scripts de gestion Wingtip Tickets.

## <a name="tutorial-overview"></a>Vue d’ensemble du didacticiel
Dans ce didacticiel, vous utilisez d’abord la géoréplication pour créer des réplicas de l’application Wingtip Tickets et de ses bases de données dans une autre région. Ensuite, vous basculez vers cette région pour simuler la récupération en cas de panne. Lorsque vous avez terminé, l’application est totalement fonctionnelle dans la région de récupération.

Ensuite, lors d’une étape distincte de rapatriement, vous basculez les bases de données de catalogue et de locataire dans la région de récupération vers la région d’origine. L’application et les bases de données restent disponibles tout au long du rapatriement. Lorsque vous avez terminé, l’application est totalement fonctionnelle dans la région d’origine.

> [!Note]
> L’application est récupérée dans la _région jumelée_ à celle dans laquelle l’application est déployée. Pour plus d’informations, consultez [Régions jumelées Azure](../../best-practices-availability-paired-regions.md).

## <a name="review-the-healthy-state-of-the-application"></a>Examiner l’état d’intégrité de l’application

Avant de lancer le processus de récupération, examinez l’état d’intégrité normale de l’application.
1. Dans votre navigateur web, ouvrez le hub d’événements Wingtip Tickets (http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net, remplacez &lt; user&gt; par la valeur d’utilisateur de votre déploiement).
    * Faites défiler la page vers le bas et notez le nom et l’emplacement du serveur de catalogue dans le pied de page. L’emplacement correspond à la région dans laquelle vous avez déployé l’application.
    *CONSEIL : placez le pointeur de la souris sur l’emplacement pour agrandir l’affichage.* 
    ![État intègre d’Event Hub dans la région d’origine](./media/saas-dbpertenant-dr-geo-replication/events-hub-original-region.png)

2. Cliquez sur le locataire Contoso Concert Hall et ouvrez sa page d’événement.
    * Dans le pied de page, notez le nom du serveur du locataire. L’emplacement sera le même que celui du serveur de catalogue.

3. Dans le [Portail Azure](https://portal.azure.com), ouvrez le groupe de ressources dans lequel l’application est déployée.
    * Notez que la région dans laquelle les serveurs sont déployés. 

## <a name="sync-tenant-configuration-into-catalog"></a>Synchroniser la configuration du locataire dans le catalogue

Cette tâche vous permet de démarrer un processus de synchronisation de la configuration des serveurs, des pools élastiques et des bases de données dans le catalogue du locataire. Le processus maintient ces informations à jour dans le catalogue.  Le processus fonctionne avec le catalogue actif, que ce soit dans la région d’origine ou dans la région de récupération. Les informations de configuration sont utilisées dans le cadre du processus de récupération pour vérifier que l’environnement de récupération est cohérent avec l’environnement d’origine, puis lors du rapatriement pour vérifier que la région d’origine est cohérente avec les modifications apportées dans l’environnement de récupération. Le catalogue permet également d’assurer le suivi de l’état de récupération des ressources de locataire.

> [!IMPORTANT]
> Par souci de simplicité, le processus de synchronisation et les autres processus longs de récupération et de rapatriement sont implémentés dans ces didacticiels sous la forme de sessions ou de travaux PowerShell locaux qui s’exécutent sous votre ID d’utilisateur client. Les jetons d’authentification émis lors de la connexion expireront après quelques heures, puis les travaux échoueront. Dans un scénario de production, les processus longs doivent être implémentés comme des services Azure fiables et exécutés sous un principal de service. Consultez [Utiliser Azure PowerShell pour créer un principal du service avec un certificat](../../active-directory/develop/howto-authenticate-service-principal-powershell.md).

1. Dans l’_ISE PowerShell_, ouvrez le fichier ...\Learning Modules\UserConfig.psm1. Remplacez `<resourcegroup>` et `<user>` sur les lignes 10 et 11 par la valeur utilisée lors du déploiement de l’application.  Enregistrez le fichier.

2. Dans l’*ISE PowerShell*, ouvrez le script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1 et configurez :
    * **$DemoScenario = 1**, pour démarrer un travail en arrière-plan qui synchronise le serveur de locataire et les informations de configuration des pools dans le catalogue

3. Appuyez sur **F5** pour exécuter le script de synchronisation. Une nouvelle session PowerShell est ouverte pour synchroniser la configuration des ressources du locataire.
![Capture d’écran montrant la nouvelle session PowerShell qui est ouverte pour synchroniser la configuration des ressources du locataire.](./media/saas-dbpertenant-dr-geo-replication/sync-process.png)

Laissez la fenêtre PowerShell ouverte en arrière-plan et poursuivez le didacticiel. 

> [!Note]
> Le processus de synchronisation se connecte au catalogue via un alias de DNS. Cet alias est modifié lors de la restauration et du rapatriement pour pointer vers le catalogue actif. Le processus de synchronisation maintient le catalogue à jour en fonction des modifications apportées à la configuration des bases de données ou des pools dans la région de récupération.  Pendant le rapatriement, ces modifications sont appliquées aux ressources équivalentes dans la région d’origine.

## <a name="create-secondary-database-replicas-in-the-recovery-region"></a>Créer des réplicas de base de données secondaires dans la région de récupération

Dans cette tâche, vous démarrez un processus qui déploie une instance d’application en double et réplique le catalogue ainsi que toutes les bases de données de locataire dans une région de récupération.

> [!Note]
> Ce didacticiel ajoute une protection de la géoréplication dans l’application exemple Wingtip Tickets. Dans un scénario de production, pour une application qui utilise la géoréplication, chaque locataire est approvisionné avec une base de données géorépliquée dès le début. Consultez [Conception de services hautement disponibles à l’aide d’Azure SQL Database](designing-cloud-solutions-for-disaster-recovery.md#scenario-1-using-two-azure-regions-for-business-continuity-with-minimal-downtime).

1. Dans l’*ISE PowerShell*, ouvrez le script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1 et configurez la valeur suivante :
    * **$DemoScenario = 2**, pour créer l’environnement de récupération d’image miroir et répliquer les bases de données de catalogue et de locataire.

2. Appuyez sur **F5** pour exécuter le script. Une nouvelle session PowerShell est ouverte pour créer les réplicas.
![Processus de synchronisation](./media/saas-dbpertenant-dr-geo-replication/replication-process.png)  

## <a name="review-the-normal-application-state"></a>Examiner l’état d’application normal

À ce stade, l’application s’exécute normalement dans la région d’origine et est maintenant protégée par la géoréplication.  Il existe des réplicas secondaires en lecture seule dans la région de récupération pour toutes les bases de données. 

1. Dans le portail Azure, examinez vos groupes de ressources et notez qu’un groupe de ressources a été créé avec le suffixe -recovery dans la région de récupération. 

2. Explorez les ressources du groupe de ressources de récupération.  

3. Cliquez sur la base de données Concert Contoso Hell sur le serveur _tenants1-dpt -&lt;utilisateur&gt;-recovery_.  Cliquez sur la géoréplication sur le côté gauche. 

    ![Lien de géoréplication de Contoso Concert](./media/saas-dbpertenant-dr-geo-replication/contoso-geo-replication.png) 

Dans le mappage des régions Azure, notez le lien de géoréplication entre la base de données primaire dans la région d’origine et la base de données secondaire dans la région de récupération.  

## <a name="fail-over-the-application-into-the-recovery-region"></a>Basculer l’application vers la région de récupération

### <a name="geo-replication-recovery-process-overview"></a>Vue d’ensemble du processus de récupération par géoréplication

Le script de récupération effectue les tâches suivantes :

1. Il désactive le point de terminaison Traffic Manager de l’application web dans la région d’origine. La désactivation du point de terminaison empêche les utilisateurs de se connecter à l’application dans un état non valide, si la région d’origine revient en ligne pendant la récupération.

1. Il force le basculement de la base de données de catalogue dans la région de récupération pour en faire la base de données primaire, et met à jour l’alias _activecatalog_ pour qu’il pointe vers le serveur de catalogue de récupération.

1. Il met à jour l’alias _newtenant_ pour qu’il pointe vers le serveur de locataire dans la région de récupération. La modification de cet alias permet d’approvisionner les bases de données des nouveaux locataires dans la région de récupération. 

1. Il marque tous les locataires existants dans le catalogue de récupération comme étant hors connexion pour empêcher l’accès aux bases de données de locataire avant leur basculement.

1. Il met à jour la configuration de tous les pools élastiques et de toutes les bases de données uniques répliquées dans la région de récupération pour la mise en miroir de leur configuration dans la région d’origine. (Cette tâche n’est nécessaire que si les pools ou les bases de données répliquées dans l’environnement de récupération sont diminués pendant les opérations normales pour réduire les coûts.)

1. Il active le point de terminaison Traffic Manager de l’application web dans la région de récupération. L’activation de ce point de terminaison permet à l’application d’approvisionner de nouveaux locataires. À ce stade, les locataires existants sont toujours hors connexion.

1. Il envoie des lots de requêtes pour forcer le basculement des bases de données dans leur ordre de priorité.
    * Les lots sont organisés de sorte que les bases de données soient restaurées en parallèle entre tous les pools.
    * Les requêtes de basculement sont envoyées de manière asynchrone pour accélérer leur envoi et permettre de traiter simultanément plusieurs requêtes.

   > [!Note]
   > En cas de panne, les bases de données primaires dans la région d’origine sont hors connexion.  Le basculement forcé sur la base de données secondaire interrompt la connexion à la base de données primaire, sans tenter d’appliquer les transactions en file d’attente résiduelles. Dans un scénario de récupération d’urgence comme ce didacticiel, une activité de mise à jour lors du basculement peut entraîner une perte de données. Ensuite, pendant le rapatriement, lorsque vous basculez des bases de données de la région de récupération vers la région d’origine, un basculement normal permet d’éviter la perte de données.

1. Il analyse le service pour déterminer le moment du basculement des bases de données. Lorsqu’une base de données de locataire est basculée, le script met à jour le catalogue pour enregistrer l’état de récupération de la base de données du locataire et marquer le locataire en ligne.
    * L’application a accès aux bases de données de locataire, dès qu’elles sont marquées en ligne dans le catalogue.
    * Une somme des valeurs rowversion dans la base de données de locataire est stockée dans le catalogue. Cette somme joue le rôle d’empreinte digitale et permet au processus de rapatriement de déterminer si la base de données a été mise à jour dans la région de récupération.

### <a name="run-the-script-to-fail-over-to-the-recovery-region"></a>Exécuter le script pour basculer vers la région de récupération

À présent, imaginez qu’une panne se produise dans la région où l’application est déployée et exécutez le script de récupération :

1. Dans l’*ISE PowerShell*, ouvrez le script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1 et configurez la valeur suivante :
    * **$DemoScenario = 3**, pour récupérer l’application dans une région de récupération en basculant vers les réplicas.

2. Appuyez sur **F5** pour exécuter le script.  
    * Le script s’ouvre dans une nouvelle fenêtre PowerShell, puis démarre une série de travaux PowerShell qui s’exécutent en parallèle. Ces travaux basculent les bases de données de locataire vers la région de récupération.
    * La région de récupération correspond à la _région jumelée_ à la région Azure où vous avez déployé l’application. Pour plus d’informations, consultez [Régions jumelées Azure](../../best-practices-availability-paired-regions.md). 

3. Surveillez l’état du processus de récupération dans la fenêtre PowerShell.
    ![Processus de basculement](./media/saas-dbpertenant-dr-geo-replication/failover-process.png)

> [!Note]
> Pour explorer le code des travaux de récupération, examinez les scripts PowerShell dans le dossier ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\RecoveryJobs.

### <a name="review-the-application-state-during-recovery"></a>Examiner l’état de l’application pendant la récupération

Pendant que le point de terminaison de l’application est désactivé dans Traffic Manager, l’application n’est pas disponible. Une fois le catalogue basculé vers la région de récupération et tous les locataires marqués hors connexion, l’application repasse en ligne. Bien que l’application soit disponible, chaque locataire apparaît hors connexion dans le hub d’événements jusqu’à ce que sa base de données soit basculée. Il est important de concevoir votre application pour qu’elle gère les bases de données de locataire hors connexion.

1. Juste après la récupération de la base de données de catalogue, actualisez le hub d’événements Wingtip Tickets dans votre navigateur web.
   * Dans le pied de page, remarquez que le nom du serveur de catalogue porte maintenant un suffixe _-recovery_ et se trouve dans la région de récupération.
   * Remarquez que les locataires non encore restaurés sont marqués hors connexion et ne sont pas sélectionnables.  

     > [!Note]
     > Avec quelques bases de données seulement à récupérer, vous ne pourrez peut-être pas actualiser le navigateur avant la fin de la récupération. Donc il se peut que vous ne voyiez pas les locataires lorsqu’ils sont hors connexion. 

     ![Hub d’événements hors connexion](./media/saas-dbpertenant-dr-geo-replication/events-hub-offlinemode.png) 

   * Si vous ouvrez directement la page des événements d’un locataire hors connexion, elle affiche une notification indiquant que le locataire est hors connexion. Par exemple, si Contoso Concert Hall est hors connexion, essayez d’ouvrir http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net/contosoconcerthall ![ Page Contoso hors connexion](./media/saas-dbpertenant-dr-geo-replication/dr-in-progress-offline-contosoconcerthall.png) 

### <a name="provision-a-new-tenant-in-the-recovery-region"></a>Approvisionner un nouveau locataire dans la région de récupération
Même avant le basculement de toutes les bases de données de locataire, vous pouvez approvisionner de nouveaux locataires dans la région de récupération.  

1. Dans l’*ISE PowerShell*, ouvrez le script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1 et configurez la propriété suivante :
    * **$DemoScenario = 4**, pour approvisionner un nouveau locataire dans la région de récupération

2. Appuyez sur **F5** pour exécuter le script et approvisionner le nouveau locataire. 

3. La page d’événements Hawthorn Hall s’ouvre dans le navigateur, une fois l’opération terminée. Dans le pied de page, remarquez que la base de données Hawthorn Hall est approvisionnée dans la région de récupération.
    ![Page d’événements Hawthorn Hall](./media/saas-dbpertenant-dr-geo-replication/hawthornhallevents.png) 

4. Dans le navigateur, actualisez la page du hub d’événements Wingtip Tickets pour voir Hawthorn Hall. 
    * Si vous avez approvisionné Hawthorn Hall sans attendre la restauration des autres locataires, ces derniers sont peut-être encore hors connexion.


## <a name="review-the-recovered-state-of-the-application"></a>Examiner l’état récupéré de l’application

À l’issue du processus de récupération, l’application et tous les locataires sont entièrement opérationnels dans la région de récupération. 

1. Lorsque l’affichage dans la console PowerShell indique que tous les locataires sont récupérés, actualisez le hub d’événements.  Les locataires apparaissent tous en ligne, y compris le nouveau, Hawthorn Hall.

    ![Locataires récupérés et nouveaux dans le hub d’événements](./media/saas-dbpertenant-dr-geo-replication/events-hub-with-hawthorn-hall.png)

2. Dans le [portail Azure](https://portal.azure.com), ouvrez la liste des groupes de ressources.  
    * Remarquez le groupe de ressources que vous avez déployé, en plus du groupe de ressources de récupération, avec le suffixe _-recovery_.  Le groupe de ressources de récupération contient toutes les ressources créées pendant le processus de récupération, plus les nouvelles ressources créées pendant la panne.  

3. Ouvrez le groupe de ressources de récupération et notez les éléments suivants :
   * Les versions de récupération des serveurs de catalogue et tenants1, avec le suffixe _-recovery_.  Toutes les bases de données de catalogue et de locataire restaurées sur ces serveurs portent les noms utilisés dans la région d’origine.

   * Le serveur SQL _tenants2-dpt-&lt;utilisateur&gt;-recovery_ est utilisé  Ce serveur sert à approvisionner les nouveaux locataires pendant la panne.
   * Le service App Service nommé _events-wingtip-dpt-&lt;région_récupération&gt;-&lt;utilisateur&gt_;, qui est l’instance de récupération de l’application Événements. 

     ![Ressources de récupération Azure](./media/saas-dbpertenant-dr-geo-replication/resources-in-recovery-region.png) 

4. Ouvrez le serveur SQL _tenants2-dpt-&lt;utilisateur&gt;-recovery_.  Remarquez qu’il contient la base de données _hawthornhall_ et le pool élastique _Pool1_.  La base de données _hawthornhall_ est configurée comme une base de données élastique dans le pool élastique _Pool1_.

5. Revenez au groupe de ressources et cliquez sur la base de données Contoso Concert Hall sur le serveur _tenants1-dpt-&lt;utilisateur&gt;-recovery_. Cliquez sur la géoréplication sur le côté gauche.

    ![Base de données Contoso après le basculement](./media/saas-dbpertenant-dr-geo-replication/contoso-geo-replication-after-failover.png)

## <a name="change-tenant-data"></a>Modifier les données de locataire 
Dans cette tâche, vous mettez à jour l’une des bases de données de locataire. 

1. Dans votre navigateur, recherchez la liste des événements de Contoso Concert Hall et notez le nom du dernier événement.
2. Dans l’*ISE PowerShell*, dans le script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1, configurez la valeur suivante :
    * **$DemoScenario = 5** pour supprimer un événement d’un locataire dans la région de récupération.
3. Appuyez sur **F5** pour exécuter le script.
4. Actualisez la page d’événements Concert Contoso Hall (http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net/contosoconcerthall - substitute &lt; utilisateur&gt; avec la valeur d’utilisateur de votre déploiement) et notez que le dernier événement a été supprimé.

## <a name="repatriate-the-application-to-its-original-production-region"></a>Rapatrier l’application dans sa région de production d’origine

Cette tâche rapatrie l’application dans sa région d’origine. Dans un scénario réel, vous déclenchez le rapatriement lorsque la panne est résolue.

### <a name="repatriation-process-overview"></a>Vue d’ensemble du processus de rapatriement

![Architecture de rapatriement](./media/saas-dbpertenant-dr-geo-replication/repatriation-architecture.png)

Le processus de rapatriement :
1. Annule toutes les requêtes de restauration de base de données en attente ou en cours.
2. Met à jour l’alias _newtenant_ pour qu’il pointe vers le serveur des locataires dans la région d’origine. La modification de cet alias garantit que les bases de données des nouveaux locataires seront désormais approvisionnées dans la région de récupération.
3. Relie les données de locataire modifiées à la région d’origine.
4. Bascule de bases de données de locataire dans l’ordre de priorité.

Le basculement déplace efficacement la base de données vers la région d’origine. Lorsque la base de données bascule, toutes les connexions ouvertes sont supprimées et la base de données n’est pas disponible pendant quelques secondes. Les applications doivent être écrites avec une logique de nouvelle tentative pour leur permettre de se reconnecter.  Bien que cette brève déconnexion passe souvent inaperçue, vous pouvez choisir de rapatrier les bases de données en dehors des heures de bureau. 


### <a name="run-the-repatriation-script"></a>Exécuter le script de rapatriement
Considérons à présent que la panne est résolue et que vous exécutez le script de rapatriement.

1. Dans l’*ISE PowerShell*, exécutez le script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-FailoverToReplica\Demo-FailoverToReplica.ps1.

2. Vérifiez que le processus de synchronisation de catalogue est toujours en cours dans son instance PowerShell.  Au besoin, relancez-le en définissant :
    * **$DemoScenario = 1** pour démarrer la synchronisation des informations de configuration du serveur de locataire, du pool et de la base de données dans le catalogue.
    * Appuyez sur **F5** pour exécuter le script.

3.  Ensuite, pour lancer le processus de rapatriement, définissez :
    * **$DemoScenario = 6** pour rapatrier l’application dans sa région d’origine.
    * Appuyez sur **F5** pour exécuter le script de récupération dans une nouvelle fenêtre PowerShell.  Le rapatriement prend plusieurs minutes, et vous pouvez le surveiller dans la fenêtre PowerShell.
    ![Processus de rapatriement](./media/saas-dbpertenant-dr-geo-replication/repatriation-process.png)

4. Pendant l’exécution du script, actualisez la page du hub d’événements (http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net)
    * Remarquez que tous les locataires sont en ligne et accessibles pendant ce processus.

5. Une fois le rapatriement terminé, actualisez le hub d’événements et ouvrez la page d’événements de Hawthorn Hall. Notez que cette base de données a été rapatriée vers la région d’origine.
    ![Hub d’événements rapatrié](./media/saas-dbpertenant-dr-geo-replication/events-hub-repatriated.png)


## <a name="designing-the-application-to-ensure-app-and-database-are-colocated"></a>Concevoir l’application pour garantir la colocalisation de l’application et de la base de données 
L’application est conçue pour toujours se connecter à partir d’une instance située dans la même région que la base de données de locataire. Cette conception réduit la latence entre l’application et la base de données. Cette optimisation suppose que l’interaction entre l’application et la base de données génère davantage d’échanges que celle entre l’utilisateur et l’application.  

Les bases de données de locataire peuvent être réparties entre les régions d’origine et de récupération pendant quelque temps lors du rapatriement. Pour chaque base de données, l’application recherche le DNS sur le serveur du locataire, dans la région où la base de données se trouve. Dans SQL Database, le nom du serveur est un alias. L’alias du nom du serveur contient le nom de la région. Si l’application ne se trouve pas dans la même région que la base de données, elle se redirige vers l’instance située dans la même région que le serveur. Cette redirection vers l’instance dans la même région que celle de la base de données minimise la latence entre l’application et la base de données. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à effectuer les opérations suivantes :
> [!div class="checklist"]
> 
> * Synchroniser les informations de configuration des bases de données et du pool élastique dans le catalogue du locataire
> * Configurer un environnement de récupération dans une autre région, comprenant une application, des serveurs et des pools
> * Utiliser la _géoréplication_ pour répliquer les bases de données de catalogue et de locataire dans la région de récupération
> * Basculer l’application et les bases de données de catalogue et de locataire vers la région de récupération 
> * Restaurer l’application ainsi que les bases de données de catalogue et de locataire dans la région d’origine, une fois la panne résolue

Pour plus d’informations sur les technologies fournies par Azure SQL Database pour assurer la continuité des activités, consultez la section [Vue d’ensemble de la continuité de l’activité](business-continuity-high-availability-disaster-recover-hadr-overview.md).

## <a name="additional-resources"></a>Ressources supplémentaires

* [Autres didacticiels reposant sur l’application SaaS Wingtip](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)