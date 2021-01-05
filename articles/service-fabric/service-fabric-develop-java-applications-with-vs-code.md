---
title: Développer des applications Java avec Visual Studio Code
description: Cet article montre comment générer, déployer et déboguer des applications Service Fabric en Java à l’aide de Visual Studio Code.
author: peterpogorski
ms.topic: article
ms.date: 06/29/2018
ms.custom: devx-track-java
ms.author: pepogors
ms.openlocfilehash: cc65deb924a9f3367c2ea1d7c71544743ccf2697
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87327359"
---
# <a name="develop-java-service-fabric-applications-with-visual-studio-code"></a>Développer des applications Service Fabric en Java avec Visual Studio Code

[L’extension Service Fabric Reliable Services pour VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-service-fabric-reliable-services) facilite la génération d’applications Service Fabric en Java sur les systèmes d’exploitation Windows, Linux et macOS.

Cet article vous montre comment générer, déployer et déboguer une application Service Fabric en Java à l’aide de Visual Studio Code.

> [!IMPORTANT]
> Vous pouvez développer des applications Service Fabric en Java sur des machines Windows, mais ne pouvez les déployer que sur des clusters Azure Linux. Le débogage des applications Java n’est pas pris en charge sur Windows.

## <a name="prerequisites"></a>Prérequis

Cet article suppose que vous avez déjà installé VS Code, l’extension Service Fabric Reliable Services pour VS Code et toutes les dépendances requises pour votre environnement de développement. Pour plus d’informations, consultez [Bien démarrer](./service-fabric-get-started-vs-code.md#prerequisites).

## <a name="download-the-sample"></a>Télécharger l’exemple
Cet article utilise l’application Voting dans le [dépôt GitHub des exemples de démarrage rapide des applications Service Fabric en Java](https://github.com/Azure-Samples/service-fabric-java-quickstart). 

Pour cloner le dépôt sur votre machine de développement, exécutez la commande suivante à partir d’une fenêtre de terminal (fenêtre de commande sur Windows) :

```sh
git clone https://github.com/Azure-Samples/service-fabric-java-quickstart.git
```

## <a name="open-the-application-in-vs-code"></a>Ouvrir l’application dans VS Code

Ouvrez Visual Studio Code.  Cliquez sur l’icône Explorer dans la **barre d’activités** et cliquez sur **Ouvrir un dossier**, ou cliquez sur **Fichier -> Ouvrir un dossier**. Accédez au répertoire *./service-fabric-java-quickstart/Voting* dans le dossier où vous avez cloné le dépôt, puis cliquez sur **OK**. L’espace de travail doit contenir les mêmes fichiers que ceux indiqués dans la capture d’écran ci-dessous.

![Application Voting en Java dans l’espace de travail](./media/service-fabric-develop-java-applications-with-vs-code/java-voting-application.png)

## <a name="build-the-application"></a>Créer l’application

1. Appuyez sur (Ctrl+Maj+p) pour ouvrir la **Palette de commandes** dans VS Code.
2. Recherchez puis sélectionnez la commande **Service Fabric : Générer une application**. La sortie de la génération est envoyée au terminal intégré.

   ![Commande Générer une application dans VS Code](./media/service-fabric-develop-java-applications-with-vs-code/sf-build-application.png)

## <a name="deploy-the-application-to-the-local-cluster"></a>Déployer l’application sur le cluster local
Une fois que vous avez généré l’application, vous pouvez la déployer sur le cluster local. 

> [!IMPORTANT]
> Le déploiement d’applications Java sur le cluster local n’est pas pris en charge sur les machines Windows.

1. Dans la **Palette de commandes**, sélectionnez la commande **Service Fabric : Déployer l’application (Localhost)** . La sortie du processus d’installation est envoyée au terminal intégré.

   ![Commande Déployer une application dans VS Code](./media/service-fabric-develop-java-applications-with-vs-code/sf-deploy-application.png)

4. Quand le déploiement est terminé, lancez un navigateur et ouvrez Service Fabric Explorer : `http://localhost:19080/Explorer`. Vous devez voir que l’application est en cours d’exécution. Cette opération pouvant prendre un certain temps, soyez patient. 

   ![Application Voting dans Service Fabric Explorer](./media/service-fabric-develop-java-applications-with-vs-code/sfx-localhost-java.png)

4. Une fois que vous avez vérifié que l’application est en cours d’exécution, lancez un navigateur et ouvrez cette page : `http://localhost:8080`. Il s’agit du frontend web de l’application. Vous pouvez ajouter des éléments et cliquer dessus pour voter.

   ![Application Voting dans le navigateur](./media/service-fabric-develop-java-applications-with-vs-code/voting-sample-in-browser.png)

5. Pour supprimer l'application du cluster, sélectionnez la commande **Service Fabric : Supprimer l'application** à partir de la **Palette de commandes**. La sortie du processus de désinstallation est envoyée au terminal intégré. Vous pouvez utiliser Service Fabric Explorer pour vérifier que l’application a été supprimée du cluster local.

## <a name="debug-the-application"></a>Déboguer l’application
Quand vous déboguez une application dans VS Code, celle-ci doit être en cours d’exécution sur un cluster local. Vous pouvez ajouter des points d’arrêt au code.

> [!IMPORTANT]
> Le débogage des applications Java n’est pas pris en charge sur les machines Windows.

Pour préparer le service de données de vote et l’application Voting au débogage, effectuez les étapes suivantes :

1. Mettez à jour le fichier *Voting/VotingApplication/VotingDataServicePkg/Code/entryPoint.sh*.
Commentez la commande sur la ligne 6 (utilisez « # ») et ajoutez la commande suivante en bas du fichier :

   ```
   java -Xdebug -Xrunjdwp:transport=dt_socket,address=8001,server=y,suspend=n -Djava.library.path=$LD_LIBRARY_PATH -jar VotingDataService.jar
   ```

2. Mettez à jour le fichier *Voting/VotingApplication/ApplicationManifest.xml*. Définissez les attributs **MinReplicaSetSize** et **TargetReplicaSetSize** sur « 1 » dans l’élément **StatefulService** :
   
   ```xml
         <StatefulService MinReplicaSetSize="1" ServiceTypeName="VotingDataServiceType" TargetReplicaSetSize="1">
   ```

3. Cliquez sur l’icône de débogage dans la **barre d’activités** pour ouvrir la vue du débogueur dans VS Code. Cliquez sur l’icône en forme d’engrenage en haut de la vue du débogueur et sélectionnez **Java** dans la liste déroulante des environnements. Le fichier launch.json s’ouvre. 

   ![Icône de débogage dans l’espace de travail VS Code](./media/service-fabric-develop-java-applications-with-vs-code/debug-icon-workspace.png)

3. Dans le fichier launch.json, définissez la valeur de port dans la configuration nommée **Debug (Attach)** (Déboguer (Attacher)) sur **8001**. Enregistrez le fichier .

   ![Configuration de débogage pour le fichier launch.json](./media/service-fabric-develop-java-applications-with-vs-code/launch-json-java.png)

4. Déployez l'application sur le cluster local à l'aide de la commande **Service Fabric : Déployer une application (localhost)** . Vérifiez que l’application est en cours d’exécution dans Service Fabric Explorer. Votre application est maintenant prête à être déboguée.

Pour définir un point d’arrêt, effectuez les étapes suivantes :

1. Dans Explorer, ouvrez le fichier */Voting/VotingDataService/src/statefulservice/VotingDataService.java*. Définissez un point d’arrêt sur la première ligne de code du bloc `try` de la méthode `addItem` (ligne 80).
   
   ![Définir un point d’arrêt dans le service de données de vote](./media/service-fabric-develop-java-applications-with-vs-code/breakpoint-set.png)

   > [!IMPORTANT]
   > Veillez à définir les points d’arrêt sur des lignes de code exécutables. Par exemple, les points d’arrêt définis sur des déclarations de méthode, des instructions `try` ou des instructions `catch` sont ignorés par le débogueur.
2. Pour commencer le débogage, cliquez sur l’icône de débogage dans la **barre d’activités**, sélectionnez la configuration **Debug (Attach)** (Déboguer (Attacher)) dans le menu Déboguer, puis cliquez sur le bouton d’exécution (flèche verte).

   ![Configuration Debug (Attach) (Déboguer (Attacher))](./media/service-fabric-develop-java-applications-with-vs-code/debug-attach-java.png)

3. Dans un navigateur web, accédez à `http://localhost:8080`. Tapez un nouvel élément dans la zone de texte et cliquez sur **+ Ajouter**. Votre point d’arrêt doit être atteint. Vous pouvez utiliser la barre d’outils de débogage en haut de VS Code pour poursuivre l’exécution, sauter des lignes, effectuer un pas à pas détaillé dans des méthodes ou sortir de la méthode actuelle. 
   
   ![Atteindre le point d’arrêt](./media/service-fabric-develop-java-applications-with-vs-code/breakpoint-hit.png)
       
4. Pour mettre fin à la session de débogage, cliquez sur l’icône en forme de prise dans la barre d’outils de débogage en haut de VS Code.
   
   ![Se déconnecter du débogueur](./media/service-fabric-develop-java-applications-with-vs-code/debug-bar-disconnect.png)
       
5. Lorsque vous avez terminé le débogage, vous pouvez utiliser la commande **Service Fabric : Supprimer l'application** pour supprimer l'application Voting de votre cluster local. 

## <a name="next-steps"></a>Étapes suivantes

* Découvrez comment [développer et déboguer des applications Service Fabric en C# avec VS Code](./service-fabric-develop-csharp-applications-with-vs-code.md).
