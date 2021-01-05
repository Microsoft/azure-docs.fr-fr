---
title: Tutoriel pour connecter, configurer et activer l’appareil Azure Stack Edge Pro avec GPU dans le portail Azure | Microsoft Docs
description: Découvrez comment se connecter à votre appareil Azure Stack Edge avec GPU embarqué en utilisant l’interface utilisateur web locale.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 08/29/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to connect and activate Azure Stack Edge Pro so I can use it to transfer data to Azure.
ms.openlocfilehash: f069f711ccf5672041ea6f3b15a52fa2b3ba2022
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90900038"
---
# <a name="tutorial-connect-to-azure-stack-edge-pro-with-gpu"></a>Tutoriel : Se connecter à Azure Stack Edge Pro avec GPU

Ce tutoriel explique comment se connecter à votre appareil Azure Stack Edge Pro avec GPU embarqué en utilisant l’interface utilisateur web locale.

Le processus de connexion peut prendre environ 5 minutes.

Ce tutoriel vous fournira des informations sur :

> [!div class="checklist"]
>
> * Prérequis
> * Se connecter à un appareil physique


## <a name="prerequisites"></a>Prérequis

Avant de configurer et d’activer votre appareil Azure Stack Edge Pro avec GPU, vérifiez que :

* Vous avez installé l’appareil physique, comme indiqué dans [Installer Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-install.md).


## <a name="connect-to-the-local-web-ui-setup"></a>Se connecter à la configuration de l’interface utilisateur web locale

1. Configurez l’adaptateur Ethernet sur votre ordinateur pour vous connecter à l’appareil Azure Stack Edge Pro avec l’adresse IP statique 192.168.100.5 et le sous-réseau 255.255.255.0.

2. Connectez l’ordinateur au PORT 1 de votre appareil. Si vous connectez l’ordinateur directement à l’appareil (sans commutateur), utilisez un câble croisé ou une carte Ethernet USB. Utilisez l’illustration ci-dessous pour identifier le PORT 1 sur votre appareil.

    ![Fond de panier d’un appareil câblé](./media/azure-stack-edge-gpu-deploy-install/ase-two-pci-slots.png)

    Le fond de panier de l’appareil peut différer légèrement selon le modèle exact que vous avez reçu. Pour plus d’informations, consultez [Raccordement de l’appareil](azure-stack-edge-gpu-deploy-install.md#cable-the-device).


3. Ouvrez une fenêtre de navigateur et accédez à l’interface utilisateur web locale de l’appareil à l’adresse `https://192.168.100.10`.  
    Cette action peut prendre quelques minutes après la mise sous tension de l’appareil.

    Une erreur ou un avertissement vous indique que le certificat de sécurité du site web présente un problème. 
   
    ![Message erreur de certificat de sécurité du site web](./media/azure-stack-edge-deploy-connect-setup-activate/image2.png)

4. Sélectionnez **Poursuivre sur cette page web**.  
    Ces étapes peuvent varier en fonction du navigateur que vous utilisez.

5. Connectez-vous à l’interface utilisateur web de votre appareil physique. Le mot de passe par défaut est *Password1*. 
   
    ![Page de connexion de l’appareil Azure Stack Edge Pro](./media/azure-stack-edge-deploy-connect-setup-activate/image3.png)

6. À l’invite, modifiez le mot de passe d’administrateur de l’appareil.  
    Le nouveau mot de passe doit contenir entre 8 et 16 caractères. Il doit contenir trois des types de caractères suivants : majuscule, minuscule, chiffre et caractère spécial.

Vous êtes maintenant sur la page **Vue d’ensemble** de votre appareil. L’étape suivante consiste à configurer les paramètres réseau de votre appareil.


## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel vous à présenté les point suivants :

> [!div class="checklist"]
> * Prérequis
> * Se connecter à un appareil physique


Pour savoir comment configurer les paramètres réseau sur votre appareil Azure Stack Edge Pro, consultez :

> [!div class="nextstepaction"]
> [Configurer le réseau](./azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md)
