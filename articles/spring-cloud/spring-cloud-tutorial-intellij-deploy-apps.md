---
title: 'Tutoriel : Utiliser IntelliJ pour déployer des applications Azure Spring Cloud'
description: Utilisez IntelliJ pour déployer des applications sur Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 03/26/2020
ms.custom: devx-track-java
ms.openlocfilehash: 91edbbda43365b17b8276572804ba788dbee9f13
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92426726"
---
# <a name="use-intellij-to-deploy-azure-spring-cloud-applications"></a>Utiliser IntelliJ pour déployer des applications Azure Spring Cloud

**Cet article s’applique à :** ✔️ Java

Le plug-in IntelliJ pour Azure Spring Cloud prend en charge le déploiement d’applications à partir d’IntelliJ IDEA.  

Avant d’exécuter cet exemple, vous pouvez essayer le [démarrage rapide de base](spring-cloud-quickstart.md).

## <a name="prerequisites"></a>Prérequis
* [JDK 8 Azul Zulu](/java/azure/jdk/java-jdk-install?preserve-view=true&view=azure-java-stable)
* [Maven 3.5.0 (ou version supérieure)](https://maven.apache.org/download.cgi)
* [IntelliJ IDEA, édition Community/Ultimate, version 2020.1/2020.2](https://www.jetbrains.com/idea/download/#section=windows)

## <a name="install-the-plug-in"></a>Installer le plug-in
Vous pouvez ajouter Azure Toolkit for IntelliJ IDEA 3.43.0 à partir de l’interface utilisateur **Plugins** d’IntelliJ.

1. Démarrez IntelliJ.  Si un projet est déjà ouvert, fermez-le pour afficher la boîte de dialogue d’accueil. Sélectionnez **Configure** en bas à droite, cliquez sur **Plugins** pour ouvrir la boîte de dialogue de configuration de plug-ins, puis sélectionnez **Install Plugins from disk** (Installer des plug-ins à partir du disque).

    ![Sélectionnez Configurer](media/spring-cloud-intellij-howto/configure-plugin-1.png)

1. Recherchez Azure Toolkit for IntelliJ.  Cliquez sur **Installer**.

    ![Installer le plug-in](media/spring-cloud-intellij-howto/install-plugin.png)

1. Cliquez sur **Restart IDE** (Redémarrer l’IDE).

## <a name="tutorial-procedures"></a>Procédures du tutoriel
Les procédures suivantes permettent de déployer une application Hello World à l’aide d’IntelliJ IDEA.

* Ouvrir le projet gs-spring-boot
* Déployer sur Azure Spring Cloud
* Afficher les journaux de streaming

## <a name="open-gs-spring-boot-project"></a>Ouvrir le projet gs-spring-boot

1. Téléchargez et décompressez le dépôt source pour ce tutoriel, ou clonez-le à l’aide de Git: git clone https://github.com/spring-guides/gs-spring-boot.git 
1. Accédez avec la commande cd à gs-spring-boot\complete.
1. Ouvrez la boîte de dialogue **Welcome** d’IntelliJ et sélectionnez **Import Project** pour ouvrir l’Assistant Importation.
1. Sélectionnez le dossier `gs-spring-boot\complete`.

    ![Importer un projet](media/spring-cloud-intellij-howto/import-project-1.png)

## <a name="deploy-to-azure-spring-cloud"></a>Déployer sur Azure Spring Cloud
Pour déployer sur Azure, vous devez vous connecter avec votre compte Azure et choisir votre abonnement.  Pour plus d’informations sur la connexion, consultez [Installation et connexion](/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in).

1. Cliquez avec le bouton droit sur votre projet dans l’explorateur de projets IntelliJ, puis sélectionnez **Azure** -> **Deploy to Azure Spring Cloud**.

    ![Déployer sur Azure 1](media/spring-cloud-intellij-howto/deploy-to-azure-1.png)

1. Acceptez le nom de l’application dans le champ **Name**. **Name** fait référence à la configuration, pas au nom de l’application. Les utilisateurs n’ont généralement pas besoin de le changer.
1. Acceptez l’identificateur du projet pour l’ **Artifact**.
1. Sélectionnez **App:** , puis cliquez sur **Create app...** (Créer une application).

    ![Déployer sur Azure 2](media/spring-cloud-intellij-howto/deploy-to-azure-2.png)

1. Entrez le nom de l’application dans le champ **App name** , puis cliquez sur **OK**.

    ![Déployer sur Azure OK](media/spring-cloud-intellij-howto/deploy-to-azure-2a.png)

1. Démarrez le déploiement en cliquant sur le bouton **Run** (Exécuter). 

    ![Déployer sur Azure 3](media/spring-cloud-intellij-howto/deploy-to-azure-3.png)

1. Le plug-in exécutera la commande `mvn package` sur le projet, puis créera la nouvelle application et déploiera le fichier jar généré par la commande `package`.

1. Si l’URL de l’application n’est pas affichée dans la fenêtre de sortie, récupérez-la à partir du portail Azure. Accédez à l’instance Azure Spring Cloud à partir de votre groupe de ressources.  Cliquez ensuite sur **Applications**.  L’application en cours d’exécution sera listée.

    ![Obtenir l’URL de test](media/spring-cloud-intellij-howto/get-test-url.png)

1. Accédez à l’URL dans un navigateur.

    ![Naviguer dans le navigateur 2](media/spring-cloud-intellij-howto/navigate-in-browser-2.png)

## <a name="show-streaming-logs"></a>Afficher les journaux de streaming
Pour obtenir les journaux
1. Sélectionnez **Azure Explorer** , puis **Spring Cloud**.
1. Cliquez avec le bouton droit sur l’application en cours d’exécution.
1. Sélectionnez **Journaux de streaming** dans la liste déroulante.

    ![Sélectionnez Journaux de streaming](media/spring-cloud-intellij-howto/streaming-logs.png)

1. Sélectionnez une instance.

    ![Sélectionnez une instance](media/spring-cloud-intellij-howto/select-instance.png)

1. Le journal de streaming est visible dans la fenêtre de sortie.

    ![Sortie de journal de streaming](media/spring-cloud-intellij-howto/streaming-log-output.png)

## <a name="next-steps"></a>Étapes suivantes
* [Préparer une application Spring pour Azure Spring Cloud](./spring-cloud-tutorial-prepare-app-deployment.md)
* [En savoir plus sur Azure Toolkit for IntelliJ](/azure/developer/java/toolkit-for-intellij/)
