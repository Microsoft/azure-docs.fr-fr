---
title: Tutoriel pour installer un appareil physique Azure Stack Edge Mini R | Microsoft Docs
description: Le deuxième tutoriel sur l’installation d’un appareil Azure Stack Edge Mini R implique le raccordement de l’appareil physique à une source d’alimentation et au réseau.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 10/20/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to install Azure Stack Edge Mini R device in datacenter so I can use it to transfer data to Azure.
ms.openlocfilehash: 4f7656337b12cf477c5c71d861d031919e0d55d6
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96464228"
---
# <a name="tutorial-install-azure-stack-edge-mini-r"></a>Tutoriel : Installer Azure Stack Edge Mini R

Ce tutoriel explique comment installer un appareil physique Azure Stack Edge Mini R. La procédure d’installation implique de brancher l’appareil.

L’installation peut prendre environ 30 minutes.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Inspecter l’appareil
> * Brancher l’appareil

## <a name="prerequisites"></a>Prérequis

Voici les prérequis pour l’installation d’un appareil physique :

### <a name="for-the-azure-stack-edge-resource"></a>Pour la ressource Azure Stack Edge

Avant de commencer, assurez-vous que :

* Vous avez terminé toutes les étapes décrites dans [Préparer le déploiement d’Azure Stack Edge Mini R](azure-stack-edge-mini-r-deploy-prep.md).
    * Vous avez créé une ressource Azure Stack Edge pour déployer votre appareil.
    * Vous avez créé la clé d’activation pour activer votre appareil auprès de la ressource Azure Stack Edge.

 
### <a name="for-the-azure-stack-edge-mini-r-physical-device"></a>Pour l’appareil physique Azure Stack Edge Mini R

Avant de déployer un appareil :

- Vérifiez que l’appareil est posé en toute sécurité sur une surface de travail plate, stable et de niveau.
- Vérifiez que le lieu prévu pour l’installation dispose des éléments suivants :
    - Alimentation secteur standard provenant d’une source indépendante OU
    - Une unité de distribution d’alimentation (PDU) pour rack. 
    

### <a name="for-the-network-in-the-datacenter"></a>Pour le réseau dans le centre de données

Avant de commencer :

- Passez en revue les conditions réseau requises pour déployer Azure Stack Edge Mini R, puis configurez le réseau du centre de données conformément à ces conditions. Pour plus d’informations, consultez [Configuration réseau pour Azure Stack Edge](azure-stack-edge-mini-r-system-requirements.md#networking-port-requirements).

- Assurez-vous que la bande passante Internet minimale est de 20 Mbits/s pour un fonctionnement optimal de l’appareil. <!-- engg TBC -->


## <a name="inspect-the-device"></a>Inspecter l’appareil

Cet appareil est livré en un seul morceau. Procédez comme suit pour déballer votre appareil.

1. Placez la boîte sur une surface plane et droite.
2. Inspectez la boîte de l’appareil pour voir si elle est abîmée. Ouvrez la boîte et inspectez l’appareil. Si la boîte ou l’appareil ont l’air endommagé, contactez le support Microsoft pour vous aider à déterminer si l’appareil est en bon état de fonctionnement.
3. Une fois la boîte ouverte, vérifiez que vous disposez des éléments suivants :
    - Un appareil Azure Stack Edge Mini R portable avec des protections latérales en place
    - Une batterie et le capot arrière fixé sur l’appareil 
    - Un cordon d’alimentation pour brancher la batterie à la source d’alimentation 

Si vous n’avez pas reçu tous les éléments énumérés ici, contactez le support technique Azure Stack Edge. L’étape suivante consiste à brancher votre appareil.


## <a name="cable-the-device"></a>Brancher l’appareil

Les procédures suivantes expliquent comment effectuer le câblage d’alimentation et de réseau de votre appareil Azure Stack Edge.

Avant de commencer à câbler votre appareil, vous avez besoin des éléments suivants :

- Votre appareil physique Azure Stack Edge Mini R sur le site d’installation.
- Un câble d’alimentation.
- Au moins un câble réseau 1-GbE RJ-45 à brancher à l’interface de gestion. Il existe deux interfaces de réseau 1-GbE, une interface de gestion, et une interface de données, sur l’appareil.
- Un câble en cuivre 10-GbE SFP+ pour chaque interface réseau de données à configurer. Au moins une interface réseau de données entre le PORT 3 et le PORT 4 a besoin d’être connectée à Internet (avec connectivité à Azure).  
- Accès à une seule unité de distribution d’alimentation (recommandé).

> [!NOTE]
> - Si vous ne connectez qu’une seule interface réseau de données, nous vous recommandons d’utiliser une interface réseau 10-GbE telle que PORT 3 ou PORT 4 pour envoyer des données à Azure. 
> - Pour de meilleures performances et pour gérer d’importants volumes de données, pensez à connecter tous les ports de données.
> - L’appareil Azure Stack Edge doit être connecté au réseau du centre de données pour pouvoir ingérer les informations provenant des serveurs des sources de données. <!-- engg TBC -->

Sur votre appareil Azure Stack Edge :

- Le panneau avant a un support SSD. 

    - L’appareil a 1 disque SSD dans l’emplacement. 
    - L’appareil dispose aussi d’une carte CFx qui sert de stockage pour le disque du système d’exploitation.
    
- Le panneau avant a des interfaces réseau et accès au Wi-Fi.

    - 2 interfaces réseau 1 GbE RJ 45. Il s’agit de PORT 1 et PORT 2 sur l’interface utilisateur locale de l’appareil.
    - 2 interfaces réseau 10 GbE SFP+. Il s’agit de PORT 3 et PORT 4 sur l’interface utilisateur locale de l’appareil. 
    - Un seul port Wi-Fi avec un transmetteur Wi-Fi branché.

- Le panneau avant comporte aussi un bouton d’alimentation. 

- Le panneau arrière comprend une batterie et un capot déjà installés sur l’appareil. 


Effectuez les étapes suivantes pour brancher l’alimentation et le réseau de votre appareil.

1. Identifiez les différents composants réseau et stockage sur la face avant de votre appareil.

    ![Interfaces réseau et de stockage sur l’appareil](./media/azure-stack-edge-mini-r-deploy-install/ports-front-plane.png)

2. Repérez le bouton d’alimentation en bas à gauche sur l’avant de l’appareil. 

    ![Face avant d’un appareil avec un bouton d’alimentation sur l’appareil](./media/azure-stack-edge-mini-r-deploy-install/device-power-button.png)

3. La batterie est raccordée au fond de panier de votre appareil. Repérez le second bouton d’alimentation situé sur la batterie. 

    ![Face avant d’un appareil avec un bouton d’alimentation sur la batterie](./media/azure-stack-edge-mini-r-deploy-install/battery-power-button.png)


    Branchez une extrémité du cordon d’alimentation à la batterie et l’autre à la prise d’alimentation. 

    ![Raccordement du cordon d’alimentation](./media/azure-stack-edge-mini-r-deploy-install/power-cord-connector-1.png) 

    En cas de fonctionnement uniquement sur batterie (la batterie n’est pas branchée sur la source d’alimentation), l’interrupteur situé à l’avant et celui de la batterie doivent être placés en position ON. Quand la batterie est branchée sur une source d’alimentation, seul le bouton d’alimentation situé à l’avant de l’appareil doit être placé en position ON. 

4. Appuyez sur le bouton d’alimentation situé sur la face avant pour mettre l’appareil sous tension. 
    
    > [!NOTE]
    > Pour mettre l’appareil sous ou hors tension, vous devez enfoncer le bouton noir situé au dessus du bouton d’alimentation, puis mettre le bouton d’alimentation en position ON ou OFF. 

5. Si vous configurez le Wi-Fi sur cet appareil, utilisez le schéma de câblage suivant :

    ![Câblage pour le Wi-Fi](./media/azure-stack-edge-mini-r-deploy-install/wireless-cabled.png)  

    - Connectez l’interface réseau 1-GbE PORT 1 à l’ordinateur utilisé pour configurer l’appareil physique. Le PORT 1 constitue l’interface de la gestion dédiée.


    Si vous utilisez une configuration filaire pour cet appareil, utilisez le schéma suivant :
     
    ![Câblage pour une configuration filaire](./media/azure-stack-edge-mini-r-deploy-install/wired-cabled.png)     
    - Connectez l’interface réseau 1-GbE PORT 1 à l’ordinateur utilisé pour configurer l’appareil physique. Le PORT 1 constitue l’interface de la gestion dédiée.
    - Connectez un ou plusieurs interfaces PORT 2, PORT 3 ou PORT 4 au réseau du centre de données/à Internet.
    
        - Si vous connectez le PORT 2, utilisez le câble réseau RJ-45.
        - Pour les interfaces réseau 10-GbE, utilisez des câbles en cuivre SFP+.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez approfondi vos connaissances sur Azure Stack Edge et avez notamment appris à :

> [!div class="checklist"]
> * Déballez l’appareil
> * Câblage de l’appareil

Passez au didacticiel suivant pour savoir comment connecter, configurer et activer votre appareil.

> [!div class="nextstepaction"]
> [Connecter et configurer Azure Stack Edge](./azure-stack-edge-mini-r-deploy-connect.md)
