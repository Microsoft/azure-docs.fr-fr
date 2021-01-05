---
title: Déployer un module GPU sur votre appareil Microsoft Azure Stack Edge Pro à partir de la Place de marché Azure | Microsoft Docs
description: Explique comment déployer un module IoT compatible GPU sur votre appareil Azure Stack Edge Pro avec GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 09/09/2020
ms.author: alkohli
ms.openlocfilehash: 64d028892298a70e7588863bf9a3f4fc6f4ca609
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91760057"
---
# <a name="deploy-a-gpu-enabled-iot-module-from-azure-marketplace-on-azure-stack-edge-pro-gpu-device"></a>Déployer un module IoT compatible GPU à partir de la Place de marché Azure sur un appareil Azure Stack Edge Pro avec GPU

Cet article explique comment déployer un module IoT Edge compatible GPU (Graphics Processing Unit) à partir de la Place de marché Azure sur votre appareil Azure Stack Edge Pro. 

Dans cet article, vous apprendrez comment :
  - Préparer Azure Stack Edge Pro pour exécuter un module GPU.
  - Téléchargez et déployez le module IoT compatible GPU à partir de la Place de marché Azure.
  - Supervisez la sortie du module.

## <a name="about-sample-module"></a>À propos de l’exemple de module

L’exemple de module GPU mentionné dans cet article comprend un exemple de code d’évaluation PyTorch et TensorFlow pour comparer processeur et GPU.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que vous disposez  des éléments suivants :

- Vous avez accès à un appareil Azure Stack Edge à 1 nœud compatible avec un GPU. Cet appareil est activé avec une ressource dans Azure. 
- Vous avez configuré le calcul sur cet appareil. Suivez les étapes décrites dans [Tutoriel : Configurer le calcul sur votre appareil Azure Stack Edge](azure-stack-edge-gpu-deploy-configure-compute.md).
- Les ressources de développement suivantes sur un client Windows :
    - [Visual Studio Code](https://code.visualstudio.com/)  
    - [Extension Azure IoT Edge pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-edge).   


## <a name="get-module-from-azure-marketplace"></a>Obtenir le module à partir de la Place de marché Azure

1. Parcourez toutes les [applications dans la Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps).

    ![Parcourir les applications dans la Place de marché Azure](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/browse-apps-marketplace-1.png)

2. Recherchez **Bien démarrer avec les GPU**.

    ![Rechercher l’exemple de module GPU](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/search-gpu-sample-module-1.png)

3. Sélectionnez **Obtenir maintenant**.

    ![Obtenir l’exemple de module](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/get-sample-module-1.png)

4. Sélectionnez  **Continuer** pour accepter les conditions d’utilisation et la politique de confidentialité du fournisseur. 

    ![Obtenir l’exemple de module 2](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/terms-of-use-1.png)

5. Choisissez l’abonnement que vous avez utilisé pour déployer votre appareil Azure Stack Edge Pro.

    ![Sélectionner un abonnement](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/select-subscription-1.png)

6. Entrez le nom du service IoT Hub que vous avez créé lors de la configuration de votre appareil Azure Stack Edge Pro. Pour trouver ce nom de service IoT Hub, accédez à la ressource Azure Stack Edge associée à votre appareil dans le portail Azure. 

    1. Dans les options de menu du volet gauche, accédez à **Computing en périphérie > Commencer**. 

    1. Dans la vignette **Configurer le computing en périphérie**, sélectionnez **Configuration de la vue**. 

        ![Afficher la configuration de calcul](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/view-config-1.png)

    1. Dans le panneau **Configuration du computing en périphérie** :

        1. Prenez note du service IoT Hub qui a été créé au moment de la configuration du calcul sur votre appareil Azure Stack Edge Pro.
        2. Notez le nom de l’appareil IoT Edge qui a été créé lorsque vous avez configuré le calcul. Vous allez utiliser ce nom à l’étape suivante.

        ![Configuration du computing en périphérie](media/azure-stack-edge-gpu-deploy-sample-module/view-compute-config-1.png)

10. Choisissez **Déployer sur un appareil**.

11. Entrez le nom de l’appareil IoT Edge ou sélectionnez  **Rechercher un appareil** pour parcourir les appareils inscrits auprès du hub.

    ![Rechercher un appareil](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/find-device-1.png)

12. Sélectionnez  **Créer** pour continuer le processus standard de la configuration d’un manifeste de déploiement, y compris l’ajout d’autres modules si vous le souhaitez. Les détails du nouveau module, tels que l’URI de l’image, les options de création et les propriétés souhaitées, sont prédéfinis mais peuvent être modifiés.

    ![Sélectionner Créer](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/target-devices-iot-edge-module-1.png)


13. Vérifiez que le module est déployé dans votre IoT Hub dans le Portail Azure. Sélectionnez votre appareil, sélectionnez  **Définir des modules** et le module doit être listé dans la section  **Modules IoT Edge**. 

    ![Sélectionner Créer 2](media/azure-stack-edge-gpu-deploy-sample-module-marketplace/running-module-iotres-1.png)

## <a name="monitor-the-module"></a>Superviser le module  

1. Dans la palette de commandes VS Code, exécutez **Azure IoT Hub: Select IoT Hub** (Azure IoT Hub : Sélectionner l’IoT Hub).

2. Choisissez l’abonnement et l’IoT Hub qui contiennent l’appareil IoT Edge que vous souhaitez configurer. Dans ce cas, sélectionnez l’abonnement utilisé pour déployer l’appareil Azure Stack Edge Pro, puis sélectionnez l’appareil IoT Edge créé pour votre appareil Azure Stack Edge Pro. Cela se produit lorsque vous avez configuré le calcul via le portail Azure dans les étapes précédentes.

3. Dans l’Explorateur VS Code, développez la section Appareils Azure IoT Hub. Sous **Appareils**, vous devez voir l’appareil IoT Edge correspondant à votre appareil Azure Stack Edge Pro. 

    1. Sélectionnez cet appareil, cliquez dessus avec le bouton droit, puis sélectionnez **Démarrer la supervision du point de terminaison d’événement intégré**.
  
        ![Démarrer la surveillance](media/azure-stack-edge-gpu-deploy-sample-module/monitor-builtin-event-endpoint-1.png)  

    2. Accédez à **Appareils > Modules**. Vous devriez voir votre **module GPU** en cours d’exécution.

    3. Le terminal VS Code doit également afficher les événements IoT Hub en tant que sortie de supervision pour votre appareil Azure Stack Edge Pro.

        ![Sortie de supervision](media/azure-stack-edge-gpu-deploy-sample-module/monitor-events-output-1.png) 

        Vous pouvez voir que le temps nécessaire pour exécuter le même ensemble d’opérations (5 000 itérations de transformation de forme) par GPU est sensiblement inférieur à celui dont a besoin un processeur.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez en détail comment [configurer le GPU pour utiliser un module](azure-stack-edge-j-series-configure-gpu-modules.md).
