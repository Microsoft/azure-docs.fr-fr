---
title: Configurer un environnement de développement Windows
description: Installez le runtime, le kit de développement logiciel et créez un cluster de développement local. Une fois la configuration terminée, vous serez prêt à générer des applications sur Windows.
author: peterpogorski
ms.topic: conceptual
ms.date: 06/16/2020
ms.custom: sfrev, devx-track-azurepowershell
ms.openlocfilehash: c252a1dc86c50327e32816abd99bc206f3b74608
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96902882"
---
# <a name="prepare-your-development-environment-on-windows"></a>Préparer votre environnement de développement sur Windows

> [!div class="op_single_selector"]
> * [Windows](service-fabric-get-started.md) 
> * [Linux](service-fabric-get-started-linux.md)
> * [Mac OS X](service-fabric-get-started-mac.md)
>
>

Pour générer et exécuter des [applications Azure Service Fabric][1] sur votre machine de développement Windows, installez le runtime, le Kit de développement logiciel (SDK) et les outils Service Fabric. Vous devez également [activer l’exécution des scripts Windows PowerShell](#enable-powershell-script-execution) inclus dans le Kit de développement logiciel (SDK).

## <a name="prerequisites"></a>Prérequis

### <a name="supported-operating-system-versions"></a>Versions du système d'exploitation prises en charge

Les versions de système d’exploitation prises en charge pour le développement sont les suivantes :

* Windows 7
* Windows 8 et Windows 8.1
* Windows Server 2012 R2
* Windows Server 2016
* Windows 10

> [!NOTE]
> Windows 7 prend en charge :
> - Windows 7 inclut uniquement Windows PowerShell 2.0 par défaut. Les applets de commande PowerShell de Service Fabric nécessitent PowerShell 3.0 ou version ultérieure. Vous pouvez [télécharger Windows PowerShell 5.1][powershell5-download] à partir du Centre de téléchargement Microsoft.
> - Le Proxy inverse de Service Fabric n’est pas disponible sur Windows 7.

## <a name="install-the-sdk-and-tools"></a>Installer le Kit de développement logiciel (SDK) et les outils

Web Platform Installer (WebPI) est la méthode recommandée pour installer le Kit de développement logiciel (SDK) et les outils. Si vous recevez des erreurs de runtime utilisant WebPI, vous trouverez également des liens directs vers les programmes d’installation dans les notes de publication pour une version spécifique de Service Fabric. Les notes de publication figurent dans les diverses annonces de version sur le [blog de l’équipe Service Fabric](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric).

> [!NOTE]
> Les mises à niveau du cluster de développement Service Fabric local ne sont pas prises en charge.

### <a name="to-use-visual-studio-2017-or-2019"></a>Pour utiliser Visual Studio 2017 ou 2019

Les outils Service Fabric font partie de la charge de travail de développement Azure dans Visual Studio 2017 et 2019. Activez cette charge de travail dans le cadre de votre installation de Visual Studio.
En outre, vous devez installer le Kit de développement logiciel (SDK) et le runtime Microsoft Azure Service Fabric à l’aide de Web Platform Installer.

* [Installez le Kit de développement logiciel (SDK) Microsoft Azure Service Fabric][core-sdk]

### <a name="sdk-installation-only"></a>Installation du Kit de développement logiciel (SDK) uniquement

Si vous avez uniquement besoin du SDK, vous pouvez installer ce package :

* [Installez le Kit de développement logiciel (SDK) Microsoft Azure Service Fabric][core-sdk]

Les versions actuelles sont les suivantes :

* SDK et outils Service Fabric 4.2.445
* Runtime Service Fabric 7.2.445

Pour obtenir la liste des versions prises en charge, consultez [Versions de Service Fabric](service-fabric-versions.md).

> [!NOTE]
> Les clusters de machine unique (OneBox) ne sont pas pris en charge pour les mises à niveau d’Application ou de Cluster. Si vous avez besoin d’effectuer une mise à niveau de Cluster ou rencontrez des problèmes de mise à niveau d’Application, supprimez le cluster OneBox, puis recréez-le. 

## <a name="enable-powershell-script-execution"></a>Activer l'exécution du script PowerShell

Service Fabric utilise des scripts Windows PowerShell pour créer un cluster de développement local et déployer des applications à partir de Visual Studio. Par défaut, Windows bloque l’exécution de ces scripts. Pour les activer, vous devez modifier votre stratégie d'exécution PowerShell. Ouvrez PowerShell en tant qu'administrateur et entrez la commande suivante :

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force -Scope CurrentUser
```

## <a name="install-docker-optional"></a>Installer Docker (facultatif)

[Service Fabric est un orchestrateur de conteneurs](service-fabric-containers-overview.md) qui permet de déployer des microservices sur un cluster de machines. Pour exécuter des applications de conteneur Windows sur votre cluster de développement local, vous devez d’abord installer Docker pour Windows. Obtenez [Docker CE pour Windows (stable)](https://store.docker.com/editions/community/docker-ce-desktop-windows?tab=description). Après l’installation et le démarrage de Docker, cliquez avec le bouton droit de la souris sur l’icône de la barre d’état, puis sélectionnez **Switch to Windows containers** (Basculer vers les conteneurs Windows). Cette étape est nécessaire pour exécuter des images Docker basées sur Windows.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez fini de configurer votre environnement de développement, commencez à créer et à exécuter des applications.

* [Découvrez comment créer, déployer et gérer des applications](service-fabric-tutorial-create-dotnet-app.md)
* [Découvrir les modèles de programmation : Reliable Services et Reliable Actors](service-fabric-choose-framework.md)
* [Consulter les exemples de code Service Fabric sur GitHub](/samples/browse/?products=azure)
* [Visualiser votre cluster à l’aide de l’outil Service Fabric Explorer](service-fabric-visualizing-your-cluster.md)
* [Préparer un environnement de développement Linux sur Windows](service-fabric-local-linux-cluster-windows.md)
* En savoir plus sur les [options de prise en charge de Service Fabric](service-fabric-support.md)

[1]: https://azure.microsoft.com/campaigns/service-fabric/ "Page de campagne Service Fabric"
[2]: https://go.microsoft.com/fwlink/?LinkId=517106 "VS RC"
[full-bundle-vs2015]:https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-VS2015 "Lien WebPI VS 2015"
[full-bundle-dev15]:https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-Dev15 "Lien WebPI Dev15"
[core-sdk]:https://www.microsoft.com/web/handlers/webpi.ashx?command=getinstallerredirect&appid=MicrosoftAzure-ServiceFabric-CoreSDK "Lien WebPI du Kit de développement logiciel principal"
[powershell5-download]:https://www.microsoft.com/download/details.aspx?id=54616
