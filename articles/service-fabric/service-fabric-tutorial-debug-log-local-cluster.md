---
title: Déboguer une application Java sur un cluster Service Fabric local
description: Dans ce didacticiel, découvrez comment déboguer et obtenir des journaux d’activité à partir d’une application Java Service Fabric s’exécutant sur un cluster local.
ms.topic: tutorial
ms.date: 02/26/2018
ms.custom: mvc, devx-track-java
ms.openlocfilehash: 801702d43bae6d925fa4f7fbc37cf44bf585fa6d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91531988"
---
# <a name="tutorial-debug-a-java-application-deployed-on-a-local-service-fabric-cluster"></a>Tutoriel : Déboguer une application Java déployée sur un cluster Service Fabric local

Ce tutoriel est le deuxième d’une série. Vous découvrez comment attacher un débogueur distant à l’aide d’Eclipse pour l’application Service Fabric. En outre, vous apprenez à rediriger les journaux des applications en cours d’exécution vers un emplacement pratique pour le développeur.

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * [Générer une application Reliable Services Service Fabric de Java](service-fabric-tutorial-create-java-app.md)
> * Déployer et déboguer l’application sur un cluster local
> * [Déployer l’application sur un cluster Azure](service-fabric-tutorial-java-deploy-azure.md)
> * [Configurer la surveillance et les diagnostics pour l’application](service-fabric-tutorial-java-elk.md)
> * [Configurer CI/CD](service-fabric-tutorial-java-jenkins.md)


Dans ce deuxième volet, vous apprenez à :
> [!div class="checklist"]
> * Déboguer une application Java à l’aide d’Eclipse
> * Rediriger les journaux d’activité vers un emplacement configurable


## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :

* Configurez votre environnement de développement pour [Mac](service-fabric-get-started-mac.md) ou [Linux](service-fabric-get-started-linux.md). Suivez les instructions pour installer le plug-in Eclipse, Gradle, le Kit de développement logiciel (SDK) Service Fabric et l’interface CLI Service Fabric (sfctl).

## <a name="download-the-voting-sample-application"></a>Télécharger l’exemple d’application de vote

Si vous n’avez pas généré l’exemple d’application de vote lors de la [première partie de cette série de didacticiels](service-fabric-tutorial-create-java-app.md), vous pouvez le télécharger. Dans une fenêtre Commande, exécutez la commande suivante pour cloner le référentiel de l’exemple d’application sur votre ordinateur local.

```bash
git clone https://github.com/Azure-Samples/service-fabric-java-quickstart
```

[Générez et déployez](service-fabric-tutorial-create-java-app.md#deploy-application-to-local-cluster) l’application sur le cluster de développement local.

## <a name="debug-java-application-using-eclipse"></a>Déboguer une application Java à l’aide d’Eclipse

1. Ouvrez l’IDE Eclipse sur votre ordinateur, puis cliquez sur **Fichier -> Importer...**

2. Dans la fenêtre contextuelle, sélectionnez l’option **Général ->Existing Projects into Workspace** (Projets existants dans l’espace de travail) et appuyez sur Suivant.

3. Dans la fenêtre d’importation des projets, choisissez l’option **Select root directory** (Sélectionner le répertoire racine), puis sélectionnez le répertoire **Voting**. Si vous avez suivi la série de didacticiels, le répertoire **Voting** se trouve dans le répertoire **Eclipse-workspace**.

4. Mettez à jour entryPoint.sh du service que vous souhaitez déboguer, afin qu’il démarre le processus Java avec les paramètres de débogage à distance. Pour ce didacticiel, le serveur frontal sans état est utilisé : *Voting/VotingApplication/VotingWebPkg/Code/entryPoint.sh*. Le port 8001 est défini pour le débogage dans cet exemple.

    ```bash
    java -Xdebug -Xrunjdwp:transport=dt_socket,address=8001,server=y,suspend=n -Djava.library.path=$LD_LIBRARY_PATH -jar VotingWeb.jar
    ```

5. Mettez à jour le manifeste de l’application en définissant le nombre d’instances ou le nombre de réplicas pour le service en cours de débogage sur 1. Ce paramètre évite les conflits pour le port utilisé pour le débogage. Par exemple, pour les services sans état, définissez ``InstanceCount="1"`` et pour les services avec état, définissez les tailles cible et de jeu de réplicas minimales sur 1 comme suit : ``TargetReplicaSetSize="1" MinReplicaSetSize="1"``.

6. Dans l’IDE Eclipse, sélectionnez **Exécuter-> Déboguer des Configurations-> Application Java distante**, appuyez sur le bouton **Nouveau**, définissez les propriétés comme suit et cliquez sur **Appliquer**.

    ```
    Name: Voting
    Project: Voting
    Connection Type: Standard
    Host: localhost
    Port: 8001
    ```

7. Placez un point d’arrêt sur la ligne 109 du fichier *Voting/VotingWeb/src/statelessservice/HttpCommunicationListener.java*.

8. Dans l’Explorateur de packages, cliquez avec le bouton droit sur le projet **Voting** et cliquez sur **Service Fabric -> Publish Application...** (Publier une application...)

9. Dans la fenêtre **Publish Application** (Publier une application), sélectionnez **Local.json** dans la liste déroulante, puis cliquez sur **Publier**.

10. Dans l’IDE Eclipse, sélectionnez **Exécuter -> Déboguer des Configurations -> Application Java distante**, cliquez sur la configuration **Voting** que vous avez créée et cliquez sur **Déboguer**.

11. Accédez à votre navigateur web puis à **localhost : 8080**. Vous atteignez automatiquement le point d’arrêt et Eclipse ouvre la **perspective Déboguer**.

Vous pouvez maintenant appliquer ces mêmes étapes pour déboguer une application Service Fabric dans Eclipse.

## <a name="redirect-application-logs-to-custom-location"></a>Rediriger les journaux des applications vers un emplacement personnalisé

Les étapes suivantes décrivent comment rediriger les journaux des applications de l’emplacement */var/log/syslog* par défaut vers un emplacement personnalisé.

1. Actuellement, les applications s’exécutant dans les clusters Linux Service Fabric prennent uniquement en charge la sélection d’un seul fichier journal. Pour configurer une application de telle sorte que les journaux aillent toujours à */tmp/mysfapp0.0.log*, créez un fichier nommé logging.properties à l’emplacement *Voting/VotingApplication/VotingWebPkg/Code/logging.properties*, pis ajoutez le contenu ci-après.

    ```
    handlers = java.util.logging.FileHandler

    java.util.logging.FileHandler.level = ALL
    java.util.logging.FileHandler.formatter = java.util.logging.SimpleFormatter

    # This value specifies your custom location.
    # You will have to ensure this path has read and write access by the process running the SF Application
    java.util.logging.FileHandler.pattern = /tmp/mysfapp0.0.log
    ```

2. Ajoutez le paramètre suivant dans *Voting/VotingApplication/VotingWebPkg/Code/entryPoint.sh* pour la commande d’exécution Java :

    ```bash
    -Djava.util.logging.config.file=logging.properties
    ```

    L’exemple suivant montre un exemple d’exécution avec le débogueur attaché, similaire à l’exécution dans la section précédente.

    ```bash
    java -Xdebug -Xrunjdwp:transport=dt_socket,address=8001,server=y,suspend=n -Djava.library.path=$LD_LIBRARY_PATH -Djava.util.logging.config.file=logging.properties -jar VotingWeb.jar
    ```

À ce stade, vous avez appris comment déboguer et accéder aux journaux des applications lors du développement de vos applications Java Service Fabric.

## <a name="next-steps"></a>Étapes suivantes

Dans cette partie du tutoriel, vous avez appris à :

> [!div class="checklist"]
> * Déboguer une application Java à l’aide d’Eclipse
> * Rediriger les journaux d’activité vers un emplacement configurable

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Déployer une application Java Service Fabric sur un cluster dans Azure](service-fabric-tutorial-java-deploy-azure.md)