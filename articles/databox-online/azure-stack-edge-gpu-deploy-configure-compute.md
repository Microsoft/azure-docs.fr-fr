---
title: Tutoriel expliquant comment filtrer et analyser des données avec le rôle de calcul sur Azure Stack Edge Pro avec GPU | Microsoft Docs
description: Apprenez à configurer le rôle de calcul sur Azure Stack Edge Pro avec GPU et à l'utiliser pour transformer des données avant de les envoyer à Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 08/28/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to configure compute on Azure Stack Edge Pro so I can use it to transform the data before sending it to Azure.
ms.openlocfilehash: 711da24b3edf08f4867109d0d70165955236c39a
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184652"
---
# <a name="tutorial-configure-compute-on-azure-stack-edge-pro-gpu-device"></a>Tutoriel : Configurer le calcul sur un appareil Azure Stack Edge Pro avec GPU

<!--[!INCLUDE [applies-to-skus](../../includes/azure-stack-edge-applies-to-all-sku.md)]-->

Ce tutoriel vous explique comment configurer un rôle de calcul et créer un cluster Kubernetes sur votre appareil Azure Stack Edge Pro. 

Cette procédure peut prendre environ 20 à 30 minutes.


Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer le calcul
> * Obtenir les points de terminaison Kubernetes

 
## <a name="prerequisites"></a>Prérequis

Avant de configurer un rôle de calcul sur votre appareil Azure Stack Edge Pro, vérifiez que :

- Vous avez activé votre appareil Azure Stack Edge Pro, comme décrit dans [Activer Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-activate.md).
- Assurez-vous que vous avez suivi les instructions dans [Activer le réseau de calcul](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md#enable-compute-network) et :
    - Vous avez activé une interface réseau pour le calcul.
    - Affecté des adresses IP de nœud Kubernetes et adresses IP de service externe Kubernetes.

## <a name="configure-compute"></a>Configurer le calcul

Pour configurer le calcul sur votre appareil Azure Stack Edge Pro, vous allez créer une ressource IoT Hub via le portail Azure.

1. Dans le portail Azure de votre ressource Azure Stack Edge, accédez à **Vue d’ensemble**. Dans le volet droit, sur la vignette **Calculer**, sélectionnez **Bien démarrer**.

    ![Bien démarrer avec le calcul](./media/azure-stack-edge-j-series-deploy-configure-compute/configure-compute-1.png)

2. Sur la vignette **Configurer le computing en périphérie**, sélectionnez **Configurer le calcul**.

    ![Configurer le calcul](./media/azure-stack-edge-j-series-deploy-configure-compute/configure-compute-2.png)

3. Sur le panneau **Configurer le computing en périphérie**, entrez ce qui suit :

   
    |Champ  |Valeur  |
    |---------|---------|
    |IoT Hub     | Choisissez **Nouveau** ou **Existant**. <br> Par défaut, un niveau Standard (S1) est utilisé pour créer une ressource IoT. Pour utiliser une ressource IoT de niveau gratuit, créez-en une, puis sélectionnez-la. <br> Dans chaque cas, la ressource IoT Hub utilise les mêmes abonnement et groupe de ressources que la ressource Azure Stack Edge.     |
    |Nom     |Entrez un nom pour votre ressource IoT Hub.         |

    ![Bien démarrer avec le calcul 2](./media/azure-stack-edge-j-series-deploy-configure-compute/configure-compute-3.png)

4. Sélectionnez **Create** (Créer). La création de ressources IoT Hub prend plusieurs minutes. Une fois la ressource IoT Hub créée, la vignette **Configurer le calcul** est mise à jour pour afficher la configuration du calcul. 

    ![Bien démarrer avec le calcul 3](./media/azure-stack-edge-j-series-deploy-configure-compute/configure-compute-4.png)

5. Pour vérifier que le rôle de computing en périphérie a été configuré, sélectionnez **Voir le computing** sur la vignette **Configurer le calcul**.
    
    ![Bien démarrer avec le calcul 4](./media/azure-stack-edge-j-series-deploy-configure-compute/configure-compute-5.png)

    > [!NOTE]
    > Si la boîte de dialogue **Configurer le calcul** est fermée avant que l'instance d'IoT Hub soit associée à l'appareil Azure Stack Edge Pro, l'instance d'IoT Hub est créée mais elle n'apparaît pas dans la configuration du calcul. 
    
Quand le rôle de calcul Edge est configuré sur l’appareil Edge, il crée deux appareils : un appareil IoT et un appareil IoT Edge. Ces deux appareils peuvent être visualisés dans la ressource IoT Hub. Un runtime IoT Edge est également exécuté sur cet appareil IoT Edge. À ce stade, seule la plateforme Linux est disponible pour votre appareil IoT Edge.

La configuration du calcul en arrière-plan peut prendre 20-30 minutes, le temps que les machines virtuelles et le cluster Kubernetes soient créés. 

Une fois le calcul correctement configuré sur le portail Azure, un cluster Kubernetes et un utilisateur par défaut associé à l'espace de noms IoT (espace de noms système contrôlé par Azure Stack Edge Pro) sont disponibles. 

## <a name="get-kubernetes-endpoints"></a>Obtenir les points de terminaison Kubernetes

Pour configurer un client pour accéder au cluster Kubernetes, vous avez besoin du point de terminaison Kubernetes. Procédez comme suit pour récupérer le point de terminaison de l'API Kubernetes à partir de l'interface utilisateur locale de votre appareil Azure Stack Edge Pro.

1. Dans l’interface utilisateur web locale de votre appareil, accédez à la page **Appareils**.
2. Sous **Points de terminaison de l’appareil**, copiez le point de terminaison du **service d’API Kubernetes**. Ce point de terminaison est une chaîne au format suivant : `https://compute.<device-name>.<DNS-domain>[Kubernetes-cluster-IP-address]`. 

    ![Page Appareil dans l’interface utilisateur locale](./media/azure-stack-edge-j-series-create-kubernetes-cluster/device-kubernetes-endpoint-1.png)

3. Enregistrez la chaîne de point de terminaison. Vous allez l’utiliser ultérieurement lors de la configuration d’un client pour accéder au cluster Kubernetes via kubectl.

4. Lorsque vous êtes dans l’interface utilisateur web locale, vous pouvez :

    - Accéder à l’API Kubernetes, sélectionner **Paramètres avancés** et télécharger un fichier de configuration avancée pour Kubernetes. 

        ![Page Appareil dans l’interface utilisateur locale 1](./media/azure-stack-edge-gpu-deploy-configure-compute/download-advanced-config-1.png)

        Si vous avez reçu une clé auprès de Microsoft (certains utilisateurs peuvent en avoir une), vous pouvez utiliser ce fichier de configuration.

        ![Page Appareil dans l’interface utilisateur locale 2](./media/azure-stack-edge-gpu-deploy-configure-compute/download-advanced-config-2.png)

    - Vous pouvez également accéder au point de terminaison **Tableau de bord Kubernetes** et télécharger un fichier de configuration `aseuser`. 
    
        ![Page Appareil dans l’interface utilisateur locale 3](./media/azure-stack-edge-gpu-deploy-configure-compute/download-aseuser-config-1.png)

        Vous pouvez utiliser ce fichier de configuration pour vous connecter au tableau de bord Kubernetes ou pour déboguer les problèmes dans votre cluster Kubernetes. Pour plus d’informations, consultez [Accéder au tableau de bord Kubernetes](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md#access-dashboard). 


## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Configurer le calcul
> * Obtenir les points de terminaison Kubernetes


Pour savoir comment administrer votre appareil Azure Stack Edge Pro, consultez :

> [!div class="nextstepaction"]
> [Administrer un appareil Azure Stack Edge Pro via l'interface utilisateur web locale](azure-stack-edge-manage-access-power-connectivity-mode.md)
