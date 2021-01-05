---
title: Installer un client autonome Service Fabric
description: Dans ce tutoriel, découvrez comment installer le client autonome Service Fabric sur le cluster.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc
ms.openlocfilehash: ae0b343be986f4d8d5176c1f39eef6b23ca81278
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91840640"
---
# <a name="tutorial-install-and-create-service-fabric-cluster"></a>Didacticiel : installer et créer un cluster Service Fabric

Les clusters autonomes Service Fabric vous permettent de choisir votre propre environnement et de créer un cluster dans le cadre de l’approche « n’importe quel SE, n’importe quel cloud » suivie par Service Fabric. Dans le cadre de cette série de tutoriels, vous créez un cluster autonome hébergé sur AWS ou Azure et vous installez une application dans celui-ci.

Ce tutoriel est le deuxième d’une série. Ce tutoriel vous guide tout au long du processus de création d’un cluster autonome Service Fabric.

Dans cet article, vous allez apprendre à :

> [!div class="checklist"]
> * Télécharger et installer le package autonome Service Fabric
> * Créer le cluster Service Fabric
> * Vous connecter au cluster Service Fabric

## <a name="download-the-service-fabric-for-windows-server-package"></a>Télécharger le package Service Fabric pour Windows Server

Service Fabric fournit un package d’installation pour créer des clusters Service Fabric autonomes.  [Téléchargez le package d’installation](https://go.microsoft.com/fwlink/?LinkId=730690) sur votre ordinateur local.  Après avoir téléchargé le package, copiez-le par le biais de la connexion RDP à votre machine virtuelle et collez-le sur le Bureau.

Sélectionnez le fichier zip, puis ouvrez le menu contextuel et sélectionnez **Extraire tout** > **Extraire**.  Lorsque vous extrayez les fichiers, vous générez sur le Bureau un dossier du même nom que le fichier zip.

Pour plus d’informations, consultez [l’article décrivant le contenu du package d’installation](service-fabric-cluster-standalone-package-contents.md).

## <a name="set-up-your-configuration-file"></a>Préparation de votre fichier de configuration

Étant donné que vous êtes en train de générer un cluster Windows à trois nœuds, vous devez modifier le fichier `ClusterConfig.Unsecure.MultiMachine.json`.

Ensuite, remplacez les trois lignes ipAddress du fichier (lignes 8, 15 et 22) par les adresses IP de chacune des instances.

Après leur mise à jour, les nœuds apparaissent comme suit :

```json
        {
            "nodeName": "vm0",
            "ipAddress": "172.31.27.1",
            "nodeTypeRef": "NodeType0",
            "faultDomain": "fd:/dc1/r0",
            "upgradeDomain": "UD0"
        }
```

Ensuite, vous devez mettre à jour quelques propriétés.  Sur la ligne 34, vous devez modifier la chaîne de connexion pour le magasin de diagnostics, qui doit se présenter comme ceci : `"connectionstring": "C:\\ProgramData\\SF\\DiagnosticsStore"`

Enfin, dans la section `nodeTypes` de la configuration, ajoutez une nouvelle section pour mapper les ports éphémères que Windows utilisera.  Le fichier de configuration doit se présenter comme suit :

```json
"applicationPorts": {
    "startPort": "20001",
    "endPort": "20031"
},
"ephemeralPorts": {
    "startPort": "20606",
    "endPort": "20861"
},
"isPrimary": true
```

## <a name="validate-the-environment"></a>Valider l’environnement

Le script *TestConfiguration.ps1* contenu dans le package autonome est utilisé pour analyser les meilleures pratiques, afin de vérifier si un cluster peut être déployé dans un environnement donné. La section [Préparation du déploiement](service-fabric-cluster-standalone-deployment-preparation.md) répertorie les conditions préalables et les exigences de l’environnement. Exécutez le script pour vérifier si vous pouvez créer le cluster de développement :

```powershell
cd .\Desktop\Microsoft.Azure.ServiceFabric.WindowsServer.6.2.274.9494\
.\TestConfiguration.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json
```

Vous devez obtenir un résultat semblable à l’exemple ci-dessous. Si le dernier champ « Passed » est renvoyé avec la valeur `True`, les contrôles d’intégrité ont réussi, et le cluster semble être déployable à l’aide de la configuration d’entrée.

```powershell
Trace folder already exists. Traces will be written to existing trace folder: C:\Users\Administrator\Desktop\Microsoft.Azure.ServiceFabric.WindowsServer.6.2.274.9494\DeploymentTraces
Running Best Practices Analyzer...
Best Practices Analyzer completed successfully.


LocalAdminPrivilege        : True
IsJsonValid                : True
IsCabValid                 :
RequiredPortsOpen          : True
RemoteRegistryAvailable    : True
FirewallAvailable          : True
RpcCheckPassed             : True
NoConflictingInstallations : True
FabricInstallable          : True
DataDrivesAvailable        : True
NoDomainController         : True
Passed                     : True
```

## <a name="create-the-cluster"></a>Créer le cluster

Après avoir validé la configuration du cluster, exécutez le script *CreateServiceFabricCluster.ps1* pour déployer le cluster Service Fabric sur les machines virtuelles du fichier de configuration.

```powershell
.\CreateServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json -AcceptEULA
```

Si tout fonctionne correctement, vous obtiendrez une sortie semblable à ceci :

```powershell
Your cluster is successfully created! You can connect and manage your cluster using Microsoft Azure Service Fabric Explorer or PowerShell. To connect through PowerShell, run 'Connect-ServiceFabricCluster [ClusterConnectionEndpoint]'.
```

> [!NOTE]
> Les traces de déploiement sont écrites sur la machine virtuelle/l’ordinateur où vous avez exécuté le script PowerShell CreateServiceFabricCluster.ps1. Elles ne se trouvent pas dans le sous-dossier DeploymentTraces, situé dans le répertoire à partir duquel le script a été exécuté. Pour voir si Service Fabric a été déployé correctement sur un ordinateur, recherchez les fichiers installés dans le répertoire FabricDataRoot, comme indiqué dans la section FabricSettings du fichier de configuration du cluster (par défaut c:\ProgramData\SF). De même, vous pouvez consulter l’exécution des processus FabricHost.exe et Fabric.exe dans le Gestionnaire des tâches.
>
>

### <a name="open-service-fabric-explorer"></a>Ouvrir Service Fabric Explorer

Désormais, vous pouvez vous connecter au cluster avec Service Fabric Explorer directement à partir de l’un des ordinateurs avec http:\//localhost:19080/Explorer/index.html ou à distance avec http:\//<*adresse_IP_ordinateur*>:19080/Explorer/index.html.

## <a name="add-and-remove-nodes"></a>Ajouter et supprimer des nœuds

Vous pouvez ajouter des nœuds à votre cluster Service Fabric autonome ou en supprimer en fonction de l’évolution de vos besoins. Consultez l’article [Ajouter ou supprimer des nœuds à/d’un cluster Service Fabric autonome](service-fabric-cluster-windows-server-add-remove-nodes.md) pour obtenir des instructions détaillées.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à charger de grandes quantités de données aléatoires dans un compte de stockage en parallèle, notamment à :

> [!div class="checklist"]
> * Configurer la chaîne de connexion
> * Créer l’application
> * Exécution de l'application
> * Valider le nombre de connexions

Passez au troisième volet de la série de tutoriels pour installer une application dans le cluster que vous avez créé.

> [!div class="nextstepaction"]
> [Installer l’application dans le cluster Service Fabric](service-fabric-tutorial-standalone-install-an-application.md)

<!--Image references-->
[Trusted Zone]: ./media/service-fabric-cluster-creation-for-windows-server/TrustedZone.png
