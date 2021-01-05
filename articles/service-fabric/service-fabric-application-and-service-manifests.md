---
title: Description des services et applications Azure Service Fabric
description: Explique l’utilisation de manifestes pour décrire les services et les applications Service Fabric.
ms.topic: conceptual
ms.date: 8/12/2019
ms.openlocfilehash: fcf4c7611f0a6f52c28b234717b9244ac58ad2d4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86248218"
---
# <a name="service-fabric-application-and-service-manifests"></a>Manifestes des services et applications Service Fabric
Cet article explique comment les services et les applications Service Fabric sont définis et créés dans différentes versions à l’aide des fichiers ApplicationManifest.xml et ServiceManifest.xml.  Pour plus d’exemples, consultez les [exemples de manifeste de service et d’application](service-fabric-manifest-examples.md).  Le schéma XML pour ces fichiers manifestes est détaillé dans [Documentation relative au schéma ServiceFabricServiceModel.xsd](service-fabric-service-model-schema.md).

> [!WARNING]
> Le schéma du fichier manifeste XML applique l’ordre correct des éléments enfants.  À tire de solution de contournement partielle, ouvrez « C:\Program Files\Microsoft SDKs\Service Fabric\schemas\ServiceFabricServiceModel.xsd » dans Visual Studio lors de la création ou de la modification des manifestes de Service Fabric. Ceci vous permet de vérifier l’ordre des éléments enfants et fournit la fonctionnalité IntelliSense.

## <a name="describe-a-service-in-servicemanifestxml"></a>Décrire un service dans ServiceManifest.xml
Le manifeste de service définit de manière déclarative le type de service et la version. Il spécifie les métadonnées de service telles que le type de service, les propriétés du contrôle d’intégrité, des mesures d’équilibrage de charge, des fichiers binaires de service et des fichiers de configuration.  Autrement dit, il décrit les packages de code, de configuration et de données qui composent un package de service pour prendre en charge un ou plusieurs types de service. Un manifeste de service peut contenir plusieurs packages de code, de configuration et de données pouvant être créés dans différentes versions indépendamment. Voici un manifeste de service pour le service web frontal ASP.NET Core de [l’exemple d’application de vote](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart) (et voici [quelques exemples plus détaillés](service-fabric-manifest-examples.md)) :

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="VotingWebPkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="https://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType. 
         This name must match the string used in RegisterServiceType call in Program.cs. -->
    <StatelessServiceType ServiceTypeName="VotingWebType" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>VotingWeb.exe</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <!-- Config package is the contents of the Config directory under PackageRoot that contains an 
       independently-updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port on which to 
           listen. Please note that if your service is partitioned, this port is shared with 
           replicas of different partitions that are placed in your code. -->
      <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" Port="8080" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```

**Version** sont des chaînes non structurées et ne sont pas analysés par le système. Les attributs de version permettent de gérer les versions de chaque composant au moment des mises à niveau.

**ServiceTypes** déclare les types de service pris en charge par des packages de code (**CodePackage**) dans ce manifeste. Quand un service est instancié par rapport à un de ces types de service, tous les packages de code déclarés dans ce manifeste sont activés en exécutant leurs points d'entrée. Les processus qui en résultent sont censés inscrire les types de service pris en charge au moment de l'exécution. Les types de service sont déclarés au niveau du manifeste et non au niveau du package de code. Ainsi, s'il existe plusieurs packages de code, ceux-ci sont tous activés chaque fois que le système recherche l'un des types de service déclarés.

Le fichier exécutable spécifié par **EntryPoint** est généralement l’hôte de service à exécution longue. **SetupEntryPoint** est un point d'entrée privilégié qui s'exécute avec les mêmes informations d'identification que Service Fabric (généralement le compte *LocalSystem* ) avant tout autre point d'entrée.  La présence d’un point d’entrée de configuration distinct évite d’avoir à exécuter l’hôte de service avec des privilèges élevés pendant de longues périodes de temps. Le fichier exécutable spécifié par **EntryPoint** est exécuté une fois que **SetupEntryPoint** se termine correctement. En cas d’interruption ou de défaillance du processus, le processus résultant fait l’objet d’une surveillance et est redémarré (à partir de **SetupEntryPoint**).  

**SetupEntryPoint** est généralement utilisé lorsque vous exécutez un fichier exécutable avant le démarrage du service ou si vous effectuez une opération avec des privilèges élevés. Par exemple :

* la configuration et l’initialisation de variables d'environnement dont le fichier exécutable du service a besoin, sans limitation aux seuls exécutables écrits via les modèles de programmation de Service Fabric. Par exemple, npm.exe a besoin de certaines variables d’environnement configurées pour le déploiement d’une application node.js.
* La configuration d’un contrôle d’accès via l’installation de certificats de sécurité.

Pour plus d’informations sur la configuration de SetupEntryPoint, consultez [Configurer la stratégie pour un point d’entrée de configuration de service](service-fabric-application-runas-security.md)

**EnvironmentVariables** (non défini dans l’exemple précédent) fournit une liste de variables d’environnement définies pour ce package de code. Les variables d’environnement peuvent être remplacées dans le fichier `ApplicationManifest.xml` en vue de fournir différentes valeurs pour différentes instances de service. 

**DataPackage** (non défini dans l’exemple précédent) déclare un dossier, nommé par l’attribut **Name**, qui contient des données statiques arbitraires destinées à être consommées par le processus pendant l’exécution.

**ConfigPackage** déclare un dossier, nommé par l’attribut **Name**, qui contient un fichier *Settings.xml*. Ce fichier de paramètres contient des sections de paramètres clé-valeur définis par l’utilisateur, que le processus lit pendant l’exécution. Le processus en cours d’exécution n’est pas redémarré pendant la mise à niveau si seule la **version** de **ConfigPackage** a changé. Au lieu de cela, un rappel indique au processus que les paramètres de configuration ont été modifiés afin qu'ils puissent être rechargés dynamiquement. Voici un exemple de fichier *Settings.xml* :

```xml
<Settings xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Section Name="MyConfigurationSection">
    <Parameter Name="MySettingA" Value="Example1" />
    <Parameter Name="MySettingB" Value="Example2" />
  </Section>
</Settings>
```

Un **point de terminaison** de service Service Fabric est un exemple de ressource Service Fabric. Une ressource Service Fabric peut être déclarée/modifiée sans changer le code compilé. L'accès aux ressources Service Fabric spécifiées dans le manifeste de service peut être contrôlé par le biais de **SecurityGroup** dans le manifeste de l'application. Lorsqu'une ressource Endpoint est définie dans le manifeste de service, Service Fabric alloue des ports de la plage de ports réservés de l'application si aucun port n'est spécifié de manière explicite. En savoir plus sur la [spécification ou la substitution des ressources de point de terminaison](service-fabric-service-manifest-resources.md).

 
> [!WARNING]
> De par leur conception, les ports statiques ne doivent pas chevaucher la plage de ports d’application spécifiée dans ClusterManifest. Si vous spécifiez un port statique, affectez-le hors de la plage de ports d’application ; sinon, des conflits de port se produiront. Avec la version 6.5CU2, nous émettons un **avertissement d’intégrité** quand nous détectons un tel conflit, mais nous laissons le déploiement continuer de façon synchronisée avec le comportement de la version 6.5 expédiée. Toutefois, nous pouvons empêcher le déploiement de l’application à partir des versions majeures suivantes.
>

<!--
For more information about other features supported by service manifests, refer to the following articles:

*TODO: LoadMetrics, PlacementConstraints, ServicePlacementPolicies
*TODO: Health properties
*TODO: Trace filters
*TODO: Configuration overrides
-->

## <a name="describe-an-application-in-applicationmanifestxml"></a>Décrire une application dans ApplicationManifest.xml
Le manifeste d’application décrit le type et la version de manière déclarative. Il spécifie les métadonnées de composition de service telles que les noms stables, le schéma de partitionnement, le nombre d'instances/facteur de réplication, la stratégie de sécurité/d'isolation, les contraintes de placement, les remplacements de configuration et les types de service constitutifs. Les domaines d'équilibrage de charge dans lesquels l'application est placée sont également décrits.

Ainsi, un manifeste d'application décrit les éléments au niveau de l'application et fait référence à un ou plusieurs des manifestes de service pour composer un type d'application. Voici le manifeste d’application pour [l’exemple d’application de vote](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart) (et voici quelques [exemples plus détaillés](service-fabric-manifest-examples.md)) :

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VotingType" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="VotingData_MinReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingData_PartitionCount" DefaultValue="1" />
    <Parameter Name="VotingData_TargetReplicaSetSize" DefaultValue="3" />
    <Parameter Name="VotingWeb_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion 
       should match the Name and Version attributes of the ServiceManifest element defined in the 
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingDataPkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
  </ServiceManifestImport>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="VotingWebPkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types, when an instance of this 
         application type is created. You can also create one or more instances of service type using the 
         ServiceFabric PowerShell module.
         
         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="VotingData">
      <StatefulService ServiceTypeName="VotingDataType" TargetReplicaSetSize="[VotingData_TargetReplicaSetSize]" MinReplicaSetSize="[VotingData_MinReplicaSetSize]">
        <UniformInt64Partition PartitionCount="[VotingData_PartitionCount]" LowKey="0" HighKey="25" />
      </StatefulService>
    </Service>
    <Service Name="VotingWeb" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="VotingWebType" InstanceCount="[VotingWeb_InstanceCount]">
        <SingletonPartition />
         <PlacementConstraints>(NodeType==NodeType0)</PlacementConstraints
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```

Comme dans le cas des manifestes de service, les attributs **Version** sont des chaînes non structurées et ne sont pas analysés par le système. Les attributs de version permettent également de gérer les versions de chaque composant au moment des mises à niveau.

**Parameters** définit les paramètres utilisés dans le manifeste d’application. Les valeurs de ces paramètres peuvent être fournies quand l’application est instanciée ; ils peuvent se substituer aux paramètres de configuration du service ou de l’application.  La valeur du paramètre par défaut est utilisée si la valeur n’est pas modifiée lors de l’instanciation de l’application. Pour savoir comment mettre à jour différents paramètres d’application et de service dans des environnements individuels, consultez [Gestion des paramètres d’application pour plusieurs environnements](service-fabric-manage-multiple-environment-app-configuration.md)

**ServiceManifestImport** contient des références aux manifestes de service qui composent ce type d'application. Un manifeste d’application peut contenir plusieurs importations de manifeste de service, chacune d’elles pouvant être créée dans une version différente indépendamment. Les manifestes de service importés déterminent les types de service qui sont valides dans ce type d'application. Dans ServiceManifestImport, vous remplacez les valeurs de configuration dans le fichier Settings.xml, et les variables d’environnement dans le fichier ServiceManifest.xml. **Policies** (non défini dans l’exemple précédent) pour la liaison de point de terminaison, la sécurité et l’accès ; le partage de package peut être défini dans les manifestes de service importés.  Pour plus d’informations, consultez [Configurer les stratégies de sécurité pour votre application](service-fabric-application-runas-security.md).

**DefaultServices** déclare des instances de service qui sont créées automatiquement chaque fois qu'une application est instanciée par rapport à ce type d'application. Les services par défaut peuvent s'avérer pratiques et se comportent à tous égards comme des services normaux une fois qu'ils ont été créés. Ils sont mis à niveau avec tous les autres services dans l'instance d'application et peuvent également être supprimés. Un manifeste d’application peut contenir plusieurs services par défaut.

**Certificates** (non défini dans l’exemple précédent) déclare les certificats utilisés pour [configurer les points de terminaison HTTPS](service-fabric-service-manifest-resources.md#example-specifying-an-https-endpoint-for-your-service) ou [chiffrer des secrets dans le manifeste d’application](service-fabric-application-secret-management.md).

Les **contraintes de placement** sont les instructions qui définissent où les services doivent s’exécuter. Ces instructions sont attachées aux différents services que vous sélectionnez pour une ou plusieurs propriétés de nœud. Pour plus d’informations, consultez [Syntaxe des contraintes de placement et des propriétés de nœud](./service-fabric-cluster-resource-manager-cluster-description.md#placement-constraints-and-node-property-syntax)

**Policies** (non défini dans l’exemple précédent) décrit les stratégies de collecte de journaux, d’[« exécuter en tant que » par défaut](service-fabric-application-runas-security.md), d’[intégrité](service-fabric-health-introduction.md#health-policies) et de [sécurité d’accès](service-fabric-application-runas-security.md) à définir au niveau de l’application, et indique notamment si les services ont accès au runtime Service Fabric.

> [!NOTE] 
> Par défaut, les applications Service Fabric ont accès au runtime Service Fabric, sous la forme d’un point de terminaison acceptant les demandes propres à l’application et de variables d’environnement pointant vers les chemins d’accès aux fichiers sur l’hôte contenant des fichiers Fabric et spécifiques à l’application. Pensez à désactiver cet accès si l’application héberge un code non fiable (autrement dit, un code dont la provenance est inconnue ou duquel le propriétaire de l’application sait que l’exécution n’est pas sécurisée). Pour plus d’informations, consultez les [bonnes pratiques de sécurité dans Service Fabric](service-fabric-best-practices-security.md#platform-isolation). 
>

**Principals** (non défini dans l’exemple précédent) décrit les principaux de sécurité (utilisateurs ou groupes) nécessaires à l’[exécution des services et à la sécurisation des ressources de service](service-fabric-application-runas-security.md).  Les principaux sont référencés dans les sections **Policies**.





<!--
For more information about other features supported by application manifests, refer to the following articles:

*TODO: Security, Principals, RunAs, SecurityAccessPolicy
*TODO: Service Templates
-->



## <a name="next-steps"></a>Étapes suivantes
- [Empaquetez une application](service-fabric-package-apps.md) et préparez-la pour le déploiement.
- [Déployer et supprimer des applications](service-fabric-deploy-remove-applications.md).
- [Configurer les paramètres et les variables d’environnement pour différentes instances d’application](service-fabric-manage-multiple-environment-app-configuration.md).
- [Configurer les stratégies de sécurité pour votre application](service-fabric-application-runas-security.md).
- [Configurer les points de terminaison HTTPS](service-fabric-service-manifest-resources.md#example-specifying-an-https-endpoint-for-your-service).
- [Chiffrer des secrets dans le manifeste d’application](service-fabric-application-secret-management.md)

<!--Image references-->
[appmodel-diagram]: ./media/service-fabric-application-model/application-model.png
[cluster-imagestore-apptypes]: ./media/service-fabric-application-model/cluster-imagestore-apptypes.png
[cluster-application-instances]: media/service-fabric-application-model/cluster-application-instances.png
