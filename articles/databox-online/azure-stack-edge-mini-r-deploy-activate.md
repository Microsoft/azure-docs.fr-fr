---
title: Tutoriel pour chiffrer et activer un appareil Azure Stack Edge Mini R dans le portail Azure | Microsoft Docs
description: Le tutoriel pour déployer Azure Stack Edge Mini R vous apprend à chiffrer et activer votre appareil physique.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 10/22/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to activate Azure Stack Edge Mini R so I can use it to transfer data to Azure.
ms.openlocfilehash: e5076888e692ad16e89f30d455e7f8c4e22a41b9
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96464306"
---
# <a name="tutorial-activate-azure-stack-edge-mini-r"></a>Tutoriel : Activer Azure Stack Edge Mini R

Ce tutoriel explique comment activer votre appareil Azure Stack Edge Mini R en utilisant l’interface utilisateur web locale.

Le processus d’activation peut prendre 15 minutes environ.

Ce didacticiel vous à présenté les point suivants :

> [!div class="checklist"]
> * Prérequis
> * Activer l’appareil physique

## <a name="prerequisites"></a>Prérequis

Avant de configurer et d’activer votre appareil Azure Stack Edge Mini R, vérifiez que :

* Pour votre appareil physique : 
    
    - Vous avez installé l’appareil physique, comme indiqué dans [Installer Azure Stack Edge Mini R](azure-stack-edge-mini-r-deploy-install.md).
    - Vous avez configuré le réseau et les paramètres réseau de calcul comme détaillé dans [Configurer le réseau, le réseau de calcul et le proxy Web](azure-stack-edge-mini-r-deploy-configure-network-compute-web-proxy.md)
    - Vous avez chargé vos propres certificats sur votre appareil sur votre appareil si vous avez modifié le nom de l’appareil ou le domaine DNS par le biais de la page **Appareil**. Ces étapes sont détaillées dans [Configurer des certificats, un VPN et un chiffrement au repos](azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption.md). Si vous n’avez pas effectué cette étape, l’activation sera bloquée.
    - Vous avez configuré le chiffrement au repos pour votre appareil. Si vous n’avez pas effectué cette étape, vous verrez une erreur pendant l’activation de l’appareil, et l’activation sera bloquée. Pour plus d’informations, consultez [Configurer des certificats, un VPN et un chiffrement au repos](azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption.md).
    
* Vous disposez de la clé d’activation du service Azure Stack Edge que vous avez créé pour gérer l’appareil Azure Stack Edge Mini R. Pour plus d’informations, consultez [Préparer le déploiement d’Azure Stack Edge Mini R](azure-stack-edge-mini-r-deploy-prep.md).


## <a name="activate-the-device"></a>Activer l’appareil

1. Dans l’interface utilisateur Web locale de l’appareil, accédez à la page **Bien démarrer**.
2. Dans la vignette **Activation**, sélectionnez **Activer**. 

    ![Page « Détails du cloud » 1 de l’interface utilisateur web locale](./media/azure-stack-edge-mini-r-deploy-activate/activation-1.png)
    
3. Dans le volet **Activer** :
    1. Entrez la **clé d’activation** que vous avez obtenue à l’étape [Obtenir la clé d’activation pour Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-prep.md#get-the-activation-key).

    1. Vous pouvez activer une collecte de journaux proactive pour permettre à Microsoft de collecter les journaux en fonction de l’état d’intégrité de l’appareil. Les journaux collectés de cette façon sont chargés sur un compte de stockage Azure.
    
    1. Sélectionnez **Appliquer**.

    ![Page « Détails du cloud » de l’interface utilisateur web locale 2](./media/azure-stack-edge-mini-r-deploy-activate/activation-2.png)


5. Tout d’abord, l’appareil est activé. Vous êtes ensuite invité à télécharger le fichier de clé.
    
    ![Page « Détails du cloud » de l’interface utilisateur web locale 3](./media/azure-stack-edge-mini-r-deploy-activate/activation-3.png)
    
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
      "SEDEncryptionExternalKey": "xZM/vC7GxjqHZ3VMo7xSs/wH9rRsF/PNM+mOsZ+GaL0=",
      "ServiceEncryptionKey": "<Azure service encryption key>"
    }
    ```        
 
    Le tableau suivant décrit les différentes clés :
    
    |Champ  |Description  |
    |---------|---------|
    |`Id`    | Il s’agit de l’ID de l’appareil.        |
    |`DataVolumeBitLockerExternalKeys`|Il s’agit des clés BitLocker pour les disques de données. Elles sont utilisées pour récupérer les données locales sur votre appareil.|
    |`SystemVolumeBitLockerRecoveryKey`| Il s’agit de la clé BitLocker pour le volume système. Cette clé aide à la récupération de la configuration système et des données système pour votre appareil. |
    |`SEDEncryptionExternalKey`| Cette clé fournie par l’utilisateur ou générée par le système est utilisée pour protéger les lecteurs de données à chiffrement automatique dont le chiffrement est prédéfini. |
    |`ServiceEncryptionKey`| Cette clé protège les données transitant par le service Azure. Cette clé garantit la sécurité des informations stockées en cas de compromission du service Azure. |

6. Allez à la page **Vue d’ensemble**. L’état de l’appareil doit s’afficher comme étant **Activé**.

    ![Page « Détails du cloud » de l’interface utilisateur web locale 4](./media/azure-stack-edge-mini-r-deploy-activate/activation-4.png)
 
L’activation de l’appareil est terminée. Vous pouvez maintenant ajouter des partages sur votre appareil.

Si vous rencontrez des problèmes lors de l’activation, consultez [Résoudre les problèmes avec l’activation et Azure Key Vault](azure-stack-edge-gpu-troubleshoot-activation.md#activation-errors).

## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel vous à présenté les point suivants :

> [!div class="checklist"]
> * Prérequis
> * Activer l’appareil physique

Pour apprendre à transférer des données avec votre appareil Azure Stack Edge Mini R, consultez :

> [!div class="nextstepaction"]
> [Transférer des données avec Azure Stack Edge Mini R](./azure-stack-edge-j-series-deploy-add-shares.md)
