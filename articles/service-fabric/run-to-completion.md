---
title: Sémantique de RunToCompletion dans Service Fabric
description: Décrit la sémantique de RunToCompletion dans Service Fabric.
author: shsha-msft
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: shsha
ms.openlocfilehash: 6f2f6aa4380fcf6909957118bf682275350ce68c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000264"
---
# <a name="runtocompletion"></a>RunToCompletion

Depuis la version 7.1, Service Fabric prend en charge la sémantique de **RunToCompletion** pour les [conteneurs][containers-introduction-link] et les applications [exécutables invité][guest-executables-introduction-link]. Cette sémantique permet d’utiliser des applications et services qui accomplissent une tâche puis se ferment, par opposition aux applications et services toujours en cours d’exécution.

Avant de poursuivre la lecture de cet article, nous vous recommandons de vous familiariser avec le [modèle d’application Service Fabric][application-model-link] et le [modèle d’hébergement Service Fabric][hosting-model-link].

> [!NOTE]
> La sémantique de RunToCompletion n’est actuellement pas prise en charge pour les services écrits à l’aide du modèle de programmation [Reliable Services][reliable-services-link].
 
## <a name="runtocompletion-semantics-and-specification"></a>Sémantique et spécification de RunToCompletion
La sémantique de RunToCompletion peut être spécifiée comme **ExecutionPolicy** lors de l’[importation du ServiceManifest][application-and-service-manifests-link]. La stratégie spécifiée est héritée par tous les CodePackages composant le ServiceManifest. L’extrait de code du fichier ApplicationManifest.xml suivant fournit un exemple.

```xml
<ServiceManifestImport>
  <ServiceManifestRef ServiceManifestName="RunToCompletionServicePackage" ServiceManifestVersion="1.0"/>
  <Policies>
    <ExecutionPolicy Type="RunToCompletion" Restart="OnFailure"/>
  </Policies>
</ServiceManifestImport>
```
**ExecutionPolicy** autorise les deux attributs suivants :
* **Type :** **RunToCompletion** est actuellement la seule valeur autorisée pour cet attribut.
* **Restart :** Cet attribut spécifie la stratégie de redémarrage appliquée aux CodePackages composant le ServicePackage en cas d’échec. Un CodePackage qui se ferme avec un **code de sortie différent de zéro** est considéré comme ayant échoué. Les valeurs autorisées pour cet attribut sont **OnFailure** et **Never**, **OnFailure** étant la valeur par défaut.

Avec une stratégie de redémarrage définie sur **OnFailure**, si un CodePackage échoue **(code de sortie différent de zéro**), il est redémarré, avec des interruptions entre les échecs répétés. Avec une stratégie de redémarrage définie sur **Never**, en cas de défaillance d’un CodePackage, l’état de déploiement du DeployedServicePackage est marqué comme **Failed** mais l’exécution des autres CodePackages peut continuer. Si tous les CodePackages composant le ServicePackage s’exécutent avec succès **(code de sortie 0)** , l’état de déploiement du DeployedServicePackage est marqué comme **RanToCompletion**. 

## <a name="complete-example-using-runtocompletion-semantics"></a>Exemple complet utilisant la sémantique de RunToCompletion

Examinons un exemple complet utilisant la sémantique de RunToCompletion.

> [!IMPORTANT]
> L’exemple suivant part du principe que vous êtes familiarisé avec la création d’[applications de conteneur Windows à l’aide de Service Fabric et de Docker][containers-getting-started-link].
>
> Cet exemple fait référence à mcr.microsoft.com/windows/nanoserver :1809. Les conteneurs Windows Server ne sont pas compatibles avec toutes les versions d’un système d’exploitation hôte. Pour plus d’informations, consultez [Compatibilité des versions avec les conteneurs Windows](/virtualization/windowscontainers/deploy-containers/version-compatibility).

Le fichier ServiceManifest.xml suivant décrit un ServicePackage constitué de deux CodePackages qui représentent des conteneurs. La commande *RunToCompletionCodePackage1* enregistre simplement un message sur **stdout**, puis se ferme. La commande *RunToCompletionCodePackage2* effectue un test ping de l’adresse de bouclage pendant un certain temps, puis se ferme avec un code de sortie **0**, **1** ou **2**.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ServiceManifest Name="WindowsRunToCompletionServicePackage" Version="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <Description>Windows RunToCompletion Service</Description>
  <ServiceTypes>
    <StatelessServiceType ServiceTypeName="WindowsRunToCompletionServiceType"  UseImplicitHost="true"/>
  </ServiceTypes>
  <CodePackage Name="RunToCompletionCodePackage1" Version="1.0">
    <EntryPoint>
      <ContainerHost>
        <ImageName>mcr.microsoft.com/windows/nanoserver:1809</ImageName>
        <Commands>/c,echo Hi from RunToCompletionCodePackage1 &amp;&amp; exit 0</Commands>
        <EntryPoint>cmd</EntryPoint>
      </ContainerHost>
    </EntryPoint>
  </CodePackage>

  <CodePackage Name="RunToCompletionCodePackage2" Version="1.0">
    <EntryPoint>
      <ContainerHost>
        <ImageName>mcr.microsoft.com/windows/nanoserver:1809</ImageName>
        <Commands>/v,/c,ping 127.0.0.1 &amp;&amp; set /a exitCode=%random% % 3 &amp;&amp; exit !exitCode!</Commands>
        <EntryPoint>cmd</EntryPoint>
      </ContainerHost>
    </EntryPoint>
  </CodePackage>
</ServiceManifest>
```

Le fichier ApplicationManifest.xml suivant décrit une application basée sur le fichier ServiceManifest.xml mentionné ci-dessus. Il spécifie **RunToCompletion** **ExecutionPolicy** pour *WindowsRunToCompletionServicePackage* avec une stratégie de redémarrage **OnFailure**. Lors de l’activation de *WindowsRunToCompletionServicePackage*, ses CodePackages constitutifs sont démarrés. La commande *RunToCompletionCodePackage1* doit se fermer correctement lors de la première activation. En revanche, la commande *RunToCompletionCodePackage2* peut échouer **(code de sortie différent de zéro**), auquel cas elle est redémarrée parce que la stratégie de redémarrage est **OnFailure**.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ApplicationManifest ApplicationTypeName="WindowsRunToCompletionApplicationType" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

  <Description>Windows RunToCompletion Application</Description>

  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="WindowsRunToCompletionServicePackage" ServiceManifestVersion="1.0"/>
    <Policies>
      <ExecutionPolicy Type="RunToCompletion" Restart="OnFailure"/>
    </Policies>
  </ServiceManifestImport>

  <DefaultServices>
    <Service Name="WindowsRunToCompletionService" ServicePackageActivationMode="ExclusiveProcess">
      <StatelessService ServiceTypeName="WindowsRunToCompletionServiceType" InstanceCount="1">
        <SingletonPartition />
      </StatelessService>
    </Service>
  </DefaultServices>
</ApplicationManifest>
```
## <a name="querying-deployment-status-of-a-deployedservicepackage"></a>Interrogation de l’état de déploiement d’un DeployedServicePackage
Vous pouvez interroger l’état de déploiement d’un DeployedServicePackage à partir de PowerShell à partir de PowerShell en utilisant [Get-ServiceFabricDeployedServicePackage][deployed-service-package-link] ou à partir de C# en utilisant l’API [FABRICCLIENT][fabric-client-link] [GetDeployedServicePackageListAsync(String, Uri, String)][deployed-service-package-fabricclient-link]

## <a name="considerations-when-using-runtocompletion-semantics"></a>Considérations relatives à l’utilisation de la sémantique de RunToCompletion

Veuillez noter les points suivants pour la prise en charge actuelle de RunToCompletion.
* Cette sémantique n’est prise en charge que pour des [conteneurs][containers-introduction-link] et des applications [exécutables invité][guest-executables-introduction-link].
* Les scénarios de mise à niveau pour applications avec la sémantique de RunToCompletion ne sont pas autorisés. Les utilisateurs doivent supprimer et recréer ces applications si nécessaire.
* Des événements de basculement peuvent entraîner la réexécution de CodePackages une fois ceux-ci correctement exécutés, sur le même nœud ou d’autres nœuds du cluster. Des événements de basculement sont, par exemple, des redémarrages de nœud et des mises à niveau du runtime Service Fabric sur un nœud.

## <a name="next-steps"></a>Étapes suivantes

Pour accéder à des informations connexes, consultez les articles suivants.

* [Service Fabric et conteneurs.][containers-introduction-link]
* [Service Fabric et exécutables invité.][guest-executables-introduction-link]

<!-- Links -->
[containers-introduction-link]: service-fabric-containers-overview.md
[containers-getting-started-link]: service-fabric-get-started-containers.md
[guest-executables-introduction-link]: service-fabric-guest-executables-introduction.md
[reliable-services-link]: service-fabric-reliable-services-introduction.md
[application-model-link]: service-fabric-application-model.md
[hosting-model-link]: service-fabric-hosting-model.md
[application-and-service-manifests-link]: service-fabric-application-and-service-manifests.md
[setup-entry-point-link]: service-fabric-run-script-at-service-startup.md
[deployed-service-package-working-with-link]: service-fabric-hosting-model.md#work-with-a-deployed-service-package
[deployed-code-package-link]: /powershell/module/servicefabric/get-servicefabricdeployedcodepackage
[deployed-service-package-link]: /powershell/module/servicefabric/get-servicefabricdeployedservicepackage
[fabric-client-link]: /dotnet/api/system.fabric.fabricclient
[deployed-service-package-fabricclient-link]: /dotnet/api/system.fabric.fabricclient.queryclient.getdeployedservicepackagelistasync
