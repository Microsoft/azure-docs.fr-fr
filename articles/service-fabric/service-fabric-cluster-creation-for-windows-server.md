---
title: Créer un cluster Azure Service Fabric autonome
description: Créez un cluster Azure Service Fabric sur n’importe quel ordinateur (physique ou virtuel) exécutant Windows Server, qu’il soit local ou dans un cloud.
ms.topic: conceptual
ms.date: 2/21/2019
ms.openlocfilehash: 36883f2c8b09fa3f8f013e0267dafa2a8220e5d2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91843190"
---
# <a name="create-a-standalone-cluster-running-on-windows-server"></a>Créer un cluster autonome s’exécutant sur Windows Server
Vous pouvez utiliser Azure Service Fabric pour créer des clusters Service Fabric sur toute machine virtuelle ou tout ordinateur exécutant Windows Server. Cela signifie que vous pouvez déployer et exécuter des applications Service Fabric dans n’importe quel environnement contenant un ensemble d’ordinateurs Windows Server interconnectés, que ce soit en local ou avec un fournisseur cloud. Service Fabric fournit un package d’installation pour créer des clusters Service Fabric, appelé package Windows Server autonome. Les clusters Service Fabric traditionnels sur Azure sont disponibles en tant que service managé, tandis que les clusters Service Fabric autonomes sont en libre-service. Pour en savoir plus sur les différences, consultez [Comparaison entre Azure et des clusters Service Fabric autonomes](./service-fabric-deploy-anywhere.md).

Cet article vous guide tout au long du processus de création d’un cluster Service Fabric autonome.

> [!NOTE]
> Ce package de Windows Server autonome est publié gratuitement et peut être utilisé pour les déploiements de production. Ce package peut contenir de nouvelles fonctionnalités Service Fabric en version « préliminaire ». Faites défiler jusqu’à la section « [Fonctionnalités préliminaires incluses dans ce package](#previewfeatures_anchor). » pour obtenir la liste des fonctionnalités préliminaires. Vous pouvez [télécharger une copie du CLUF](https://go.microsoft.com/fwlink/?LinkID=733084) maintenant.
> 
> 

<a id="getsupport"></a>

## <a name="get-support-for-the-service-fabric-for-windows-server-package"></a>Obtenir de l’aide pour le package Service Fabric pour Windows Server
* Interrogez la communauté sur le package autonome Service Fabric pour Windows Server via la [page de questions Microsoft Q&A pour Azure Service Fabric](/answers/topics/azure-service-fabric.html).
* Ouvrez un ticket pour obtenir le [support professionnel Service Fabric](https://support.microsoft.com/oas/default.aspx?prid=16146).  En savoir plus sur le support professionnel Microsoft [ici](https://support.microsoft.com/en-us/gp/offerprophone?wa=wsignin1.0).
* Vous pouvez également bénéficier du support pour ce package dans le cadre du [Support Premier Microsoft](https://support.microsoft.com/en-us/premier).
* Pour plus d’informations, consultez [Options de support d’Azure Service Fabric](./service-fabric-support.md).
* Pour collecter des journaux d’activité à des fins de support, exécutez le [collecteur de journaux d’activité de Service Fabric autonome](service-fabric-cluster-standalone-package-contents.md).

<a id="downloadpackage"></a>

## <a name="download-the-service-fabric-for-windows-server-package"></a>Télécharger le package Service Fabric pour Windows Server
Pour créer le cluster, utilisez le package Service Fabric pour Windows Server (Windows Server 2012 R2 et versions ultérieures) disponible ici : <br>
[Lien de téléchargement - Package autonome Service Fabric - Windows Server](https://go.microsoft.com/fwlink/?LinkId=730690)

Plus d’informations sur le contenu du package [ici](service-fabric-cluster-standalone-package-contents.md).

Le package de runtime Service Fabric est automatiquement téléchargé lors de la création du cluster. En cas de déploiement à partir d’un ordinateur non connecté à Internet, téléchargez le package de runtime hors bande à partir d’ici : <br>
[Lien de téléchargement - Runtime Service Fabric - Windows Server](https://go.microsoft.com/fwlink/?linkid=839354)

Recherchez des exemples de configuration de cluster autonome sous : <br>
[Exemples de configuration de cluster autonome](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples)

<a id="createcluster"></a>

## <a name="create-the-cluster"></a>Créer le cluster
Plusieurs fichiers d’exemples de configuration de cluster sont installés avec le package d’installation. *ClusterConfig.Unsecure.DevCluster.json* correspond à la configuration de cluster la plus simple : un cluster à trois nœuds non sécurisé, s’exécutant sur un seul ordinateur.  Les autres fichiers de configuration décrivent des clusters uniques ou de plusieurs ordinateurs, sécurisés à l’aide de certificats X.509 ou de la sécurité Windows.  Vous n’avez pas besoin de modifier les paramètres de configuration par défaut pour ce didacticiel, mais parcourez le fichier de configuration et familiarisez-vous avec les paramètres.  La section **Nœuds** décrit les trois nœuds du cluster : nom, adresse IP, [type de nœud, domaine d’erreur et domaine de mise à niveau](service-fabric-cluster-manifest.md#nodes-on-the-cluster).  La section **Propriétés** définit la [sécurité, le niveau de fiabilité, la collecte des diagnostics et les types de nœuds](service-fabric-cluster-manifest.md#cluster-properties) pour le cluster.

Le cluster créé dans cet article n’est pas sécurisé.  N’importe qui peut se connecter anonymement et effectuer des opérations de gestion. Les clusters de production doivent donc toujours être sécurisés à l’aide de certificats X.509 ou via la sécurité Windows.  La sécurité peut uniquement être configurée au moment de la création du cluster et il n’est pas possible d’activer la sécurité une fois le cluster créé. La mise à jour du fichier de configuration active la [sécurité par certificat](service-fabric-windows-cluster-x509-security.md) ou la [sécurité Windows](service-fabric-windows-cluster-windows-security.md). Lisez [Sécuriser un cluster](service-fabric-cluster-security.md) pour en savoir plus sur la sécurité du cluster Service Fabric.

### <a name="step-1-create-the-cluster"></a>Étape 1 : Créer le cluster

#### <a name="scenario-a-create-an-unsecured-local-development-cluster"></a>Scénario A : Créer un cluster de développement local non sécurisé
Service Fabric peut être déployé vers un cluster de développement avec un ordinateur à l’aide du fichier *ClusterConfig.Unsecure.DevCluster.json* inclus dans les [exemples](https://github.com/Azure-Samples/service-fabric-dotnet-standalone-cluster-configuration/tree/master/Samples).

Décompressez le package autonome sur votre ordinateur, copiez l’exemple de fichier de configuration sur l’ordinateur local, puis exécutez le script *CreateServiceFabricCluster.ps1* par le biais d’une session administrateur PowerShell, à partir du dossier du package autonome.

```powershell
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.DevCluster.json -AcceptEULA
```

Consultez la section Configuration de l’environnement sous [Planifier et préparer le déploiement de cluster](service-fabric-cluster-standalone-deployment-preparation.md) pour obtenir plus d’informations de dépannage.

Si vous avez fini d’exécuter les scénarios de développement, vous pouvez supprimer le cluster Service Fabric de l’ordinateur en vous reportant aux étapes de la section « [Supprimer un cluster](#removecluster_anchor) ». 

#### <a name="scenario-b-create-a-multi-machine-cluster"></a>Scénario B : Créer un cluster de plusieurs ordinateurs
Après avoir effectué les tâches de planification et de préparation détaillées dans [Planifier et préparer le déploiement de cluster](service-fabric-cluster-standalone-deployment-preparation.md), vous êtes prêt à créer votre cluster de production à l’aide de votre fichier de configuration de cluster.

L’administrateur de cluster déployant et configurant le cluster doit disposer de privilèges d’administrateur sur l’ordinateur. Vous ne pouvez pas installer Service Fabric sur un contrôleur de domaine.

1. Le script *TestConfiguration.ps1* contenu dans le package autonome est utilisé pour analyser les meilleures pratiques, afin de vérifier si un cluster peut être déployé dans un environnement donné. La section [Préparation du déploiement](service-fabric-cluster-standalone-deployment-preparation.md) répertorie les conditions préalables et les exigences de l’environnement. Exécutez le script pour vérifier si vous pouvez créer le cluster de développement :  

    ```powershell
    .\TestConfiguration.ps1 -ClusterConfigFilePath .\ClusterConfig.json
    ```

    Le résultat ressemble à ce qui suit. Si le champ du bas « Passed » (Transmis) est retourné avec la valeur « True », les contrôles de validité ont réussi et le cluster semble être déployable à l’aide de la configuration d’entrée.

    ```powershell
    Trace folder already exists. Traces will be written to existing trace folder: C:\temp\Microsoft.Azure.ServiceFabric.WindowsServer\DeploymentTraces
    Running Best Practices Analyzer...
    Best Practices Analyzer completed successfully.
    
    LocalAdminPrivilege        : True
    IsJsonValid                : True
    IsCabValid                 : True
    RequiredPortsOpen          : True
    RemoteRegistryAvailable    : True
    FirewallAvailable          : True
    RpcCheckPassed             : True
    NoConflictingInstallations : True
    FabricInstallable          : True
    Passed                     : True
    ```

2. Créer le cluster :  exécutez le script *CreateServiceFabricCluster.ps1* pour déployer le cluster Service Fabric sur chaque ordinateur dans la configuration. 
    ```powershell
    .\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.json -AcceptEULA
    ```

> [!NOTE]
> Les traces de déploiement sont écrites sur la machine virtuelle/l’ordinateur où vous avez exécuté le script PowerShell CreateServiceFabricCluster.ps1. Elles ne se trouvent pas dans le sous-dossier DeploymentTraces, situé dans le répertoire à partir duquel le script a été exécuté. Pour voir si Service Fabric a été déployé correctement sur un ordinateur, recherchez les fichiers installés dans le répertoire FabricDataRoot, comme indiqué dans la section FabricSettings du fichier de configuration du cluster (par défaut c:\ProgramData\SF). De même, vous pouvez consulter l’exécution des processus FabricHost.exe et Fabric.exe dans le Gestionnaire des tâches.
> 
> 

#### <a name="scenario-c-create-an-offline-internet-disconnected-cluster"></a>Scénario C : Créer un cluster hors connexion (déconnecté d’Internet)
Le package de runtime Service Fabric est automatiquement téléchargé lors de la création du cluster. Lorsque vous déployez un cluster sur des ordinateurs qui ne sont pas connectés à Internet, vous devrez télécharger le package de runtime Service Fabric séparément et fournir le chemin d’accès à celui-ci lors de la création du cluster.
Le package de runtime peut être téléchargé séparément, à partir d’un autre ordinateur connecté à Internet, en cliquant sur le [lien de téléchargement - Runtime Service Fabric - Windows Server](https://go.microsoft.com/fwlink/?linkid=839354). Copiez le package de runtime à l’endroit où vous déployez le cluster hors connexion et créez le cluster en exécutant `CreateServiceFabricCluster.ps1` avec le paramètre `-FabricRuntimePackagePath` inclus, comme indiqué dans cet exemple : 

```powershell
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.json -FabricRuntimePackagePath .\MicrosoftAzureServiceFabric.cab
```

*.\ClusterConfig.json* et *.\MicrosoftAzureServiceFabric.cab* sont, respectivement, les chemins de la configuration du cluster et du fichier .cab de runtime.

### <a name="step-2-connect-to-the-cluster"></a>Étape 2 : Se connecter au cluster
Connectez-vous au cluster pour vérifier que le cluster est en cours d’exécution et disponible. Le module Service Fabric PowerShell est installé avec le runtime.  Vous pouvez vous connecter au cluster à partir de l’un des nœuds de cluster ou d’un ordinateur distant avec le runtime Service Fabric.  L’applet de commande [Connect-ServiceFabricCluster](/powershell/module/servicefabric/connect-servicefabriccluster?view=azureservicefabricps) établit une connexion au cluster.

Exécutez la commande PowerShell suivante pour vous connecter à un cluster non sécurisé :

```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint <*IPAddressofaMachine*>:<Client connection end point port>
```

Par exemple :
```powershell
Connect-ServiceFabricCluster -ConnectionEndpoint 192.13.123.234:19000
```

Pour obtenir des exemples de connexion à un cluster, consultez [Se connecter à un cluster sécurisé](service-fabric-connect-to-secure-cluster.md) . Une fois connecté au cluster, utilisez l’applet de commande [Get-ServiceFabricNode](/powershell/module/servicefabric/get-servicefabricnode?view=azureservicefabricps) pour afficher une liste des nœuds du cluster et les informations d’état pour chaque nœud. **HealthState** doit être à l’état *OK* pour chaque nœud.

```powershell
PS C:\temp\Microsoft.Azure.ServiceFabric.WindowsServer> Get-ServiceFabricNode |Format-Table

NodeDeactivationInfo NodeName IpAddressOrFQDN NodeType  CodeVersion  ConfigVersion NodeStatus NodeUpTime NodeDownTime HealthState
-------------------- -------- --------------- --------  -----------  ------------- ---------- ---------- ------------ -----------
                     vm2      localhost       NodeType2 5.6.220.9494 0                     Up 00:03:38   00:00:00              OK
                     vm1      localhost       NodeType1 5.6.220.9494 0                     Up 00:03:38   00:00:00              OK
                     vm0      localhost       NodeType0 5.6.220.9494 0                     Up 00:02:43   00:00:00              OK
```

### <a name="step-3-visualize-the-cluster-using-service-fabric-explorer"></a>Étape 3 : Visualiser votre cluster à l’aide de l’outil Service Fabric Explorer
[Service Fabric Explorer](service-fabric-visualizing-your-cluster.md) est un bon outil pour visualiser votre cluster et gérer les applications.  Service Fabric Explorer est un service qui s’exécute dans le cluster, auquel vous accédez à l’aide d’un navigateur en utilisant l’adresse `http://localhost:19080/Explorer`.

Le tableau de bord de cluster fournit une vue d’ensemble de votre cluster, y compris un résumé de l’intégrité de l’application et du nœud. L’affichage des nœuds montre la disposition physique du cluster. Vous pouvez identifier les applications ayant déployé du code sur un nœud donné.

![Service Fabric Explorer][service-fabric-explorer]

## <a name="add-and-remove-nodes"></a>Ajouter et supprimer des nœuds
Vous pouvez ajouter des nœuds à votre cluster Service Fabric autonome ou en supprimer en fonction de l’évolution de vos besoins. Consultez l’article [Ajouter ou supprimer des nœuds à/d’un cluster Service Fabric autonome](service-fabric-cluster-windows-server-add-remove-nodes.md) pour obtenir des instructions détaillées.

<a id="removecluster" name="removecluster_anchor"></a>
## <a name="remove-a-cluster"></a>Supprimer un cluster
Pour supprimer un cluster, exécutez le script PowerShell *RemoveServiceFabricCluster.ps1* à partir du dossier de package et transmettez le chemin du fichier de configuration JSON. Vous pouvez éventuellement spécifier un emplacement pour le journal de la suppression.

Ce script peut être exécuté sur n’importe quel ordinateur disposant d’un accès administrateur à tous les ordinateurs qui sont répertoriés en tant que nœuds dans le fichier de configuration du cluster. L’ordinateur sur lequel ce script est exécuté ne doit pas nécessairement faire partie du cluster.

```powershell
# Removes Service Fabric from each machine in the configuration
.\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.json -Force
```

```powershell
# Removes Service Fabric from the current machine
.\CleanFabric.ps1
```

<a id="telemetry"></a>

## <a name="telemetry-data-collected-and-how-to-opt-out-of-it"></a>Données de télémétrie recueillies et comment se désabonner
Par défaut, le produit collecte les données de télémétrie sur l’utilisation de Service Fabric en vue de l’améliorer. L’outil Best Practice Analyzer qui s’exécute dans le cadre de l’installation contrôle la connectivité à [https://vortex.data.microsoft.com/collect/v1](https://vortex.data.microsoft.com/collect/v1). Si le service n’est pas joignable, la configuration échoue, sauf si vous vous désabonnez de la télémétrie.

1. Le pipeline de télémétrie tente de charger les données suivantes vers [https://vortex.data.microsoft.com/collect/v1](https://vortex.data.microsoft.com/collect/v1) une fois par jour. Ce meilleur téléchargement n’a aucun impact sur la fonctionnalité de cluster. Les données de télémétrie sont envoyées uniquement à partir du nœud qui exécute le service Failover Manager principal. Aucun autre nœud n’envoie des données de télémétrie.
2. La télémétrie se compose des éléments suivants :

* Nombre de services
* Nombre de ServiceTypes
* Nombre de Applications
* Nombre de ApplicationUpgrades
* Nombre de FailoverUnits
* Nombre de InBuildFailoverUnits
* Nombre de UnhealthyFailoverUnits
* Nombre de Replicas
* Nombre de InBuildReplicas
* Nombre de StandByReplicas
* Nombre de OfflineReplicas
* CommonQueueLength
* QueryQueueLength
* FailoverUnitQueueLength
* CommitQueueLength
* Nombre de nœuds
* IsContextComplete : True/False
* ClusterId : GUID généré de manière aléatoire pour chaque cluster
* ServiceFabricVersion
* Adresse IP de la machine virtuelle ou de l’ordinateur à partir de laquelle/duquel les données de télémétrie sont téléchargées

Pour désactiver les données de télémétrie, ajoutez ce qui suit à *properties* dans votre configuration de cluster : *enableTelemetry: false*.

<a id="previewfeatures" name="previewfeatures_anchor"></a>

## <a name="preview-features-included-in-this-package"></a>Fonctionnalités préliminaires incluses dans ce package
Aucun.


> [!NOTE]
> À partir de la nouvelle [version mise à la disposition générale du cluster autonome pour Windows Server (version 5.3.204.x)](https://azure.microsoft.com/blog/azure-service-fabric-for-windows-server-now-ga/), vous pouvez mettre à niveau votre cluster vers des versions ultérieures, manuellement ou automatiquement. Reportez-vous au document [Mettre à niveau une version autonome du cluster Service Fabric](service-fabric-cluster-upgrade-windows-server.md) pour plus d’informations.
> 
> 

## <a name="next-steps"></a>Étapes suivantes
* [Déployer et supprimer des applications avec PowerShell](service-fabric-deploy-remove-applications.md)
* [Paramètres de configuration pour un cluster Windows autonome](service-fabric-cluster-manifest.md)
* [Ajouter ou supprimer des nœuds à/d’un cluster Service Fabric autonome](service-fabric-cluster-windows-server-add-remove-nodes.md)
* [Mettre à niveau une version autonome du cluster Service Fabric](service-fabric-cluster-upgrade-windows-server.md)
* [Créer un cluster Service Fabric autonome avec des machines virtuelles Azure sous Windows](./service-fabric-cluster-creation-via-arm.md)
* [Sécuriser un cluster autonome sur Windows à l’aide de la sécurité Windows](service-fabric-windows-cluster-windows-security.md)
* [Sécuriser un cluster autonome sur Windows à l’aide de certificats X509](service-fabric-windows-cluster-x509-security.md)

<!--Image references-->
[Trusted Zone]: ./media/service-fabric-cluster-creation-for-windows-server/TrustedZone.png
[service-fabric-explorer]: ./media/service-fabric-cluster-creation-for-windows-server/sfx.png
