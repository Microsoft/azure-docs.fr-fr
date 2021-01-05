---
title: Mise à niveau des applications Service Fabric
description: Cet article fournit une introduction à la mise à niveau d'une application Service Fabric, y compris le choix des modes de mise à niveau et les vérifications d'intégrité exécutées.
ms.topic: conceptual
ms.date: 8/5/2020
ms.openlocfilehash: 8eecd923b009ecbe9f4e607ad57a99b3f20955b9
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92309848"
---
# <a name="service-fabric-application-upgrade"></a>Mise à niveau des applications Service Fabric
Une application Azure Service Fabric est une collection de services. Pendant une mise à niveau, Service Fabric compare le nouveau [manifeste d'application](service-fabric-application-and-service-manifests.md) à la version précédente et détermine les services qui, dans l'application, nécessitent des mises à jour. Service Fabric compare les numéros des versions dans les manifestes de service avec les numéros des versions dans la version précédente. Si un service n'a pas changé, ce service n'est pas mis à niveau.

> [!NOTE]
> Les [ApplicationParameter](/dotnet/api/system.fabric.description.applicationdescription.applicationparameters?view=azure-dotnet#System_Fabric_Description_ApplicationDescription_ApplicationParameters) ne sont pas conservés dans une mise à niveau d’application. Pour conserver les paramètres d’application actuels, l’utilisateur doit d’abord récupérer les paramètres et les transmettre à l’appel d’API de mise à niveau, comme ci-dessous :
```powershell
$myApplication = Get-ServiceFabricApplication -ApplicationName fabric:/myApplication
$appParamCollection = $myApplication.ApplicationParameters

$applicationParameterMap = @{}
foreach ($pair in $appParamCollection)
{
    $applicationParameterMap.Add($pair.Name, $pair.Value);
}

Start-ServiceFabricApplicationUpgrade -ApplicationName fabric:/myApplication -ApplicationTypeVersion 2.0.0 -ApplicationParameter $applicationParameterMap -Monitored -FailureAction Rollback
```

## <a name="rolling-upgrades-overview"></a>Vue d'ensemble des mises à niveau propagées
Une mise à niveau d'application propagée s'effectue par étapes. À chaque étape, la mise à niveau est appliquée à un sous-ensemble de nœuds du cluster, appelé domaine de mise à jour. Ainsi, l'application demeure disponible tout au long de la mise à niveau. Au cours de la mise à niveau, le cluster peut contenir une combinaison des versions ancienne et nouvelle.

De ce fait, les deux versions doivent être mutuellement compatibles. Si ce n'est pas le cas, l'administrateur d'application doit effectuer une mise à niveau échelonnée pour maintenir la disponibilité. Dans une mise à niveau à plusieurs phases, la première étape consiste à effectuer une mise à niveau vers une version intermédiaire de l’application qui est compatible avec la version précédente. La deuxième étape consiste à mettre à niveau la version finale qui interrompt la compatibilité avec la version antérieure à la mise à jour, mais qui est compatible avec la version intermédiaire.

Les domaines de mise à jour sont spécifiés dans le manifeste de cluster quand vous configurez le cluster. Les domaines de mise à jour ne reçoivent pas les mises à jour dans un ordre particulier. Un domaine de mise à jour est une unité logique de déploiement pour une application. Grâce aux domaines de mise à jour, les services demeurent hautement disponibles pendant une mise à niveau.

Une mise à niveau non propagée est possible si elle est appliquée à tous les nœuds du cluster, ce qui est le cas quand l'application n'a qu'un seul domaine de mise à jour. Cette approche n’est pas recommandée, car le service est arrêté et indisponible au moment de la mise à niveau. En outre, Azure ne fournit aucune garantie quand un cluster est configuré avec un seul domaine de mise à jour.

Une fois la mise à niveau terminée, tous les services et les réplicas (instances) restent dans la même version ; en d’autres termes, si la mise à niveau réussit, ils sont mis à jour vers la nouvelle version, alors que si la mise à niveau échoue et est restaurée, ils sont restaurés vers l’ancienne version.

## <a name="health-checks-during-upgrades"></a>Vérifications d'intégrité pendant les mises à niveau
Pour une mise à niveau, vous devez définir des stratégies d'intégrité (ou utiliser éventuellement des valeurs par défaut). Une mise à niveau est considérée comme ayant réussi quand tous les domaines de mise à jour sont mis à niveau dans les délais spécifiés et que tous les domaines de mise à jour sont considérés comme intègres.  Un domaine de mise à jour intègre signifie que le domaine de mise à jour a réussi toutes les vérifications d'intégrité spécifiées dans la stratégie de contrôle d'intégrité. Par exemple, une stratégie d'intégrité peut imposer que tous les services dans une instance d'application doivent être *intègres*, car l'intégrité est définie par Service Fabric.

Pendant une mise à jour effectuée par Service Fabric, les stratégies et vérifications d'intégrité sont indépendantes du service et de l'application. Autrement dit, aucun test spécifique au service n'est exécuté.  Par exemple, votre service peut avoir une exigence de débit, mais Service Fabric ne dispose pas des informations pour vérifier le débit. Consultez les [articles sur l’intégrité](service-fabric-health-introduction.md) pour connaître les vérifications qui sont effectuées. Les contrôles qui se produisent pendant une mise à niveau incluent des tests pour savoir si le package d'application a été copié correctement, si l'instance a été démarrée, etc.

L'intégrité de l'application est un regroupement des entités enfants de l'application. En bref, Service Fabric évalue l'intégrité de l'application au travers de l'intégrité établie au sujet de celle-ci. Il évalue également l'intégrité de la totalité des services de l'application de cette façon. De plus, Service Fabric évalue l'intégrité des services d'application par l'agrégation de l'intégrité de leurs enfants, tels que le réplica de service. Une fois que la stratégie d'intégrité d'application est satisfaite, la mise à niveau peut continuer. Si la stratégie d'intégrité n'est pas respectée, la mise à niveau de l'application échoue.

## <a name="upgrade-modes"></a>Modes de mise à niveau
Le mode que nous recommandons pour la mise à niveau d’application est le mode surveillé, qui est couramment utilisé. Le mode surveillé effectue la mise à niveau sur un domaine de mise à jour spécifique, puis, si toutes les vérifications d'intégrité sont satisfaites (suivant la stratégie spécifiée), il passe automatiquement au domaine de mise à jour suivant.  Si les vérifications d’intégrité échouent et/ou que les délais d’expiration sont atteints, la mise à niveau est restaurée pour le domaine de mise à jour ou le système bascule sur le mode manuel non surveillé. Vous pouvez configurer la mise à niveau pour choisir l’un de ces deux modes pour les mises à niveau ayant échoué. 

Le mode manuel non surveillé nécessite une intervention manuelle après chaque mise à niveau sur un domaine de mise à jour pour lancer la mise à niveau sur le domaine de mise à jour suivant. Aucune vérification de l'intégrité de Service Fabric n'est effectuée. L'administrateur effectue les vérifications d'intégrité ou d'état avant de commencer la mise à niveau dans le domaine de mise à jour suivant.

## <a name="upgrade-default-services"></a>Mettre à niveau les services par défaut
Certains paramètres de service par défaut définis dans le [manifeste de l’application](service-fabric-application-and-service-manifests.md) peuvent aussi être mis à niveau dans le cadre d’une mise à niveau de l’application. Seuls les paramètres de service qui prennent une modification par le biais de [Update-ServiceFabricService](/powershell/module/servicefabric/update-servicefabricservice?view=azureservicefabricps) peuvent être modifiés dans le cadre d’une mise à niveau. Le comportement de modification des services par défaut au cours de la mise à niveau de l’application est le suivant :

1. Les services par défaut du nouveau manifeste de l’application qui n’existent pas déjà dans le cluster sont créés.
2. Les services par défaut qui existent dans les manifestes de l’application précédent et nouveau sont mis à jour. Les paramètres du service par défaut dans le nouveau manifeste de l’application remplacent ceux du service existant. La mise à niveau de l’application est restaurée automatiquement en cas d’échec de la mise à jour d’un service par défaut.
3. Les services par défaut qui n’existent pas dans le nouveau manifeste de l’application sont supprimés s’ils existent dans le cluster. **Notez que la suppression d’un service par défaut entraîne la suppression de l’état de tout ce service, et que cette opération est définitive.**

Quand vous restaurez une mise à niveau de l’application, les paramètres de service par défaut sont rétablis à leurs valeurs d’avant le démarrage de la mise à niveau, mais les services supprimés ne peuvent pas être recréés avec leur ancien état.

> [!TIP]
> Le paramètre de configuration de cluster [EnableDefaultServicesUpgrade](service-fabric-cluster-fabric-settings.md) doit avoir la valeur *true* pour activer les règles 2) et 3) ci-dessus (mise à jour et suppression de service par défaut). Cette fonctionnalité est prise en charge à partir de Service Fabric version 5.5.

## <a name="upgrading-multiple-applications-with-https-endpoints"></a>Mise à niveau de plusieurs applications avec des points de terminaison HTTPS
Vous devez veiller à ne pas utiliser le **même port** pour différentes instances de la même application lors de l’utilisation de HTTP**S**. En effet, Service Fabric ne pourra plus mettre à niveau le certificat pour l’une des instances de l’application. Par exemple, si l’application 1 ou l’application 2 souhaitent mettre à niveau leur certificat 1 vers le certificat 2. Lors de la mise à niveau, Service Fabric peut avoir nettoyé l’inscription de certificat 1 auprès de http.sys même si l’autre application l’utilise toujours. Pour éviter cela, Service Fabric détecte l’existence d’une autre instance d’application inscrite sur le port avec le certificat (en raison de http.sys) et l’opération échoue.

Par conséquent, Service Fabric ne prend pas en charge la mise à niveau de deux services utilisant **le même port** dans des instances d’application différentes. En d’autres termes, vous ne pouvez pas utiliser le même certificat sur les différents services au niveau du même port. Si vous avez besoin d’un certificat partagé sur le même port, vous devez vous assurer que les services sont placés sur des machines différentes avec des contraintes de placement. Vous pouvez aussi envisager d’utiliser le cas échéant les ports dynamiques Service Fabric pour chaque service de chaque instance d’application. 

Si une mise à niveau avec https échoue, un avertissement d’erreur indique que l’API du serveur HTTP Windows ne prend pas en charge plusieurs certificats pour les applications qui partagent le même port.

## <a name="application-upgrade-flowchart"></a>Organigramme de la mise à niveau d'application
L’organigramme suivant ce paragraphe peut vous aider à comprendre le processus de mise à niveau d’une application Service Fabric. En particulier, le flux indique dans quelle mesure les délais, notamment *HealthCheckStableDuration*, *HealthCheckRetryTimeout* et *UpgradeHealthCheckInterval*, déterminent l’échec ou la réussite de la mise à niveau dans un domaine de mise à jour donné.

![Processus de mise à niveau d'une application Service Fabric][image]

## <a name="next-steps"></a>Étapes suivantes
[mise à niveau de votre application à l’aide de Visual Studio](service-fabric-application-upgrade-tutorial.md) vous guide à travers une mise à niveau de l’application à l’aide de Visual Studio.

[Mise à niveau de votre application à l’aide de PowerShell](service-fabric-application-upgrade-tutorial-powershell.md) vous guide à travers une mise à niveau de l’application à l’aide de PowerShell.

Contrôlez les mises à niveau de votre application à l'aide des [Paramètres de mise à niveau](service-fabric-application-upgrade-parameters.md).

Rendez les mises à niveau de votre application compatibles en apprenant à utilisez la [Sérialisation des données](service-fabric-application-upgrade-data-serialization.md).

Apprenez à utiliser les fonctionnalités avancées lors de la mise à niveau de votre application en consultant les [Rubriques avancées](service-fabric-application-upgrade-advanced.md).

Résolvez les problèmes courants de mise à niveau de l’application en vous reportant aux étapes de [Résolution des problèmes de mise à niveau des applications](service-fabric-application-upgrade-troubleshooting.md).

[image]: media/service-fabric-application-upgrade/service-fabric-application-upgrade-flowchart.png