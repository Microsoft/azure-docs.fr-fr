---
title: Déploiement d’Azure Service Fabric à l’aide de PowerShell
description: Découvrez comment supprimer et déployer des applications dans Azure Service Fabric et comment effectuer ces actions dans PowerShell.
ms.topic: conceptual
ms.date: 01/19/2018
ms.openlocfilehash: 8bc4557c5d0d59330c7e91a4b3fdce83cfbf334c
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91827426"
---
# <a name="deploy-and-remove-applications-using-powershell"></a>Déployer et supprimer des applications avec PowerShell

> [!div class="op_single_selector"]
> * [Resource Manager](service-fabric-application-arm-resource.md)
> * [PowerShell](service-fabric-deploy-remove-applications.md)
> * [Interface de ligne de commande de Service Fabric](service-fabric-application-lifecycle-sfctl.md)
> * [API FabricClient](service-fabric-deploy-remove-applications-fabricclient.md)

<br/>

Après avoir [packagé un type d’application][10], il peut être déployé sur un cluster Azure Service Fabric. Le déploiement implique les trois étapes suivantes :

1. Charger le package d’application dans le magasin d’images.
2. Inscrire le type d’application avec le chemin d’accès relatif du magasin d’images.
3. Créer l’instance d’application.

Une fois déployée, l’application n’est plus nécessaire, vous pouvez supprimer l’instance d’application et son type d’application. La suppression complète d’une application du cluster implique les étapes suivantes :

1. Supprimer l’instance d’application en cours d’exécution.
2. Désinscrire le type d’application si vous n’en avez plus besoin.
3. Supprimer le package d’application du magasin d’images.

Si vous utilisez Visual Studio pour déployer et déboguer des applications dans votre cluster de développement local, toutes les étapes précédentes sont gérées automatiquement à l’aide d’un script PowerShell.  Ce script se trouve dans le dossier *Scripts* du projet d’application. Cet article fournit des précisions sur les actions de ce script afin que vous puissiez effectuer les mêmes opérations en dehors de Visual Studio. 

Une autre façon de déployer une application consiste à utiliser un approvisionnement externe. Le package d’application peut être [conditionné en tant que `sfpkg`](service-fabric-package-apps.md#create-an-sfpkg) et chargé dans un magasin externe. Dans ce cas, le chargement dans le magasin d’images n’est pas nécessaire. Le déploiement nécessite les étapes suivantes :

1. Charger le `sfpkg` dans un magasin externe. Le magasin externe peut être n’importe quel magasin qui expose un point de terminaison HTTP ou HTTPS REST.
2. Inscrire le type d’application à l’aide de l’URI de téléchargement externe et les informations sur le type d’application.
2. Créer l’instance d’application.

Pour le nettoyage, supprimez les instances de l’application et annulez l’inscription du type d’application. Le package n’ayant pas été copié dans le magasin d’images, il n’existe aucun emplacement temporaire à nettoyer. L’approvisionnement à partir d’un magasin externe est possible avec Service Fabric version 6.1.

>[!NOTE]
> Visual Studio ne prend actuellement pas en charge l’approvisionnement externe.

 

## <a name="connect-to-the-cluster"></a>Se connecter au cluster

Avant d’exécuter des commandes PowerShell dans le cadre de cet article, commencez toujours par vous connecter au cluster Service Fabric à l’aide de la commande [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps). Pour vous connecter au cluster de développement local, exécutez la commande suivante :

```powershell
Connect-ServiceFabricCluster
```

Pour obtenir des exemples de connexion à un cluster distant ou à un cluster sécurisé à l’aide d’Azure Active Directory, de certificats X509 ou de Windows Active Directory, consultez [Se connecter à un cluster sécurisé](service-fabric-connect-to-secure-cluster.md).

## <a name="upload-the-application-package"></a>Télécharger le package d'application

Quand vous chargez le package d’application, celui-ci est placé dans un dossier accessible aux composants internes de Service Fabric.
Si vous souhaitez vérifier le package d’application en local, utilisez l’applet de commande [Test-ServiceFabricApplicationPackage](/powershell/module/servicefabric/test-servicefabricapplicationpackage?view=azureservicefabricps).

La commande [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps) charge le package d’application dans le magasin d’images du cluster.

Supposons que vous génériez une application nommée *MyApplication* et que vous créiez un package pour cette application dans Visual Studio 2015. Par défaut, le nom du type application répertorié dans le fichier ApplicationManifest.xml est « MyApplicationType ».  Le package d’application, qui contient le manifeste de l’application, les manifestes de service et les packages de code/configuration/données requis, se trouve sous *C:\Users\<username\>\Documents\Visual Studio 2015\Projects\MyApplication\MyApplication\pkg\Debug*. 

La commande suivante répertorie le contenu du package d’application :

```powershell
$path = 'C:\Users\<user\>\Documents\Visual Studio 2015\Projects\MyApplication\MyApplication\pkg\Debug'
tree /f $path
```

```Output
Folder PATH listing for volume OSDisk
Volume serial number is 0459-2393
C:\USERS\USER\DOCUMENTS\VISUAL STUDIO 2015\PROJECTS\MYAPPLICATION\MYAPPLICATION\PKG\DEBUG
│   ApplicationManifest.xml
│
└───Stateless1Pkg
    │   ServiceManifest.xml
    │
    ├───Code
    │       Microsoft.ServiceFabric.Data.dll
    │       Microsoft.ServiceFabric.Data.Interfaces.dll
    │       Microsoft.ServiceFabric.Internal.dll
    │       Microsoft.ServiceFabric.Internal.Strings.dll
    │       Microsoft.ServiceFabric.Services.dll
    │       ServiceFabricServiceModel.dll
    │       Stateless1.exe
    │       Stateless1.exe.config
    │       Stateless1.pdb
    │       System.Fabric.dll
    │       System.Fabric.Strings.dll
    │
    └───Config
            Settings.xml
```

Si le package d’application est volumineux ou contient de nombreux fichiers, vous pouvez le [compresser](service-fabric-package-apps.md#compress-a-package). La compression réduit la taille et le nombre de fichiers.
Cela permet d’accélérer l’inscription et la désinscription du type d’application. Actuellement, le chargement peut demander plus de temps, notamment si vous incluez le temps nécessaire à la compression du package. 

Pour compresser un package, utilisez la même commande [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps). La compression peut être effectuée indépendamment du chargement, en utilisant l’indicateur `SkipCopy`, ou conjointement avec l’opération de chargement. L’application d’une compression sur un package compressé n’a aucun effet.
Pour décompresser un package compressé, utilisez la même commande [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps) avec le commutateur `UncompressPackage`.

L’applet de commande suivante permet de compresser le package sans le copier dans le magasin d’images. Le package inclut désormais les fichiers compressés pour les packages `Code` et `Config`. Les manifestes de l’application et de service ne sont pas compressés, car ils sont requis pour de nombreuses opérations internes (comme le partage de package ou l’extraction du nom du type de l’application et de la version pour certaines validations). La compression des manifestes rendrait ces opérations inefficaces.

```powershell
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath $path -CompressPackage -SkipCopy
tree /f $path
```

```Output
Folder PATH listing for volume OSDisk
Volume serial number is 0459-2393
C:\USERS\USER\DOCUMENTS\VISUAL STUDIO 2015\PROJECTS\MYAPPLICATION\MYAPPLICATION\PKG\DEBUG
|   ApplicationManifest.xml
|
└───Stateless1Pkg
       Code.zip
       Config.zip
       ServiceManifest.xml
```

Pour les packages d’application volumineux, la compression demande un certain temps. Pour obtenir de meilleurs résultats, utilisez un disque SSD rapide. Le temps de compression et la taille du package compressé varient également selon le contenu du package.
Par exemple, voici les statistiques de compression pour certains packages, qui indiquent la taille initiale et la taille du package compressé, avec le temps de compression.

|Taille initiale (Mo)|Nombre de fichiers|Temps de compression|Taille du package compressé (Mo)|
|----------------:|---------:|---------------:|---------------------------:|
|100|100|00:00:03.3547592|60|
|512|100|00:00:16.3850303|307|
|1 024|500|00:00:32.5907950|615|
|2 048|1 000|00:01:04.3775554|1231|
|5012|100|00:02:45.2951288|3074|

Une fois qu’un package est compressé, il peut être chargé dans un ou plusieurs clusters Service Fabric suivant les besoins. Le mécanisme de déploiement est le même pour les packages compressés et non compressés. Les packages compressés sont stockés tels quels dans le magasin d’images du cluster. Les packages sont décompressées sur le nœud, avant l’exécution de l’application.


L’exemple suivant charge le package dans le magasin d’images, dans un dossier nommé « MyApplicationV1 » :

```powershell
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath $path -ApplicationPackagePathInImageStore MyApplicationV1 -TimeoutSec 1800
```

Si vous ne spécifiez pas le paramètre *-ApplicationPackagePathInImageStore*, le package d’application est copié dans le dossier « Debug » du magasin d’images.

>[!NOTE]
>**Copy-ServiceFabricApplicationPackage** détecte automatiquement la chaîne de connexion du magasin d’images appropriée si la session PowerShell est connectée à un cluster Service Fabric. Pour les versions de Service Fabric antérieures à la version 5.6, l’argument **-ImageStoreConnectionString** doit être fourni explicitement.
>
>```powershell
>PS C:\> Copy-ServiceFabricApplicationPackage -ApplicationPackagePath $path -ApplicationPackagePathInImageStore MyApplicationV1 -ImageStoreConnectionString (Get-ImageStoreConnectionStringFromClusterManifest(Get-ServiceFabricClusterManifest)) -TimeoutSec 1800
>```
>
>L’applet de commande **Get-ImageStoreConnectionStringFromClusterManifest** , qui fait partie du module PowerShell du SDK de Service Fabric, sert à obtenir la chaîne de connexion au magasin d’images.  Pour importer le module du kit SDK, exécutez :
>
>```powershell
>Import-Module "$ENV:ProgramFiles\Microsoft SDKs\Service Fabric\Tools\PSModule\ServiceFabricSDK\ServiceFabricSDK.psm1"
>```
>
>Pour plus d’informations sur le magasin d’images et sur ImageStoreConnectionString, consultez la page [Comprendre la chaîne de connexion du magasin d’images](service-fabric-image-store-connection-string.md).
>
>
>

Le temps nécessaire pour charger un package varie selon plusieurs facteurs : nombre de fichiers dans le package, taille du package, taille des fichiers, etc. La vitesse de la connexion réseau entre l’ordinateur source et le cluster Service Fabric a également un impact sur le temps de chargement. Le délai d’expiration par défaut pour la commande [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps) est de 30 minutes.
En fonction des facteurs décrits, il se peut que vous deviez augmenter ce délai. Si vous compressez le package dans l’appel de copie, vous devez également prendre en compte le temps de compression.



## <a name="register-the-application-package"></a>Enregistrer le package d'application

Le type et la version de l’application déclarés dans le manifeste de l’application deviennent utilisables à l’inscription du package d’application. Le système lit le package chargé à l’étape précédente, vérifie le package, traite le contenu du package et copie le package traité dans un emplacement système interne.  

Exécutez l’applet de commande [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype?view=azureservicefabricps) pour inscrire le type d’application dans le cluster et le rendre disponible pour le déploiement :

### <a name="register-the-application-package-copied-to-image-store"></a>Inscrire le package d’application copié dans le magasin d’images

Lorsqu’un package a précédemment été copié dans le magasin d’images, l’opération d’inscription spécifie le chemin d’accès relatif dans le magasin d’images.

```powershell
Register-ServiceFabricApplicationType -ApplicationPathInImageStore MyApplicationV1
```

```Output
Register application type succeeded
```

« MyApplicationV1 » est le dossier du magasin d’images qui contient le package d’application. Le type d’application présentant le nom « MyApplicationType » et la version « 1.0.0 » (ces deux valeurs se trouvent dans le manifeste de l’application) est à présent inscrit dans le cluster.

### <a name="register-the-application-package-copied-to-an-external-store"></a>Inscrire le package d’application copié dans un magasin externe

Depuis Service Fabric version 6.1, l’approvisionnement prend en charge le téléchargement du package à partir d’un magasin externe. L’URI de téléchargement représente le chemin d’accès au [`sfpkg`package d’application](service-fabric-package-apps.md#create-an-sfpkg) à partir duquel le package d’application peut être téléchargé à l’aide des protocoles HTTP ou HTTPS. Le package doit avoir été précédemment chargé dans cet emplacement externe. L’URI doit autoriser un accès en lecture pour que Service Fabric puisse télécharger le fichier. Le fichier `sfpkg` doit avoir l’extension « .sfpkg ». L’opération d’approvisionnement doit inclure les informations sur le type d’application, telles que trouvées dans le manifeste de l’application.

```powershell
Register-ServiceFabricApplicationType -ApplicationPackageDownloadUri "https://sftestresources.blob.core.windows.net:443/sfpkgholder/MyAppPackage.sfpkg" -ApplicationTypeName MyApp -ApplicationTypeVersion V1 -Async
```

La commande [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype?view=azureservicefabricps) ne renvoie un résultat que lorsque le package d’application a été correctement inscrit par le système. La durée de l’enregistrement dépend de la taille et du contenu du package de l’application. Si nécessaire, le paramètre **-TimeoutSec** peut être utilisé pour fournir un délai d’expiration plus long (le délai d’expiration par défaut est de 60 secondes).

Si vous disposez d’un package d’application volumineux et que vous rencontrez des problèmes d’expiration du délai, utilisez le paramètre **-Async**. La commande s’exécute lorsque le cluster accepte la commande d’inscription. L’opération d’inscription se poursuit suivant les besoins.
La commande [Get-ServiceFabricApplicationType](/powershell/module/servicefabric/get-servicefabricapplicationtype?view=azureservicefabricps) répertorie toutes les versions de types d’applications et l’état de leur inscription. Vous pouvez utiliser cette commande pour déterminer quand l’inscription est effectuée.

```powershell
Get-ServiceFabricApplicationType
```

```Output
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : 1.0.0
Status                 : Available
DefaultParameters      : { "Stateless1_InstanceCount" = "-1" }
```

## <a name="remove-an-application-package-from-the-image-store"></a>Supprimer un package d’application du magasin d’images

Si un package a été copié dans le magasin d’images, vous devez le supprimer de l’emplacement temporaire une fois l’application inscrite avec succès. La suppression de packages d’application du magasin d’images libère des ressources système. La conservation des packages d’application inutilisés consomme du stockage sur disque et affecte le niveau de performance des applications.

```powershell
Remove-ServiceFabricApplicationPackage -ApplicationPackagePathInImageStore MyApplicationV1
```

## <a name="create-the-application"></a>Création de l'application

Vous pouvez instancier une application à partir de n’importe quelle version de type d’application correctement inscrite à l’aide de l’applet de commande [New-ServiceFabricApplication](/powershell/module/servicefabric/new-servicefabricapplication?view=azureservicefabricps). Le nom de chaque application doit commencer par le schéma *« fabric: »* et être unique pour chaque instance d’application. Les éventuels services par défaut définis dans le manifeste de l’application du type de l’application cible sont également créés.

```powershell
New-ServiceFabricApplication fabric:/MyApp MyApplicationType 1.0.0
```

```Output
ApplicationName        : fabric:/MyApp
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : 1.0.0
ApplicationParameters  : {}
```

Plusieurs instances d'application peuvent être créées pour une version donnée d'un type d'application enregistré. Chaque instance de l’application s’exécute en isolement, avec ses propres répertoire de travail et processus.

Pour connaître les applications et services nommés en cours d’exécution dans le cluster, exécutez les applets de commande [Get-ServiceFabricApplication](/powershell/module/servicefabric/get-servicefabricapplication) et [Get-ServiceFabricService](/powershell/module/servicefabric/get-servicefabricservice?view=azureservicefabricps) :

```powershell
Get-ServiceFabricApplication  
```

```Output
ApplicationName        : fabric:/MyApp
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : 1.0.0
ApplicationStatus      : Ready
HealthState            : Ok
ApplicationParameters  : {}
```

```powershell
Get-ServiceFabricApplication | Get-ServiceFabricService
```

```Output
ServiceName            : fabric:/MyApp/Stateless1
ServiceKind            : Stateless
ServiceTypeName        : Stateless1Type
IsServiceGroup         : False
ServiceManifestVersion : 1.0.0
ServiceStatus          : Active
HealthState            : Ok
```

## <a name="remove-an-application"></a>Supprimer une application

Lorsque vous n’avez plus besoin d’une instance d’application, vous pouvez la supprimer définitivement à l’aide de son nom en utilisant l’applet de commande [Remove-ServiceFabricApplication](/powershell/module/servicefabric/remove-servicefabricapplication?view=azureservicefabricps). [Remove-ServiceFabricApplication](/powershell/module/servicefabric/remove-servicefabricapplication?view=azureservicefabricps) supprime également automatiquement tous les services qui appartiennent à l’application, supprimant ainsi définitivement tous les états de service. 

> [!WARNING]
> Cette opération ne peut pas être annulée et l’état de l’application ne peut pas être récupéré.

```powershell
Remove-ServiceFabricApplication fabric:/MyApp
```

```Output
Confirm
Continue with this operation?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"):
Remove application instance succeeded
```

```powershell
Get-ServiceFabricApplication
```

## <a name="unregister-an-application-type"></a>Désinscrire un type d’application

Lorsque vous n’avez plus besoin d’un type d’application, il est recommandé de le désinscrire à l’aide de l’applet de commande [Unregister-ServiceFabricApplicationType](/powershell/module/servicefabric/unregister-servicefabricapplicationtype?view=azureservicefabricps). La désinscription des types d’applications inutilisés libère l’espace de stockage utilisé par le magasin d’images par la suppression les fichiers de type d’application. La désinscription d’un type d’application ne supprime pas le package d’application copié dans l’emplacement temporaire du magasin d’image si la copie dans le magasin d’images a été utilisée. Vous pouvez désinscrire un type d’application s’il ne contient aucune instance d’application et s’il n’est référencé par aucune mise à niveau d’application en attente.

Pour afficher les types d’applications actuellement inscrits dans le cluster, exécutez [Get-ServiceFabricApplicationType](/powershell/module/servicefabric/get-servicefabricapplicationtype?view=azureservicefabricps) :

```powershell
Get-ServiceFabricApplicationType
```

```Output
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : 1.0.0
Status                 : Available
DefaultParameters      : { "Stateless1_InstanceCount" = "-1" }
```

Pour désinscrire un type d’application spécifique, exécutez [Unregister-ServiceFabricApplicationType](/powershell/module/servicefabric/unregister-servicefabricapplicationtype?view=azureservicefabricps) :

```powershell
Unregister-ServiceFabricApplicationType MyApplicationType 1.0.0
```

## <a name="troubleshooting"></a>Dépannage

### <a name="copy-servicefabricapplicationpackage-asks-for-an-imagestoreconnectionstring"></a>Copy-ServiceFabricApplicationPackage demande un ImageStoreConnectionString

L'environnement du SDK Service Fabric doit déjà être configuré avec les valeurs par défaut correctes. Toutefois, si besoin, l’ImageStoreConnectionString de toutes les commandes doit correspondre à celui utilisé par le cluster Service Fabric. ImageStoreConnectionString se trouve dans le manifeste de cluster récupéré à l’aide des commandes [Get-ServiceFabricClusterManifest](/powershell/module/servicefabric/get-servicefabricclustermanifest?view=azureservicefabricps) et Get-ImageStoreConnectionStringFromClusterManifest :

```powershell
Get-ImageStoreConnectionStringFromClusterManifest(Get-ServiceFabricClusterManifest)
```

L’applet de commande **Get-ImageStoreConnectionStringFromClusterManifest** , qui fait partie du module PowerShell du SDK de Service Fabric, sert à obtenir la chaîne de connexion au magasin d’images.  Pour importer le module du kit SDK, exécutez :

```powershell
Import-Module "$ENV:ProgramFiles\Microsoft SDKs\Service Fabric\Tools\PSModule\ServiceFabricSDK\ServiceFabricSDK.psm1"
```

ImageStoreConnectionString se trouve dans le manifeste de cluster :

```xml
<ClusterManifest xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Name="Server-Default-SingleNode" Version="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric">

    [...]

    <Section Name="Management">
      <Parameter Name="ImageStoreConnectionString" Value="file:D:\ServiceFabric\Data\ImageStore" />
    </Section>

    [...]
```

Pour plus d’informations sur le magasin d’images et sur ImageStoreConnectionString, consultez la page [Comprendre la chaîne de connexion du magasin d’images](service-fabric-image-store-connection-string.md).

### <a name="deploy-large-application-package"></a>Déployer un package d’application volumineux

Problème : la commande [Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps) expire pour un package d’application volumineux (de l’ordre du gigaoctet).
Essayez de procéder comme suit :
- Spécifiez un délai d’expiration supérieur pour la commande[Copy-ServiceFabricApplicationPackage](/powershell/module/servicefabric/copy-servicefabricapplicationpackage?view=azureservicefabricps) avec le paramètre `TimeoutSec`. Par défaut, le délai d’expiration est de 30 minutes.
- Vérifiez la connexion réseau entre votre ordinateur source et le cluster. Si la connexion est lente, envisagez d’utiliser un ordinateur offrant une meilleure connexion réseau.
Si l’ordinateur client se trouve dans une autre région que le cluster, envisagez d’utiliser un ordinateur client se trouvant dans la région du cluster ou dans une région plus proche de celle-ci.
- Vérifiez si vous êtes confronté à des limitations externes. Par exemple, lorsque le magasin d’images est configuré pour utiliser le stockage Azure, le chargement peut être limité.

Problème : le chargement du package s’est terminé avec succès, mais la commande [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype?view=azureservicefabricps) expire. Essayez de procéder comme suit :
- [Compressez le package](service-fabric-package-apps.md#compress-a-package) avant de le copier dans le magasin d’images.
La compression réduit la taille et le nombre de fichiers, ce qui a pour effet de réduire la quantité de trafic et la charge de travail pour Service Fabric. L’opération de chargement peut demander plus de temps (surtout si vous incluez le temps de compression), mais l’inscription et la désinscription du type d’application sont plus rapides.
- Spécifiez un délai d’expiration supérieur pour la commande[Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype?view=azureservicefabricps) avec le paramètre `TimeoutSec`.
- Spécifiez le commutateur `Async` pour la commande [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype?view=azureservicefabricps). La commande procède au renvoi lorsque le cluster accepte la commande et l’inscription du type d’application se poursuit de façon asynchrone. Par conséquent, il est inutile de spécifier un délai d’expiration plus élevé dans ce cas. La commande [Get-ServiceFabricApplicationType](/powershell/module/servicefabric/get-servicefabricapplicationtype?view=azureservicefabricps) répertorie toutes les versions de types d’applications correctement inscrites et l’état de leur inscription. Vous pouvez utiliser cette commande pour déterminer quand l’inscription est effectuée.

```powershell
Get-ServiceFabricApplicationType
```

```Output
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : 1.0.0
Status                 : Available
DefaultParameters      : { "Stateless1_InstanceCount" = "-1" }
```

### <a name="deploy-application-package-with-many-files"></a>Déployer un package d’application contenant de nombreux fichiers

Problème : la commande [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype?view=azureservicefabricps) expire pour un package d’application contenant un grand nombre de fichiers (de l’ordre de plusieurs milliers).
Essayez de procéder comme suit :
- [Compressez le package](service-fabric-package-apps.md#compress-a-package) avant de le copier dans le magasin d’images. La compression réduit le nombre de fichiers.
- Spécifiez un délai d’expiration supérieur pour la commande[Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype?view=azureservicefabricps) avec le paramètre `TimeoutSec`.
- Spécifiez le commutateur `Async` pour la commande [Register-ServiceFabricApplicationType](/powershell/module/servicefabric/register-servicefabricapplicationtype?view=azureservicefabricps). La commande procède au renvoi lorsque le cluster accepte la commande et l’inscription du type d’application se poursuit de façon asynchrone.
Par conséquent, il est inutile de spécifier un délai d’expiration plus élevé dans ce cas. La commande [Get-ServiceFabricApplicationType](/powershell/module/servicefabric/get-servicefabricapplicationtype?view=azureservicefabricps) répertorie toutes les versions de types d’applications correctement inscrites et l’état de leur inscription. Vous pouvez utiliser cette commande pour déterminer quand l’inscription est effectuée.

```powershell
Get-ServiceFabricApplicationType
```

```Output
ApplicationTypeName    : MyApplicationType
ApplicationTypeVersion : 1.0.0
Status                 : Available
DefaultParameters      : { "Stateless1_InstanceCount" = "-1" }
```

## <a name="next-steps"></a>Étapes suivantes

[Empaqueter une application](service-fabric-package-apps.md)

[Mise à niveau des applications Service Fabric](service-fabric-application-upgrade.md)

[Présentation de l’intégrité de Service Fabric](service-fabric-health-introduction.md)

[Diagnostiquer et résoudre les problèmes d'un service Service Fabric](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

[Modéliser une application dans Service Fabric](service-fabric-application-model.md)

<!--Link references--In actual articles, you only need a single period before the slash-->
[10]: service-fabric-package-apps.md
[11]: service-fabric-application-upgrade.md
