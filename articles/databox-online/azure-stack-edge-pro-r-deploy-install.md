---
title: Tutoriel pour installer un appareil physique Azure Stack Edge Pro R | Microsoft Docs
description: Le deuxième tutoriel sur l’installation d’un appareil Azure Stack Edge Pro R implique le raccordement de l’appareil physique à une source d’alimentation et au réseau.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 10/18/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to install Azure Stack Edge Pro R in datacenter so I can use it to transfer data to Azure.
ms.openlocfilehash: b67da2607d206424f69f53645eda148495ea58ec
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96464000"
---
# <a name="tutorial-install-azure-stack-edge-pro-r"></a>Tutoriel : Installer Azure Stack Edge Pro R

Ce tutoriel explique comment installer un appareil physique Azure Stack Edge Pro R. La procédure d’installation implique de brancher l’appareil.

L’installation peut prendre environ 30 minutes.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Inspecter l’appareil
> * Câblage de l’appareil

## <a name="prerequisites"></a>Prérequis

Voici les prérequis pour l’installation d’un appareil physique :

### <a name="for-the-azure-stack-edge-resource"></a>Pour la ressource Azure Stack Edge

Avant de commencer, assurez-vous que :

* Vous avez terminé toutes les étapes décrites dans [Préparer le déploiement d’Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-prep.md).
    * Vous avez créé une ressource Azure Stack Edge pour déployer votre appareil.
    * Vous avez créé la clé d’activation pour activer votre appareil auprès de la ressource Azure Stack Edge.

 
### <a name="for-the-azure-stack-edge-pro-r-physical-device"></a>Pour l’appareil physique Azure Stack Edge Pro R

Avant de déployer un appareil :

- Vérifiez que l’appareil est posé en toute sécurité sur une surface de travail plate, stable et de niveau.
- Vérifiez que le lieu prévu pour l’installation dispose des éléments suivants :
    - Alimentation secteur standard provenant d’une source indépendante

        OU
    - Une unité de distribution d’alimentation (PDU) pour rack. L’appareil est livré avec un onduleur.
    

### <a name="for-the-network-in-the-datacenter"></a>Pour le réseau dans le centre de données

Avant de commencer :

- Passez en revue les conditions réseau requises pour déployer Azure Stack Edge Pro R, puis configurez le réseau du centre de données conformément à ces conditions. Pour plus d’informations, consultez [Conditions réseau requises pour Azure Stack Edge Pro R](azure-stack-edge-pro-r-system-requirements.md#networking-port-requirements).

- Assurez-vous que la bande passante Internet minimale est de 20 Mbits/s pour un fonctionnement optimal de l’appareil.


## <a name="inspect-the-device"></a>Inspecter l’appareil

Cet appareil est livré en un seul morceau. Procédez comme suit pour déballer votre appareil.

1. Placez la boîte sur une surface plane et droite.
2. Inspectez la boîte de l’appareil pour voir si elle est abîmée. Ouvrez la boîte et inspectez l’appareil. Si la boîte ou l’appareil ont l’air endommagé, contactez le support Microsoft pour vous aider à déterminer si l’appareil est en bon état de fonctionnement.
3. Une fois la boîte ouverte, vérifiez que vous disposez des éléments suivants :
    - Un appareil Azure Stack Edge Pro R dans son boîtier
    - Un onduleur
    - 2 câbles d’alimentation courts pour connecter l’appareil à l’onduleur
    - 1 câble d’alimentation pour connecter l’onduleur à la source d’alimentation

Si vous n’avez pas reçu tous les éléments listés ici, contactez le support Azure Stack Edge Pro R. L’étape suivante consiste à brancher votre appareil.


## <a name="cable-the-device"></a>Câblage de l’appareil

Les procédures suivantes expliquent comment raccorder votre appareil Azure Stack Edge Pro R à la source d’alimentation et au réseau.

Avant de commencer à câbler votre appareil, vous avez besoin des éléments suivants :

- Votre appareil physique Azure Stack Edge Pro R sur le site d’installation.
- Un câble d’alimentation.
- Au moins un câble réseau 1-GbE RJ-45 à brancher à l’interface de gestion. Il existe deux interfaces de réseau 1-GbE, une interface de gestion, et une interface de données, sur l’appareil.
- Un câble en cuivre 10/25-GbE SFP+ pour chaque interface réseau de données à configurer. Au moins une interface réseau de données entre le PORT 3 et le PORT 4 a besoin d’être connectée à Internet (avec connectivité à Azure).  
- Accès à une seule unité de distribution d’alimentation (recommandé).

> [!NOTE]
> - Si vous ne connectez qu’une seule interface réseau de données, nous vous recommandons d’utiliser une interface réseau 25/10-GbE telle que PORT 3 ou PORT 4 pour envoyer des données à Azure. 
> - Pour de meilleures performances et pour gérer d’importants volumes de données, pensez à connecter tous les ports de données.
> - L’appareil Azure Stack Edge Pro R doit être connecté au réseau du centre de données pour pouvoir ingérer les informations provenant des serveurs des sources de données.

Sur votre appareil Azure Stack Edge Pro R :

- Le panneau avant comprend des lecteurs de disque et un bouton d’alimentation.

    - 8 emplacements de disque se trouvent à l’avant de votre appareil.
    - L’appareil inclut également 2 disques SATA M.2 qui servent de disques de système d’exploitation. 

- Le fond de panier de l’appareil comprend des unités d’alimentation (PSU) redondantes à l’arrière.
- Le fond de panier comprend quatre interfaces réseau :

    - Deux interfaces de 1 Gbit/s
    - Deux interfaces de 25 Gbits/s qui peuvent également faire office d’interfaces de 10 Gbits/s.
    - Un contrôleur de gestion de la carte de base (BMC).

<!--- The back plane has two network cards corresponding to the 4 ports:

    - QLogic FastLinQ 41264
    - QLogic FastLinQ 41262

For a full list of supported cables, switches, and transceivers for these network cards, go to [Cavium FastlinQ 41000 Series Interoperability Matrix](https://www.marvell.com/documents/xalflardzafh32cfvi0z/).-->
 
Effectuez les étapes suivantes pour brancher l’alimentation et le réseau de votre appareil.

1. Identifiez les différents ports sur le fond de panier de votre appareil.

    ![Fond de panier d’un appareil câblé](./media/azure-stack-edge-pro-r-deploy-install/backplane-cabled.png)

2. Localisez les emplacements de disque et le bouton d’alimentation à l’avant de l’appareil.

    ![Plan avant d’un appareil](./media/azure-stack-edge-pro-r-deploy-install/device-front-plane-labeled-1.png)

3. Branchez l’une des extrémités du cordon d’alimentation à l’onduleur. Branchez l’autre extrémité du cordon d’alimentation à l’unité de distribution d’alimentation pour rack. 
5. Appuyez sur le bouton d’alimentation pour allumer l’appareil.
6. Connectez l’interface réseau 1-GbE PORT 1 à l’ordinateur utilisé pour configurer l’appareil physique. Le PORT 1 constitue l’interface de la gestion dédiée.
7. Connectez un ou plusieurs interfaces PORT 2, PORT 3 ou PORT 4 au réseau du centre de données/à Internet.

    - Si vous connectez le PORT 2, utilisez le câble réseau RJ-45.
    - Pour les interfaces de réseau 10/25-GbE, utilisez les câbles en cuivre SFP+.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez approfondi vos connaissances sur Azure Stack Edge Pro R et avez notamment appris à :

> [!div class="checklist"]
> * Déballer l’appareil
> * Brancher l’appareil

Passez au tutoriel suivant pour savoir comment vous connecter à votre appareil.

> [!div class="nextstepaction"]
> [Se connecter à Azure Stack Edge Pro R](./azure-stack-edge-pro-r-deploy-connect.md)
