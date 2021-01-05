---
title: Packager une application Azure Service Fabric
description: Découvrez la mise en package d’une application Azure Service Fabric et sa préparation au déploiement sur un cluster.
ms.topic: conceptual
ms.date: 2/23/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 11a3fdd5dbaef53af321342952f786ed8119689c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021059"
---
# <a name="package-an-application"></a>Empaqueter une application

Cet article explique comment empaqueter une application Service Fabric et la préparer pour le déploiement.

## <a name="package-layout"></a>Disposition du package

Le manifeste d’application, un ou plusieurs manifestes de service et les autres fichiers de package nécessaires doivent être organisés selon une disposition spécifique en vue du déploiement dans un cluster Service Fabric. Les exemples de manifestes dans cet article doivent être organisés selon la structure de répertoires suivante :

```
tree /f .\MyApplicationType
```

```Output
D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
    │   ServiceManifest.xml
    │
    ├───MyCode
    │       MyServiceHost.exe
    │
    ├───MyConfig
    │       Settings.xml
    │
    └───MyData
            init.dat
```

Les dossiers sont nommés d'après les attributs **Name** de chaque élément correspondant. Par exemple, si le manifeste de service contient deux packages de code nommés **MyCodeA** et **MyCodeB**, deux dossiers de même nom contiennent les fichiers binaires nécessaires pour chaque package de code.

## <a name="use-setupentrypoint"></a>Utilisation de SetupEntrypoint

**SetupEntryPoint** est généralement utilisé lorsque vous avez besoin d’exécuter un fichier exécutable avant le démarrage du service ou si vous devez effectuer une opération avec des privilèges élevés. Par exemple :

* la configuration et l’initialisation de variables d'environnement dont le fichier exécutable du service a besoin, sans limitation aux seuls exécutables écrits via les modèles de programmation de Service Fabric. Par exemple, npm.exe a besoin de certaines variables d’environnement configurées pour le déploiement d’une application node.js.
* La configuration d’un contrôle d’accès via l’installation de certificats de sécurité.

Pour plus d’informations sur la configuration de **SetupEntryPoint**, consultez [Configurer la stratégie pour un point d’entrée de configuration de service](service-fabric-application-runas-security.md).

<a id="Package-App"></a>

## <a name="configure"></a>Configurer

### <a name="build-a-package-by-using-visual-studio"></a>Création d'un package à l'aide de Visual Studio

Si vous avez utilisé Visual Studio pour créer votre application, vous pouvez utiliser la commande *Package* pour créer automatiquement un package qui correspond à la disposition décrite ci-dessus.

Pour créer un package, cliquez avec le bouton droit sur le projet d’application dans l’*Explorateur de solutions*, puis choisissez la commande **Package** :

![Empaquetage d'une application avec Visual Studio][vs-package-command]

Quand la création du package est terminée, l'emplacement du package est indiqué dans la fenêtre **Sortie**. L’étape de création du package se produit automatiquement quand vous déployez ou déboguez votre application dans Visual Studio.

### <a name="build-a-package-by-command-line"></a>Développer un package par ligne de commande

Vous pouvez également empaqueter votre application par programme en utilisant `msbuild.exe`. Sous le capot, Visual Studio l’exécute pour obtenir le même résultat.

```shell
D:\Temp> msbuild HelloWorld.sfproj /t:Package
```

## <a name="test-the-package"></a>Test du package

Vous pouvez vérifier la structure du package localement via PowerShell à l’aide de la commande [Test-ServiceFabricApplicationPackage](/powershell/module/servicefabric/test-servicefabricapplicationpackage?view=azureservicefabricps) .
Cette commande vérifie la présence de problèmes liés à l’analyse du manifeste, ainsi que toutes les références. Cette commande vérifie uniquement l’exactitude structurelle des répertoires et des fichiers du package.
Elle ne vérifie pas le code ou les données du contenu du package, mais uniquement si tous les fichiers nécessaires sont présents.

```powershell
Test-ServiceFabricApplicationPackage .\MyApplicationType
```

```Output
False
Test-ServiceFabricApplicationPackage : The EntryPoint MySetup.bat is not found.
FileName: C:\Users\servicefabric\AppData\Local\Temp\TestApplicationPackage_7195781181\nrri205a.e2h\MyApplicationType\MyServiceManifest\ServiceManifest.xml
```

Cette erreur indique que le fichier *MySetup.bat* référencé dans le manifeste de service **SetupEntryPoint** est manquant dans le package de code. Une fois le fichier manquant ajouté, la phase de vérification est terminée :

```
tree /f .\MyApplicationType
```

```Output
D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
    │   ServiceManifest.xml
    │
    ├───MyCode
    │       MyServiceHost.exe
    │       MySetup.bat
    │
    ├───MyConfig
    │       Settings.xml
    │
    └───MyData
            init.dat
```

```powershell
Test-ServiceFabricApplicationPackage .\MyApplicationType
```

```Output
True
```

Si des [paramètres d’application](service-fabric-manage-multiple-environment-app-configuration.md) sont définis pour votre application, vous pouvez les transmettre dans [Test-ServiceFabricApplicationPackage](/powershell/module/servicefabric/test-servicefabricapplicationpackage?view=azureservicefabricps) pour assurer une validation correcte.

Si vous savez dans quel cluster l’application sera déployée, il est recommandé de transmettre le paramètre `ImageStoreConnectionString`. Dans ce cas, le package est également validé par rapport aux versions précédentes de l’application qui sont déjà exécutées dans le cluster. Par exemple, la validation permet de détecter si un package présentant la même version mais un contenu différent a déjà été déployé.  

Une fois que l’application a été correctement empaquetée et a passé la validation, pensez à compresser le package pour accélérer les opérations de déploiement.

## <a name="compress-a-package"></a>Compresser un package

Lorsqu’un package est volumineux ou contient de nombreux fichiers, vous pouvez le compresser afin d’accélérer le déploiement. La compression réduit le nombre de fichiers et la taille du package.
Pour un package d’application compressé, le [chargement du package d’application](service-fabric-deploy-remove-applications.md#upload-the-application-package) peut prendre plus de temps que le chargement du package non compressé (surtout si la compression est effectuée lors de la copie). La compression a pour effet secondaire d’accélérer [l’inscription](service-fabric-deploy-remove-applications.md#register-the-application-package) et la [désinscription du type d’application](service-fabric-deploy-remove-applications.md#unregister-an-application-type).

Le mécanisme de déploiement est le même pour les packages compressés et non compressés. Si le package est compressé, il est stocké tel quel dans le magasin d’images du cluster et il est décompressé sur le nœud avant l’exécution de l’application.
La compression remplace le package Service Fabric valide par la version compressée. Le dossier doit autoriser l’accès en écriture. L’exécution d’une compression sur un package déjà compressé ne produit aucun effet.

Vous pouvez compresser un package en exécutant la commande Powershell [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps) avec le commutateur `CompressPackage`. Vous pouvez décompresser le package avec la même commande, en utilisant le commutateur `UncompressPackage`.

La commande suivante permet de compresser le package sans le copier dans le magasin d’images. Vous pouvez copier un package compressé dans un ou plusieurs clusters Service Fabric, selon vos besoins, en utilisant la commande [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps) sans l’indicateur `SkipCopy`.
Le package inclut désormais les fichiers compressés pour les packages `code`, `config` et `data`. Le manifeste de l’application et les manifestes de service ne sont pas compressés, car ils sont exigés pour de nombreuses opérations internes. Par exemple, le partage de package ou l’extraction du nom du type de l’application et de la version pour certaines validations doivent accéder aux manifestes. La compression des manifestes rendrait ces opérations inefficaces.

```
tree /f .\MyApplicationType
```

```Output
D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
    │   ServiceManifest.xml
    │
    ├───MyCode
    │       MyServiceHost.exe
    │       MySetup.bat
    │
    ├───MyConfig
    │       Settings.xml
    │
    └───MyData
            init.dat
```

```powershell
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath .\MyApplicationType -CompressPackage -SkipCopy
tree /f .\MyApplicationType
```

```Output
D:\TEMP\MYAPPLICATIONTYPE
│   ApplicationManifest.xml
│
└───MyServiceManifest
       ServiceManifest.xml
       MyCode.zip
       MyConfig.zip
       MyData.zip

```

Vous pouvez également compresser et copier le package en une seule fois avec la commande [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps).
Si le package est volumineux, prévoyez un délai d’expiration suffisamment long pour que la compression du package et le chargement dans le cluster aient le temps de se terminer.

```powershell
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath .\MyApplicationType -ApplicationPackagePathInImageStore MyApplicationType -ImageStoreConnectionString fabric:ImageStore -CompressPackage -TimeoutSec 5400
```

En interne, Service Fabric calcule les sommes de contrôle des packages d’application à des fins de validation. Lorsque la compression est utilisée, les sommes de contrôle sont calculées dans les versions compressées de chaque package. La génération d’un nouveau fichier zip à partir du même package d’application crée des sommes de contrôle différentes. Pour éviter les erreurs de validation, utilisez le [provisionnement différentiel](service-fabric-application-upgrade-advanced.md). Avec cette option, n’incluez pas les packages inchangés dans la nouvelle version. Au lieu de cela, référencez-les directement à partir du nouveau manifeste de service.

Si le provisionnement différentiel n’est pas une option et que vous devez inclure les packages, générez de nouvelles versions des packages `code`, `config` et `data` pour éviter la non-concordance des sommes de contrôle. La génération de nouvelles versions pour les packages inchangés est nécessaire quand un package compressé est utilisé, que la version précédente utilise la compression ou non.

Le package est maintenant empaqueté correctement, validé et compressé (si nécessaire). Il est donc prêt pour le [déploiement](service-fabric-deploy-remove-applications.md) dans un ou plusieurs clusters Service Fabric.

### <a name="compress-packages-when-deploying-using-visual-studio"></a>Compresser les packages lors du déploiement via Visual Studio

Si vous le souhaitez, Visual Studio peut compresser des packages lors du déploiement, en ajoutant l’élément `CopyPackageParameters` à votre profil de publication, et définir l’attribut `CompressPackage` sur `true`.

``` xml
    <PublishProfile xmlns="http://schemas.microsoft.com/2015/05/fabrictools">
        <ClusterConnectionParameters ConnectionEndpoint="mycluster.westus.cloudapp.azure.com" />
        <ApplicationParameterFile Path="..\ApplicationParameters\Cloud.xml" />
        <CopyPackageParameters CompressPackage="true"/>
    </PublishProfile>
```

## <a name="create-an-sfpkg"></a>Créer un sfpkg

À compter de la version 6.1, Service Fabric autorise le provisionnement à partir d’un magasin externe.
Cette option vous évite de devoir copier le package d’application dans le magasin d’images. Au lieu de cela, créez un `sfpkg` et chargez-le sur un magasin externe, puis fournissez l’URI de téléchargement à Service Fabric au moment du provisionnement. Le même package peut être provisionné sur plusieurs clusters. Le provisionnement à partir du magasin externe vous fait gagner du temps puisqu’il est inutile de copier le package sur chaque cluster.

`sfpkg` est un fichier zip qui contient le package d’application initial avec l’extension « .sfpkg ».
Dans le fichier zip, le package d’application peut être compressé ou non compressé. La compression du package d’application dans le fichier zip est effectuée aux niveaux du code, de la configuration et du package de données, comme [mentionné précédemment](service-fabric-package-apps.md#compress-a-package).

Pour créer un `sfpkg`, commencez par un dossier qui contient le package d’application d’origine, compressé ou non. Ensuite, utilisez n’importe quel utilitaire pour compresser le dossier avec l’extension « .sfpkg ». Par exemple, utilisez [ZipFile.CreateFromDirectory](/dotnet/api/system.io.compression.zipfile.createfromdirectory?view=netcore-3.1#System_IO_Compression_ZipFile_CreateFromDirectory_System_String_System_String_System_IO_Compression_CompressionLevel_System_Boolean_).

```csharp
ZipFile.CreateFromDirectory(appPackageDirectoryPath, sfpkgFilePath);
```

Le `sfpkg` doit être chargé sur le magasin externe hors bande, en dehors de Service Fabric. Le magasin externe peut être n’importe quel magasin qui expose un point de terminaison HTTP ou HTTPS REST. Durant le provisionnement, Service Fabric exécute une opération GET pour télécharger le package d’application `sfpkg`. Le magasin doit donc autoriser l’accès en lecture au package.

Pour provisionner le package, utilisez un provisionnement externe, ce qui nécessite l’URI de téléchargement URI et le type d’application.

>[!NOTE]
> Le provisionnement basé sur le chemin relatif du magasin d’images ne prend pas actuellement en charge les fichiers `sfpkg`. Le `sfpkg` ne doit donc pas être copié dans le magasin d’images.

## <a name="next-steps"></a>Étapes suivantes

[Déployer et supprimer des applications][10] décrit comment utiliser PowerShell pour gérer des instances d’application.

[Gestion des paramètres d’application pour plusieurs environnements][11] décrit comment configurer les paramètres et les variables d’environnement pour différentes instances d’application.

[Configurer les stratégies de sécurité de votre application][12] décrit comment exécuter des services dans le cadre de stratégies de sécurité pour restreindre l’accès.

<!--Image references-->
[vs-package-command]: ./media/service-fabric-package-apps/vs-package-command.png

<!--Link references--In actual articles, you only need a single period before the slash-->
[10]: service-fabric-deploy-remove-applications.md
[11]: service-fabric-manage-multiple-environment-app-configuration.md
[12]: service-fabric-application-runas-security.md
