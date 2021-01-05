---
title: Tutoriel pour configurer des certificats pour un appareil Azure Stack Edge Pro R dans le portail Azure | Microsoft Docs
description: Le tutoriel sur le déploiement d’Azure Stack Edge Pro R vous apprend à configurer des certificats pour votre appareil physique.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: tutorial
ms.date: 10/19/2020
ms.author: alkohli
Customer intent: As an IT admin, I need to understand how to configure certificates for Azure Stack Edge Pro R so I can use it to transfer data to Azure.
ms.openlocfilehash: fad3e5dcb0ecda82f3fb35cadf1719a62c99bd97
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96464060"
---
# <a name="tutorial-configure-certificates-for-your-azure-stack-edge-pro-r"></a>Tutoriel : Configurer des certificats pour votre appareil Azure Stack Edge Pro R

Ce tutoriel explique comment configurer des certificats pour votre appareil Azure Stack Edge Pro R en utilisant l’interface utilisateur web locale.

Le temps nécessaire pour cette étape peut varier en fonction de l’option spécifique que vous choisissez et de la façon dont le workflow de certificat est établi dans votre environnement.

Ce tutoriel vous fournira des informations sur :

> [!div class="checklist"]
>
> * Prérequis
> * Configurer des certificats pour l’appareil physique
> * Configurer le VPN
> * Configurer le chiffrement au repos

## <a name="prerequisites"></a>Prérequis

Avant de configurer et d’activer votre appareil Azure Stack Edge Pro R, vérifiez que :

* Vous avez installé l’appareil physique, comme indiqué dans [Installer Azure Stack Edge Pro R](azure-stack-edge-pro-r-deploy-install.md).
* Si vous envisagez d’apporter vos propres certificats :
    - Vos certificats doivent être prêts au format approprié, y compris le certificat de chaîne de signature. Pour plus d’informations sur les certificats, accédez à [Gérer des certificats](azure-stack-edge-j-series-manage-certificates.md).



## <a name="configure-certificates-for-device"></a>Configurer des certificats pour un appareil

1. Dans la page **Certificats** , vous allez configurer vos certificats. Selon que vous avez ou non modifié le nom de l’appareil ou le domaine DNS dans la page **Configuration de l’appareil**, vous pouvez choisir l’une des options suivantes pour vos certificats.

    - Si vous n’avez pas modifié le nom de l’appareil ou le domaine DNS à l’étape précédente, vous pouvez ignorer cette étape et passer à la suivante. L’appareil a généré automatiquement des certificats auto-signés pour commencer. 

    - Si vous avez modifié le nom de l’appareil ou le domaine DNS, vous noterez que l’état des certificats indique **Non valide**. 

        ![Page 2 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/generate-certificate-1.png)    

        Sélectionnez un certificat pour afficher les détails relatifs à l’état.

        ![Page 3 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/generate-certificate-1a.png)  

        Cela est dû au fait que les certificats ne reflètent pas le nom de l’appareil et le domaine DNS mis à jour (qui sont utilisés dans le nom de l’objet et l’autre nom de l’objet). Pour activer correctement votre appareil, vous pouvez apporter vos propres certificats de point de terminaison signés et les chaînes de signature correspondantes. Vous ajoutez d’abord la chaîne de signature, puis vous chargez les certificats de point de terminaison. Pour plus d’informations, consultez [Apporter vos propres certificats sur votre appareil Azure Stack Edge Pro R](#bring-your-own-certificates).

    - Si vous avez changé le nom de l’appareil ou le domaine DNS et que vous n’apportez pas vos propres certificats, l’**activation sera bloquée**.

    
#### <a name="bring-your-own-certificates"></a>Apportez vos propres certificats

Procédez comme suit pour ajouter vos propres certificats, y compris la chaîne de signature.

1. Pour télécharger le certificat, sur la page **Certificat**, sélectionnez **+ Ajouter un certificat**.

    ![Page 4 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/add-certificate-1.png)

2. Chargez d’abord la chaîne de signature, puis sélectionnez **Valider et ajouter**.

    ![Page 5 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/add-certificate-2.png)

3. Vous pouvez maintenant télécharger d’autres certificats. Par exemple, vous pouvez télécharger les certificats de point de terminaison de stockage d’objets Blob et Azure Resource Manager.

    ![Page 6 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/add-certificate-3.png)

    Vous pouvez également télécharger le certificat de l’interface utilisateur web locale. Après avoir téléchargé ce certificat, vous devez démarrer votre navigateur et effacer le cache. Vous devrez ensuite vous connecter à l’interface utilisateur web locale de l’appareil.  

    ![Page 7 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/add-certificate-5.png)

    Vous pouvez également télécharger le certificat de nœud.

    ![Page 8 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/add-certificate-4.png)

    Enfin, vous pouvez charger le certificat VPN.

    ![Page 9 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/add-certificate-4.png)

    À tout moment, vous pouvez sélectionner un certificat et afficher les détails pour vous assurer qu’ils correspondent au certificat que vous avez chargé.

    La page du certificat devrait s’actualiser pour refléter les nouveaux certificats ajoutés.

    ![Page 10 « Certificats » de l’interface utilisateur web locale](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/add-certificate-7.png)  

    > [!NOTE]
    > À l’exception du cloud public Azure, les certificats de chaîne de signature doivent être apportés avant l’activation de toutes les configurations cloud (Azure Government ou Azure Stack).

1. Sélectionnez **< Revenir à Démarrer**.

## <a name="configure-vpn"></a>Configurer le VPN

1. Dans la vignette **Sécurité**, sélectionnez **Configurer** pour le VPN.

    ![Page « VPN » de l’interface utilisateur web locale](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/configure-vpn-local-ui-1.png)  

    Pour configurer un VPN, vous devez d’abord veiller à disposer de toute la configuration nécessaire dans Azure. Pour plus d’informations, consultez [Configurer les prérequis](azure-stack-edge-placeholder.md) et [Configurer des ressources Azure pour un VPN](azure-stack-edge-placeholder.md). Une fois cette vérification faite, vous pouvez effectuer la configuration dans l’interface utilisateur locale.
    
    1. Dans la page VPN, sélectionnez **Configurer**.
    2. Dans le panneau **Configurer un VPN** :

    - Activez **Paramètres du VPN**.
    - Indiquez le **secret partagé du VPN**. Il s’agit de la clé partagée que vous avez fournie lors de la création de l’objet de connexion VPN Azure.
    - Indiquez l’**adresse IP de la passerelle VPN**. Il s’agit de l’adresse IP de la passerelle réseau locale Azure.
    - Pour **Groupe PFS**, sélectionnez **Aucun**. 
    - Pour **Groupe DH**, sélectionnez **Group2**.
    - Pour **Méthode d’intégrité IPsec**, sélectionnez **SHA256**.
    - Pour **Constantes de transformation IPseccipher**, sélectionnez **GCMAES256**.
    - Pour **Constantes de transformation d’authentification IPsec**, sélectionnez **GCMAES256**.
    - Pour **Méthode de chiffrement IKE**, sélectionnez **AES256**.
    - Sélectionnez **Appliquer**.

        ![Configurer l’interface utilisateur locale 2](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/configure-vpn-local-ui-2.png)

    3. Pour charger le fichier de configuration de route VPN, sélectionnez **Charger**. 
    
        ![Configurer l’interface utilisateur locale 3](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/configure-vpn-local-ui-3.png)
    
        - Accédez au fichier *json* de configuration VPN que vous avez téléchargé sur votre système local à l’étape précédente.
        - Sélectionnez la région Azure associée à l’appareil, au réseau virtuel et aux passerelles.
        - Sélectionnez **Appliquer**.
    
            ![Configurer l’interface utilisateur locale 4](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/configure-vpn-local-ui-4.png)
    
    4. Pour ajouter des routes propres au client, configurez des plages d’adresses IP accessibles uniquement à l’aide du VPN. 
    
        - Sous **Plages d’adresses IP accessibles uniquement à l’aide du VPN**, sélectionnez **Configurer**.
        - Indiquez une plage IPv4 valide et sélectionnez **Ajouter**. Répétez les étapes pour ajouter d’autres plages.
        - Sélectionnez **Appliquer**.
    
            ![Configurer l’interface utilisateur locale 5](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/configure-vpn-local-ui-5.png)

1. Sélectionnez **< Revenir à Démarrer**.

## <a name="configure-encryption-at-rest"></a>Configurer le chiffrement au repos

1. Dans la vignette **Sécurité**, sélectionnez **Configurer** pour le chiffrement au repos. Il s’agit d’un paramètre obligatoire. Tant qu’il n’est pas configuré correctement, vous ne pouvez pas activer l’appareil. 

    En usine, une fois les appareils imagés, le chiffrement BitLocker du niveau de volume est activé. Une fois que vous recevez l’appareil, vous devez configurer le chiffrement au repos. Le pool de stockage et les volumes sont recréés et vous pouvez fournir des clés BitLocker pour activer le chiffrement au repos et ainsi créer une deuxième couche de chiffrement pour vos données au repos.

1. Dans le volet **Chiffrement au repos**, fournissez une clé encodée en base 64 de 32 caractères. Il s’agit d’une configuration qui a lieu une seule fois et cette clé est utilisée pour protéger la clé de chiffrement réelle. Vous pouvez choisir de générer automatiquement cette clé ou d’en entrer une.

    ![Volet « Chiffrement au repos » de l’interface utilisateur web locale](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/encryption-key-1.png)

    La clé est enregistrée dans un fichier de clé dans la page **Détails du cloud** après l’activation de l’appareil.

1. Sélectionnez **Appliquer**. Cette opération prend plusieurs minutes et l’état de l’opération s’affiche dans la vignette **Sécurité**.

    ![Page « Chiffrement au repos » de l’interface utilisateur web locale](./media/azure-stack-edge-pro-r-deploy-configure-certificates-vpn-encryption/encryption-at-rest-status-1.png)

1. Une fois que l’état indique **Terminé**, sélectionnez **< Revenir à Démarrer**.

Votre appareil est maintenant prêt à être activé. 


## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel vous fournira des informations sur :

> [!div class="checklist"]
>
> * Prérequis
> * Configurer des certificats pour l’appareil physique
> * Configurer le VPN
> * Configurer le chiffrement au repos

Pour savoir comment activer votre appareil Azure Stack Edge Pro R, consultez :

> [!div class="nextstepaction"]
> [Activer un appareil Azure Stack Edge Pro R](./azure-stack-edge-pro-r-deploy-activate.md)
