---
title: Tutoriel pour activer un appareil Azure Stack Edge Pro R dans le portail Azure
description: Le tutoriel sur le déploiement d’Azure Stack Edge Pro R vous apprend à activer votre appareil physique.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 10/19/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to activate Azure Stack Edge Pro R device so I can use it to transfer data to Azure.
ms.openlocfilehash: c1408c25ea7e5d805d17e084899b0ffeb6cfec28
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96463904"
---
# <a name="tutorial-activate-azure-stack-edge-pro-r-device"></a>Tutoriel : Activer un appareil Azure Stack Edge Pro R

Ce tutoriel explique comment activer votre appareil Azure Stack Edge Pro R en utilisant l’interface utilisateur web locale.

Le processus d’activation peut prendre 5 minutes environ.

Ce didacticiel vous à présenté les point suivants :

> [!div class="checklist"]
> * Prérequis
> * Activer l’appareil physique

## <a name="prerequisites"></a>Prérequis

Avant de configurer et d’activer votre appareil Azure Stack Edge Pro R, vérifiez que :

* Pour votre appareil physique : 
    
    - Vous avez installé l’appareil physique, comme indiqué dans [Installer Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-install.md).
    - Vous avez configuré le réseau et les paramètres réseau de calcul comme détaillé dans [Configurer le réseau, le réseau de calcul et le proxy Web](azure-stack-edge-pro-r-deploy-configure-network-compute-web-proxy.md)
    - Vous avez téléchargé votre propre certificat ou généré les certificats de l’appareil sur votre appareil si vous avez modifié le nom de l’appareil ou le domaine DNS via la page **Appareil**. Si vous n’avez pas effectué cette étape, vous verrez une erreur pendant l’activation de l’appareil, et l’activation sera bloquée. Pour plus d’informations, consultez la page [Configurer les certificats](azure-stack-edge-placeholder.md).
    
* Vous disposez de la clé d’activation du service Azure Stack Edge que vous avez créé pour gérer l’appareil Azure Stack Edge Pro R. Pour plus d’informations, consultez [Préparer le déploiement d’Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-prep.md).


## <a name="activate-the-device"></a>Activer l’appareil

1. Dans l’interface utilisateur Web locale de l’appareil, accédez à la page **Bien démarrer**.
2. Dans la vignette **Activation**, sélectionnez **Activer**. 

    ![Page « Détails du cloud » 1 de l’interface utilisateur web locale](./media/azure-stack-edge-pro-r-deploy-activate/activate-1.png)
    
3. Dans le volet **Activer** :
    1. Entrez la **clé d’activation** que vous avez obtenue à l’étape [Obtenir la clé d’activation pour Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-prep.md#get-the-activation-key).

    1. Vous pouvez activer une collecte de journaux proactive pour permettre à Microsoft de collecter les journaux en fonction de l’état d’intégrité de l’appareil. Les journaux collectés de cette façon sont chargés sur un compte de stockage Azure.
    
    1. Sélectionnez **Appliquer**.

    ![Page « Détails du cloud » de l’interface utilisateur web locale 2](./media/azure-stack-edge-pro-r-deploy-activate/activate-2.png)


5. Tout d’abord, l’appareil est activé. Vous êtes ensuite invité à télécharger le fichier de clé.
    
    ![Page « Détails du cloud » de l’interface utilisateur web locale 3](./media/azure-stack-edge-pro-r-deploy-activate/activate-3.png)
    
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

Pour apprendre à transférer des données avec votre appareil Azure Stack Edge Pro R, consultez :

> [!div class="nextstepaction"]
> [Transférer des données avec Azure Stack Edge Pro R](./azure-stack-edge-j-series-deploy-add-shares.md)
