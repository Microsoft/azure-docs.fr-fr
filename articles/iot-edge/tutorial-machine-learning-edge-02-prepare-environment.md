---
title: 'Tutoriel : Configurer un environnement - Machine Learning sur Azure IoT Edge'
description: 'Tutoriel : Préparez votre environnement pour le développement et le déploiement de modules pour l’apprentissage automatique à la périphérie.'
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 3/12/2020
ms.topic: tutorial
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: d86a273a69203a92a9b437f021486feb3bcb31da
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96932368"
---
# <a name="tutorial-set-up-an-environment-for-machine-learning-on-iot-edge"></a>Tutoriel : Configurer un environnement - Machine Learning sur Azure IoT Edge

Cet article vous aide à préparer votre environnement pour le développement et le déploiement. Tout d’abord, configurez une machine de développement avec tous les outils dont vous avez besoin. Ensuite, créez les ressources cloud nécessaires dans Azure.

Dans cette section du tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Configurer une machine virtuelle pour le développement.
> * Configurer un hub IoT et un stockage cloud à utiliser dans votre environnement de développement.

## <a name="prerequisites"></a>Prérequis

Cet article fait partie d’une série décrivant l’utilisation d’Azure Machine Learning sur IoT Edge. Chaque article de la série s’appuie sur le travail réalisé dans le cadre de l’article précédent. Si vous êtes tombé directement sur cet article, reportez-vous au [premier article](tutorial-machine-learning-edge-01-intro.md) de la série.

## <a name="set-up-the-development-vm"></a>Configurer la machine virtuelle de développement

Cette étape est généralement effectuée par un développeur cloud. Certains logiciels peuvent également être utiles pour un scientifique des données.

Nous avons créé un script PowerShell qui crée une machine virtuelle Azure avec plusieurs des prérequis déjà configurés. La machine virtuelle que vous créez doit pouvoir gérer la [virtualisation imbriquée](../virtual-machines/windows/nested-virtualization.md), c’est pourquoi nous avons choisi la taille de machine [Standard_D8s_v3](../virtual-machines/dv3-dsv3-series.md).

La machine virtuelle de développement sera mise en place avec :

* Windows 10
* [Chocolatey](https://chocolatey.org/)
* [Docker Desktop pour Windows](https://www.docker.com/products/docker-desktop)
* [Git pour Windows](https://gitforwindows.org/)
* [GIT Credential Manager pour Windows](https://github.com/Microsoft/Git-Credential-Manager-for-Windows)
* [Kit de développement logiciel (SDK) .NET Core](https://dotnet.microsoft.com/)
* [Python 3](https://www.python.org/)
* [Visual Studio Code](https://code.visualstudio.com/)
* [Azure PowerShell](/powershell/azure/)
* [Extensions de VS Code](https://marketplace.visualstudio.com/search?target=VSCode)
  * [Outils IoT Azure](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  * [Python](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  * [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
  * [Docker](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)
  * [PowerShell](https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell)

La machine de développement n’est pas strictement nécessaire - tous les outils de développement peuvent être exécutés sur une machine locale. Cependant, nous recommandons fortement d’utiliser la machine virtuelle pour assurer des règles du jeu équitables.

Il faut environ 30 minutes pour créer et configurer la machine virtuelle.

1. Clonez ou téléchargez l’exemple de dépôt [Machine Learning and IoT Edge](https://github.com/Azure-Samples/IoTEdgeAndMlSample) sur votre ordinateur local.

1. Ouvrez PowerShell en tant qu’administrateur et accédez au répertoire **\IoTEdgeAndMlSample\DevVM** situé sous le répertoire racine dans lequel vous avez téléchargé le code. Nous utiliserons `srcdir` pour faire référence au répertoire racine de votre source.

    ```powershell
    cd c:\srcdir\IoTEdgeAndMlSample\DevVM
    ```

   Le répertoire DevVM contient les fichiers nécessaires à la création d’une machine virtuelle Azure appropriée pour compléter ce tutoriel.

1. Exécutez la commande suivante pour permettre l’exécution des scripts. Cliquez sur **Oui à tout** lorsque vous y êtes invité.

    ```powershell
    Set-ExecutionPolicy Bypass -Scope Process
    ```

1. Exécutez Create-AzureDevVM.ps1.

    ```powershell
    .\Create-AzureDevVm.ps1
    ```

    À l’invite, fournissez les informations suivantes :

    * **ID d’abonnement Azure** : votre ID d’abonnement, disponible dans [Abonnements Azure](https://ms.portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) dans le portail.
    * **Nom du groupe de ressources** : nom d’un groupe de ressources nouveau ou existant dans Azure.
    * **Emplacement** : Choisissez un emplacement Azure où la machine virtuelle sera créée. Par exemple, « USA Ouest 2 » ou « Europe Nord ». Pour plus d’informations, consultez [Emplacements Azure](https://azure.microsoft.com/global-infrastructure/locations/).
    * **Nom d’utilisateur** : entrez un nom facile à retenir pour le compte administrateur de la machine virtuelle.
    * **Mot de passe** : définissez un mot de passe pour le compte administrateur de la machine virtuelle.

   Pendant plusieurs minutes, le script exécute les étapes suivantes :

    1. Installe le [module Azure PowerShell Az](/powershell/azure/new-azureps-module-az).
    1. Vous invite à vous connecter à Azure.
    1. Confirme les informations relatives à la création de votre machine virtuelle. Appuyez sur **y** ou **Entrée** pour continuer.
    1. Crée le groupe de ressources s’il n’existe pas.
    1. Déploie la machine virtuelle.
    1. Active Hyper-V sur la machine virtuelle.
    1. Installe les logiciels nécessaires au développement et clone l’exemple de dépôt.
    1. Redémarre la machine virtuelle.
    1. Crée un fichier RDP sur votre bureau pour se connecter à la machine virtuelle.

   Si vous êtes invité à entrer le nom de la machine virtuelle pour la redémarrer, vous pouvez copier son nom à partir de la sortie du script. La sortie indique également le chemin du fichier RDP pour se connecter à la machine virtuelle.

### <a name="set-auto-shutdown-schedule"></a>Définir la planification d’arrêt automatique

Pour vous aider à réduire les coûts, la machine virtuelle de développement a été créée avec une planification d’arrêt automatique définie à 1900 PST. Vous devrez peut-être mettre à jour ce paramètre en fonction de votre localisation et de votre planification. Pour mettre à jour la planification d’arrêt :

1. Dans le portail Azure, accédez à la machine virtuelle créée par le script.

1. Dans le menu du volet de gauche, sous **Opérations**, sélectionnez **Arrêt automatique**.

1. Ajustez les options **Arrêt planifié** et **Fuseau horaire** selon vos besoins, puis sélectionnez **Enregistrer**.

## <a name="connect-to-the-development-vm"></a>Se connecter à la machine virtuelle de développement

Maintenant que nous avons créé une machine virtuelle, nous devons terminer l’installation du logiciel nécessaire pour compléter le tutoriel.

1. Double-cliquez sur le fichier RDP que le script a créé sur votre bureau.

1. Une boîte de dialogue vous indique que l’éditeur de la connexion distante est inconnu. Comme c’est acceptable, sélectionnez **Se connecter**.

1. Entrez le mot de passe d’administrateur que vous avez fourni pour créer la machine virtuelle, puis cliquez sur **OK**.

1. Une invite vous demande d’accepter le certificat de la machine virtuelle. Sélectionnez **Oui**.

## <a name="install-visual-studio-code-extensions"></a>Installer les extensions Visual Studio Code

Maintenant que vous êtes connecté à la machine de développement, ajoutez quelques extensions utiles à Visual Studio Code pour faciliter l’expérience de développement.

1. Connectez-vous à la machine virtuelle de développement, ouvrez une fenêtre PowerShell et accédez au répertoire **C:\source\IoTEdgeAndMlSample\DevVM**. Ce répertoire a été créé par le script de création de la machine virtuelle.

    ```powershell
    cd C:\source\IoTEdgeAndMlSample\DevVM
    ```

1. Exécutez la commande suivante pour permettre l’exécution des scripts. Cliquez sur **Oui à tout** lorsque vous y êtes invité.

    ```powershell
    Set-ExecutionPolicy Bypass -Scope Process
    ```

1. Exécutez le script des extensions Visual Studio Code.

    ```powershell
    .\Enable-CodeExtensions.ps1
    ```

1. Le script s’exécutera pendant quelques minutes en installant des extensions de code VS :

    * Azure IoT Tools
    * Python
    * C#
    * Docker
    * PowerShell

## <a name="set-up-iot-hub-and-storage"></a>Configurer IoT Hub et le stockage

Ces étapes sont généralement effectuées par un développeur de cloud.

Azure IoT Hub est le cœur de toute application IoT dans la mesure où il gère la communication sécurisée entre les appareils IoT et le cloud. C’est le principal point de coordination pour l’exploitation de la solution d’apprentissage automatique IoT Edge.

* IoT Hub utilise des itinéraires pour diriger les données entrantes des appareils IoT vers d’autres services en aval. Nous allons tirer parti des routes IoT Hub pour envoyer des données d’appareil à Stockage Azure. Dans Stockage Azure, les données d’appareil sont consommées par Azure Machine Learning pour entraîner notre classificateur de durée de vie restante (RUL).

* Plus loin dans le tutoriel, nous utiliserons IoT Hub pour configurer et gérer notre appareil Azure IoT Edge.

Dans cette section, vous utilisez un script pour créer un hub IoT Azure et un compte Stockage Azure. Ensuite, dans le portail Azure, vous configurez une route qui transfère les données reçues par le hub à un conteneur Stockage Azure. Ces étapes prennent environ 10 minutes.

1. Connectez-vous à la machine virtuelle de développement, ouvrez une fenêtre PowerShell et accédez au répertoire **IoTHub**.

    ```powershell
    cd C:\source\IoTEdgeAndMlSample\IoTHub
    ```

1. Exécutez le script de création. Utilisez les mêmes valeurs pour l’ID d’abonnement, la localisation et le groupe de ressources que lors de la création de la machine virtuelle de développement.

    ```powershell
    .\New-HubAndStorage.ps1 -SubscriptionId <subscription id> -Location <location> -ResourceGroupName <resource group>
    ```

    * Vous serez invité à vous connecter à Azure.
    * Le script confirme les informations pour la création de votre compte Hub et Stockage. Appuyez sur **y** ou **Entrée** pour continuer.

L’exécution du script prend deux minutes. Une fois terminé, le script affiche le nom du hub IoT et du compte de stockage.

## <a name="review-route-to-storage-in-iot-hub"></a>Passer en revue l’itinéraire vers le stockage dans IoT Hub

Dans le cadre de la création du hub IoT, le script que nous avons exécuté à la section précédente a également créé un point de terminaison personnalisé et un itinéraire. Les itinéraires IoT Hub se composent d’une expression de requête et d’un point de terminaison. Si un message correspond à l’expression, les données sont envoyées jusqu’au point final associé via l’itinéraire. Les points de terminaison peuvent être des Event Hubs, des files d’attente de Service Bus et des rubriques. Dans ce cas, le point de terminaison est un conteneur d’objets blob dans un compte de stockage. Nous allons utiliser le portail Azure pour passer en revue la route créée par notre script.

1. Ouvrez le [portail Azure](https://portal.azure.com) et accédez au groupe de ressources que vous utilisez pour ce tutoriel.

1. Dans la liste des ressources, sélectionnez le hub IoT que le script a créé. Son nom se termine par des caractères aléatoires tels que `IotEdgeAndMlHub-jrujej6de6i7w`.

1. Dans le menu du volet de gauche, sous **Messagerie**, sélectionnez **Routage des messages**.

1. Dans la page **Routage des messages**, sélectionnez l’onglet **Points de terminaison personnalisés**.

1. Développez la section **Stockage** :

   ![Vérifier que turbofanDeviceStorage se trouve dans la liste des points de terminaison personnalisés](media/tutorial-machine-learning-edge-02-prepare-environment/custom-endpoints.png)

   Nous voyons que **turbofanDeviceStorage** figure dans la liste des points de terminaison personnalisés. Ce point de terminaison présente les caractéristiques suivantes :

   * Il pointe vers le conteneur de stockage d’objets blob que vous avez créé (`devicedata`), comme indiqué par **Nom du conteneur**.
   * Son **Format de nom de fichier** a partition comme dernier élément du nom. Ce format est plus pratique pour les opérations de fichier que nous effectuerons avec Azure Notebooks plus loin dans le tutoriel.
   * Son **État** doit être sain.

1. Sélectionnez l’onglet **Itinéraires**.

1. Sélectionnez l’itinéraire **turbofanDeviceDataToStorage**.

1. Dans la page **Détails de la route**, notez que le point de terminaison de la route est **turbofanDeviceStorage**.

   ![Passer en revue les détails de la route turbofanDeviceDataToStorage](media/tutorial-machine-learning-edge-02-prepare-environment/route-details.png)

1. Examinez la **requête de routage**, qui a la valeur **true**. Ce paramètre signifie que tous les messages de télémétrie de l’appareil correspondront à cette route et que tous les messages seront envoyés au point de terminaison **turbofanDeviceStorage**.

1. Dans la mesure où aucune modification n’a été apportée, fermez cette page.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Ce tutoriel fait partie d’un ensemble où chaque article s’appuie sur le travail effectué dans les articles précédents. Ne nettoyez pas les ressources avant d’avoir terminé le dernier tutoriel.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, nous avons créé un hub IoT et configuré un itinéraire vers un compte Stockage Azure. Nous allons à présent envoyer les données d’un ensemble d’appareils simulés par le biais du hub IoT au compte de stockage. Plus tard dans le tutoriel, après avoir configuré notre appareil et nos modules IoT Edge, nous revisiterons les itinéraires et examinerons un peu plus la requête de routage.

Passez à l’article suivant pour créer un appareil simulé à surveiller.

> [!div class="nextstepaction"]
> [Générer les données d’appareil](tutorial-machine-learning-edge-03-generate-data.md)
