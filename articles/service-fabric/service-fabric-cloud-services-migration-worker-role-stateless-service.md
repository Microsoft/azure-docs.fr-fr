---
title: Convertir des applications Azure Cloud Services en Service Fabric
description: Ce guide compare les rôles web et de travail des services cloud aux services sans état Service Fabric afin de faciliter la migration des services cloud vers Service Fabric.
ms.topic: conceptual
ms.date: 11/02/2017
ms.custom: devx-track-csharp
ms.openlocfilehash: cf593f793aabf2a0650684ed8d02fe02d756ec2b
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575735"
---
# <a name="guide-to-converting-web-and-worker-roles-to-service-fabric-stateless-services"></a>Guide de conversion des rôles web et de travail en services sans état Service Fabric
Cet article explique comment migrer vos rôles web et de travail des services cloud vers les services sans état Service Fabric. Il s’agit de la manière la plus simple de migrer des applications dont l’architecture globale va rester quasi identique des services cloud vers Service Fabric.

## <a name="cloud-service-project-to-service-fabric-application-project"></a>Passer d’un projet de service cloud à un projet d’application Service Fabric
 Les projets de service cloud et les projets d’application Service Fabric présentent une structure similaire. Les deux représentent l’unité de déploiement de votre application : autrement dit, ils définissent tous deux l’ensemble du package déployé pour exécuter votre application. Un projet de service cloud contient un ou plusieurs rôles web ou de travail. De la même façon, un projet d’application Service Fabric regroupe un ou plusieurs services. 

La différence est que le projet de service cloud associe le déploiement d’application à un déploiement de machine virtuelle et inclut donc les paramètres de configuration de la machine virtuelle, tandis que le projet d’application Service Fabric définit uniquement une application qui sera déployée sur un ensemble de machines virtuelles existantes dans un cluster Service Fabric. Le cluster Service Fabric est déployé une seule fois, par le biais d’un modèle Resource Manager ou du portail Azure, et plusieurs applications Service Fabric peuvent y être déployées.

![Comparaison de projet entre Service Fabric et les services cloud][3]

## <a name="worker-role-to-stateless-service"></a>Rôle de travail en service sans état
D’un point de vue conceptuel, un rôle de travail représente une charge de travail sans état, ce qui signifie que toutes les instances de la charge de travail sont identiques et que les requêtes peuvent être acheminées vers n’importe quelle instance à tout moment. Chaque instance n’est pas censée se souvenir de la requête précédente. L’état auquel la charge de travail s’exécute est géré par un magasin d’état externe, tel qu’Azure Stockage Table ou Azure Cosmos DB. Dans Service Fabric, ce type de charge de travail est représenté par un service sans état. Le plus simple pour migrer un rôle de travail dans Service Fabric est de convertir un code de rôle de travail en service sans état.

![Rôle de travail en service sans état][4]

## <a name="web-role-to-stateless-service"></a>Rôle web en service sans état
Comme pour le rôle de travail, un rôle web représente également une charge de travail sans état, et, d’un point de vue conceptuel, il peut donc lui aussi être mappé vers un service sans état Service Fabric. Toutefois, contrairement aux rôles web, Service Fabric ne prend pas en charge les IIS. Pour migrer une application web d’un rôle web vers un service sans état, vous devez d’abord passer à une infrastructure web auto-hébergée et qui ne dépend pas d’IIS ou de System.Web, comme ASP.NET Core 1.

| **Application** | **Pris en charge** | **Chemin de migration** |
| --- | --- | --- |
| Formulaires web ASP.NET |Non |Convertir en ASP.NET Core 1 MVC |
| ASP.NET MVC |Avec migration |Mettre à niveau vers ASP.NET Core 1 MVC |
| API Web ASP.NET |Avec migration |Utiliser un serveur auto-hébergé ou ASP.NET Core 1 |
| ASP.NET Core 1 |Oui |N/A |

## <a name="entry-point-api-and-lifecycle"></a>API de point d’entrée et cycle de vie
Les points d’entrée des API de rôle de travail et de Service Fabric sont semblables : 

| **Point d’entrée** | **Instances de** | **Service Service Fabric** |
| --- | --- | --- |
| Traitement en cours |`Run()` |`RunAsync()` |
| Démarrer la machine virtuelle |`OnStart()` |N/A |
| Arrêter la machine virtuelle |`OnStop()` |N/A |
| Ouvrir l’écouteur pour les requêtes du client |N/A |<ul><li> Sans état : `CreateServiceInstanceListener()`</li><li>Avec état : `CreateServiceReplicaListener()`</li></ul> |

### <a name="worker-role"></a>Instances de
```csharp

using Microsoft.WindowsAzure.ServiceRuntime;

namespace WorkerRole1
{
    public class WorkerRole : RoleEntryPoint
    {
        public override void Run()
        {
        }

        public override bool OnStart()
        {
        }

        public override void OnStop()
        {
        }
    }
}

```

### <a name="service-fabric-stateless-service"></a>Services sans état Service Fabric
```csharp

using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.ServiceFabric.Services.Communication.Runtime;
using Microsoft.ServiceFabric.Services.Runtime;

namespace Stateless1
{
    public class Stateless1 : StatelessService
    {
        protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
        {
        }

        protected override Task RunAsync(CancellationToken cancelServiceInstance)
        {
        }
    }
}

```

Les deux disposent d’un remplacement « Exécuter » principal où commencer le traitement. Les services Service Fabric associent `Run`, `Start` et `Stop` en un point d’entrée unique, `RunAsync`. Votre service doit démarrer en même temps que `RunAsync` et doit cesser de fonctionner quand le CancellationToken de la méthode `RunAsync` est signalé. 

Il existe plusieurs différences majeures entre le cycle de vie et la durée de vie des rôles de travail et des services Service Fabric :

* **Cycle de vie :** la plus grande différence est qu’un rôle de travail est une machine virtuelle. Son cycle de vie est donc lié à la machine virtuelle, ce qui inclut les événements de démarrage et d’arrêt de la machine virtuelle. Le cycle de vie des services Service Fabric est distinct du cycle de vie de la machine virtuelle. Il n’inclut donc pas d’événements lorsque l’ordinateur ou la machine virtuelle hôte démarre et s’arrête, dans la mesure où ils ne sont pas liés.
* **Durée de vie :** les instances de rôle de travail sont recyclées quand la méthode `Run` se ferme. La méthode `RunAsync` dans un service Service Fabric peut cependant s’exécuter jusqu’à la fin et l’instance de service continuera de s’exécuter. 

Service Fabric fournit un point d’entrée de configuration de la communication en option pour les services qui écoutent les requêtes des clients. RunAsync et le point d’entrée de communication constituent tous deux des remplacements en option dans les services Service Fabric. Votre service peut choisir d’écouter les requêtes des clients, d’exécuter uniquement une boucle de traitement, ou les deux. C’est pourquoi la méthode RunAsync peut se fermer sans redémarrer l’instance de service : elle peut continuer à écouter les requêtes des clients.

## <a name="application-api-and-environment"></a>API d’application et environnement
L’API d’environnement des services cloud fournit des informations et des fonctionnalités pour l’instance de machine virtuelle active ainsi que des informations sur les autres instances de rôle de machine virtuelle. Service Fabric fournit des informations concernant son exécution et le nœud sur lequel un service s’exécute actuellement. 

| **Tâche d’environnement** | **Cloud Services** | **Service Fabric** |
| --- | --- | --- |
| Paramètres de configuration et notification de modification |`RoleEnvironment` |`CodePackageActivationContext` |
| Stockage local |`RoleEnvironment` |`CodePackageActivationContext` |
| Informations sur le point de terminaison |`RoleInstance` <ul><li>Instance actuelle : `RoleEnvironment.CurrentRoleInstance`</li><li>Autres rôles et instances : `RoleEnvironment.Roles`</li> |<ul><li>`NodeContext` pour l’adresse du nœud actuel</li><li>`FabricClient` et `ServicePartitionResolver` pour la découverte de point de terminaison de service</li> |
| Émulation de l’environnement |`RoleEnvironment.IsEmulated` |N/A |
| Événement de modification simultanée |`RoleEnvironment` |N/A |

## <a name="configuration-settings"></a>Paramètres de configuration
Les paramètres de configuration des services cloud sont définis pour un rôle de machine virtuelle et s’appliquent à toutes les instances de ce rôle de machine virtuelle. Ces paramètres correspondent à des paires clé-valeur définies dans les fichiers ServiceConfiguration.*.cscfg. Ils sont accessibles directement via RoleEnvironment. Dans Service Fabric, les paramètres s’appliquent individuellement à chaque service et chaque application, plutôt qu’à une machine virtuelle. En effet, une machine virtuelle peut héberger plusieurs applications et services. Un service se compose de trois packages :

* **Code :** contient les exécutables du service, ses fichiers binaires, ses DLL et tous les autres fichiers dont un service a besoin pour s’exécuter.
* **Config :** tous les fichiers de configuration et les paramètres d’un service.
* **Data :** fichiers de données statiques associés au service.

Chacun de ces packages peut être individuellement mis à niveau et faire l’objet d’un contrôle de version indépendant. Comme pour les services cloud, les packages de configuration sont accessibles par programme via une API et des événements permettent d’informer le service de toute modification apportée au package de configuration. Un fichier Settings.xml peut être utilisé pour la configuration clé-valeur et l’accès par programmation de manière similaire à la section appSetttings d’un fichier App.config. Toutefois, contrairement aux services cloud, les packages de configuration Service Fabric peuvent contenir des fichiers de configuration à n’importe quel format : XML, JSON, YAML ou tout format binaire personnalisé. 

### <a name="accessing-configuration"></a>Accès à la configuration
#### <a name="cloud-services"></a>Cloud Services
Les paramètres de configuration de ServiceConfiguration.*.cscfg sont accessibles par le biais de `RoleEnvironment`. Toutes les instances de rôle d’un même déploiement de service cloud peuvent accéder à ces paramètres.

```csharp

string value = RoleEnvironment.GetConfigurationSettingValue("Key");

```

#### <a name="service-fabric"></a>Service Fabric
Chaque service dispose de son propre package de configuration individuel. Il n’existe aucun mécanisme intégré permettant à toutes les applications d’un cluster d’accéder à l’ensemble des paramètres de configuration. Lorsque vous utilisez le fichier de configuration spécial Settings.xml de Service Fabric dans un package de configuration, les valeurs du fichier Settings.xml peuvent être remplacées au niveau de l’application, permettant ainsi l’utilisation de paramètres de configuration au niveau de l’application.

Les paramètres de configuration constituent des accès au sein de chaque instance de service par le biais du `CodePackageActivationContext`du service.

```csharp

ConfigurationPackage configPackage = this.Context.CodePackageActivationContext.GetConfigurationPackageObject("Config");

// Access Settings.xml
KeyedCollection<string, ConfigurationProperty> parameters = configPackage.Settings.Sections["MyConfigSection"].Parameters;

string value = parameters["Key"]?.Value;

// Access custom configuration file:
using (StreamReader reader = new StreamReader(Path.Combine(configPackage.Path, "CustomConfig.json")))
{
    MySettings settings = JsonConvert.DeserializeObject<MySettings>(reader.ReadToEnd());
}

```

### <a name="configuration-update-events"></a>Événements de mise à jour de la configuration
#### <a name="cloud-services"></a>Cloud Services
L’événement `RoleEnvironment.Changed` sert à informer toutes les instances de rôle quand une modification est apportée à l’environnement, telle qu’une modification de configuration. Cela permet d’effectuer les mises à jour de la configuration sans recycler les instances de rôle, ni redémarrer un processus de travail.

```csharp

RoleEnvironment.Changed += RoleEnvironmentChanged;

private void RoleEnvironmentChanged(object sender, RoleEnvironmentChangedEventArgs e)
{
   // Get the list of configuration changes
   var settingChanges = e.Changes.OfType<RoleEnvironmentConfigurationSettingChange>();
foreach (var settingChange in settingChanges) 
   {
      Trace.WriteLine("Setting: " + settingChange.ConfigurationSettingName, "Information");
   }
}

```

#### <a name="service-fabric"></a>Service Fabric
Chacun des trois types de package d’un service (Code, Config et Data) inclut des événements qui informent une instance de service lorsqu’un package est mis à jour, ajouté ou supprimé. Un service peut contenir plusieurs packages de chaque type. Par exemple, un service peut avoir plusieurs packages de configuration, chacun pouvant être géré et mis à niveau individuellement. 

Ces événements permettent d’apporter des modifications aux packages de service sans avoir à redémarrer l’instance de service.

```csharp

this.Context.CodePackageActivationContext.ConfigurationPackageModifiedEvent +=
                    this.CodePackageActivationContext_ConfigurationPackageModifiedEvent;

private void CodePackageActivationContext_ConfigurationPackageModifiedEvent(object sender, PackageModifiedEventArgs<ConfigurationPackage> e)
{
    this.UpdateCustomConfig(e.NewPackage.Path);
    this.UpdateSettings(e.NewPackage.Settings);
}

```

## <a name="startup-tasks"></a>Tâches de démarrage
Les tâches de démarrage sont des actions effectuées avant le démarrage d’une application. Elles sont généralement utilisées pour exécuter des scripts de configuration avec des privilèges élevés. Les services cloud et Service Fabric prennent tous deux en charge les tâches de démarrage. La principale différence est que dans les services cloud, les tâches de démarrage sont liées à une machine virtuelle, car elles font partie d’une instance de rôle, tandis que dans Service Fabric, les tâches de démarrage sont liées à un service, qui n’est lui-même pas lié à une machine virtuelle en particulier.

| Service Fabric | Cloud Services |
| --- | --- |
| Emplacement de la configuration |ServiceDefinition.csdef |
| Privilèges |« limités » ou « élevés » |
| Séquencement |« simple », « en arrière-plan », « au premier plan » |

### <a name="cloud-services"></a>Cloud Services
Dans les services cloud, un point d’entrée de démarrage est configuré pour chaque rôle dans le fichier ServiceDefinition.csdef. 

```xml

<ServiceDefinition>
    <Startup>
        <Task commandLine="Startup.cmd" executionContext="limited" taskType="simple" >
            <Environment>
                <Variable name="MyVersionNumber" value="1.0.0.0" />
            </Environment>
        </Task>
    </Startup>
    ...
</ServiceDefinition>

```

### <a name="service-fabric"></a>Service Fabric
Dans Service Fabric, un point d’entrée de démarrage est configuré pour chaque service dans le fichier ServiceManifest.xml :

```xml

<ServiceManifest>
  <CodePackage Name="Code" Version="1.0.0">
    <SetupEntryPoint>
      <ExeHost>
        <Program>Startup.bat</Program>
      </ExeHost>
    </SetupEntryPoint>
    ...
</ServiceManifest>

``` 

## <a name="a-note-about-development-environment"></a>Remarque sur l’environnement de développement
Les services cloud et Service Fabric sont tous deux intégrés à Visual Studio avec les modèles de projet et la prise en charge du débogage, de la configuration et du déploiement, localement et sur Azure. Les services cloud et Service Fabric fournissent également tous deux un environnement local d’exécution de développement. La différence est que, tandis que l’exécution de développement du service cloud émule l’environnement Azure sur lequel il s’exécute, Service Fabric n’utilise pas d’émulateur, mais l’exécution Service Fabric dans son intégralité. L’environnement Service Fabric que vous exécutez sur votre machine de développement locale est le même que celui qui s’exécute en production.

## <a name="next-steps"></a>Étapes suivantes
Découvrez plus en détail les services fiables Service Fabric et les différences fondamentales entre les services cloud et l’architecture d’application Service Fabric, afin de comprendre comment tirer parti de l’ensemble des fonctionnalités de Service Fabric.

* [Découvrir les services fiables Service Fabric](service-fabric-reliable-services-quick-start.md)
* [Guide conceptuel sur les différences entre les services cloud et Service Fabric](service-fabric-cloud-services-migration-differences.md)

<!--Image references-->
[3]: ./media/service-fabric-cloud-services-migration-worker-role-stateless-service/service-fabric-cloud-service-projects.png
[4]: ./media/service-fabric-cloud-services-migration-worker-role-stateless-service/worker-role-to-stateless-service.png
