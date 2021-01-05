---
title: Tutoriel pour configurer des certificats pour un appareil Azure Stack Edge Mini R dans le portail Azure | Microsoft Docs
description: Le tutoriel sur le déploiement d’Azure Stack Edge Mini R vous apprend à configurer des certificats pour votre appareil physique.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 10/21/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to configure certificates for Azure Stack Edge Mini R  so I can use it to transfer data to Azure.
ms.openlocfilehash: 1e81ab61b04f9cca6aff57de1736eac25a871c97
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96464285"
---
# <a name="tutorial-configure-certificates-vpn-encryption-for-your-azure-stack-edge-mini-r"></a>Tutoriel : Configurer des certificats, un VPN et un chiffrement pour votre appareil Azure Stack Edge Mini R

Ce tutoriel explique comment configurer des certificats, un VPN et un chiffrement au repos pour votre appareil Azure Stack Edge Mini R en utilisant l’interface utilisateur web locale.

Le temps nécessaire pour cette étape peut varier en fonction de l’option spécifique que vous choisissez et de la façon dont le workflow de certificat est établi dans votre environnement.

Ce tutoriel vous fournira des informations sur :

> [!div class="checklist"]
>
> * Prérequis
> * Configurer des certificats pour l’appareil physique
> * Configurer le VPN
> * Configurer le chiffrement au repos

## <a name="prerequisites"></a>Prérequis

Avant de configurer et d’activer votre appareil Azure Stack Edge Mini R, vérifiez que :

* Vous avez installé l’appareil physique, comme indiqué dans [Installer Azure Stack Edge Mini R](azure-stack-edge-mini-r-deploy-install.md).

* Si vous envisagez d’apporter vos propres certificats :
    - Vos certificats doivent être prêts au format approprié, y compris le certificat de chaîne de signature. Pour plus d’informations sur les certificats, accédez à [Gérer des certificats](azure-stack-edge-j-series-manage-certificates.md).

    - Si votre appareil est déployé sur Azure Government, Azure Government Secret ou le cloud top secret Azure Government et pas sur le cloud public Azure, un certificat de chaîne de signature est nécessaire pour pouvoir activer votre appareil. 
    Pour plus d’informations sur les certificats, accédez à [Gérer des certificats](azure-stack-edge-j-series-manage-certificates.md).


## <a name="configure-certificates-for-device"></a>Configurer des certificats pour un appareil

1. Dans la page **Certificats** , vous allez configurer vos certificats. Selon que vous avez ou non modifié le nom de l’appareil ou le domaine DNS dans la page **Configuration de l’appareil**, vous pouvez choisir l’une des options suivantes pour vos certificats.

    - Si vous n’avez pas changé le nom de l’appareil par défaut ou le domaine DNS par défaut à l’étape précédente et que vous ne souhaitez pas apporter vos propres certificats, vous pouvez ignorer cette étape et passer à la suivante. L’appareil a généré automatiquement des certificats auto-signés pour commencer. 

       <!-- ![Local web UI "Certificates" page](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/certificate-1.png)-->

    - Si vous avez modifié le nom de l’appareil ou le domaine DNS, vous noterez que l’état des certificats indique **Non valide**. 

        ![Page 2 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/certificate-2.png)    

        Sélectionnez un certificat pour afficher les détails relatifs à l’état.

        <!--![Local web UI "Certificates" page 3](./media/azure-stack-edge-gpu-deploy-configure-certificates/generate-certificate-1a.png)-->  

        L’état du certificat est **Non valide** car les certificats ne reflètent pas le nom de l’appareil et le domaine DNS mis à jour (qui sont utilisés dans le nom de l’objet et l’autre nom de l’objet). Pour activer correctement votre appareil, vous pouvez apporter vos propres certificats de point de terminaison signés et les chaînes de signature correspondantes. Vous ajoutez d’abord la chaîne de signature, puis vous chargez les certificats de point de terminaison. Pour plus d’informations, consultez [Apporter vos propres certificats sur votre appareil Azure Stack Edge Mini R](#bring-your-own-certificates).


    - Si vous avez modifié le nom de l’appareil ou le domaine DNS et que vous n’apportez pas vos propres certificats, l’**activation sera bloquée**.


#### <a name="bring-your-own-certificates"></a>Apportez vos propres certificats

Vous avez déjà ajouté la chaîne de signature sur cet appareil lors d’une étape précédente. Vous pouvez maintenant charger les certificats de point de terminaison, de nœud, de l’interface utilisateur locale et VPN. Pour ajouter vos propres certificats, effectuez les étapes suivantes.

1. Pour télécharger le certificat, sur la page **Certificat**, sélectionnez **+ Ajouter un certificat**.

    [![Page 4 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-1.png)](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-1.png#lightbox)


1. Vous pouvez charger d’autres certificats. Par exemple, vous pouvez télécharger les certificats de point de terminaison de stockage d’objets Blob et Azure Resource Manager.

    ![Page 6 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-3.png)

1. Vous pouvez également télécharger le certificat de l’interface utilisateur web locale. Après avoir téléchargé ce certificat, vous devez démarrer votre navigateur et effacer le cache. Vous devrez ensuite vous connecter à l’interface utilisateur web locale de l’appareil.  

    ![Page 7 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-4.png)

1. Vous pouvez également télécharger le certificat de nœud.


    ![Page 8 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-5.png)

1. Enfin, vous pouvez charger le certificat VPN.
        
    ![Page « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-6.png)

1. À tout moment, vous pouvez sélectionner un certificat et afficher les détails pour vous assurer qu’ils correspondent au certificat que vous avez chargé.

    [![Page 9 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-7.png)](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-7.png#lightbox)

    La page du certificat devrait s’actualiser pour refléter les nouveaux certificats ajoutés.

    [![Page 10 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-8.png)](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/add-certificate-8.png#lightbox)  

    > [!NOTE]
    > À l’exception du cloud public Azure, les certificats de chaîne de signature doivent être apportés avant l’activation de toutes les configurations cloud (Azure Government ou Azure Stack Hub).


## <a name="configure-vpn"></a>Configurer le VPN

1. Dans la vignette **Sécurité**, sélectionnez **Configurer** pour le VPN. 

    Pour configurer un VPN, vous devez d’abord veiller à disposer de toute la configuration nécessaire dans Azure. Pour plus d’informations, consultez [Configurer un VPN par le biais de PowerShell pour votre appareil Azure Stack Edge Mini R](azure-stack-edge-placeholder.md). Une fois cette vérification faite, vous pouvez effectuer la configuration dans l’interface utilisateur locale.
    
    1. Dans la page VPN, sélectionnez **Configurer**.
        ![Configurer l’interface utilisateur locale du VPN 1](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/configure-vpn-1.png)

    1. Dans le panneau **Configurer un VPN** :

        1. Indiquez l’annuaire téléphonique comme entrée.
        2. Fournissez le fichier JSON de la plage d’adresses IP Azure Data Center comme entrée. Téléchargez ce fichier à partir de : [https://www.microsoft.com/download/details.aspx?id=56519](https://www.microsoft.com/download/details.aspx?id=56519).
        3. Sélectionnez **eastus** comme région.
        4. Sélectionnez **Appliquer**.

        ![Configurer l’interface utilisateur locale du VPN 2](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/configure-vpn-2.png)
    
    1. Configurez des plages d’adresses IP accessibles uniquement avec le VPN. 
    
        - Sous **Plages d’adresses IP accessibles uniquement à l’aide du VPN**, sélectionnez **Configurer**.
        - Entrez la plage IPv4 du réseau virtuel que vous avez choisie pour votre réseau virtuel Azure.
        - Sélectionnez **Appliquer**.
    
        ![Configurer l’interface utilisateur locale du VPN 3](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/configure-vpn-3.png)

Votre appareil est maintenant prêt à être chiffré. Configurez le chiffrement au repos.


## <a name="enable-encryption"></a>Activer le chiffrement

1. Dans la vignette **Sécurité**, sélectionnez **Configurer** pour le chiffrement au repos. Il s’agit d’un paramètre obligatoire. Tant qu’il n’est pas configuré correctement, vous ne pouvez pas activer l’appareil. 

    ![Page « Chiffrement au repos » 1 de l’interface utilisateur web locale](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/encryption-at-rest-1.png)

    En usine, une fois les appareils imagés, le chiffrement BitLocker du niveau de volume est activé. Une fois que vous recevez l’appareil, vous devez configurer le chiffrement au repos. Le pool de stockage et les volumes sont recréés et vous pouvez fournir des clés BitLocker pour activer le chiffrement au repos et ainsi créer une deuxième couche de chiffrement pour vos données au repos.

1. Dans le volet **Chiffrement au repos**, entrez une clé codée en base 64 de 32 caractères (AES-256 bits). Il s’agit d’une configuration qui a lieu une seule fois et cette clé est utilisée pour protéger la clé de chiffrement réelle. 

    ![Page « Chiffrement au repos » 2 de l’interface utilisateur web locale](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/encryption-at-rest-2.png)

    Vous pouvez aussi choisir de générer automatiquement cette clé.

    ![Page « Chiffrement au repos » 3 de l’interface utilisateur web locale](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/encryption-at-rest-3.png)

1. Sélectionnez **Appliquer**. Cette opération prend plusieurs minutes et l’état de l’opération s’affiche dans la vignette **Sécurité**.

    ![Page « Chiffrement au repos » 4 de l’interface utilisateur web locale](./media/azure-stack-edge-mini-r-deploy-configure-certificates-vpn-encryption/encryption-at-rest-3.png)

1. Une fois que l’état indique **Terminé**, revenez à **Démarrer**.

Votre appareil est maintenant prêt à être activé.

## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel vous fournira des informations sur :

> [!div class="checklist"]
>
> * Prérequis
> * Configurer des certificats pour l’appareil physique
> * Configurer le VPN
> * Configurer le chiffrement au repos

Pour savoir comment activer votre appareil Azure Stack Edge Mini R, consultez :

> [!div class="nextstepaction"]
> [Activer un appareil Azure Stack Edge Mini R](./azure-stack-edge-mini-r-deploy-activate.md)
