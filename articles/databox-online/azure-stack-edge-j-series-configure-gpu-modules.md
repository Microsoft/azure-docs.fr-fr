---
title: Exécuter un module GPU sur un appareil GPU Microsoft Azure Stack Edge Pro | Microsoft Docs
description: Décrit comment configurer et exécuter un module GPU sur un appareil Azure Stack Edge Pro via le portail Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/25/2020
ms.author: alkohli
ms.openlocfilehash: 8b9f1180639f638e72fdea2f87958628a2e9e86b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90891463"
---
# <a name="configure-and-run-a-module-on-gpu-on-azure-stack-edge-pro-device"></a>Configurer et exécuter un module GPU sur un appareil Azure Stack Edge Pro

Votre appareil Azure Stack Edge Pro contient une ou plusieurs unités de traitement graphique (GPU, Graphics Processing Unit). Les GPU sont souvent choisis pour les calculs IA, car ils offrent des capacités de traitement parallèle et sont plus rapides au rendu d’image que les processeurs (ou CPU). Pour plus d’informations sur le GPU de votre appareil Azure Stack Edge Pro, accédez aux [spécifications techniques des appareils Azure Stack Edge Pro](azure-stack-edge-gpu-technical-specifications-compliance.md).

Cet article explique comment configurer et exécuter un module sur le GPU de votre appareil Azure Stack Edge Pro. Dans cet article, vous allez utiliser un module de conteneur disponible publiquement, **Digits**, écrit pour les GPU Nvidia T4. Cette procédure peut être utilisée pour configurer tout autre module publié par Nvidia pour ces GPU.


## <a name="prerequisites"></a>Prérequis

Avant de commencer, assurez-vous que :

1. Vous avez accès à un appareil Azure Stack Edge Pro à 1 nœud compatible avec GPU. Cet appareil est activé avec une ressource dans Azure.  

## <a name="configure-module-to-use-gpu"></a>Configurer le module pour utiliser le GPU

Pour configurer un module afin qu’il utilise le GPU sur votre appareil Azure Stack Edge Pro, procédez comme suit.

1. Dans le portail Azure, accédez à la ressource associée à votre appareil. 

2. Accédez à **Computing en périphérie > Bien démarrer**. Dans la vignette **Configurer le computing en périphérie**, sélectionnez Configurer.

    ![Configurer le module pour utiliser le GPU 1](media/azure-stack-edge-j-series-configure-gpu-modules/configure-compute-1.png)

3. Dans le panneau **Configurer le computing en périphérie** :

    1. Pour **IoT Hub**, choisissez **Créer**.
    2. Fournissez un nom pour la ressource IoT Hub à créer pour votre appareil. Pour utiliser un niveau gratuit, sélectionnez une ressource existante. 
    3. Prenez note de l’appareil IoT Edge et de l’appareil de la passerelle IoT qui sont créés avec la ressource IoT Hub. Vous allez utiliser ces informations dans les étapes ultérieures.

    ![Configurer le module pour utiliser le GPU 2](media/azure-stack-edge-j-series-configure-gpu-modules/configure-compute-2.png)

4. La création de la ressource IoT Hub prend plusieurs minutes. Une fois la ressource créée, dans la vignette **Configurer le computing en périphérie**, sélectionnez **Configuration de la vue** pour voir les détails de la ressource IoT Hub.

    ![Configurer le module pour utiliser le GPU 4](media/azure-stack-edge-j-series-configure-gpu-modules/configure-compute-4.png)

5. Accédez à **Gestion automatique des appareils > IoT Edge**.

    ![Configurer le module pour utiliser le GPU 6](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-2.png)

    Dans le volet droit, vous voyez l’appareil IoT Edge associé à votre appareil Azure Stack Edge Pro. Celui-ci correspond à l’appareil IoT Edge que vous avez créé à l’étape précédente en créant la ressource IoT Hub. 
    
6. Sélectionnez cet appareil IoT Edge.

   ![Configurer le module pour utiliser le GPU 7](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-3.png)

7.  Sélectionnez **Définir des modules**.

    ![Configurer le module pour utiliser le GPU 8](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-4.png)

8. Sélectionnez **+ Ajouter**, puis **+ Module IoT Edge**. 

    ![Configurer le module pour utiliser le GPU 9](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-5.png)

9. Sous l’onglet **Ajouter un module IoT Edge** :

    1. Fournissez l’**URI de l’image**. Vous allez utiliser le module Nvidia publiquement disponible **Digits** ici. 
    
    2. Définissez **Stratégie de redémarrage** sur **toujours**.
    
    3. Définissez **État souhaité** sur **en cours d’exécution**.
    
    ![Configurer le module pour utiliser le GPU 10](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-6.png)

10. Sous l’onglet **Variables d’environnement**, indiquez le nom de la variable et la valeur correspondante. 

    1. Pour que le module actuel utilise un seul GPU sur cet appareil, utilisez NVIDIA_VISIBLE_DEVICES. 

    2. Définissez cette condition sur la valeur 0 ou 1. Cela garantit qu’au moins un GPU est utilisé par l’appareil pour ce module. Quand vous définissez la valeur sur 0, 1, cela signifie que ces deux GPU sur votre appareil sont utilisés par ce module.

        ![Configurer le module pour utiliser le GPU 11](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-7.png)

        Pour plus d’informations sur les variables d’environnement que vous pouvez utiliser avec le GPU Nvidia, accédez au [runtime de conteneur nVidia](https://github.com/NVIDIA/nvidia-container-runtime#environment-variables-oci-spec).

    > [!NOTE]
    > Un GPU ne peut être mappé qu’à un seul module. Un module peut toutefois utiliser un seul GPU, deux GPU, voire aucun. 

11. Entrez un nom pour votre module. À ce stade, vous pouvez choisir de fournir une option de création de conteneur et de modifier les paramètres de jumeau de module ou, si vous avez terminé, sélectionnez **Ajouter**. 

    ![Configurer le module pour utiliser le GPU 12](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-8.png)

12. Vérifiez que le module est en cours d’exécution et sélectionnez **Vérifier + créer**.    

    ![Configurer le module pour utiliser le GPU 13](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-9.png)

13. Sous l’onglet **Vérifier + créer**, les options de déploiement que vous avez sélectionnées s’affichent. Vérifiez les options et sélectionnez **Créer**.
    
    ![Configurer le module pour utiliser le GPU 14](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-10.png)

14. Prenez note de l’**état du runtime** du module. 
    
    ![Configurer le module pour utiliser le GPU 15](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-11.png)

    Le déploiement du module prend quelques minutes. Sélectionnez **Actualiser** pour voir l’**état du runtime** passer à **en cours d’exécution**.

    ![Configurer le module pour utiliser le GPU 16](media/azure-stack-edge-j-series-configure-gpu-modules/configure-gpu-12.png)


## <a name="next-steps"></a>Étapes suivantes

- Découvrez plus en détail les [variables d’environnement](https://github.com/NVIDIA/nvidia-container-runtime#environment-variables-oci-spec) que vous pouvez utiliser avec le GPU Nvidia.
