---
title: 'Applications SaaS : Sauvegardes géoredondantes pour la récupération d’urgence'
description: Apprendre à utiliser des sauvegardes géoredondantes Azure SQL Database pour récupérer une application SaaS multilocataire en cas de panne
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 01/14/2019
ms.openlocfilehash: 3fe6095595f5270b18536e6ef46afe4a0a5b3268
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97607709"
---
# <a name="use-geo-restore-to-recover-a-multitenant-saas-application-from-database-backups"></a>Utiliser la géorestauration pour récupérer une application SaaS multilocataire à partir de sauvegardes de bases de données
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Ce e didacticiel examine un scénario complet de récupération d’urgence pour une application SaaS multilocataire implémentée avec le modèle de base de données par locataire. Vous utilisez la [géorestauration](recovery-using-backups.md) pour récupérer les bases de données de catalogue et de locataire à partir de sauvegardes géoredondantes automatiquement conservées dans une autre région de récupération. Une fois la panne résolue, vous utilisez la [géoréplication](active-geo-replication-overview.md) pour rapatrier les bases de données modifiées dans leur région d’origine.

![Le diagramme présente les régions d’origine et de récupération, qui disposent toutes deux d’une application, d’un catalogue, d’images originales ou miroir de serveurs et de pools, de sauvegardes automatiques vers le stockage, avec la région de récupération qui accepte la géo-réplication de sauvegarde et dispose d’un serveur et d’un pool pour les nouveaux locataires.](./media/saas-dbpertenant-dr-geo-restore/geo-restore-architecture.png)

La géorestauration est la solution de reprise d’activité la plus économique pour Azure SQL Database. Toutefois, la restauration à partir de sauvegardes géo-redondantes peut entraîner une perte de données allant jusqu'à une heure. Elle peut prendre beaucoup de temps, selon la taille de la base de données. 

> [!NOTE]
> Récupérer des applications avec l’objectif de point de récupération (RPO) et l’objectif de délai de récupération (RTO) les plus faibles possible en utilisant la géoréplication au lieu de la géorestauration.

Ce tutoriel étudie à la fois les workflows de restauration et de rapatriement. Vous allez apprendre à effectuer les actions suivantes :
> [!div class="checklist"]
> 
> * Synchroniser les informations de configuration des bases de données et du pool élastique dans le catalogue du locataire.
> * Configurer l’environnement d’une image miroir dans une région de récupération, comprenant l’application, les serveurs et les pools.   
> * Récupérer des bases de données de catalogue et de locataire en utilisant la géorestauration.
> * Utiliser la géoréplication pour rapatrier le catalogue du locataire et les bases de données de locataire modifiées une fois la panne résolue.
> * Mettre à jour le catalogue au fur et à mesure que chaque base de données est restaurée (ou rapatriée) pour suivre l’emplacement actuel de la copie active de la base de données de chaque locataire.
> * Vérifier que l’application et la base de données du locataire sont toujours colocalisées dans la même région Azure pour réduire la latence. 
 

Avant de commencer ce didacticiel, terminez les éléments suivants :
* Déployer l’application de base de données Wingtip Tickets SaaS par client. Pour procéder à un déploiement en moins de cinq minutes, consultez [Déployer et explorer une application multi-locataire SaaS qui illustre le modèle de base de données par locataire avec Azure SQL Database](saas-dbpertenant-get-started-deploy.md). 
* Installez Azure PowerShell. Pour plus d’informations, consultez [Prise en main d’Azure PowerShell](/powershell/azure/get-started-azureps).

## <a name="introduction-to-the-geo-restore-recovery-pattern"></a>Présentation du modèle de récupération par géorestauration

La récupération d’urgence est un élément important à prendre en compte pour de nombreuses applications, que ce soit pour des raisons de conformité ou de continuité de l’activité. En cas d’interruption prolongée du service, un plan de récupération d’urgence bien préparé peut minimiser la perturbation de l’activité. Un plan de reprise d’activité qui s’appuie sur la géorestauration doit remplir plusieurs objectifs :
 * Réserver toute la capacité nécessaire dans la région de récupération choisie aussi rapidement que possible pour garantir la disponibilité pour restaurer les bases de données de locataire.
 * Établir un environnement de récupération d’image miroir qui reflète la configuration du pool et de la base de données d’origine. 
 * Permettre l’annulation du processus de restauration pendant l’exécution si la région d’origine est remise en ligne.
 * Activer rapidement le provisionnement de locataires pour que leur intégration puisse redémarrer dès que possible.
 * Être optimisé pour restaurer les locataires dans leur ordre de priorité.
 * Être optimisé pour l’obtention des locataires en ligne dès que possible en effectuant d’éventuelles étapes pratiques en parallèle.
 * Être résilient à l’échec, redémarrable et idempotent.
 * Rapatrier les bases de données dans leur région d’origine avec un impact minimal sur les locataires quand la panne est résolue.  

> [!NOTE]
> L’application est récupérée dans la région jumelée à celle dans laquelle l’application est déployée. Pour plus d’informations, consultez [Régions jumelées Azure](../../best-practices-availability-paired-regions.md).   

Ce tutoriel utilise les fonctionnalités Azure SQL Database et la plateforme Azure pour effectuer ces tâches :

* [Modèles Azure Resource Manager](../../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md) pour réserver toute la capacité nécessaire aussi rapidement que possible. Les modèles Azure Resource Manager sont utilisés pour provisionner une image miroir des serveurs et pools élastiques d’origine dans la région de récupération. Un serveur et un pool distincts sont également créés pour le provisionnement de nouveaux locataires.
* [Bibliothèque cliente de base de données élastique](elastic-database-client-library.md) (EDCL) pour créer et tenir à jour un catalogue de bases de données de locataire. Le catalogue augmenté inclut des informations de configuration de pool et de base de données régulièrement actualisées.
* [Fonctionnalités de récupération de gestion de partition](elastic-database-recovery-manager.md) de l’EDCL pour tenir à jour les entrées d’emplacement de base de données dans le catalogue pendant la récupération et le rapatriement.  
* [Géorestauration](../../key-vault/general/disaster-recovery-guidance.md) pour récupérer les bases de données de catalogue et de locataire à partir de sauvegardes géoredondantes automatiquement conservées. 
* [Opérations de restauration asynchrone](../../azure-resource-manager/management/async-operations.md) envoyées dans l’ordre de priorité des locataires, lesquels sont mis en file d’attente pour chaque pool par le système et traités par lots pour que le pool ne soit pas surchargé. Ces opérations peuvent être annulées avant ou pendant l’exécution si nécessaire.   
* [Géoréplication](active-geo-replication-overview.md) pour rapatrier les bases de données dans leur région d’origine après la panne. L’utilisation de la géoréplication garantit l’absence de perte de données et un impact minimal sur le locataire.
* [Alias DNS du serveur SQL](./dns-alias-overview.md) pour permettre au processus de synchronisation de catalogue de se connecter au catalogue actif, quel que soit son emplacement.  

## <a name="get-the-disaster-recovery-scripts"></a>Obtenir des scripts de récupération d’urgence

Les scripts de reprise d’activité utilisés dans ce tutoriel sont disponibles dans le [dépôt GitHub de la base de données par locataire SaaS Wingtip Tickets](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant). Consultez les [conseils généraux](saas-tenancy-wingtip-app-guidance-tips.md) pour télécharger et débloquer les scripts de gestion Wingtip Tickets.

> [!IMPORTANT]
> Comme tous les scripts de gestion Wingtip Tickets, les scripts de récupération d’urgence sont fournis à titre d’exemple et n’ont pas vocation à être utilisés en production.

## <a name="review-the-healthy-state-of-the-application"></a>Examiner l’état d’intégrité de l’application
Avant de lancer le processus de récupération, examinez l’état d’intégrité normale de l’application.

1. Dans votre navigateur web, ouvrez le hub d’événements Wingtip Tickets (http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net, remplacez &lt; user&gt; par la valeur d’utilisateur de votre déploiement).
    
   Faites défiler la page vers le bas et notez le nom et l’emplacement du serveur de catalogue dans le pied de page. L’emplacement correspond à la région dans laquelle vous avez déployé l’application.    

   > [!TIP]
   > Placez le curseur de la souris au-dessus de l’emplacement pour agrandir l’affichage.

   ![État sain du hub d’événements dans la région d’origine](./media/saas-dbpertenant-dr-geo-restore/events-hub-original-region.png)

2. Sélectionnez le locataire Contoso Concert Hall et ouvrez sa page d’événement.

   Dans le pied de page, notez le nom du serveur du locataire. L’emplacement est le même que celui du serveur de catalogue.

   ![Région d’origine de Contoso Concert Hall](./media/saas-dbpertenant-dr-geo-restore/contoso-original-location.png)   

3. Dans le [portail Azure](https://portal.azure.com), passez en revue et ouvrez le groupe de ressources dans lequel vous avez déployé l’application.

   Notez les ressources et la région où sont déployés les composants du service d’application et SQL Database.

## <a name="sync-the-tenant-configuration-into-the-catalog"></a>Synchroniser la configuration du locataire dans le catalogue

Cette tâche vous permet de démarrer un processus de synchronisation de la configuration des serveurs, pools élastiques et bases de données dans le catalogue du locataire. Ces informations sont utilisées ultérieurement pour configurer l’environnement d’une image miroir dans la région de récupération.

> [!IMPORTANT]
> Par souci de simplicité, le processus de synchronisation et d’autres processus longs de récupération et de rapatriement sont implémentés dans ces exemples en tant que travaux ou sessions Powershell locaux qui s’exécutent sous votre connexion utilisateur cliente. Les jetons d’authentification émis lors de la connexion expirent à l’issue de quelques heures, puis les travaux échouent. Dans un scénario de production, les processus longs doivent être implémentés comme des services Azure fiables et exécutés sous un principal de service. Consultez [Utiliser Azure PowerShell pour créer un principal du service avec un certificat](../../active-directory/develop/howto-authenticate-service-principal-powershell.md). 

1. Dans l’ISE PowerShell, ouvrez le fichier ...\Learning Modules\UserConfig.psm1. Remplacez `<resourcegroup>` et `<user>` sur les lignes 10 et 11 par la valeur utilisée lors du déploiement de l’application. Enregistrez le fichier .

2. Dans PowerShell ISE, ouvrez le script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1.

    Dans ce didacticiel, vous exécutez chacun des scénarios inclus dans ce script PowerShell, alors laissez ce fichier ouvert.

3. Spécifiez les paramètres suivants :

    $DemoScenario = 1 : pour démarrer un travail en arrière-plan qui synchronise le serveur de locataire et les informations de configuration des pools dans le catalogue.

4. Appuyez sur F5 pour exécuter le script de synchronisation. 

    Ces informations sont utilisées ultérieurement pour veiller à ce que la récupération crée une image miroir des serveurs, pools et bases de données dans la région de récupération.  

    ![Processus de synchronisation](./media/saas-dbpertenant-dr-geo-restore/sync-process.png)

Laissez la fenêtre PowerShell en cours d’exécution en arrière-plan et poursuivez le tutoriel.

> [!NOTE]
> Le processus de synchronisation se connecte au catalogue via un alias de DNS. L’alias est modifié au cours de la restauration et du rapatriement pour pointer vers le catalogue actif. Le processus de synchronisation maintient le catalogue à jour en fonction des modifications apportées à la configuration des bases de données ou des pools dans la région de récupération. Pendant le rapatriement, ces modifications sont appliquées aux ressources équivalentes dans la région d’origine.

## <a name="geo-restore-recovery-process-overview"></a>Vue d’ensemble du processus de récupération par géorestauration

Le processus de récupération par géorestauration déploie l’application et restaure les bases de données à partir de sauvegardes dans la région de récupération.

Le processus de récupération effectue les opérations suivantes :

1. Il désactive le point de terminaison Azure Traffic Manager de l’application web dans la région d’origine. La désactivation du point de terminaison empêche les utilisateurs de se connecter à l’application dans un état non valide, si la région d’origine revient en ligne pendant la récupération.

2. Configure un serveur de catalogue de récupération dans la région de récupération, géorestaure la base de données de catalogue et met à jour l’alias activecatalog pour qu’il pointe vers le serveur de catalogue restauré. La modification de l’alias du catalogue garantit que le processus de synchronisation de catalogue se synchronise toujours avec le catalogue actif.

3. Marque tous les locataires existants dans le catalogue de récupération comme hors connexion pour empêcher l’accès aux bases de données de locataire avant leur restauration.

4. Provisionne une instance de l’application dans la région de récupération et la configure pour utiliser le catalogue restauré dans cette région. Pour maintenir la latence à un niveau minimal, l’exemple d’application est conçu de sorte à toujours se connecter à une base de données de locataire située dans la même région.

5. Configure un serveur et un pool élastique dans lesquels les nouveaux locataires sont provisionnés. La création de ces ressources garantit que le provisionnement de nouveaux locataires n’interfère pas avec la récupération de locataires existants.

6. Met à jour l’alias du nouveau locataire pour qu’il pointe vers le serveur des nouvelles bases de données de locataire situées dans la région de récupération. La modification de cet alias permet de veiller à ce que les bases de données de tous les nouveaux locataires soient provisionnées dans la région de récupération.
        
7. Provisionne des serveurs et des pools élastiques dans la région de récupération pour restaurer les bases de données de locataire. Ces serveurs et pools sont une image miroir de la configuration dans la région d’origine. Le provisionnement de pools en amont permet de réserver la capacité nécessaire pour restaurer toutes les bases de données.

    Une panne survenant dans une région peut mettre une pression importante sur les ressources disponibles dans la région jumelée. Si vous comptez sur la géorestauration pour la reprise d’activité, alors une réservation rapide de ressources est recommandée. Envisagez la géoréplication si une application doit impérativement être récupérée dans une région spécifique. 

8. Il active le point de terminaison Traffic Manager de l’application web dans la région de récupération. L’activation de ce point de terminaison permet à l’application d’approvisionner de nouveaux locataires. À ce stade, les locataires existants sont toujours hors connexion.

9. Envoie des lots de requêtes pour restaurer les bases de données dans leur ordre de priorité. 

    * Les lots sont organisés de sorte à ce que les bases de données soient restaurées en parallèle dans tous les pools.  

    * Les requêtes de restauration sont envoyées de manière asynchrone si bien qu’elles sont envoyées rapidement et mises en file d’attente à des fins d’exécution dans chaque pool.

    * Étant donné que les requêtes de restauration sont traitées en parallèle dans tous les pools, il est préférable de distribuer les locataires importants dans plusieurs pools. 

10. Supervise le service pour déterminer à quel moment les bases de données sont restaurées. Une fois qu’une base de données de locataire est restaurée, elle est marquée comme en ligne dans le catalogue et une somme de version de ligne est enregistrée pour elle. 

    * L’application a accès aux bases de données de locataire, dès qu’elles sont marquées en ligne dans le catalogue.

    * Une somme des valeurs rowversion dans la base de données de locataire est stockée dans le catalogue. Cette somme joue le rôle d’empreinte digitale qui permet au processus de rapatriement de déterminer si la base de données a été mise à jour dans la région de récupération.

## <a name="run-the-recovery-script"></a>Exécuter le script de récupération

> [!IMPORTANT]
> Ce tutoriel restaure des bases de données à partir de sauvegardes géoredondantes. Bien que ces sauvegardes soient généralement disponibles dans un délai de 10 minutes, cela peut prendre jusqu’à une heure. Le script s’interrompt jusqu'à ce qu’elles sont disponibles.

Imaginez qu’une panne se produise dans la région où l’application est déployée et exécutez le script de récupération :

1. Dans PowerShell ISE, dans le script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1, définissez la valeur suivante :

    $DemoScenario = 2 : récupérer l’application dans une région de récupération en restaurant à partir de sauvegardes géoredondantes.

2. Appuyez sur F5 pour exécuter le script.  

    * Le script s’ouvre dans une nouvelle fenêtre PowerShell, puis démarre un ensemble de travaux PowerShell qui s’exécutent en parallèle. Ces travaux restaurent les serveurs, pools et bases de données dans la région de récupération.

    * La région de récupération correspond à la région jumelée à la région Azure où vous avez déployé l’application. Pour plus d’informations, consultez [Régions jumelées Azure](../../best-practices-availability-paired-regions.md). 

3. Surveillez l’état du processus de récupération dans la fenêtre PowerShell.

    ![Capture d’écran montrant la fenêtre PowerShell dans laquelle vous pouvez superviser l’état du processus de récupération.](./media/saas-dbpertenant-dr-geo-restore/dr-in-progress.png)

> [!NOTE]
> Pour explorer le code des travaux de récupération, examinez les scripts PowerShell dans le dossier ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\RecoveryJobs.

## <a name="review-the-application-state-during-recovery"></a>Examiner l’état de l’application pendant la récupération
Pendant que le point de terminaison de l’application est désactivé dans Traffic Manager, l’application n’est pas disponible. Le catalogue est restauré et tous les locataires sont marqués hors connexion. Le point de terminaison d’application dans la région de récupération est alors activé et l’application est remise en ligne. Bien que l’application soit disponible, les locataires apparaissent hors connexion dans le hub d’événements jusqu’à ce que leurs bases de données soient restaurées. Il est important de concevoir votre application pour qu’elle gère les bases de données de locataire hors connexion.

* Une fois que la base de données de catalogue a été récupérée, mais avant que les locataires ne soient à nouveau en ligne, actualisez le hub d’événements Wingtip Tickets dans votre navigateur web.

  * Dans le pied de page, remarquez que le nom du serveur de catalogue porte maintenant un suffixe -recovery et se trouve dans la région de récupération.

  * Remarquez que les locataires qui ne sont pas encore restaurés sont marqués comme hors connexion et ne peuvent pas être sélectionnés.   
 
    ![Processus de récupération](./media/saas-dbpertenant-dr-geo-restore/events-hub-tenants-offline-in-recovery-region.png)  

  * Si vous ouvrez la page Événements d’un locataire directement alors qu’il est hors connexion, la page présente une notification indiquant que le locataire est hors connexion. Par exemple, si Contoso Concert Hall est hors connexion, essayez d’ouvrir http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net/contosoconcerthall.

    ![Capture d’écran affichant une page d’événements hors connexion.](./media/saas-dbpertenant-dr-geo-restore/dr-in-progress-offline-contosoconcerthall.png)

## <a name="provision-a-new-tenant-in-the-recovery-region"></a>Approvisionner un nouveau locataire dans la région de récupération
Avant même que les bases de données de locataire ne soient restaurées, vous pouvez provisionner de nouveaux locataires dans la région de récupération. Les nouvelles bases de données de locataire provisionnées dans la région de récupération sont rapatriées ultérieurement avec les bases de données récupérées.   

1. Dans PowerShell ISE, dans le script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1, définissez la propriété suivante :

    $DemoScenario = 3 : Approvisionner un nouveau locataire dans la région de récupération.

2. Appuyez sur F5 pour exécuter le script.

3. La page des événements Hawthorn Hall s’ouvre dans le navigateur quand le provisionnement est terminé. 

    Remarquez que la base de données Hawthorn Hall se trouve dans la région de récupération.

    ![Hall hawthorn provisionné dans la région de récupération](./media/saas-dbpertenant-dr-geo-restore/hawthorn-hall-provisioned-in-recovery-region.png)

4. Dans le navigateur, actualisez la page du hub d’événements Wingtip Tickets pour voir Hawthorn Hall. 

    Si vous avez approvisionné Hawthorn Hall sans attendre la restauration des autres locataires, ces derniers sont peut-être encore hors connexion.

## <a name="review-the-recovered-state-of-the-application"></a>Examiner l’état récupéré de l’application

À l’issue du processus de récupération, l’application et tous les locataires sont entièrement opérationnels dans la région de récupération. 

1. Une fois que l’affichage dans la console PowerShell indique que tous les locataires sont récupérés, actualisez le hub d’événements. 

    Les locataires apparaissent tous en ligne, y compris le nouveau, Hawthorn Hall.

    ![Locataires récupérés et nouveaux dans le hub d’événements](./media/saas-dbpertenant-dr-geo-restore/events-hub-with-hawthorn-hall.png)

2. Cliquez sur Contoso Concert Hall pour ouvrir sa page Événements. 

    Dans le pied de page, remarquez que la base de données se trouve sur le serveur de récupération situé dans la région de récupération.

    ![Contoso dans la région de récupération](./media/saas-dbpertenant-dr-geo-restore/contoso-recovery-location.png)

3. Dans le [portail Azure](https://portal.azure.com), ouvrez la liste des groupes de ressources.  

    Remarquez le groupe de ressources que vous avez déployé, en plus du groupe de ressources de récupération, avec le suffixe -recovery. Le groupe de ressources de récupération contient toutes les ressources créées pendant le processus de récupération, plus les nouvelles ressources créées pendant la panne. 

4. Ouvrez le groupe de ressources de récupération et notez les éléments suivants :

   * Les versions de récupération des serveurs de catalogue et tenants1, avec le suffixe -recovery. Toutes les bases de données de catalogue et de locataire restaurées sur ces serveurs portent les noms utilisés dans la région d’origine.

   * Le serveur SQL tenants2-dpt-&lt;utilisateur&gt;-recovery est utilisé. Ce serveur sert à approvisionner les nouveaux locataires pendant la panne.

   * Le service App Service nommé events-wingtip-dpt-&lt;région_récupération&gt;-&lt;utilisateur&gt;, qui est l’instance de récupération de l’application Événements.

     ![Ressources Contoso dans la région de récupération](./media/saas-dbpertenant-dr-geo-restore/resources-in-recovery-region.png) 
    
5. Ouvrez le serveur SQL tenants2-dpt-&lt;utilisateur&gt;-recovery. Remarquez qu’il contient la base de données hawthornhall et le pool élastique Pool1. La base de données hawthornhall est configurée comme une base de données élastique dans le pool élastique Pool1.

## <a name="change-the-tenant-data"></a>Modifier les données de locataire 
Cette tâche permet de mettre à jour l’une des bases de données de locataire restaurées. Le processus de rapatriement copie les bases de données restaurées qui ont été modifiées dans la région d’origine. 

1. Dans votre navigateur, recherchez la liste des événements de Contoso Concert Hall, faites-les défiler et remarquez le dernier, Seriously Strauss.

2. Dans PowerShell ISE, dans le script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1, définissez la valeur suivante :

    $DemoScenario = 4 : pour supprimer un événement d’un locataire dans la région de récupération.

3. Appuyez sur F5 pour exécuter le script.

4. Actualisez la page des événements de Contoso Concert Hall (http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net/contosoconcerthall) et remarquez que l’événement Seriously Strauss est manquant.

À ce stade du tutoriel, vous avez récupéré l’application, qui est à présent en cours d’exécution dans la région de récupération. Vous avez provisionné un nouveau locataire dans la région de récupération et modifié les données de l’un des locataires restaurés.  

> [!NOTE]
> Les autres tutoriels de l’exemple ne sont pas conçus pour s’exécuter avec l’application dans un état de récupération. Pour découvrir les autres tutoriels, veillez à rapatrier d’abord l’application.

## <a name="repatriation-process-overview"></a>Vue d’ensemble du processus de rapatriement

Le processus de rapatriement rétablit l’application et ses bases de données dans leur région d’origine une fois qu’une panne est résolue.

![Rapatriement par géorestauration](./media/saas-dbpertenant-dr-geo-restore/geo-restore-repatriation.png)   

Le processus :

1. Arrête toute activité de restauration en cours et annule toutes les requêtes de restauration de base de données en attente ou en cours d’exécution.

2. Réactive dans la région d’origine les bases de données de locataire qui n’ont pas été modifiées depuis la panne. Ces bases de données incluent celles qui ne sont pas encore récupérées ou celles qui le sont, mais qui n’ont pas été modifiées par la suite. Les bases de données réactivées sont exactement comme elles étaient la dernière fois que leurs locataires y ont accédé.

3. Provisionne une image miroir du serveur et du pool élastique des locataires dans la région d’origine. Une fois que cette action est terminée, l’alias du nouveau locataire est mis à jour pour pointer vers ce serveur. La mise à jour de l’alias entraîne l’intégration du nouveau locataire dans la région d’origine au lieu de la région de récupération.

3. Utilise la géoréplication pour déplacer le catalogue vers la région d’origine depuis la région de récupération.

4. Met à jour la configuration du pool dans la région d’origine pour qu’elle soit cohérente avec les modifications apportées dans la région de récupération pendant la panne.

5. Crée les serveurs et pools nécessaires pour héberger toutes les nouvelles bases de données créées pendant la panne.

6. Utilise la géoréplication pour rapatrier les bases de données de locataire restaurées qui ont été mises à jour après la restauration, ainsi que toutes les nouvelles bases de données de locataire provisionnées pendant la panne. 

7. Nettoie les ressources créées dans la région de récupération pendant le processus de restauration.

Pour limiter le nombre de bases de données de locataire à rapatrier, les étapes 1 à 3 sont effectuées rapidement.  

L’étape 4 est uniquement effectuée si le catalogue situé dans la région de récupération a été modifié pendant la panne. Le catalogue est mis à jour si de nouveaux locataires sont créés ou si la configuration d’une base de données ou d’un pool est modifiée dans la région de récupération.

Il est important que l’étape 7 n’entraîne qu’une interruption minime pour les locataires et qu’aucune donnée ne soit perdue. Pour remplir cet objectif, le processus utilise la géoréplication.

Avant de géorépliquer chaque base de données, la base de données correspondante dans la région d’origine est supprimée. La base de données située dans la région de récupération est alors géorépliquée, ce qui crée un réplica secondaire dans la région d’origine. Une fois la réplication terminée, le locataire est marqué hors connexion dans le catalogue, ce qui met fin à toutes les connexions à la base de données dans la région de récupération. Ensuite, la base de données bascule, ce qui entraîne le traitement de toutes les transactions en attente dans la base de données secondaire afin de ne perdre aucune donnée. 

Pendant le basculement, les rôles de base de données sont inversés. La base de données secondaire dans la région d’origine devient la base de données primaire en lecture-écriture, et la base de données dans la région de récupération devient une base de données secondaire en lecture seule. L’entrée du locataire dans le catalogue est mise à jour pour faire référence à la base de données dans la région d’origine et le locataire est marqué comme en ligne. À ce stade, le rapatriement de la base de données est terminé. 

Il est préférable d’écrire les applications avec une logique de nouvelle tentative pour veiller à ce qu’elles se reconnectent automatiquement quand les connexions sont interrompues. Quand elles utilisent le catalogue pour négocier la reconnexion, elles se connectent à la base de données rapatriée dans la région d’origine. Bien que la brève déconnexion n’est souvent pas remarquée, vous pouvez choisir de rapatrier les bases de données en dehors des heures de bureau.

Une fois qu’une base de données est rapatriée, la base de données secondaire dans la région de récupération peut être supprimée. La base de données dans la région d’origine compte alors sur la géorestauration pour obtenir à nouveau une protection de reprise d’activité.

À l’étape 8, les ressources incluses dans la région de récupération, notamment les serveurs de récupération et les pools, sont supprimés.

## <a name="run-the-repatriation-script"></a>Exécuter le script de rapatriement
Considérons que la panne est résolue et que vous exécutez le script de rapatriement.

Si vous avez suivi le tutoriel, le script réactive immédiatement Fabrikam Jazz Club et Dogwood Dojo dans leur région d’origine, car ils n’ont pas changé. Ensuite, il rapatrie le nouveau locataire, Hawthorn Hall, puis Contoso Concert Hall, car ce dernier a été modifié. Le script rapatrie également le catalogue qui a été mis à jour lors du provisionnement de Hawthorn Hall.
  
1. Dans PowerShell ISE, dans le script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1, vérifiez que le processus synchronisation de catalogue est toujours en cours d’exécution dans son instance PowerShell. Au besoin, relancez-le en définissant :

    $DemoScenario = 1 : pour démarrer la synchronisation des informations de configuration du serveur de locataire, du pool et de la base de données dans le catalogue.

    Appuyez sur F5 pour exécuter le script.

2.  Ensuite, pour lancer le processus de rapatriement, définissez :

    $DemoScenario = 5 : pour rapatrier l’application dans sa région d’origine.

    Appuyez sur F5 pour exécuter le script de récupération dans une nouvelle fenêtre PowerShell. Le rapatriement prend plusieurs minutes, et vous pouvez le surveiller dans la fenêtre PowerShell.

3. Pendant l’exécution du script, actualisez la page du hub d’événements (http://events.wingtip-dpt.&lt;user&gt;.trafficmanager.net).

    Remarquez que tous les locataires sont en ligne et accessibles pendant ce processus.

4. Sélectionnez Fabrikam Jazz Club pour l’ouvrir. Si vous n’avez pas modifié ce locataire, remarquez, dans le pied de page, que le serveur d’origine est déjà rétabli.

5. Ouvrez ou actualisez la page d’événements de salle de concert Contoso. À partir du pied de page, notez que, la base de données est toujours sur le serveur -recovery au départ. 

6. Actualisez la page des événements de Contoso Concert Hall une fois que le processus de rapatriement est terminé et remarquez que la base de données se trouve à présent dans votre région d’origine.

7. Réactualisez le hub d’événements et ouvrez Hawthorn Hall. Notez que sa base de données se trouve également dans la région d’origine. 

## <a name="clean-up-recovery-region-resources-after-repatriation"></a>Nettoyer les ressources de la région de récupération après le rapatriement
Une fois le rapatriement terminé, il est plus prudent de supprimer les ressources dans la région de récupération. 

> [!IMPORTANT]
> Supprimez rapidement ces ressources pour stopper leur facturation.

Le processus de restauration crée toutes les ressources de récupération dans un groupe de ressources de récupération. Le processus de nettoyage supprime ce groupe de ressources du catalogue, ainsi que toutes les références qui s’y rapportent. 

1. Dans PowerShell ISE, dans le script ...\Learning Modules\Business Continuity and Disaster Recovery\DR-RestoreFromBackup\Demo-RestoreFromBackup.ps1, définissez :
    
    $DemoScenario = 6 : supprimer les ressources obsolètes de la région de récupération.

2. Appuyez sur F5 pour exécuter le script.

Une fois les scripts nettoyés, l’application a retrouvé son emplacement de démarrage. À ce stade, vous pouvez réexécuter le script ou tester d’autres didacticiels.

## <a name="designing-the-application-to-ensure-that-the-app-and-the-database-are-co-located"></a>Concevoir l’application pour garantir la colocalisation de l’application et de la base de données 
L’application est conçue pour toujours se connecter à partir d’une instance située dans la même région que la base de données de locataire. Cette conception réduit la latence entre l’application et la base de données. Cette optimisation suppose que l’interaction entre l’application et la base de données génère davantage d’échanges que celle entre l’utilisateur et l’application.  

Les bases de données de locataire peuvent être réparties entre les régions d’origine et de récupération pendant quelque temps lors du rapatriement. Pour chaque base de données, l’application recherche le DNS sur le serveur du locataire, dans la région où la base de données se trouve. Le nom du serveur est un alias. L’alias du nom du serveur contient le nom de la région. Si l’application ne se trouve pas dans la même région que la base de données, elle se redirige vers l’instance située dans la même région que le serveur. Cette redirection vers l’instance dans la même région que celle de la base de données minimise la latence entre l’application et la base de données.  

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :
> [!div class="checklist"]
> 
> * Utiliser le catalogue de locataire pour conserver régulièrement les informations de configuration actualisées, et ainsi créer un environnement de récupération d’image miroir dans une autre région.
> * Récupérer des bases de données dans la région de récupération en utilisant la géorestauration.
> * Mettre à jour le catalogue de locataire pour refléter les emplacements de bases de données de locataire restaurées. 
> * Utiliser un alias DNS pour permettre à une application de se connecter au catalogue de locataire sans reconfiguration.
> * Utiliser la géoréplication pour rapatrier des bases de données récupérées dans leur région d’origine après la résolution d’une panne.

Intéressez-vous au didacticiel [Reprise d’activité pour une application SaaS multilocataire à l’aide de la géoréplication de bases de données](./saas-dbpertenant-dr-geo-replication.md) pour apprendre à utiliser la géoréplication pour réduire considérablement le temps nécessaire pour récupérer une application multilocataire à grande échelle.

## <a name="additional-resources"></a>Ressources supplémentaires

[Autres didacticiels reposant sur l’application SaaS Wingtip](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)