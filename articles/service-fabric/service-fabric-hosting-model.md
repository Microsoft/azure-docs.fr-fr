---
title: Modèle d’hébergement Azure Service Fabric
description: Décrit la relation entre les réplicas (ou instances) d’un service Service Fabric déployé et le processus hôte du service.
author: harahma
ms.topic: conceptual
ms.date: 04/15/2017
ms.author: harahma
ms.custom: devx-track-csharp
ms.openlocfilehash: 5f3f6238bb72704d13fef4a7171aeaebee5f9141
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91708694"
---
# <a name="azure-service-fabric-hosting-model"></a>Modèle d’hébergement Azure Service Fabric
Cet article fournit une vue d’ensemble des modèles d’hébergement d’applications fournis par Azure Service Fabric et décrit les différences entre les modèles à **processus partagé** et à **processus exclusif**. Il décrit le fonctionnement d’une application déployée sur un nœud Service Fabric, et présente la relation entre les réplicas (ou instances) du service et le processus hôte du service.

Avant de continuer, assurez-vous de bien avoir compris les différents concepts et relations expliqués dans [Modéliser une application dans Service Fabric][a1]. 

> [!NOTE]
> Dans cet article, à moins qu’un sens différent n’ait été explicitement fourni :
>
> - Le terme *réplica* fait référence à la fois au réplica d’un service avec état et à une instance d’un service sans état.
> - *CodePackage* est traité de la même manière que le processus *ServiceHost*, qui inscrit un *ServiceType*, et héberge des réplicas de services du même *ServiceType*.
>

Pour comprendre le modèle d’hébergement, commençons par un exemple. Imaginons que nous ayons un *ApplicationType* « MyAppType », avec un *ServiceType* « MyServiceType ». « MyServiceType » est fourni par le *ServicePackage* « MyServicePackage », qui a un *CodePackage* « MyCodePackage ». « MyCodePackageB » inscrit le *ServiceType* « MyServiceType » au moment de son exécution.

Imaginons que nous ayons un cluster à trois nœuds et que nous voulions créer une *application* **fabric:/App1** de type « MyAppType ». À l’intérieur de cette application **fabric:/App1**, nous créons un service **fabric:/App1/ServiceA** de type « MyServiceType ». Ce service comprend deux partitions (par exemple, **P1** et **P2**), et trois réplicas par partition. Le schéma suivant illustre la vue de cette application lorsqu’elle est déployée sur un nœud.


![Diagramme montrant la vue de cette application quand elle est déployée sur un nœud.][node-view-one]


Service Fabric a activé « MyServicePackage », qui a démarré « MyCodePackage », lequel héberge les réplicas des deux partitions. Tous les nœuds du cluster ont la même vue puisque nous avons choisi un nombre de réplicas par partition égal au nombre de nœuds dans le cluster. Nous allons créer un autre service, **fabric:/App1/ServiceB**, dans l’application **fabric:/App1**. Ce service comprend une partition (par exemple, **P3**), et trois réplicas par partition. Le schéma suivant illustre la nouvelle vue sur le nœud :


![Diagramme montrant la nouvelle vue sur le nœud.][node-view-two]


Service Fabric a placé le nouveau réplica de la partition **P3** du service **fabric:/App1/ServiceB** dans l’activation existante de « MyServicePackage ». À présent, créons une autre application **fabric:/App2** de type « MyAppType ». Dans **fabric:/App2**, créez un service **fabric:/App2/ServiceA**. Ce service comprend deux partitions (**P4** et **P5**), et trois réplicas par partition. Le diagramme suivant montre la nouvelle vue du nœud :


![Diagramme montrant la nouvelle vue du nœud.][node-view-three]


Service Fabric active une nouvelle copie de « MyServicePackage », qui démarre une nouvelle copie de « MyCodePackage ». Les réplicas des deux partitions du service **fabric:/App2/ServiceA** (**P4** et **P5**) sont placés dans cette nouvelle copie de « MyCodePackage ».

## <a name="shared-process-model"></a>Modèle à processus partagé
La section précédente décrit le modèle d’hébergement par défaut fourni par Service Fabric, que l’on appelle le modèle à processus partagé. Dans ce modèle, pour une application donnée, seule une copie d’un *ServicePackage* donné est activée sur un nœud (qui démarre tous les *CodePackages* qu’il contient). Tous les réplicas de tous les services d’un *ServiceType* donné sont placés dans le *CodePackage* qui inscrit ce *ServiceType*. En d’autres termes, tous les réplicas de tous les services sur un nœud d’un *ServiceType* donné partagent le même processus.

## <a name="exclusive-process-model"></a>Modèle à processus exclusif
L’autre modèle d’hébergement fourni par Service Fabric est le modèle à processus exclusif. Dans ce modèle, sur un nœud donné, chaque réplica réside dans son propre processus dédié. Service Fabric active une nouvelle copie de *ServicePackage* (qui démarre tous les *CodePackages* qu’il contient). Les réplicas sont placés dans le *CodePackage* qui inscrit le *ServiceType* du service auquel appartient le réplica. 

Si vous utilisez Service Fabric 5.6 ou version ultérieure, vous pouvez choisir le Modèle à processus exclusif lorsque vous créez un service (à l’aide de [PowerShell][p1], [REST][r1] ou [FabricClient][c1]). Spécifiez **ServicePackageActivationMode** en tant que « ExclusiveProcess ».

```powershell
PS C:\>New-ServiceFabricService -ApplicationName "fabric:/App1" -ServiceName "fabric:/App1/ServiceA" -ServiceTypeName "MyServiceType" -Stateless -PartitionSchemeSingleton -InstanceCount -1 -ServicePackageActivationMode "ExclusiveProcess"
```

```csharp
var serviceDescription = new StatelessServiceDescription
{
    ApplicationName = new Uri("fabric:/App1"),
    ServiceName = new Uri("fabric:/App1/ServiceA"),
    ServiceTypeName = "MyServiceType",
    PartitionSchemeDescription = new SingletonPartitionSchemeDescription(),
    InstanceCount = -1,
    ServicePackageActivationMode = ServicePackageActivationMode.ExclusiveProcess
};

var fabricClient = new FabricClient(clusterEndpoints);
await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

Si votre manifeste d’application comporte un service par défaut, vous pouvez choisir le modèle à processus exclusif en spécifiant l’attribut **ServicePackageActivationMode** :

```xml
<DefaultServices>
  <Service Name="MyService" ServicePackageActivationMode="ExclusiveProcess">
    <StatelessService ServiceTypeName="MyServiceType" InstanceCount="1">
      <SingletonPartition/>
    </StatelessService>
  </Service>
</DefaultServices>
```
À présent, nous allons créer un autre service, **fabric:/App1/ServiceC**, dans l’application **fabric:/App1**. Ce service comprend deux partitions (par exemple, **P6** et **P7**), et trois réplicas par partition. Définissez **ServicePackageActivationMode** sur « ExclusiveProcess ». Le diagramme suivant montre la nouvelle vue du nœud :


![Diagramme de la vue du nœud de l’application déployée][node-view-four]


Comme vous pouvez le voir, Service Fabric a activé deux nouvelles copies de « MyServicePackage » (une pour chaque réplica des partitions **P6** et **P7**). Service Fabric a placé chaque réplica dans sa copie dédiée de *CodePackage*. Lorsque le modèle à processus exclusif est utilisé pour une application donnée, plusieurs copies d’un *ServicePackage* donné peuvent être actives sur un nœud. Dans l’exemple précédent, trois copies de « MyServicePackage » sont actives pour **fabric:/App1**. Chacune des copies actives de « MyServicePackage » est associée à un **ServicePackageActivationId**. Cet ID identifie la copie dans l’application **fabric:/App1**.

Lorsque vous utilisez uniquement le modèle à processus partagé pour une application, une seule copie de *ServicePackage* est active sur chaque nœud. Le **ServicePackageActivationId** de cette activation de *ServicePackage* est une chaîne vide. C’est le cas, par exemple, avec **fabric:/App2**.

> [!NOTE]
>- Dans le modèle d’hébergement à processus partagé, **ServicePackageActivationMode** est égal à **SharedProcess**. Il s’agit du modèle d’hébergement par défaut, et il n’est pas nécessaire de spécifier **ServicePackageActivationMode** au moment de la création du service.
>
>- Dans le modèle d’hébergement à processus exclusif, **ServicePackageActivationMode** est égal à **ExclusiveProcess**. Pour utiliser ce paramètre, vous devez le spécifier explicitement au moment de la création du service. 
>
>- Pour voir le Modèle d’hébergement d’un service, interrogez la [description du service][p2] et regardez la valeur de **ServicePackageActivationMode**.
>
>

## <a name="work-with-a-deployed-service-package"></a>Utiliser un package de services déployé
La copie active d’un *ServicePackage* sur un nœud est appelée [package de services déployé][p3]. Lorsque le modèle à processus exclusif est utilisé pour la création de services, il peut y avoir, pour une application donnée, plusieurs packages de services déployés pour le même *ServicePackage*. Si vous effectuez des opérations relatives à un package de services déployé, vous devez fournir le **ServicePackageActivationId** pour identifier le package de services déployé. Par exemple, vous devez fournir l’ID si vous [créez un rapport sur l’intégrité d’un package de services déployé][p4] ou si vous [redémarrez le package de code d’un package de services déployé][p5].

Pour connaître le **ServicePackageActivationId** d’un package de services déployé, interrogez la liste des [packages de services déployés][p3] sur un nœud. Lorsque vous interrogez les [types de services déployés][p6], les [réplicas déployés][p7] et les [packages de code déployés][p8] sur un nœud, le résultat de la requête contient également le **ServicePackageActivationId** du package de services parent déployé.

> [!NOTE]
>- Sous le modèle d’hébergement à processus partagé, une seule copie d’un *ServicePackage* est activée sur un nœud donné et pour une application donnée. Le **ServicePackageActivationId** est alors égal à une *chaîne vide* et il n’est pas nécessaire de le spécifier lors de l’exécution des opérations associées au package de services déployé. 
>
> - Sous le modèle d’hébergement à processus exclusif, une ou plusieurs copies d’un *ServicePackage* peuvent être actives sur un nœud donné et pour une application donnée. Chaque activation est associée à un **ServicePackageActivationId** *non vide*, spécifié lors de l’exécution des opérations associées au package de services déployé. 
>
> - Si le **ServicePackageActivationId** est omis, la valeur par défaut est une *chaîne vide*. En présence d’un package de services déployé qui a été activé sous le modèle à processus partagé, l’opération est exécutée dans ce package. Dans le cas contraire, l’opération échoue.
>
> - Vous ne devez pas interroger les données, puis mettre en cache le **ServicePackageActivationId**. L’ID est généré dynamiquement et peut changer pour différentes raisons. Avant d’effectuer une opération qui nécessite un **ServicePackageActivationId**, vous devez commencer par interroger la liste des [packages de services déployés][p3] sur un nœud. Ensuite, utilisez le **ServicePackageActivationId** généré à partir du résultat de la requête pour exécuter l’opération d’origine.
>
>

## <a name="guest-executable-and-container-applications"></a>Applications d’exécutables invités et de conteneurs
Service Fabric traite les applications d’[exécutable invité][a2] et de [conteneur][a3] en tant que services sans état, qui sont autonomes. Il n’y a pas de runtime Service Fabric dans *ServiceHost* (processus ou conteneur). Étant donné que ces services sont autonomes, le nombre de réplicas par *ServiceHost* n’est pas applicable à ces services. La configuration la plus couramment utilisée avec ces services est la partition unique avec un [InstanceCount][c2] égal à -1 (une seule copie du code de service exécutée sur chaque nœud du cluster). 

Le **ServicePackageActivationMode** par défaut pour ces services est **SharedProcess**. Dans ce cas, Service Fabric active uniquement une copie de *ServicePackage* sur un nœud pour une application donnée.  Cela signifie qu’une seule copie du code de service exécutera un nœud. Si vous souhaitez que plusieurs copies de votre code de service s’exécutent sur un nœud, spécifiez **ServicePackageActivationMode** en tant que **ExclusiveProcess** au moment de la création du service. Par exemple, vous pouvez le faire lorsque vous créez plusieurs services (*Service1* à *ServiceN*) de *ServiceType* (spécifié dans *ServiceManifest*), ou si votre service comprend plusieurs partitions. 

## <a name="change-the-hosting-model-of-an-existing-service"></a>Modifier le modèle d’hébergement d’un service existant
À l’heure actuelle, vous ne pouvez pas modifier le modèle d’hébergement d’un service existant, c’est-à-dire, passer du modèle à processus partagé au modèle à processus exclusif (et inversement).

## <a name="choose-between-the-hosting-models"></a>Choisir le modèle d’hébergement
Vous devez évaluer le modèle d’hébergement qui convient le mieux à vos besoins. Le modèle à processus partagé garantit une meilleure utilisation des ressources du système d’exploitation, car il réduit le nombre de processus générés et permet à plusieurs réplicas du même processus de partager des ports. Toutefois, si l’un des réplicas rencontre une erreur qui l’oblige à arrêter l’hôte de service, cela impacte tous les autres réplicas du même processus.

 Le modèle à processus exclusif isole mieux les réplicas en les associant à un processus dédié. Dans ce cas, si l’un des réplicas rencontre une erreur, cela n’impacte pas les autres réplicas. Ce modèle est utile lorsque le partage de port n’est pas pris en charge par le protocole de communication. Il permet également d’appliquer la gouvernance des ressources au niveau du réplica. En revanche, le modèle à processus exclusif consomme plus de ressources du système d’exploitation puisqu’il génère un processus pour chaque réplica présent sur le nœud.

## <a name="exclusive-process-model-and-application-model-considerations"></a>Considérations relatives au modèle à processus exclusif et au modèle d’application
Pour la plupart des applications, vous pouvez modéliser votre application dans Service Fabric en n’ayant qu’un seul *ServiceType* par *ServicePackage*. 

Dans certains cas d’usage, Service Fabric autorise également plusieurs *ServiceType* par *ServicePackage* (et un même *CodePackage* peut inscrire plusieurs *ServiceType*). Voici quelques scénarios dans lesquels ces configurations peuvent être utiles :

- Vous souhaitez optimiser l’utilisation des ressources du système d’exploitation en générant moins de processus et en augmentant la densité de réplica par processus.
- Les réplicas de différents *ServiceTypes* doivent partager certaines données qui présentent un coût élevé d’initialisation ou de mémoire.
- Vous disposez d’une offre de service gratuite et vous souhaitez limiter l’utilisation des ressources en plaçant tous les réplicas du service dans le même processus.

Le modèle d’hébergement à processus exclusif n’est pas cohérent avec le modèle d’application qui utilise plusieurs *ServiceTypes* par *ServicePackage*. En effet, plusieurs *ServiceTypes* par *ServicePackage* sont destinés à améliorer le partage des ressources entre les réplicas et à augmenter la densité de réplica par processus. Le modèle à processus exclusif à un usage différent.

Prenons le cas de plusieurs *ServiceTypes* par *ServicePackage* avec un *CodePackage* différent qui inscrit chaque *ServiceType*. Prenons l’exemple d’un *ServicePackage*, « MultiTypeServicePackage », comprenant deux *CodePackages* :

- « MyCodePackageA », qui inscrit le *ServiceType* « MyServiceTypeA ».
- « MyCodePackageB », qui inscrit le *ServiceType* « MyServiceTypeB ».

Maintenant, nous allons créer l’application **fabric:/SpecialApp**. Dans **fabric:/SpecialApp**, nous allons créer les deux services suivants avec le modèle à processus exclusif :

- Le service **fabric:/SpecialApp/ServiceA** de type « MyServiceTypeA » avec deux partitions (par exemple, **P1** et **P2**), et trois réplicas par partition.
- Le service **fabric:/SpecialApp/ServiceB** de type « MyServiceTypeB » avec deux partitions (par exemple, **P3** et **P4**), et trois réplicas par partition.

Sur un nœud donné, les deux services ont chacun deux réplicas. Étant donné que nous avons utilisé le modèle à processus exclusif pour créer les services, Service Fabric active une nouvelle copie de « MyServicePackage » pour chaque réplica. Chaque activation de « MultiTypeServicePackage » démarre une copie de « MyCodePackageA » et « MyCodePackageB ». Cependant, le réplica pour lequel « MultiTypeServicePackage » a été activé ne peut être hébergé que par « MyCodePackageA » ou par « MyCodePackageB ». Le diagramme qui suit montre la vue du nœud :


![Diagramme montrant la vue du nœud.][node-view-five]


Dans l’activation de « MultiTypeServicePackage » pour le réplica de la partition **P1** du service **fabric:/SpecialApp/ServiceA**, « MyCodePackageA » héberge le réplica. « MyCodePackageB » est en cours d’exécution. De même, dans l’activation de « MultiTypeServicePackage » pour le réplica de la partition **P3** du service **fabric:/SpecialApp/ServiceB**, « MyCodePackageB » héberge le réplica. « MyCodePackageA » est en cours d’exécution. Par conséquent, plus il y a de *CodePackages* (inscrivant différents *ServiceTypes*) par *ServicePackage*, plus on utilise de ressources redondantes. 
 
 En revanche, si nous créons les services **fabric:/SpecialApp/ServiceA** et **fabric:/SpecialApp/ServiceB** avec le modèle à processus partagé, Service Fabric active une seule copie de « MultiTypeServicePackage » pour l’application **fabric:/SpecialApp**. « MyCodePackageA » héberge tous les réplicas pour le service **fabric:/SpecialApp/ServiceA**. « MyCodePackageB » héberge tous les réplicas pour le service **fabric:/SpecialApp/ServiceB**. Le schéma qui suit présente la vue du nœud dans ce paramètre : 


![Diagramme de la vue du nœud de l’application déployée][node-view-six]


Dans l’exemple précédent, vous pourriez penser que si « MyCodePackageA » inscrit « MyServiceTypeA » et « MyServiceTypeB », alors qu’il n’existe pas de « MyCodePackageB », aucun *CodePackage* redondant n’est exécuté. Même si vous avez raison, ce modèle d’application n’est pas aligné sur le modèle d’hébergement à processus exclusif. Si l’objectif est de placer chaque réplica dans son propre processus dédié, il n’est pas nécessaire d’inscrire les deux *ServiceTypes* du même *CodePackage*. Il suffit de placer chaque *ServiceType* dans son propre *ServicePackage*.

### <a name="reliable-services-and-actor-forking-subprocesses"></a>Sous-processus de duplication Reliable Services et Actor

Service Fabric ne prend pas en charge les sous-processus de duplication de Reliable Services et Reliable Actors. À titre d'exemple sur la raison de cette absence de prise en charge, [CodePackageActivationContext](/dotnet/api/system.fabric.codepackageactivationcontext?view=azure-dotnet) ne peut pas être utilisé pour inscrire un sous-processus non pris en charge et les jetons d’annulation sont uniquement envoyés à des processus enregistrés, ce qui engendre toutes sortes de problèmes, telles que les échecs de mise à niveau, lorsque les sous-processus ne ferment pas une fois le jeton d’annulation reçu par le processus parent.

## <a name="next-steps"></a>Étapes suivantes
[Empaquetez une application][a4] et préparez-la pour le déploiement.

[Déployer et supprimer des applications][a5]. Cet article explique comment utiliser PowerShell pour gérer des instances d’application.

<!--Image references-->
[node-view-one]: ./media/service-fabric-hosting-model/node-view-one.png
[node-view-two]: ./media/service-fabric-hosting-model/node-view-two.png
[node-view-three]: ./media/service-fabric-hosting-model/node-view-three.png
[node-view-four]: ./media/service-fabric-hosting-model/node-view-four.png
[node-view-five]: ./media/service-fabric-hosting-model/node-view-five.png
[node-view-six]: ./media/service-fabric-hosting-model/node-view-six.png

<!--Link references--In actual articles, you only need a single period before the slash-->
[a1]: service-fabric-application-model.md
[a2]: service-fabric-guest-executables-introduction.md
[a3]: service-fabric-containers-overview.md
[a4]: service-fabric-package-apps.md
[a5]: service-fabric-deploy-remove-applications.md

[r1]: /rest/api/servicefabric/sfclient-api-createservice

[c1]: /dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync
[c2]: /dotnet/api/system.fabric.description.statelessservicedescription.instancecount

[p1]: /powershell/module/servicefabric/new-servicefabricservice
[p2]: /powershell/module/servicefabric/get-servicefabricservicedescription
[p3]: /powershell/module/servicefabric/get-servicefabricdeployedservicepackage
[p4]: /powershell/module/servicefabric/send-servicefabricdeployedservicepackagehealthreport
[p5]: /powershell/module/servicefabric/restart-servicefabricdeployedcodepackage
[p6]: /powershell/module/servicefabric/get-servicefabricdeployedservicetype
[p7]: /powershell/module/servicefabric/get-servicefabricdeployedreplica
[p8]: /powershell/module/servicefabric/get-servicefabricdeployedcodepackage
