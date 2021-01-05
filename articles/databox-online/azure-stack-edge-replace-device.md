---
title: Remplacer votre appareil Azure Stack Edge Pro | Microsoft Docs
description: Décrit comment obtenir un appareil Azure Stack Edge Pro de remplacement.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 07/20/2020
ms.author: alkohli
ms.openlocfilehash: ec16a2b42b818e96399b8fdbad4a0951f84ef825
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90893893"
---
# <a name="replace-your-azure-stack-edge-pro-device"></a>Remplacer votre appareil Azure Stack Edge Pro

Cet article explique comment obtenir un appareil Azure Stack Edge Pro de remplacement. Un appareil de remplacement est nécessaire lorsque l’appareil existant présente une défaillance matérielle ou a besoin d’une mise à niveau. 


Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
>
> * Ouvrir un ticket de support de support pour problème matériel
> * Créer une ressource pour l’appareil de remplacement dans le portail Azure
> * Installer et activer l’appareil de remplacement
> * Retourner l’appareil d’origine

## <a name="open-a-support-ticket"></a>Ouverture d’un ticket de support

Si votre appareil existant présente une défaillance matérielle, ouvrez un ticket de support. Le support Microsoft déterminera qu’une unité de remplacement de champ (FRU) n’est pas disponible pour cette instance ou que l’appareil a besoin d’une mise à niveau matérielle. Dans les deux cas, le support commandera un appareil de remplacement.

1. Ouvrez un ticket de support auprès du Support Microsoft indiquant que vous souhaitez retourner l’appareil. Sélectionnez le type de problème **Matériel Azure Stack Edge Pro**.

    ![Ouverture d’un ticket de support](media/azure-stack-edge-replace-device/open-support-ticket-1.png)  

2. Un ingénieur du Support Microsoft vous contactera. Fournissez les informations d’expédition.
<!--3. If you need a return shipping box, you can request it. Answer **Yes** to the question **Need an empty box to return**.-->


## <a name="create-a-resource-for-replacement-device"></a>Créer une ressource pour l’appareil de remplacement

Pour créer une ressource, effectuez les étapes suivantes.

1. Suivez les étapes décrites dans [Créer une ressource](azure-stack-edge-deploy-prep.md#create-a-new-resource) pour créer une ressource pour l’appareil de remplacement. 

2. Veillez à activer la case à cocher **Je possède un appareil Azure Stack Edge Pro**. 

    ![Ressource pour l’appareil de remplacement](media/azure-stack-edge-replace-device/replace-resource-1.png)  

## <a name="install-and-activate-the-replacement-device"></a>Installer et activer l’appareil de remplacement

Pour installer et activer l’appareil de remplacement, procédez comme suit :

1. [Installez votre appareil](azure-stack-edge-deploy-install.md).

2. [Activez votre appareil](azure-stack-edge-deploy-connect-setup-activate.md) par rapport à la nouvelle ressource que vous avez créée précédemment.

## <a name="return-your-existing-device"></a>Retourner votre appareil existant

Suivez toutes les étapes pour retourner l’appareil d’origine :

1. [Effacez les données sur l’appareil](azure-stack-edge-return-device.md#erase-data-from-the-device).
2. [Initiez le retour de l’appareil](azure-stack-edge-return-device.md#initiate-device-return) pour l’appareil d’origine.
3. [Planifiez un enlèvement](azure-stack-edge-return-device.md#schedule-a-pickup).
4. [Supprimez la ressource](azure-stack-edge-return-device.md#delete-the-resource) associée à l’appareil retourné.


## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [retourner un appareil Azure Stack Edge Pro](azure-stack-edge-return-device.md).
