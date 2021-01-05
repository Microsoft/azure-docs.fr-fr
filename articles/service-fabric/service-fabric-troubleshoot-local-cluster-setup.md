---
title: Dépanner votre configuration de cluster Azure Service Fabric locale
description: Cet article aborde un ensemble de suggestions relatives à la résolution des problèmes de votre cluster de développement local
ms.topic: conceptual
ms.date: 02/23/2018
ms.openlocfilehash: 20948cd1626c02d73fb6e9ef096b552bbab627fb
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575905"
---
# <a name="troubleshoot-your-local-development-cluster-setup"></a>Résoudre les problèmes d'installation de votre cluster de développement local
Si vous rencontrez un problème en interagissant avec votre cluster de développement Azure Service Fabric local, examinez les suggestions suivantes de résolution.

## <a name="cluster-setup-failures"></a>Échecs de configuration du cluster
### <a name="cannot-clean-up-service-fabric-logs"></a>Impossible de nettoyer les journaux d’activité de Service Fabric
#### <a name="problem"></a>Problème
Lors de l’exécution du script DevClusterSetup, le message d’erreur suivant s’affiche :

```output
Cannot clean up C:\SfDevCluster\Log fully as references are likely being held to items in it. Please remove those and run this script again.
At line:1 char:1 + .\DevClusterSetup.ps1
+ ~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo : NotSpecified: (:) [Write-Error], WriteErrorException
+ FullyQualifiedErrorId : Microsoft.PowerShell.Commands.WriteErrorException,DevClusterSetup.ps1
```

#### <a name="solution"></a>Solution
Fermez la fenêtre PowerShell active et ouvrez une nouvelle fenêtre PowerShell en tant qu’administrateur. Vous pouvez maintenant exécuter le script correctement.

## <a name="cluster-connection-failures"></a>Échecs de connexion au cluster

### <a name="type-initialization-exception"></a>Exception durant l’initialisation de type
#### <a name="problem"></a>Problème
Quand vous êtes connecté au cluster dans PowerShell, l’erreur TypeInitializationException apparaît pour System.Fabric.Common.AppTrace.

#### <a name="solution"></a>Solution
Votre variable PATH n’a pas été correctement définie durant l’installation. Déconnectez-vous de Windows, puis reconnectez-vous. Le chemin d’accès est alors actualisé.

### <a name="cluster-connection-fails-with-object-is-closed"></a>La connexion au cluster est mise en échec avec une indication de fermeture de l’objet
#### <a name="problem"></a>Problème
Un appel à Connect-ServiceFabricCluster est mis en échec avec une erreur de ce type :

```output
Connect-ServiceFabricCluster : The object is closed.
At line:1 char:1
+ Connect-ServiceFabricCluster
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo : InvalidOperation: (:) [Connect-ServiceFabricCluster], FabricObjectClosedException
+ FullyQualifiedErrorId : CreateClusterConnectionErrorId,Microsoft.ServiceFabric.Powershell.ConnectCluster
```

#### <a name="solution"></a>Solution
Fermez la fenêtre PowerShell active et ouvrez une nouvelle fenêtre PowerShell en tant qu’administrateur.

### <a name="fabric-connection-denied-exception"></a>Exception Connexion Fabric refusée
#### <a name="problem"></a>Problème
Pendant le débogage à partir de Visual Studio, vous obtenez une erreur FabricConnectionDeniedException.

#### <a name="solution"></a>Solution
Cette erreur se produit généralement lorsque vous tentez de démarrer un processus hôte de service manuellement.

Assurez-vous de ne pas disposer de projets de service définis en tant que projets de démarrage dans votre solution. Seuls les projets d’application Service Fabric doivent être définis en tant que projets de démarrage.

> [!TIP]
> Si, après l’installation, votre cluster local commence à se comporter de manière anormale, vous pouvez le réinitialiser à l’aide de l’application de barre d’état système de gestionnaire de cluster local. Cela supprime le cluster existant et en installe un nouveau. Notez que toutes les applications déployées et les données associées sont supprimées.
> 
> 

## <a name="next-steps"></a>Étapes suivantes
* [Comprendre votre cluster et résoudre les problèmes à l’aide des rapports d’intégrité système](service-fabric-understand-and-troubleshoot-with-system-health-reports.md)
* [Visualiser votre cluster à l’aide de l’outil Service Fabric Explorer](service-fabric-visualizing-your-cluster.md)

