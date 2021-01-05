---
title: Configurer un environnement de développement Windows pour Service Fabric Mesh
description: Configurez votre environnement de développement Windows afin de pouvoir créer une application de maillage Service Fabric et la déployer dans Azure Service Fabric Mesh.
author: georgewallace
ms.author: gwallace
ms.date: 12/12/2018
ms.topic: conceptual
ms.openlocfilehash: 479e975abd40be80e17fc187f1d13500d11cb983
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91842748"
---
# <a name="set-up-your-windows-development-environment-to-build-service-fabric-mesh-apps"></a>Configurer votre environnement de développement Windows pour créer des applications Service Fabric Mesh

Pour générer et exécuter des applications Azure Service Fabric Mesh sur votre machine de développement Windows, vous devez disposer de ce qui suit :

* Docker
* Visual Studio 2017 ou version ultérieure
* Runtime Service Fabric Mesh
* Outils et kit de développement logiciel (SDK) Service Fabric Mesh

Une des versions suivantes de Windows :

* Windows 10 (Entreprise, Professionnel ou Éducation) version 1709 (Fall Creators Update) ou 1803 (mise à jour d'avril 2018 de Windows 10)
* Windows Server version 1709
* Windows Server version 1803

Les instructions suivantes vous aideront à tout installer en fonction de la version de Windows que vous exécutez.

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="visual-studio"></a>Visual Studio

Visual Studio 2017 ou version ultérieure est requis pour déployer des applications Service Fabric Mesh. [Installez la version 15.6.0][download-visual-studio] ou ultérieure et activez les charges de travail suivantes :

* Développement web et ASP.NET
* Développement Azure

## <a name="install-docker"></a>Installation de Docker

Si vous avez déjà installé Docker, vérifiez que vous disposez de la dernière version. Docker peut vous indiquer quand une nouvelle version est disponible, mais vérifiez manuellement pour vous assurer que vous disposez de la version la plus récente.

#### <a name="install-docker-on-windows-10"></a>Installer Docker sous Windows 10

Téléchargez et installez la dernière version de [Docker Community Edition pour Windows][download-docker] pour prendre en charge les applications Service Fabric en conteneur utilisées par Service Fabric Mesh.

Pendant l’installation, sélectionnez **Utiliser des conteneurs Windows au lieu des conteneurs Linux** lorsque vous y êtes invité.

Si Hyper-V n’est pas activé sur votre ordinateur, le programme d’installation de Docker propose de l’activer. Pour ce faire, cliquez sur **OK** à l’invite.

#### <a name="install-docker-on-windows-server-2016"></a>Installer Docker sous Windows Server 2016

Si le rôle Hyper-V n’est pas activé, ouvrez PowerShell en tant qu’administrateur et exécutez la commande suivante pour activer Hyper-V, puis redémarrez votre ordinateur. Pour plus d’informations, consultez [Docker Enterprise Edition pour Windows Server][download-docker-server].

```powershell
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools
```

Redémarrez votre ordinateur.

Ouvrez PowerShell en tant qu’administrateur et exécutez la commande suivante pour installer Docker :

```powershell
Install-Module DockerMsftProvider -Force
Install-Package Docker -ProviderName DockerMsftProvider -Force
Install-WindowsFeature Containers
```

## <a name="sdk-and-tools"></a>Kit de développement logiciel (SDK) et outils

Installez le runtime, le kit de développement logiciel (SDK) et les outils Service Fabric Mesh dans l’ordre suivant.

1. Installez le [Kit de développement logiciel (SDK) Service Fabric Mesh][download-sdkmesh] à l’aide de Web Platform Installer. Cela installera également le Kit de développement logiciel (SDK) et le runtime Microsoft Azure Service Fabric.
2. Installez l’[extension Visual Studio Service Fabric Mesh Tools (préversion)][download-tools] à partir de Visual Studio Marketplace.

## <a name="build-a-cluster"></a>Générer un cluster

> [!IMPORTANT]
> Docker **doit** être en cours d’exécution pour que vous puissiez générer un cluster.
> Vérifiez que Docker est en cours d’exécution en ouvrant une fenêtre de terminal et en exécutant `docker ps` pour voir si une erreur se produit. Si la réponse n’indique aucune erreur, Docker est en cours d’exécution et vous êtes prêt à générer un cluster.

> [!Note]
> Si vous développez sur la machine dotée de Windows Fall Creators Update (version 1709), vous pouvez uniquement utiliser des images Docker de la version 1709 de Windows.
> Si vous développez sur l'ordinateur doté de la mise à jour d'avril 2018 de Windows 10 (version 1803), vous pouvez indistinctement utiliser des images Docker des versions 1709 et 1803 de Windows.

Si vous utilisez Visual Studio, vous pouvez ignorer cette section, car Visual Studio crée un cluster local pour vous si vous n’en avez pas.

Pour des performances de débogage optimales lorsque vous créez et exécutez une seule application Service Fabric à la fois, créez un cluster de développement local à nœud unique. Si vous exécutez plusieurs applications à la fois, créez un cluster de développement local à cinq nœuds. Le cluster doit être en cours d'exécution chaque fois que vous déployez ou déboguez un projet Service Fabric Mesh.

Après avoir installé le runtime, les SDK, Visual Studio Tools et Docker, et après avoir démarré ce dernier, créez un cluster de développement.

1. Fermez votre fenêtre PowerShell.
2. Ouvrez une nouvelle fenêtre PowerShell avec élévation de privilèges en tant qu’administrateur. Cette étape est nécessaire pour charger les modules Service Fabric récemment installés.
3. Exécutez la commande PowerShell suivante pour créer un cluster de développement :

    ```powershell
    . "C:\Program Files\Microsoft SDKs\Service Fabric\ClusterSetup\DevClusterSetup.ps1" -CreateMeshCluster -CreateOneNodeCluster
    ```
4. Pour démarrer le gestionnaire de cluster local, exécutez la commande PowerShell suivante :

    ```powershell
    . "C:\Program Files\Microsoft SDKs\Service Fabric\Tools\ServiceFabricLocalClusterManager\ServiceFabricLocalClusterManager.exe"
    ```
5. Une fois que vous avez démarré le gestionnaire de cluster (il s’affiche alors dans votre barre d’état système), faites un clic droit, puis cliquez sur **Démarrer le cluster Local**.

![Figure 1 : Démarrer le cluster local](./media/service-fabric-mesh-howto-setup-developer-environment-sdk/start-local-cluster.png)

Vous êtes maintenant prêt à créer des applications Service Fabric Mesh !

## <a name="next-steps"></a>Étapes suivantes

Consultez le didacticiel [Créer une application Azure Service Fabric](service-fabric-mesh-tutorial-create-dotnetcore.md).

Trouvez les réponses [aux questions fréquentes et aux problèmes connus](service-fabric-mesh-faq.md).

[azure-cli-install]: /cli/azure/install-azure-cli
[download-docker]: https://store.docker.com/editions/community/docker-ce-desktop-windows
[download-docker-server]: https://docs.docker.com/install/windows/docker-ee/
[download-runtime]: https://aka.ms/sfruntime
[download-sdk]: https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-CoreSDK
[download-sdkmesh]: https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-SDK-Mesh
[download-tools]: https://aka.ms/sfmesh_vs2017tools
[download-visual-studio]: https://www.visualstudio.com/downloads/
