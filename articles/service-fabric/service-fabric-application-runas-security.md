---
title: Exécuter un service sous des comptes système et de sécurité locale
description: Découvrez comment exécuter une application Service Fabric sous des comptes système et de sécurité locale.  Créez des principaux de sécurité et appliquez la stratégie Run-As afin d’exécuter en toute sécurité vos services.
ms.topic: conceptual
ms.date: 03/29/2018
ms.openlocfilehash: 53212f8636602705899834b6db1d3f0d80b5fe4f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75610117"
---
# <a name="run-a-service-as-a-local-user-account-or-local-system-account"></a>Exécuter un service en tant que compte d’utilisateur local ou compte système local
Avec Azure Service Fabric, vous pouvez sécuriser les applications en cours d’exécution dans le cluster sous différents comptes d’utilisateur. Par défaut, les applications Service Fabric s’exécutent sous le compte qui exécute le processus Fabric.exe. Service Fabric fournit également la possibilité d’exécuter des applications sous un compte d’utilisateur ou système local. Les types de comptes système locaux pris en charge sont **LocalUser**, **NetworkService**, **LocalService** et **LocalSystem**.  Si vous exécutez Service Fabric sur un cluster autonome Windows, vous pouvez exécuter un service avec des [comptes de domaine Active Directory](service-fabric-run-service-as-ad-user-or-group.md) ou des [comptes de service administré de groupe](service-fabric-run-service-as-gmsa.md).

Dans le manifeste de l’application, les comptes d’utilisateur nécessaires pour exécuter des services ou sécuriser des ressources sont définis dans la section **Principals**. Vous pouvez également définir et créer des groupes d’utilisateurs afin de gérer plusieurs utilisateurs en même temps. C’est utile lorsqu’il existe plusieurs utilisateurs pour des points d’entrée de service différents et qu’ils doivent disposer de certains privilèges courants disponibles au niveau du groupe.  Les utilisateurs sont ensuite référencés dans une stratégie RunAs, qui est appliquée à tous les services de l’application ou à un service en particulier. 

Par défaut, la stratégie RunAs est appliquée au point d’entrée principal.  Vous pouvez également appliquer une stratégie RunAs au point d’entrée d’installation si vous avez besoin [d’exécuter certaines opérations d’installation à privilèges élevés sous un compte système](service-fabric-run-script-at-service-startup.md), ou appliquer la stratégie à la fois au point d’entrée principal et au point d’entrée d’installation.  

> [!NOTE] 
> Si vous appliquez une stratégie Run-As à un service et que le manifeste de service déclare des ressources de point de terminaison avec le protocole HTTP, vous devez spécifier une **SecurityAccessPolicy**.  Pour plus d’informations, consultez [Assigner une stratégie d’accès de sécurité pour des points de terminaison HTTP et HTTPS](service-fabric-assign-policy-to-endpoint.md). 
>

## <a name="run-a-service-as-a-local-user"></a>Exécuter un service en tant qu’utilisateur local
Vous pouvez créer un utilisateur local qui peut être utilisé pour sécuriser un service au sein de l’application. Quand un type de compte **LocalUser** est spécifié dans la section Principals du manifeste d’application, Service Fabric crée des comptes d’utilisateurs locaux sur les ordinateurs où l’application est déployée. Par défaut, ces comptes n’ont pas les mêmes noms que ceux spécifiés dans le manifeste de l’application (par exemple, *Customer3* dans l’exemple du manifeste de l’application suivant). Ils sont générés dynamiquement et sont associés à des mots de passe aléatoires.

Dans la section **RunAsPolicy** d’un **ServiceManifestImport**, spécifiez le compte d’utilisateur de la section **Principals** qui doit être utilisé pour exécuter un package de code de service.  L’exemple suivant montre comment créer un utilisateur local et appliquer une stratégie RunAs au point d’entrée principal :

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="Application7Type" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="Web1_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Web1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="Customer3" EntryPointType="Main" />
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>    
    <Service Name="Web1" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="Web1Type" InstanceCount="[Web1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
    <Users>
      <User Name="Customer3" />
    </Users>
  </Principals>
</ApplicationManifest>
```

## <a name="create-a-local-user-group"></a>Création d'un groupe d'utilisateurs local
Vous pouvez créer des groupes d’utilisateurs et ajouter un ou plusieurs utilisateurs au groupe. Cela est utile s’il existe plusieurs utilisateurs pour des points d’entrée de service différents et s’ils doivent disposer de certains privilèges courants disponibles au niveau du groupe. L’exemple de manifeste d’application suivant montre un groupe local appelé *LocalAdminGroup* disposant de privilèges d’administrateur. Deux utilisateurs, *Customer1* et *Customer2*, deviennent membres de ce groupe local. Dans la section **ServiceManifestImport**, une stratégie RunAs est appliquée au package de code *Stateful1Pkg* en tant que *Customer2*.  Une autre stratégie RunAs est appliquée pour exécuter le package de code *Web1Pkg* en tant que *Customer1*.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="Application7Type" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="Stateful1_MinReplicaSetSize" DefaultValue="3" />
    <Parameter Name="Stateful1_PartitionCount" DefaultValue="1" />
    <Parameter Name="Stateful1_TargetReplicaSetSize" DefaultValue="3" />
    <Parameter Name="Web1_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateful1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="Customer2" EntryPointType="Main"/>
    </Policies>
  </ServiceManifestImport>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Web1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <RunAsPolicy CodePackageRef="Code" UserRef="Customer1" EntryPointType="Main"/>
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <Service Name="Stateful1" ServicePackageActivationMode="ExclusiveProcess">
      <StatefulService ServiceTypeName="Stateful1Type" TargetReplicaSetSize="[Stateful1_TargetReplicaSetSize]" MinReplicaSetSize="[Stateful1_MinReplicaSetSize]">
        <UniformInt64Partition PartitionCount="[Stateful1_PartitionCount]" LowKey="-9223372036854775808" HighKey="9223372036854775807" />
      </StatefulService>
    </Service>
    <Service Name="Web1" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="Web1Type" InstanceCount="[Web1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
    <Groups>
      <Group Name="LocalAdminGroup">
        <Membership>
          <SystemGroup Name="Administrators" />
        </Membership>
      </Group>
    </Groups>
    <Users>
      <User Name="Customer1">
        <MemberOf>
          <Group NameRef="LocalAdminGroup" />
        </MemberOf>
      </User>
      <User Name="Customer2">
        <MemberOf>
          <Group NameRef="LocalAdminGroup" />
        </MemberOf>
      </User>
    </Users>
  </Principals>
</ApplicationManifest>
```

## <a name="apply-a-default-policy-to-all-service-code-packages"></a>Application d’une stratégie par défaut à tous les packages de code de service
La section **DefaultRunAsPolicy** permet de spécifier un compte utilisateur par défaut pour tous les packages de code qui n’ont pas de stratégie **RunAsPolicy** spécifique définie. Si la plupart des packages de code spécifiés dans le manifeste de service utilisé par une application doivent s’exécuter sous le même utilisateur, l’application peut définir une stratégie RunAs par défaut avec ce compte utilisateur. L’exemple suivant spécifie que, si un package de code n’a pas de stratégie **RunAsPolicy** spécifiée, il doit être exécuté sous le compte d’utilisateur **MyDefaultAccount** spécifié dans la section Principals.  Les types de comptes pris en charge sont LocalUser, NetworkService, LocalSystem et LocalService.  Si vous utilisez un utilisateur ou un service local, spécifiez également le nom du compte et son mot de passe.

```xml
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="Application7Type" ApplicationTypeVersion="1.0.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">
  <Parameters>
    <Parameter Name="Web1_InstanceCount" DefaultValue="-1" />
  </Parameters>
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Web1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    
  </ServiceManifestImport>
  <DefaultServices>    
    <Service Name="Web1" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="Web1Type" InstanceCount="[Web1_InstanceCount]">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
  <Principals>
    <Users>
      <User Name="MyDefaultAccount" AccountType="NetworkService" />      
    </Users>
  </Principals>
  <Policies>
    <DefaultRunAsPolicy UserRef="MyDefaultAccount" />
  </Policies>
</ApplicationManifest>
```

## <a name="debug-a-code-package-locally-using-console-redirection"></a>Déboguer un package de code localement à l’aide de la redirection de la console
Parfois, à des fins de débogage, il est utile de visualiser la sortie de la console générée par l’exécution d’un service. Vous pouvez définir une stratégie de redirection de la console sur le point d’entrée dans le manifeste de service, qui écrit ensuite la sortie dans un fichier. La sortie du fichier est écrite dans le dossier d’application **log**, sur le nœud de cluster où l’application est déployée et exécutée. 

> [!WARNING]
> N’utilisez jamais la stratégie de redirection de console dans une application déployée dans un environnement de production, car cela peut affecter le basculement de l’application. N’utilisez cette stratégie *que* pour le développement local et à des fins de débogage.  
> 
> 

L’exemple de manifeste de service suivant montre comment activer la redirection de console avec une valeur FileRetentionCount :

```xml
<CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>VotingWeb.exe</Program>
        <WorkingFolder>CodePackage</WorkingFolder>
        <ConsoleRedirection FileRetentionCount="10"/>
      </ExeHost>
    </EntryPoint>
</CodePackage>

```

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>Étapes suivantes
* [Comprendre le modèle d'application](service-fabric-application-model.md)
* [Spécifier des ressources dans un manifeste de service](service-fabric-service-manifest-resources.md)
* [Déployer une application](service-fabric-deploy-remove-applications.md)

[image1]: ./media/service-fabric-application-runas-security/copy-to-output.png
