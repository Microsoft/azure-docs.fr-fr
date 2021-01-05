---
title: Bien démarrer avec Azure Service Fabric à l’aide de VS Code
description: Cet article est une vue d’ensemble de la création d’applications Service Fabric à l’aide de Visual Studio Code.
author: peterpogorski
ms.topic: article
ms.date: 06/29/2018
ms.author: pepogors
ms.custom: devx-track-js
ms.openlocfilehash: a655199d854462285ca98a2d8f454c483c061f99
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92316673"
---
# <a name="service-fabric-for-visual-studio-code"></a>Service Fabric pour Visual Studio Code

[L’extension Service Fabric Reliable Services pour VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-service-fabric-reliable-services) fournit les outils nécessaires pour créer, générer et déboguer des applications Service Fabric sur les systèmes d’exploitation Windows, Linux et macOS.

Cet article fournit une vue d’ensemble des exigences et de la configuration de l’extension, ainsi que de l’utilisation des différentes commandes qu’elle fournit. 

> [!IMPORTANT]
> Vous pouvez développer des applications Service Fabric en Java sur des machines Windows, mais ne pouvez les déployer que sur des clusters Azure Linux. Le débogage des applications Java n’est pas pris en charge sur Windows.

## <a name="prerequisites"></a>Prérequis

Les composants requis suivants doivent être installés sur tous les environnements.

* [Visual Studio Code](https://code.visualstudio.com/)
* [Node.JS](https://nodejs.org/)
* [Git](https://git-scm.com/)
* [SDK Service Fabric](./service-fabric-get-started.md)
* Générateurs Yeoman : installez les générateurs appropriés pour votre application

   ```sh
   npm install -g yo
   npm install -g generator-azuresfjava
   npm install -g generator-azuresfcsharp
   npm install -g generator-azuresfcontainer
   npm install -g generator-azuresfguest
   ```

Les composants requis suivants doivent être installés pour le développement Java :

* [SDK Java](/azure/developer/java/fundamentals/java-jdk-long-term-support) (version 1.8)
* [Gradle](https://gradle.org/install/)
* [Débogueur pour l’extension Java VS Code](https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-debug) Nécessaire pour déboguer les services Java. Le débogage des services Java n’est pris en charge que sur Linux. Vous pouvez effectuer l’installation en cliquant sur l’icône Extensions dans la **barre d’activités** de VS Code et en recherchant l’extension, ou bien à partir de la place de marché VS Code.

Les composants requis suivants doivent être installés pour le développement .NET Core/C# :

* [.NET Core](https://www.microsoft.com/net/learn/get-started) (version 2.0.0 ou ultérieure)
* [Extension VS Code C# pour Visual Studio Code (technologie OmniSharp)](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) Nécessaire pour déboguer les services C#. Vous pouvez effectuer l’installation en cliquant sur l’icône Extensions dans la **barre d’activités** de VS Code et en recherchant l’extension, ou bien à partir de la place de marché VS Code.

## <a name="setup"></a>Programme d’installation

1. Ouvrez Visual Studio Code.
2. Cliquez sur l’icône Extensions dans la **barre d’activités** située sur le côté gauche de VS Code. Recherchez « Service Fabric ». Cliquez sur **installer** pour l’extension Service Fabric Reliable Services.

## <a name="commands"></a>Commandes
L’extension Service Fabric Reliable Services pour VS Code fournit de nombreuses commandes qui permettent aux développeurs de créer et de déployer des projets Service Fabric. Vous pouvez appeler une commande à partir de la **Palette de commandes** en appuyant sur `(Ctrl + Shift + p)`, en tapant le nom de la commande dans la barre d’entrée et en sélectionnant la commande souhaitée à partir de la liste de choix. 

* Service Fabric: Create Application 
* Service Fabric: Publish Application 
* Service Fabric: Deploy Application 
* Service Fabric: Remove Application  
* Service Fabric: Build Application 
* Service Fabric: Clean Application 

### <a name="service-fabric-create-application"></a>Service Fabric: Create Application

La commande **Service Fabric: Create application** crée une application Service Fabric dans votre espace de travail actuel. Selon les générateurs yeoman installés sur votre machine de développement, vous pouvez créer plusieurs types d’application Service Fabric, y compris des projets Java, C#, conteneur et invité. 

1.  Sélectionnez la commande **Service Fabric: Create Application**.
2.  Sélectionnez le type de votre nouvelle application Service Fabric. 
3.  Entrez le nom de l’application à créer.
3.  Sélectionnez le type de service que vous souhaitez ajouter à votre application Service Fabric. 
4.  Suivez les invites pour nommer le service. 
5.  La nouvelle application Service Fabric s’affiche dans l’espace de travail.
6.  Ouvrez le dossier de la nouvelle application afin qu’il devienne le dossier racine dans l’espace de travail. Vous pouvez continuer à exécuter des commandes à partir de cet emplacement.

### <a name="service-fabric-add-service"></a>Service Fabric: Add Service
La commande **Service Fabric: Add Service** ajoute un nouveau service à une application Service Fabric existante. L’application à laquelle est ajouté le service doit être le répertoire racine de l’espace de travail. 

1.  Sélectionnez la commande **Service Fabric: Add Service**.
2.  Sélectionnez le type de votre application Service Fabric actuelle. 
3.  Sélectionnez le type de service que vous souhaitez ajouter à votre application Service Fabric. 
4.  Suivez les invites pour nommer le service. 
5.  Le nouveau service s’affiche dans le répertoire de votre projet. 

### <a name="service-fabric-publish-application"></a>Service Fabric: Publish Application
La commande **Service Fabric: Publish Application** déploie votre application Service Fabric sur un cluster distant. Le cluster cible peut être un cluster sécurisé ou non sécurisé. Si les paramètres ne sont pas définis dans Cloud.json, l’application est déployée sur le cluster local.

1.  La première fois que l’application est générée, un fichier Cloud.json est généré dans le répertoire du projet.
2.  Entrez les valeurs du cluster auquel vous souhaitez vous connecter dans le fichier Cloud.json.
3.  Sélectionnez la commande **Service Fabric: Publish Application**.
4.  Affichez le cluster cible avec Service Fabric Explorer pour vérifier que l’application a été installée. 

### <a name="service-fabric-deploy-application-localhost"></a>Service Fabric: Deploy Application (Localhost)
La commande **Service Fabric: Deploy Application** déploie votre application Service Fabric sur votre cluster local. Vérifiez que votre cluster local est en cours d’exécution avant d’utiliser la commande. 

1. Sélectionnez la commande **Service Fabric: Deploy Application**.
2. Affichez le cluster local avec Service Fabric Explorer (http:\//localhost:19080/Explorer) pour vérifier que l’application a été installée. Cette opération pouvant prendre un certain temps, soyez patient.
3. Vous pouvez également utiliser la commande **Service Fabric: Publish Application** sans aucun paramètre défini dans le fichier Cloud.json pour la déployer sur un cluster local.

> [!NOTE]
> Le déploiement d’applications Java sur le cluster local n’est pas pris en charge sur les machines Windows.

### <a name="service-fabric-remove-application"></a>Service Fabric: Remove Application
La commande **Service Fabric: Remove Application** supprime une application Service Fabric du cluster sur lequel elle avait été déployée à l’aide de l’extension VS Code. 

1.  Sélectionnez la commande **Service Fabric: Remove Application**.
2.  Affichez le cluster avec Service Fabric Explorer pour vérifier que l’application a été supprimée. Cette opération pouvant prendre un certain temps, soyez patient.

### <a name="service-fabric-build-application"></a>Service Fabric: Build Application
La commande **Service Fabric: Build Application** peut générer des applications Service Fabric Java ou C#. 

1.  Vérifiez que vous vous trouvez dans le dossier racine de l’application avant d’exécuter cette commande. La commande identifie le type d’application (C# ou Java) et génère votre application en conséquence.
2.  Sélectionnez la commande **Service Fabric: Générer une application**.
3.  La sortie du processus de génération est écrite dans le terminal intégré.

### <a name="service-fabric-clean-application"></a>Service Fabric: Clean Application
La commande **Service Fabric: Clean Application** supprime tous les fichiers jar et les bibliothèques natives issus de l’opération de génération. Valide pour les applications Java uniquement. 

1.  Vérifiez que vous vous trouvez dans le dossier racine de l’application avant d’exécuter cette commande. 
2.  Sélectionnez la commande **Service Fabric: Clean Application**.
3.  La sortie du processus de nettoyage est écrite dans le terminal intégré.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [développer et déboguer des applications Service Fabric en C# avec VS Code](./service-fabric-develop-csharp-applications-with-vs-code.md).
* Découvrez comment [développer et déboguer des applications Service Fabric en Java avec VS Code](./service-fabric-develop-java-applications-with-vs-code.md).