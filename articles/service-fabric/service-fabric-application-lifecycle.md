---
title: Cycle de vie des applications dans Service Fabric
description: Décrit le développement, le déploiement, le test, la mise à niveau, la maintenance et la suppression d’applications Service Fabric.
ms.topic: conceptual
ms.date: 1/19/2018
ms.openlocfilehash: 6a36c97c6f1be96dcb8353e886f2159929e8e794
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86248308"
---
# <a name="service-fabric-application-lifecycle"></a>Cycle de vie des applications Service Fabric
Comme pour les autres plateformes, une application sur Azure Service Fabric passe généralement par les phases suivantes : conception, développement, test, déploiement, mise à niveau, maintenance et suppression. Service Fabric offre une excellente prise en charge du cycle de vie complet des applications cloud : du développement au retrait éventuel, en passant par le déploiement, la gestion quotidienne et la maintenance. Le modèle de service permet à différents rôles de participer indépendamment au cycle de vie des applications. Cet article fournit une vue d'ensemble des API et de la façon dont elles sont utilisées par les différents rôles pendant les phases du cycle de vie des applications Service Fabric.

[!INCLUDE [links to azure cli and service fabric cli](../../includes/service-fabric-sfctl.md)]

## <a name="service-model-roles"></a>Rôles de modèle de service
Les rôles de modèle de service sont les suivants :

* **Développeur de service** : développe des services génériques et modulaires qui peuvent être réaffectés et utilisés dans plusieurs applications du même type ou de différents types. Par exemple, un service de file d'attente peut être utilisé pour la création d'une application de gestion de tickets (support technique) ou d'une application de commerce électronique (panier).
* **Développeur d’application** : crée des applications en intégrant un ensemble de services pour répondre à des scénarios ou exigences spécifiques. Par exemple, un site web de commerce électronique peut intégrer un service frontal sans état JSON, un service sans état d’enchères et un service avec état de file d’attente pour créer une solution de vente aux enchères.
* **Administrateur d’application** : prend des décisions sur la configuration de l’application (indication des paramètres de modèle de configuration), le déploiement (mappage aux ressources disponibles) et la qualité de service. Par exemple, un administrateur d’application détermine la langue locale de l’application (anglais pour les États-Unis ou japonais pour le Japon, par exemple). Une application déployée différente peut avoir différents paramètres.
* **Opérateur** : déploie des applications basées sur la configuration et les spécifications définies par l’administrateur d’application. Par exemple, un opérateur met en service et déploie l'application et s'assure qu'elle s'exécute dans Azure. Les opérateurs surveillent les informations d'intégrité et de performances des applications et gèrent l'infrastructure physique en fonction des besoins.

## <a name="develop"></a>Développer
1. Un *développeur de service* développe différents types de services à l’aide du modèle de programmation [Reliable Actors](service-fabric-reliable-actors-introduction.md) ou [Reliable Services](service-fabric-reliable-services-introduction.md).
2. Un *développeur de service* décrit de manière déclarative les types de service développés dans un fichier de manifeste de service comprenant au moins un package de code, de configuration et de données.
3. Un *développeur d'application* crée ensuite une application à l'aide de différents types de service.
4. Un *développeur d'application* décrit de manière déclarative le type d'application dans un manifeste d'application en référençant les manifestes de service des services constitutifs et en remplaçant et paramétrant correctement les différents paramètres de configuration et de déploiement des services constitutifs.

Pour obtenir des exemples, consultez [Prise en main de Reliable Actors](service-fabric-reliable-actors-get-started.md) et [Prise en main de Reliable Services](service-fabric-reliable-services-quick-start.md).

## <a name="deploy"></a>Déployer
1. Un *administrateur d’application* personnalise le type d’application en une application spécifique en vue de son déploiement sur un cluster Service Fabric en spécifiant les paramètres appropriés de l’élément **ApplicationType** du manifeste d’application.
2. Un *opérateur* charge le package d’application vers le magasin d’images du cluster à l’aide de la [méthode **CopyApplicationPackage**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient) ou de [l’applet de commande **Copy-ServiceFabricApplicationPackage**](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps). Le package d'application contient le manifeste d'application et la collection de packages de service. Service Fabric déploie les applications à partir du package d’application stocké dans le magasin d’images, qui peut être un magasin d’objets blob Azure ou le service système Service Fabric.
3. Ensuite, *l’opérateur* approvisionne le type d’application dans le cluster cible à partir du package d’application chargé à l’aide de la [méthode **ProvisionApplicationAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), de [l’applet de commande **Register-ServiceFabricApplicationType**](/powershell/module/servicefabric/register-servicefabricapplicationtype) ou de [l’opération REST **d’approvisionnement d’une application**](/rest/api/servicefabric/provision-an-application).
4. Une fois l’application approvisionnée, un *opérateur* la démarre avec les paramètres fournis par *l’administrateur d’application* à l’aide de la [méthode **CreateApplicationAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), de [l’applet de commande **New-ServiceFabricApplication**](/powershell/module/servicefabric/new-servicefabricapplication) ou de [l’opération REST de **création d’application**](/rest/api/servicefabric/create-an-application).
5. Une fois l’application déployée, un *opérateur* utilise la [méthode **CreateServiceAsync**](/dotnet/api/system.fabric.fabricclient.servicemanagementclient), [l’applet de commande **New-ServiceFabricService**](/powershell/module/servicefabric/new-servicefabricservice) ou [l’opération REST de **création de service**](/rest/api/servicefabric/create-a-service) pour créer des instances de service pour l’application en fonction des types de services disponibles.
6. À ce stade, l'application est en cours d'exécution dans le cluster Service Fabric.

Pour obtenir des exemples, consultez [Déployer une application](service-fabric-deploy-remove-applications.md) .

## <a name="test"></a>Test
1. Après le déploiement sur le cluster de développement local ou sur un cluster de test, un *développeur de service* exécute le scénario de test de basculement intégré à l’aide des classes [**FailoverTestScenarioParameters**](/dotnet/api/system.fabric.testability.scenario.failovertestscenarioparameters) et [**FailoverTestScenario**](/dotnet/api/system.fabric.testability.scenario.failovertestscenario) ou de l’applet de commande [**Invoke-ServiceFabricFailoverTestScenario**](/powershell/module/servicefabric/invoke-servicefabricfailovertestscenario?view=azureservicefabricps). Le scénario de test de basculement exécute un service spécifié en procédant à des transitions et à des basculements intensifs pour s’assurer de sa disponibilité et de son bon fonctionnement.
2. Le *développeur de service* exécute ensuite le scénario de test de chaos intégré à l’aide des classes [**ChaosTestScenarioParameters**](/dotnet/api/system.fabric.testability.scenario.chaostestscenarioparameters) et [**ChaosTestScenario**](/dotnet/api/system.fabric.testability.scenario.chaostestscenario) ou de [l’applet de commande **Invoke-ServiceFabricChaosTestScenario**](/powershell/module/servicefabric/invoke-servicefabricchaostestscenario?view=azureservicefabricps). Le scénario de test chaos provoque aléatoirement plusieurs défaillances de nœud, de package de code et de réplica dans le cluster.
3. Le *développeur de service* [teste la communication entre les services](service-fabric-testability-scenarios-service-communication.md) en créant des scénarios de test qui déplacent des réplicas principaux dans le cluster.

Consultez la rubrique [Introduction au service d’analyse des erreurs](service-fabric-testability-overview.md) pour plus d'informations.

## <a name="upgrade"></a>Mettre à niveau
1. Un *développeur de service* met à jour les services constitutifs de l’application instanciée et/ou corrige les bogues et fournit une nouvelle version du manifeste de service.
2. Un *développeur d'application* substitue et ajuste les paramètres de configuration et de déploiement des services constitutifs et fournit une nouvelle version du manifeste d'application. Le développeur d'application intègre ensuite les nouvelles versions des manifestes de service à l'application et fournit une nouvelle version du type d'application dans un package d'application mis à jour.
3. Un *administrateur d'application* incorpore la nouvelle version du type d'application dans l'application cible en mettant à jour les paramètres appropriés.
4. Un *opérateur* charge le package d’application mis à jour vers le magasin d’images du cluster à l’aide de la [méthode **CopyApplicationPackage**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient) ou de [l’applet de commande **Copy-ServiceFabricApplicationPackage**](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps). Le package d'application contient le manifeste d'application et la collection de packages de service.
5. Un *opérateur* approvisionne la nouvelle version de l’application dans le cluster cible à l’aide de la [méthode **ProvisionApplicationAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), de [l’applet de commande **Register-ServiceFabricApplicationType**](/powershell/module/servicefabric/register-servicefabricapplicationtype) ou de [l’opération REST **d’approvisionnement d’une application**](/rest/api/servicefabric/provision-an-application).
6. Un *opérateur* met à niveau l’application cible vers la nouvelle version à l’aide de la [méthode **UpgradeApplicationAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), de [l’applet de commande **Start-ServiceFabricApplicationUpgrade**](/powershell/module/servicefabric/start-servicefabricapplicationupgrade) ou de [l’opération REST de **mise à niveau d’une application**](/rest/api/servicefabric/upgrade-an-application).
7. Un *opérateur* vérifie la progression de la mise à niveau à l’aide de la [méthode **GetApplicationUpgradeProgressAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), de [l’applet de commande **Get-ServiceFabricApplicationUpgrade**](/powershell/module/servicefabric/get-servicefabricapplicationupgrade) ou de [l’opération REST **d’obtention de la progression de la mise à niveau d’une application**](/rest/api/servicefabric/get-the-progress-of-an-application-upgrade1).
8. Si nécessaire, *l’opérateur* modifie et applique de nouveau les paramètres de la mise à niveau d’application actuelle à l’aide de la [méthode **UpdateApplicationUpgradeAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), de [l’applet de commande **Update-ServiceFabricApplicationUpgrade**](/powershell/module/servicefabric/update-servicefabricapplicationupgrade) ou de [l’opération REST de **mise à jour d’une mise à niveau d’application**](/rest/api/servicefabric/update-an-application-upgrade).
9. Si nécessaire, *l’opérateur* restaure la mise à niveau d’application actuelle à l’aide de la [méthode **RollbackApplicationUpgradeAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), de [l’applet de commande **Start-ServiceFabricApplicationRollback**](/powershell/module/servicefabric/start-servicefabricapplicationrollback) ou de [l’opération REST de **restauration d’une mise à niveau d’application**](/rest/api/servicefabric/rollback-an-application-upgrade).
10. Service Fabric met à niveau l'application cible en cours d'exécution dans le cluster sans perdre la disponibilité des services qui la composent.

Pour obtenir des exemples, consultez [Didacticiel de mise à niveau d’application](service-fabric-application-upgrade-tutorial.md) .

## <a name="maintain"></a>Maintenance
1. Pour les mises à niveau et les correctifs du système d'exploitation, Service Fabric interagit avec l'infrastructure Azure pour garantir la disponibilité de toutes les applications en cours d'exécution dans le cluster.
2. Pour les mises à niveau et les correctifs de la plateforme Service Fabric, Service Fabric se met à niveau sans perdre la disponibilité des applications en cours d’exécution dans le cluster.
3. Un *administrateur d'application* approuve l'ajout ou la suppression de nœuds d'un cluster après avoir analysé les données historiques relatives à l'utilisation des capacités et estimé la demande future.
4. Un *opérateur* ajoute et supprime des nœuds spécifiés par *l’administrateur d’application*.
5. Quand de nouveaux nœuds sont ajoutés au cluster ou que des nœuds existants sont supprimés de celui-ci, Service Fabric équilibre automatiquement la charge des applications en cours d’exécution sur tous les nœuds du cluster pour obtenir des performances optimales.

## <a name="remove"></a>Supprimer
1. Un *opérateur* peut supprimer une instance spécifique d’un service en cours d’exécution dans le cluster sans supprimer l’ensemble de l’application à l’aide de la [méthode **DeleteServiceAsync**](/dotnet/api/system.fabric.fabricclient.servicemanagementclient), de [l’applet de commande **Remove-ServiceFabricService**](/powershell/module/servicefabric/remove-servicefabricservice) ou de [l’opération REST de **suppression d’un service**](/rest/api/servicefabric/delete-a-service).  
2. Un *opérateur* peut également supprimer une instance d’application et tous ses services à l’aide de la [méthode **DeleteApplicationAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), de [l’applet de commande **Remove-ServiceFabricApplication**](/powershell/module/servicefabric/remove-servicefabricapplication) ou de [l’opération REST de **suppression d’une application**](/rest/api/servicefabric/delete-an-application).
3. Une fois que l’application et les services sont arrêtés, *l’opérateur* peut annuler l’approvisionnement du type d’application à l’aide de la [méthode **UnprovisionApplicationAsync**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient), de [l’applet de commande **Unregister-ServiceFabricApplicationType**](/powershell/module/servicefabric/unregister-servicefabricapplicationtype) ou de [l’opération REST **d’annulation de l’approvisionnement d’une application**](/rest/api/servicefabric/unprovision-an-application). L’annulation de la mise en service du type d’application ne supprime pas le package d’application du magasin d’images. Vous devez donc supprimer le package d’application manuellement.
4. Un *opérateur* supprime le package d’application du magasin d’images à l’aide de la [méthode **RemoveApplicationPackage**](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient) ou de [l’applet de commande **Remove-ServiceFabricApplicationPackage**](/powershell/module/servicefabric/remove-servicefabricapplicationpackage?view=azureservicefabricps).

Pour obtenir des exemples, consultez [Déployer une application](service-fabric-deploy-remove-applications.md) .

## <a name="next-steps"></a>Étapes suivantes
Pour plus d'informations sur le développement, le test et la gestion des services et des applications Service Fabric, consultez :

* [Acteurs fiables (Reliable Actors)](service-fabric-reliable-actors-introduction.md)
* [Services fiables (Reliable Services)](service-fabric-reliable-services-introduction.md)
* [Déployer une application](service-fabric-deploy-remove-applications.md)
* [Mise à niveau de l’application](service-fabric-application-upgrade.md)
* [Vue d’ensemble de la testabilité](service-fabric-testability-overview.md)
