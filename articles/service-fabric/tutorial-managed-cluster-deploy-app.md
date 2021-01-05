---
title: Déployer une application sur un cluster Service Fabric managé via PowerShell (préversion)
description: Dans ce tutoriel, vous allez vous connecter à un cluster Service Fabric managé et déployer une application via PowerShell.
ms.topic: tutorial
ms.date: 09/28/2020
ms.openlocfilehash: 36a91d2852bcda5f958441b48ef4721d6ccc83c4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91410230"
---
# <a name="tutorial-deploy-an-app-to-a-service-fabric-managed-cluster-preview"></a>Tutoriel : Déployer une application sur un cluster Service Fabric managé (préversion)

Dans cette série de tutoriels, nous allons aborder les points suivants :

> [!div class="checklist"]
> * [Comment déployer un cluster Service Fabric managé](tutorial-managed-cluster-deploy.md)
> * [Comment effectuer le scale-out d’un cluster Service Fabric managé](tutorial-managed-cluster-scale.md)
> * [Comment ajouter et supprimer des nœuds dans un cluster Service Fabric managé](tutorial-managed-cluster-add-remove-node-type.md)
> * Comment déployer une application sur un cluster Service Fabric managé

Cette partie de la série explique comment :

> [!div class="checklist"]
> * Se connecter à votre cluster Service Fabric managé
> * Charger une application sur un cluster
> * Instancier une application dans un cluster
> * Supprimer une application dans un cluster

## <a name="prerequisites"></a>Prérequis

* Un cluster Service Fabric managé (consultez [*Déployer un cluster managé*](tutorial-managed-cluster-deploy.md)).

## <a name="connect-to-your-cluster"></a>Se connecter au cluster

Pour vous connecter à votre cluster, vous avez besoin de l’empreinte numérique du certificat de cluster. Vous pouvez trouver cette valeur dans la sortie des propriétés du cluster de votre déploiement de ressources ou en interrogeant les propriétés du cluster sur une ressource existante.

La commande suivante peut être utilisée pour interroger votre ressource de cluster afin d’obtenir l’empreinte numérique du certificat de cluster.

```powershell
$serverThumbprint = (Get-AzResource -ResourceId /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.ServiceFabric/managedclusters/mysfcluster).Properties.clusterCertificateThumbprint
```

Avec l’empreinte numérique du certificat de cluster, vous êtes prêt à vous connecter à votre cluster.

```powershell
$connectionEndpoint = "mysfcluster.eastus2.cloudapp.azure.com:19000"
Connect-ServiceFabricCluster -ConnectionEndpoint $connectionEndpoint -KeepAliveIntervalInSec 10 `
      -X509Credential `
      -ServerCertThumbprint $serverThumbprint  `
      -FindType FindByThumbprint `
      -FindValue $clientThumbprint `
      -StoreLocation CurrentUser `
      -StoreName My

```

### <a name="upload-an-application-package"></a>Charger un package d’application

Dans ce tutoriel, nous allons utiliser l’exemple d’[application de vote Service Fabric](https://github.com/Azure-Samples/service-fabric-dotnet-quickstart/tree/voting-sample-no-reverse-proxy). Pour plus d’informations sur le déploiement d’applications Service Fabric via PowerShell, consultez [déployer et supprimer des applications Service Fabric](service-fabric-deploy-remove-applications.md).

> [!NOTE]
> Dans le cluster Service Fabric managé en préversion, vous ne pouvez pas publier d’applications directement depuis Visual Studio.

Vous devez d’abord [empaqueter l’application pour le déploiement](service-fabric-package-apps.md). Pour ce tutoriel, suivez les étapes pour empaqueter une application à partir depuis Visual Studio. Il est important de noter le chemin où l’application a été empaquetée, car il sera utilisé pour le chemin ci-dessous.

Une fois que le package d’application a été créé, vous pouvez le charger sur votre cluster. Mettez à jour la valeur de `$path` pour de façon à ce qu’elle représente le chemin où se trouve votre package d’application, puis exécutez la commande suivante :

```powershell
$path = "C:\Users\<user>\Documents\service-fabric-dotnet-quickstart\Voting\pkg\Debug"
Copy-ServiceFabricApplicationPackage -ApplicationPackagePath $path -CompressPackage
Register-ServiceFabricApplicationType -ApplicationPathInImageStore Debug
```

### <a name="create-an-application"></a>Créer une application

Vous pouvez instancier une application à partir de n’importe quelle version de type d’application correctement inscrite à l’aide de l’applet de commande New-ServiceFabricApplication. Le nom de chaque application doit commencer par le schéma « fabric: » et être unique pour chaque instance d’application. Les éventuels services par défaut définis dans le manifeste de l’application du type de l’application cible sont également créés.

```powershell
New-ServiceFabricApplication fabric:/Voting VotingType 1.0.0
```

Une fois cette opération terminée, vous devez normalement voir les instances de votre application en cours d’exécution dans Service Fabric Explorer.

### <a name="remove-an-application"></a>Supprimer une application

Quand une instance d’application n’est plus nécessaire, vous pouvez la supprimer définitivement en utilisant son nom avec l’applet de commande `Remove-ServiceFabricApplication`, qui supprime également automatiquement tous les services qui appartiennent à l’application, ce qui supprime définitivement tous les états des services.

```powershell
Remove-ServiceFabricApplication fabric:/Voting
```

## <a name="next-steps"></a>Étapes suivantes

Dans cette étape, nous avons déployé une application sur un cluster Service Fabric managé. Pour plus d’informations sur les clusters Service Fabric managé, consultez :

> [!div class="nextstepaction"]
> [Questions fréquentes (FAQ) sur les clusters Service Fabric managés](faq-managed-cluster.md)
