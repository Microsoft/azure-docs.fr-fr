---
title: Déploiement Azure Service Fabric avec FabricClient
description: Utilisez les API FabricClient pour déployer et supprimer des applications dans Service Fabric.
ms.topic: conceptual
ms.date: 01/19/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 565e6b8f23f159a5c231295694830917217a3d19
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89009298"
---
# <a name="deploy-and-remove-applications-using-fabricclient"></a>Déployer et supprimer des applications avec FabricClient
> [!div class="op_single_selector"]
> * [Resource Manager](service-fabric-application-arm-resource.md)
> * [PowerShell](service-fabric-deploy-remove-applications.md)
> * [Interface de ligne de commande de Service Fabric](service-fabric-application-lifecycle-sfctl.md)
> * [API FabricClient](service-fabric-deploy-remove-applications-fabricclient.md)
> 
> 

<br/>

Après avoir [packagé un type d’application][10], il peut être déployé sur un cluster Azure Service Fabric. Le déploiement implique les trois étapes suivantes :

1. Charger le package d’application dans le magasin d’images
2. Enregistrer le type d’application
3. Télécharger le package d'application à partir du magasin d'images
4. Créer l’instance d’application

Après avoir déployé une application et exécuté une instance dans le cluster, vous pouvez supprimer l’instance d’application et son type d’application. Supprimez complètement une application du cluster en suivant les étapes ci-dessous :

1. Supprimer l’instance d’application en cours d’exécution
2. Désinscrire le type d’application si vous n’en avez plus besoin

Si vous utilisez Visual Studio pour déployer et déboguer des applications dans votre cluster de développement local, toutes les étapes précédentes sont gérées automatiquement à l’aide d’un script PowerShell.  Ce script se trouve dans le dossier *Scripts* du projet d’application. Cet article fournit des précisions sur les actions de ce script afin que vous puissiez effectuer les mêmes opérations en dehors de Visual Studio. 
 
## <a name="connect-to-the-cluster"></a>Se connecter au cluster
Connectez-vous au cluster en créant une instance [FabricClient](/dotnet/api/system.fabric.fabricclient) avant d’exécuter un des exemples de code dans cet article. Pour obtenir des exemples de connexion à un cluster de développement local, à un cluster distant ou à un cluster sécurisé à l’aide d’Azure Active Directory, de certificats X509 ou de Windows Active Directory, consultez [Se connecter à un cluster sécurisé](service-fabric-connect-to-secure-cluster.md#connect-to-a-cluster-using-the-fabricclient-apis). Pour vous connecter au cluster de développement local, exécutez l’exemple suivant :

```csharp
// Connect to the local cluster.
FabricClient fabricClient = new FabricClient();
```

## <a name="upload-the-application-package"></a>Télécharger le package d'application
Supposons que vous génériez une application nommée *MyApplication* et que vous créiez un package pour cette application dans Visual Studio. Par défaut, le nom du type application répertorié dans le fichier ApplicationManifest.xml est « MyApplicationType ».  Le package d’application, qui contient le manifeste de l’application, les manifestes de service et les packages de code / configuration / données requis, se trouve sous *C:\Utilisateurs\&lt;nom d’utilisateur&gt;\Documents\Visual Studio 2019\Projects\MyApplication\MyApplication\pkg\Debug*.

Quand vous chargez le package d’application, celui-ci est placé dans un dossier accessible aux composants internes de Service Fabric. Service Fabric vérifie le package d’application lors de l’inscription du package d’application. Toutefois, si vous souhaitez vérifier le package d’application en local (c’est-à-dire avant le chargement), utilisez l’applet de commande [Test-ServiceFabricApplicationPackage](/powershell/module/servicefabric/test-servicefabricapplicationpackage?view=azureservicefabricps).

L’API [CopyApplicationPackage](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.copyapplicationpackage) charge le package d’application dans le magasin d’images du cluster. 

Si le package d’application est volumineux ou contient de nombreux fichiers, vous pouvez le [compresser](service-fabric-package-apps.md#compress-a-package), et le copier dans le magasin d’images à l’aide de PowerShell. La compression réduit la taille et le nombre de fichiers.

Pour plus d’informations sur le magasin d’images et sur ImageStoreConnectionString, consultez la page [Comprendre la chaîne de connexion du magasin d’images](service-fabric-image-store-connection-string.md).

## <a name="register-the-application-package"></a>Enregistrer le package d'application
Le type et la version de l’application déclarés dans le manifeste de l’application deviennent utilisables à l’inscription du package d’application. Le système lit le package chargé à l’étape précédente, vérifie le package, traite le contenu du package et copie le package traité dans un emplacement système interne.  

L’API [ProvisionApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.provisionapplicationasync) inscrit le type d’application dans le cluster et le rend disponible pour le déploiement.

L’API [GetApplicationTypeListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getapplicationtypelistasync) fournit des informations sur tous les types d’application correctement inscrits. Vous pouvez utiliser cette API pour déterminer quand l’inscription est effectuée.

## <a name="remove-an-application-package-from-the-image-store"></a>Supprimer un package d’application du magasin d’images
Nous vous recommandons de supprimer le package d’application une fois que l’application est inscrite.  La suppression de packages d’application du magasin d’images libère des ressources système.  La conservation des packages d’application inutilisés consomme du stockage sur disque et affecte le niveau de performance des applications. Supprimez le package d’application du magasin d’images à l’aide de l’API [RemoveApplicationPackage](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.removeapplicationpackage).

## <a name="create-an-application-instance"></a>Créer une instance d’application
Vous pouvez instancier une application à partir de n’importe quel type d’application correctement inscrit à l’aide de l’API [CreateApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.createapplicationasync). Le nom de chaque application doit commencer par le schéma *« fabric: »* et être unique pour chaque instance d’application (dans un cluster). Les éventuels services par défaut définis dans le manifeste de l’application du type de l’application cible sont également créés.

Plusieurs instances d'application peuvent être créées pour une version donnée d'un type d'application enregistré. Chaque instance de l’application s’exécute en isolement, avec son propre répertoire de travail et son ensemble de processus.

Pour connaître les applications et services nommés en cours d’exécution dans le cluster, exécutez les API [GetApplicationListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getapplicationlistasync) et [GetServiceListAsync](/dotnet/api/system.fabric.fabricclient.queryclient.getservicelistasync).

## <a name="create-a-service-instance"></a>Créer une instance de service
Vous pouvez instancier un service à partir d’un type de service à l’aide de l’API [CreateServiceAsync](/dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync).  Si le service est déclaré comme un service par défaut dans le manifeste d’application, le service est instancié quand l’application l’est.  L’appel à l’API [CreateServiceAsync](/dotnet/api/system.fabric.fabricclient.servicemanagementclient.createserviceasync) pour un service qui est déjà instancié renvoie une exception de type FabricException. L’exception contient un code d’erreur avec la valeur FabricErrorCode.ServiceAlreadyExists.

## <a name="remove-a-service-instance"></a>Supprimer une instance de service
Lorsqu’une instance de service n’est plus nécessaire, vous pouvez la supprimer de l’instance d’application en cours d’exécution en appelant l’API[DeleteServiceAsync](/dotnet/api/system.fabric.fabricclient.servicemanagementclient.deleteserviceasync).  

> [!WARNING]
> Cette opération ne peut pas être annulée et l’état du service ne peut pas être récupéré.

## <a name="remove-an-application-instance"></a>Supprimer une instance d’application
Lorsque vous n’avez plus besoin d’une instance d’application, vous pouvez la supprimer définitivement à l’aide de son nom en utilisant l’API [DeleteApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.deleteapplicationasync). [DeleteApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.deleteapplicationasync) supprime automatiquement tous les services qui appartiennent à l’application, et supprime définitivement tous les états de service.

> [!WARNING]
> Cette opération ne peut pas être annulée et l’état de l’application ne peut pas être récupéré.

## <a name="unregister-an-application-type"></a>Désinscrire un type d’application
Lorsque vous n’avez plus besoin d’une version spécifique d’un type d’application, il est recommandé de la désinscrire à l’aide de l’API [Unregister-ServiceFabricApplicationType](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.unprovisionapplicationasync). La désinscription des versions inutilisées des types d’application libère l’espace de stockage utilisé par le magasin d’images. Une version d’un type d’application peut être désinscrite tant qu’aucune application n’est instanciée par rapport à cette version du type d’application. En outre, le type d’application ne peut avoir aucune mise à niveau d’application en attente faisant référence à cette version du type d’application.

## <a name="troubleshooting"></a>Dépannage
### <a name="copy-servicefabricapplicationpackage-asks-for-an-imagestoreconnectionstring"></a>Copy-ServiceFabricApplicationPackage demande un ImageStoreConnectionString
L'environnement du SDK Service Fabric doit déjà être configuré avec les valeurs par défaut correctes. Toutefois, si besoin, l’ImageStoreConnectionString de toutes les commandes doit correspondre à celui utilisé par le cluster Service Fabric. ImageStoreConnectionString se trouve dans le manifeste de cluster récupéré à l’aide des commandes [Get-ServiceFabricClusterManifest](/powershell/module/servicefabric/get-servicefabricclustermanifest?view=azureservicefabricps) et Get-ImageStoreConnectionStringFromClusterManifest :

```powershell
PS C:\> Get-ImageStoreConnectionStringFromClusterManifest(Get-ServiceFabricClusterManifest)
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
Problème : l’API [CopyApplicationPackage](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.copyapplicationpackage) expire pour un package d’application volumineux (de l’ordre du Go).
Essayez de procéder comme suit :
- Spécifiez un délai d’expiration supérieur pour la méthode [CopyApplicationPackage](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.copyapplicationpackage) avec le paramètre `timeout`. Par défaut, le délai d’expiration est de 30 minutes.
- Vérifiez la connexion réseau entre votre ordinateur source et le cluster. Si la connexion est lente, envisagez d’utiliser un ordinateur offrant une meilleure connexion réseau.
Si l’ordinateur client se trouve dans une autre région que le cluster, envisagez d’utiliser un ordinateur client se trouvant dans la région du cluster ou dans une région plus proche de celle-ci.
- Vérifiez si vous êtes confronté à des limitations externes. Par exemple, lorsque le magasin d’images est configuré pour utiliser le stockage Azure, le chargement peut être limité.

Problème : le chargement du package s’est terminé avec succès, mais l’API [ProvisionApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.provisionapplicationasync) expire. Essayez de procéder comme suit :
- [Compressez le package](service-fabric-package-apps.md#compress-a-package) avant de le copier dans le magasin d’images.
La compression réduit la taille et le nombre de fichiers, ce qui a pour effet de réduire la quantité de trafic et la charge de travail pour Service Fabric. L’opération de chargement peut demander plus de temps (surtout si vous incluez le temps de compression), mais l’inscription et la désinscription du type d’application sont plus rapides.
- Spécifiez un délai d’expiration supérieur pour l’API [ProvisionApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.provisionapplicationasync) avec le paramètre `timeout`.

### <a name="deploy-application-package-with-many-files"></a>Déployer un package d’application contenant de nombreux fichiers
Problème : [ProvisionApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.provisionapplicationasync) expire pour un package d’application contenant un grand nombre de fichiers (de l’ordre de plusieurs milliers).
Essayez de procéder comme suit :
- [Compressez le package](service-fabric-package-apps.md#compress-a-package) avant de le copier dans le magasin d’images. La compression réduit le nombre de fichiers.
- Spécifiez un délai d’expiration supérieur pour [ProvisionApplicationAsync](/dotnet/api/system.fabric.fabricclient.applicationmanagementclient.provisionapplicationasync) avec le paramètre `timeout`.

## <a name="code-example"></a>Exemple de code
L’exemple suivant copie un package d’application dans le magasin d’images et provisionne le type d’application. L’exemple crée ensuite une instance d’application et une instance de service. Enfin, l’exemple supprime l’instance d’application, déprovisionne le type d’application et supprime le package d’application du magasin d’images.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Reflection;
using System.Threading.Tasks;

using System.Fabric;
using System.Fabric.Description;
using System.Threading;

namespace ServiceFabricAppLifecycle
{
class Program
{
static void Main(string[] args)
{

    string clusterConnection = "localhost:19000";
    string appName = "fabric:/MyApplication";
    string appType = "MyApplicationType";
    string appVersion = "1.0.0";
    string serviceName = "fabric:/MyApplication/Stateless1";
    string imageStoreConnectionString = "file:C:\\SfDevCluster\\Data\\ImageStoreShare";
    string packagePathInImageStore = "MyApplication";
    string packagePath = "C:\\Users\\username\\Documents\\Visual Studio 2019\\Projects\\MyApplication\\MyApplication\\pkg\\Debug";
    string serviceType = "Stateless1Type";

    // Connect to the cluster.
    FabricClient fabricClient = new FabricClient(clusterConnection);

    // Copy the application package to a location in the image store
    try
    {
        fabricClient.ApplicationManager.CopyApplicationPackage(imageStoreConnectionString, packagePath, packagePathInImageStore);
        Console.WriteLine("Application package copied to {0}", packagePathInImageStore);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("Application package copy to Image Store failed: ");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Provision the application.  "MyApplicationV1" is the folder in the image store where the application package is located. 
    // The application type with name "MyApplicationType" and version "1.0.0" (both are found in the application manifest) 
    // is now registered in the cluster.            
    try
    {
        fabricClient.ApplicationManager.ProvisionApplicationAsync(packagePathInImageStore).Wait();

        Console.WriteLine("Provisioned application type {0}", packagePathInImageStore);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("Provision Application Type failed:");

        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Delete the application package from a location in the image store.
    try
    {
        fabricClient.ApplicationManager.RemoveApplicationPackage(imageStoreConnectionString, packagePathInImageStore);
        Console.WriteLine("Application package removed from {0}", packagePathInImageStore);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("Application package removal from Image Store failed: ");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    //  Create the application instance.
    try
    {
        ApplicationDescription appDesc = new ApplicationDescription(new Uri(appName), appType, appVersion);
        fabricClient.ApplicationManager.CreateApplicationAsync(appDesc).Wait();
        Console.WriteLine("Created application instance of type {0}, version {1}", appType, appVersion);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("CreateApplication failed.");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Create the stateless service description.  For stateful services, use a StatefulServiceDescription object.
    StatelessServiceDescription serviceDescription = new StatelessServiceDescription();
    serviceDescription.ApplicationName = new Uri(appName);
    serviceDescription.InstanceCount = 1;
    serviceDescription.PartitionSchemeDescription = new SingletonPartitionSchemeDescription();
    serviceDescription.ServiceName = new Uri(serviceName);
    serviceDescription.ServiceTypeName = serviceType;

    // Create the service instance.  If the service is declared as a default service in the ApplicationManifest.xml,
    // the service instance is already running and this call will fail.
    try
    {
        fabricClient.ServiceManager.CreateServiceAsync(serviceDescription).Wait();
        Console.WriteLine("Created service instance {0}", serviceName);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("CreateService failed.");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Delete a service instance.
    try
    {
        DeleteServiceDescription deleteServiceDescription = new DeleteServiceDescription(new Uri(serviceName));

        fabricClient.ServiceManager.DeleteServiceAsync(deleteServiceDescription);
        Console.WriteLine("Deleted service instance {0}", serviceName);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("DeleteService failed.");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Delete an application instance from the application type.
    try
    {
        DeleteApplicationDescription deleteApplicationDescription = new DeleteApplicationDescription(new Uri(appName));

        fabricClient.ApplicationManager.DeleteApplicationAsync(deleteApplicationDescription).Wait();
        Console.WriteLine("Deleted application instance {0}", appName);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("DeleteApplication failed.");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    // Un-provision the application type.
    try
    {
        fabricClient.ApplicationManager.UnprovisionApplicationAsync(appType, appVersion).Wait();
        Console.WriteLine("Un-provisioned application type {0}, version {1}", appType, appVersion);
    }
    catch (AggregateException ae)
    {
        Console.WriteLine("Un-provision application type failed: ");
        foreach (Exception ex in ae.InnerExceptions)
        {
            Console.WriteLine("HResult: {0} Message: {1}", ex.HResult, ex.Message);
        }
    }

    Console.WriteLine("Hit enter...");
    Console.Read();
}        
}
}

```

## <a name="next-steps"></a>Étapes suivantes
[Mise à niveau des applications Service Fabric](service-fabric-application-upgrade.md)

[Présentation de l’intégrité de Service Fabric](service-fabric-health-introduction.md)

[Diagnostiquer et résoudre les problèmes d'un service Service Fabric](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

[Modéliser une application dans Service Fabric](service-fabric-application-model.md)

<!--Link references--In actual articles, you only need a single period before the slash-->
[10]: service-fabric-package-apps.md
[11]: service-fabric-application-upgrade.md
