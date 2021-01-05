---
title: Déployer une application sans état Kubernetes sur un GPU Azure Stack Edge Pro via un module IoT Edge | Microsoft Docs
description: Décrit comment déployer une application sans état Kubernetes sur votre appareil avec GPU Azure Stack Edge Pro à l’aide d’un module IoT Edge accessible via une adresse IP externe.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/26/2020
ms.author: alkohli
ms.openlocfilehash: 4bc598080b96886e6734ac3709761465a1a28d49
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90899519"
---
# <a name="use-iot-edge-module-to-run-a-kubernetes-stateless-application-on-your-azure-stack-edge-pro-gpu-device"></a>Utiliser un module IoT Edge pour exécuter une application sans état Kubernetes sur votre appareil avec GPU Azure Stack Edge Pro

Cet article explique comment vous pouvez utiliser un module IoT Edge pour déployer une application sans état sur votre appareil Azure Stack Edge Pro.

Pour déployer l’application sans état, procédez comme suit :

- Assurez-vous que les conditions préalables sont remplies avant de déployer un module IoT Edge.
- Ajoutez un module IoT Edge pour accéder au réseau de calcul sur Azure Stack Edge Pro.
- Vérifiez que le module peut accéder à l’interface réseau activée.

Dans cet article de procédure, vous allez utiliser un module d’application de serveur web pour illustrer le scénario.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vous avez besoin des éléments suivants :

- Un appareil Azure Stack Edge Pro. Assurez-vous que :

    - Les paramètres réseau de calcul sont configurés sur l’appareil.
    - L’appareil est activé conformément aux étapes décrites dans [Tutoriel : Activer votre appareil](azure-stack-edge-gpu-deploy-activate.md).
- Vous avez terminé l’étape **Configurer le calcul** du [Tutoriel : Configurer le calcul sur votre appareil Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-configure-compute.md) sur votre propre appareil. Votre appareil doit être associé à une ressource IoT Hub, un appareil IoT et un appareil IoT Edge.


## <a name="add-webserver-app-module"></a>Ajouter le module d’application de serveur web

Procédez comme suit pour ajouter un module d’application de serveur web sur votre appareil Azure Stack Edge Pro.

1. Dans la ressource IoT Hub associée à votre appareil, accédez à **Gestion automatique des appareils > IoT Edge**.
1. Sélectionnez l’appareil IoT Edge associé à votre appareil Azure Stack Edge Pro en cliquant dessus. 

    ![Sélectionner un appareil IoT Edge](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/select-iot-edge-device-1.png)  

1. Sélectionnez **Définir des modules**. Dans **Définir des modules sur l’appareil**, sélectionnez **+ Ajouter**, puis **Module IoT Edge**.

    ![Sélectionner un module IoT Edge](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/select-iot-edge-module-1.png)

1. Dans **Ajouter un module IoT Edge** :

    1. Spécifiez un **nom** pour le module d’application de serveur web que vous souhaitez déployer.
    2. Sous l’onglet **Paramètres du module**, indiquez un **URI d’image** pour votre image de module. Un module de mise en correspondance du nom et des balises est récupéré. Dans ce cas, `nginx:stable` extrait une image nginx stable (marquée comme stable) à partir du [référentiel Docker](https://hub.docker.com/_/nginx/) public.

        ![Ajouter un module IoT Edge](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/set-module-settings-1.png)    

    3. Dans l’onglet **Options de création de conteneur**, collez l’exemple de code suivant :  

        ```
        {
            "HostConfig": {
                "PortBindings": {
                    "80/tcp": [
                        {
                            "HostPort": "8080"
                        }
                    ]
                }
            }
        }
        ```

        Cette configuration vous permet d’accéder au module à l’aide de l’adresse IP réseau de calcul sur *http* sur le port TCP 8080 (avec le port de serveur web par défaut 80). Sélectionnez **Ajouter**.

        ![Spécifier les informations de port dans le panneau de module personnalisé IoT Edge](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/verify-module-status-1.png)

    4. Sélectionnez **Revoir + créer**. Passez en revue les détails du module, puis sélectionnez **Créer**.

## <a name="verify-module-access"></a>Vérifier l’accès au module

1. Vérifiez que le module est correctement déployé et qu’il est en cours d’exécution. Dans l’onglet **Modules**, l’état du runtime du module doit être **en cours d’exécution**.  

    ![Vérifier que l’état du module est en cours d’exécution](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/verify-module-status-1.png)

1. Pour accéder au point de terminaison externe de l’application de serveur web, [accédez au tableau de bord Kubernetes](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md#access-dashboard). 
1. Dans le volet gauche du tableau de bord, filtrez par espace de noms **iotedge**. Accédez à **Découverte et équilibrage de charge > Services**. Dans la liste des services répertoriés, localisez le point de terminaison externe pour le module d’application de serveur web. 

    ![Se connecter à l’application de serveur web au niveau du point de terminaison externe](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/connect-external-endpoint-1.png)

1. Sélectionnez le point de terminaison externe pour ouvrir une nouvelle fenêtre de navigateur.

    Vous devez voir que l’application de serveur web est en cours d’exécution.

    ![Vérifier la connexion au module sur le port spécifié](media/azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module/verify-webserver-app-1.png)

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment exposer une application avec état via un module IoT Edge<!--insert link-->.
