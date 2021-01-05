---
title: CodePackages initialiseurs dans Service Fabric
description: Décrit les CodePackages initialiseurs dans Service Fabric.
author: shsha-msft
ms.topic: conceptual
ms.date: 03/10/2020
ms.author: shsha
ms.openlocfilehash: 3be079b97c2660437344f88203fdda06cc6d6740
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86258967"
---
# <a name="initializer-codepackages"></a>CodePackages initialiseurs

À partir de la version 7.1, Service Fabric prend en charge les **CodePackages initialiseurs** pour les [conteneurs][containers-introduction-link] et les applications [exécutable invité][guest-executables-introduction-link]. Les CodePackages initialiseurs offrent la possibilité d’effectuer des initialisations au niveau de l’étendue ServicePackage avant que d’autres CodePackages commencent l’exécution. Leur relation avec un ServicePackage est similaire à celle d’un [SetupEntryPoint][setup-entry-point-link] pour un CodePackage.

Avant de poursuivre la lecture de cet article, nous vous recommandons de vous familiariser avec le [modèle d’application Service Fabric][application-model-link] et le [modèle d’hébergement Service Fabric][hosting-model-link].

> [!NOTE]
> Les CodePackages initialiseurs ne sont actuellement pas pris en charge pour les services écrits à l’aide du modèle de programmation [Reliable Services][reliable-services-link].
 
## <a name="semantics"></a>Sémantique

Un CodePackage initialiseur est censé s’exécuter jusqu’à **l’achèvement réussi (code de sortie 0)** . Un CodePackage initialiseur ayant échoué est redémarré jusqu’à ce qu’il se termine correctement. Plusieurs CodePackages initialiseurs sont autorisés et sont exécutés jusqu’à **l’achèvement réussi**, **séquentiellement**, **dans un ordre spécifié** avant que d’autres CodePackages dans le ServicePackage commencent l’exécution.

## <a name="specifying-initializer-codepackages"></a>Spécification des CodePackages initialiseurs
Vous pouvez marquer un CodePackage en tant qu’initialiseur en affectant à l’attribut **Initialiseur** la valeur **true** dans le ServiceManifest. Lorsqu’il existe plusieurs CodePackages initialiseurs, leur ordre d’exécution peut être spécifié via l’attribut **ExecOrder**. **ExecOrder** doit être un entier non négatif et n’est valide que pour les CodePackages initialiseurs. Les CodePackages initialiseurs avec des valeurs **ExecOrder** inférieures sont exécutés en premier. Si **ExecOrder** n’est pas spécifié pour un CodePackage initialiseur, la valeur par défaut 0 est utilisée. L’ordre d’exécution relatif des CodePackages initialiseurs avec la même valeur **ExecOrder** n’est pas spécifié.

L’extrait de code ServiceManifest suivant décrit trois CodePackages, dont deux sont marqués comme initialiseurs. Lorsque ce ServicePackage est activé, *InitCodePackage0* est exécuté en premier, car il a la valeur **ExecOrder** la plus faible. En cas de réussite de l’opération (code de sortie 0) de *InitCodePackage0*, *InitCodePackage1* est exécuté. Enfin, en cas de réussite de l’exécution de *InitCodePackage1*, *WorkloadCodePackage* est exécuté.

```xml
<CodePackage Name="InitCodePackage0" Version="1.0" Initializer="true" ExecOrder="0">
  ...
</CodePackage>

<CodePackage Name="InitCodePackage1" Version="1.0" Initializer="true" ExecOrder="1">
  ...
</CodePackage>

<CodePackage Name="WorkloadCodePackage" Version="1.0">
  ...
</CodePackage>
```
## <a name="complete-example-using-initializer-codepackages"></a>Exemple complet utilisant des CodePackages initialiseurs

Examinons un exemple complet utilisant des CodePackages initialiseurs.

> [!IMPORTANT]
> L’exemple suivant part du principe que vous êtes familiarisé avec la création [d’applications de conteneur Windows à l’aide de Service Fabric et de Docker][containers-getting-started-link].
>
> Cet exemple fait référence à mcr.microsoft.com/windows/nanoserver:1809. Les conteneurs Windows Server ne sont pas compatibles avec toutes les versions d’un système d’exploitation hôte. Pour plus d’informations, consultez [Compatibilité des versions avec les conteneurs Windows](/virtualization/windowscontainers/deploy-containers/version-compatibility).

Le fichier ServiceManifest.xml suivant s’appuie sur l’extrait de code ServiceManifest décrit précédemment. *InitCodePackage0*, *InitCodePackage1* et *WorkloadCodePackage* sont des CodePackages qui représentent des conteneurs. Lors de l’activation, *InitCodePackage0* est exécuté en premier. Il journalise un message dans un fichier et se ferme. Ensuite, *InitCodePackage1* est exécuté. Il journalise également un message dans un fichier et se ferme. Enfin, l’exécution de *WorkloadCodePackage* commence. Il journalise également un message dans un fichier, génère le contenu du fichier sur **stdout** puis effectue un test ping indéfiniment.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ServiceManifest Name="WindowsInitCodePackageServicePackage" Version="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <Description>Windows Init CodePackage Service</Description>
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="WindowsInitCodePackageServiceType"  UseImplicitHost="true"/>
  </ServiceTypes>
  <CodePackage Name="InitCodePackage0" Version="1.0" Initializer="true" ExecOrder="0">
    <EntryPoint>
      <ContainerHost>
        <ImageName>mcr.microsoft.com/windows/nanoserver:1809</ImageName>
        <Commands>/c,echo Hi from InitCodePackage0. &gt; C:\WorkspaceOnContainer\log.txt</Commands>
        <EntryPoint>cmd</EntryPoint>
      </ContainerHost>
    </EntryPoint>
  </CodePackage>

  <CodePackage Name="InitCodePackage1" Version="1.0" Initializer="true" ExecOrder="1">
    <EntryPoint>
      <ContainerHost>
        <ImageName>mcr.microsoft.com/windows/nanoserver:1809</ImageName>
        <Commands>/c,echo Hi from InitCodePackage1. &gt;&gt; C:\WorkspaceOnContainer\log.txt</Commands>
        <EntryPoint>cmd</EntryPoint>
      </ContainerHost>
    </EntryPoint>
  </CodePackage>

  <CodePackage Name="WorkloadCodePackage" Version="1.0">
    <EntryPoint>
      <ContainerHost>
        <ImageName>mcr.microsoft.com/windows/nanoserver:1809</ImageName>
        <Commands>/c,echo Hi from WorkloadCodePackage. &gt;&gt; C:\WorkspaceOnContainer\log.txt &amp;&amp; type C:\WorkspaceOnContainer\log.txt &amp;&amp; ping -t 127.0.0.1 &gt; nul</Commands>
        <EntryPoint>cmd</EntryPoint>
      </ContainerHost>
    </EntryPoint>
  </CodePackage>
</ServiceManifest>
```

Le fichier ApplicationManifest.xml suivant décrit une application basée sur le fichier ServiceManifest.xml mentionné ci-dessus. Notez qu’il spécifie le même montage **Volume** pour tous les conteneurs, c’est-à-dire **C:\WorkspaceOnHost** est monté sur **C:\WorkspaceOnContainer** sur les trois conteneurs. L’effet net est que tous les conteneurs écrivent dans le même fichier journal dans l’ordre dans lequel ils sont activés.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ApplicationManifest ApplicationTypeName="WindowsInitCodePackageApplicationType" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

  <Description>Windows Init CodePackage Application</Description>

  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="WindowsInitCodePackageServicePackage" ServiceManifestVersion="1.0"/>
    <Policies>
      <ContainerHostPolicies CodePackageRef="InitCodePackage0" ContainersRetentionCount="2" RunInteractive="true">
        <Volume Source="C:\WorkspaceOnHost" Destination="C:\WorkspaceOnContainer" IsReadOnly="false" />
      </ContainerHostPolicies>

     <ContainerHostPolicies CodePackageRef="InitCodePackage1" ContainersRetentionCount="2" RunInteractive="true">
        <Volume Source="C:\WorkspaceOnHost" Destination="C:\WorkspaceOnContainer" IsReadOnly="false" />
      </ContainerHostPolicies>

      <ContainerHostPolicies CodePackageRef="WorkloadCodePackage" ContainersRetentionCount="2" RunInteractive="true">
        <Volume Source="C:\WorkspaceOnHost" Destination="C:\WorkspaceOnContainer" IsReadOnly="false" />
      </ContainerHostPolicies>
    </Policies>
  </ServiceManifestImport>

  <DefaultServices>
    <Service Name="WindowsInitCodePackageService" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="WindowsInitCodePackageServiceType" InstanceCount="1">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```
Une fois le ServicePackage correctement activé, le contenu de **C:\WorkspaceOnHost\log.txt** doit être le suivant.

```console
C:\Users\test>type C:\WorkspaceOnHost\log.txt
Hi from InitCodePackage0.
Hi from InitCodePackage1.
Hi from WorkloadCodePackage.
```

## <a name="next-steps"></a>Étapes suivantes

Pour accéder à des informations connexes, consultez les articles suivants.

* [Service Fabric et conteneurs.][containers-introduction-link]
* [Service Fabric et exécutables invités.][guest-executables-introduction-link]

<!-- Links -->
[containers-introduction-link]: service-fabric-containers-overview.md
[containers-getting-started-link]: service-fabric-get-started-containers.md
[guest-executables-introduction-link]: service-fabric-guest-executables-introduction.md
[reliable-services-link]: service-fabric-reliable-services-introduction.md
[application-model-link]: service-fabric-application-model.md
[hosting-model-link]: service-fabric-hosting-model.md
[setup-entry-point-link]: service-fabric-run-script-at-service-startup.md
