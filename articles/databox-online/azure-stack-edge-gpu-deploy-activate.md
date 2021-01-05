---
title: Tutoriel pour activer l’appareil Azure Stack Edge Pro avec GPU dans le portail Azure | Microsoft Docs
description: Le tutoriel sur le déploiement d’un GPU Azure Stack Edge Pro vous apprend à activer votre appareil physique.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 10/07/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to activate Azure Stack Edge Pro so I can use it to transfer data to Azure.
ms.openlocfilehash: a767f6f877ae22a311afdb00417efb9f28fdc8f0
ms.sourcegitcommit: 541bb46e38ce21829a056da880c1619954678586
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/11/2020
ms.locfileid: "91938352"
---
# <a name="tutorial-activate-azure-stack-edge-pro-with-gpu"></a>Tutoriel : Activer Azure Stack Edge Pro avec GPU

Ce tutoriel explique comment activer votre appareil Azure Stack Edge Pro avec GPU embarqué en utilisant l’interface utilisateur web locale.

Le processus d’activation peut prendre 5 minutes environ.

Ce didacticiel vous à présenté les point suivants :

> [!div class="checklist"]
> * Prérequis
> * Activer l’appareil physique

## <a name="prerequisites"></a>Prérequis

Avant de configurer et d’activer votre appareil Azure Stack Edge Pro avec GPU, vérifiez que :

* Pour votre appareil physique : 
    
    - Vous avez installé l’appareil physique, comme indiqué dans [Installer Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-install.md).
    - Vous avez configuré le réseau et les paramètres réseau de calcul comme détaillé dans [Configurer le réseau, le réseau de calcul et le proxy Web](azure-stack-edge-gpu-deploy-configure-network-compute-web-proxy.md)
    - Vous avez téléchargé votre propre certificat ou généré les certificats de l’appareil sur votre appareil si vous avez modifié le nom de l’appareil ou le domaine DNS via la page **Appareil**. Si vous n’avez pas effectué cette étape, vous verrez une erreur pendant l’activation de l’appareil, et l’activation sera bloquée. Pour plus d’informations, consultez la page [Configurer les certificats](azure-stack-edge-gpu-deploy-configure-certificates.md).
    
* Vous disposez de la clé d’activation provenant du service Azure Stack Edge que vous avez créé pour gérer l’appareil Azure Stack Edge Pro. Pour plus d’informations, consultez [Préparer le déploiement d’Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md).


## <a name="activate-the-device"></a>Activer l’appareil

1. Dans l’interface utilisateur Web locale de l’appareil, accédez à la page **Bien démarrer**.
2. Dans la vignette **Activation**, sélectionnez **Activer**. 

    ![Page « Détails du cloud » de l’interface utilisateur web locale](./media/azure-stack-edge-gpu-deploy-activate/activate-1.png)
    
3. Dans le volet **Activer**, entrez la **Clé d’activation** que vous avez obtenue dans [Obtenir la clé d’activation pour Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md#get-the-activation-key).

4. Sélectionnez **Appliquer**.

    ![Page « Détails du cloud » de l’interface utilisateur web locale 2](./media/azure-stack-edge-gpu-deploy-activate/activate-2.png)


5. Tout d’abord, l’appareil est activé. Vous êtes ensuite invité à télécharger le fichier de clé.
    
    ![Page « Détails du cloud » de l’interface utilisateur web locale 3](./media/azure-stack-edge-gpu-deploy-activate/activate-3.png)
    
    Sélectionnez **Télécharger et continuer** et enregistrez le fichier *device-serial-no.json* dans un emplacement sûr en dehors de l’appareil. **Ce fichier de clé contient les clés de récupération pour le disque du système d’exploitation et les disques de données sur votre appareil**. Ces clés peuvent être nécessaires pour récupérer le système à une date ultérieure.

    Voici le contenu du fichier *json* :

        
    ```json
    {
      "Id": "<Device ID>",
      "DataVolumeBitLockerExternalKeys": {
        "hcsinternal": "<BitLocker key for data disk>",
        "hcsdata": "<BitLocker key for data disk>"
      },
      "SystemVolumeBitLockerRecoveryKey": "<BitLocker key for system volume>",
      "ServiceEncryptionKey": "<Azure service encryption key>"
    }
    ```
        
 
    Le tableau suivant décrit les différentes clés :
    
    |Champ  |Description  |
    |---------|---------|
    |`Id`    | Il s’agit de l’ID de l’appareil.        |
    |`DataVolumeBitLockerExternalKeys`|Il s’agit des clés BitLocker pour les disques de données ; elles sont utilisées pour récupérer les données locales sur votre appareil.|
    |`SystemVolumeBitLockerRecoveryKey`| Il s’agit de la clé BitLocker pour le volume système. Cette clé aide à la récupération de la configuration système et des données système pour votre appareil. |
    |`ServiceEncryptionKey`| Cette clé protège les données transitant par le service Azure. Cette clé garantit la sécurité des informations stockées en cas de compromission du service Azure. |

6. Allez à la page **Vue d’ensemble**. L’état de l’appareil doit s’afficher comme étant **Activé**.

    ![Page « Détails du cloud » de l’interface utilisateur web locale 4](./media/azure-stack-edge-gpu-deploy-activate/activate-4.png)
 
L’activation de l’appareil est terminée. Vous pouvez maintenant ajouter des partages sur votre appareil.

Si vous rencontrez des problèmes lors de l’activation, consultez [Résoudre les problèmes avec l’activation et Azure Key Vault](azure-stack-edge-gpu-troubleshoot-activation.md#activation-errors).

## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel vous à présenté les point suivants :

> [!div class="checklist"]
> * Prérequis
> * Activer l’appareil physique

Pour savoir comment transférer des données avec votre appareil Azure Stack Edge Pro, consultez :

> [!div class="nextstepaction"]
> [Transférer des données avec Azure Stack Edge Pro](./azure-stack-edge-j-series-deploy-add-shares.md)
