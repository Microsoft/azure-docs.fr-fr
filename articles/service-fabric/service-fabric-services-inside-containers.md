---
title: Conteneuriser vos services Azure Service Fabric sur Windows
description: Découvrez comment conteneuriser vos services Service Fabric Reliable Actors et Reliable Services sur Windows.
ms.topic: conceptual
ms.date: 5/23/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: e0c5f3ecf0334ad2190d8542e54388a2a25d30fb
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95999805"
---
# <a name="containerize-your-service-fabric-reliable-services-and-reliable-actors-on-windows"></a>Conteneuriser vos services Service Fabric Reliable Actors et Reliable Services sur Windows

Service Fabric prend en charge la conteneurisation des microservices Service Fabric (services basés sur Reliable Services et Reliable Actors). Pour en savoir plus, voir [Service Fabric et conteneurs](service-fabric-containers-overview.md).

Ce document fournit des conseils pour rendre votre service opérationnel à l’intérieur d’un conteneur Windows.

> [!NOTE]
> Actuellement, cette fonctionnalité fonctionne uniquement pour Windows. Pour exécuter des conteneurs, le cluster doit être en cours d’exécution sur Windows Server 2016 avec des conteneurs.

## <a name="steps-to-containerize-your-service-fabric-application"></a>Étapes de conteneurisation de votre application Service Fabric

1. Ouvrez l’application Service Fabric dans Visual Studio.

2. Ajoutez la classe [SFBinaryLoader.cs](https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/code/SFBinaryLoaderForContainers/SFBinaryLoader.cs) à votre projet. Le code de cette classe correspond à une application auxiliaire qui gère le chargement correct des binaires du runtime Service Fabric dans l’application, lorsqu’elle est exécutée au sein d’un conteneur.

3. Pour chaque package de code que vous souhaitez conteneuriser, initialisez le chargeur au point d’entrée du programme. Ajoutez le constructeur statique indiqué dans l’extrait de code suivant à votre fichier de point d’entrée du programme.

   ```csharp
   namespace MyApplication
   {
      internal static class Program
      {
          static Program()
          {
              SFBinaryLoader.Initialize();
          }

          /// <summary>
          /// This is the entry point of the service host process.
          /// </summary>
          private static void Main()
          {
   ```

4. Générez et [créez un package](service-fabric-package-apps.md#Package-App) de votre projet. Pour générer et créer un package, cliquez avec le bouton droit sur le projet d’application dans l’Explorateur de solutions, puis choisissez la commande **Package**.

5. Pour chaque package de code que vous voulez conteneuriser, exécutez le script PowerShell suivant : [CreateDockerPackage.ps1](https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/scripts/CodePackageToDockerPackage/CreateDockerPackage.ps1). Procédez comme suit :

    .NET complet
      ```powershell
        $codePackagePath = 'Path to the code package to containerize.'
        $dockerPackageOutputDirectoryPath = 'Output path for the generated docker folder.'
        $applicationExeName = 'Name of the Code package executable.'
        CreateDockerPackage.ps1 -CodePackageDirectoryPath $codePackagePath -DockerPackageOutputDirectoryPath $dockerPackageOutputDirectoryPath -ApplicationExeName $applicationExeName
      ```
    .NET Core
      ```powershell
        $codePackagePath = 'Path to the code package to containerize.'
        $dockerPackageOutputDirectoryPath = 'Output path for the generated docker folder.'
        $dotnetCoreDllName = 'Name of the Code package dotnet Core Dll.'
        CreateDockerPackage.ps1 -CodePackageDirectoryPath $codePackagePath -DockerPackageOutputDirectoryPath $dockerPackageOutputDirectoryPath -DotnetCoreDllName $dotnetCoreDllName
      ```
      Ce script crée un dossier avec des artefacts Docker à l’emplacement $dockerPackageOutputDirectoryPath. Modifiez le fichier Dockerfile généré pour `expose` les ports, exécuter les scripts de configuration, etc. selon vos besoins.

6. Vous devez ensuite [générer](service-fabric-get-started-containers.md#Build-Containers) et [envoyer (push)](service-fabric-get-started-containers.md#Push-Containers) votre package de conteneur Docker vers votre référentiel.

7. Modifiez les fichiers ApplicationManifest.xml et ServiceManifest.xml de manière à ajouter le mappage des ports aux hôtes, l’authentification des registres, les informations sur les référentiels et l’image de votre conteneur. Pour savoir comment modifier les manifestes, voir [Créer votre première application de conteneur Service Fabric sur Windows](service-fabric-get-started-containers.md). La définition du package de code dans le manifeste de service doit être remplacée par l’image de conteneur correspondante. N’oubliez pas de remplacer la valeur du paramètre EntryPoint par un type ContainerHost.

   ```xml
   <!-- Code package is your service executable. -->
   <CodePackage Name="Code" Version="1.0.0">
   <EntryPoint>
    <!-- Follow this link for more information about deploying Windows containers to Service Fabric: https://aka.ms/sfguestcontainers -->
    <ContainerHost>
      <ImageName>myregistry.azurecr.io/samples/helloworldapp</ImageName>
    </ContainerHost>
   </EntryPoint>
   <!-- Pass environment variables to your container: -->
   </CodePackage>
   ```

8. Ajoutez le mappage des ports aux hôtes dans le cas du point de terminaison du réplicateur et du service. Comme ces ports sont affectés tous les deux lors du runtime par Service Fabric, la valeur ContainerPort est définie sur zéro, afin que le port affecté soit utilisé pour le mappage.

   ```xml
   <Policies>
   <ContainerHostPolicies CodePackageRef="Code">
    <PortBinding ContainerPort="0" EndpointRef="ServiceEndpoint"/>
    <PortBinding ContainerPort="0" EndpointRef="ReplicatorEndpoint"/>
   </ContainerHostPolicies>
   </Policies>
   ```

9. Pour configurer le mode d’isolation de conteneur, voir [Configurer le mode d’isolation]( ./service-fabric-get-started-containers.md#configure-isolation-mode). Windows prend en charge deux modes d’isolation pour les conteneurs : Processus et Hyper-V. Les extraits de code suivants montrent comment le mode d’isolation est spécifié dans le fichier manifeste de l’application.

   ```xml
   <Policies>
   <ContainerHostPolicies CodePackageRef="Code" Isolation="process">
   ...
   </ContainerHostPolicies>
   </Policies>
   ```
   ```xml
   <Policies>
   <ContainerHostPolicies CodePackageRef="Code" Isolation="hyperv">
   ...
   </ContainerHostPolicies>
   </Policies>
   ```

> [!NOTE] 
> Par défaut, les applications Service Fabric ont accès au runtime Service Fabric, sous la forme d’un point de terminaison acceptant les demandes propres à l’application. Pensez à désactiver cet accès si l’application héberge un code non fiable. Pour plus d’informations, consultez les [bonnes pratiques de sécurité dans Service Fabric](service-fabric-best-practices-security.md#platform-isolation). Pour désactiver l’accès au runtime Service Fabric, ajoutez le paramètre suivant dans la section Stratégies du manifeste de l’application correspondant au manifeste du service importé, comme suit :
>
```xml
  <Policies>
      <ServiceFabricRuntimeAccessPolicy RemoveServiceFabricRuntimeAccess="true"/>
  </Policies>
```
>

10. Pour tester cette application, vous devez la déployer sur un cluster qui exécute la version 5.7 ou plus. Pour les versions de runtime 6.1 ou inférieures, vous devez modifier et mettre à jour les paramètres de cluster pour activer cette fonctionnalité en préversion. Suivez les étapes de cet [article](service-fabric-cluster-fabric-settings.md) pour ajouter le paramètre représenté ensuite.
    ```
      {
        "name": "Hosting",
        "parameters": [
          {
            "name": "FabricContainerAppsEnabled",
            "value": "true"
          }
        ]
      }
    ```

11. Ensuite, [déployez](service-fabric-deploy-remove-applications.md) le package d’application modifié sur ce cluster.

Vous devez maintenant disposer d’une application Service Fabric en conteneur exécutant votre cluster.

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur l’exécution des [conteneurs sur Service Fabric](service-fabric-get-started-containers.md).
* En savoir plus sur le [cycle de vie des applications](service-fabric-application-lifecycle.md) Service Fabric.