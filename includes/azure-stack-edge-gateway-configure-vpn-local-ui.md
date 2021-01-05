---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 10/27/2020
ms.author: alkohli
ms.openlocfilehash: fa65a7354112a2b220686372459b348d45832dd9
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465337"
---
Procédez comme suit dans l’interface utilisateur web locale de votre appareil. Ces étapes prennent environ 15 minutes, chargement du fichier de configuration du VPN (ou du fichier d’étiquettes de service) compris. 

1. Accédez à **Configuration > VPN**. Sélectionnez **Configurer**.

    ![Configurer l’interface utilisateur locale 1](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/configure-vpn-local-ui-1.png)

2. Dans le panneau **Configurer un VPN** :

    - Activez **Paramètres du VPN**.
    - Indiquez le **secret partagé du VPN**. Il s’agit de la clé partagée que vous avez fournie lors de la création de la ressource de connexion VPN Azure.
    - Indiquez l’**adresse IP de la passerelle VPN**. Il s’agit de l’adresse IP de la passerelle réseau locale Azure.
    - Pour **Groupe PFS**, sélectionnez **Aucun**. 
    - Pour **Groupe DH**, sélectionnez **Group2**.
    - Pour **Méthode d’intégrité IPsec**, sélectionnez **SHA256**.
    - Pour **Constantes de transformation IPsec cipher**, sélectionnez **GCMAES256**.
    - Pour **Constantes de transformation d’authentification IPsec**, sélectionnez **GCMAES256**.
    - Pour **Méthode de chiffrement IKE**, sélectionnez **AES256**.
    - Sélectionnez **Appliquer**.

        ![Configurer l’interface utilisateur locale 2](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/configure-vpn-local-ui-2.png)

    Pour plus d’informations sur les algorithmes de chiffrement pris en charge, consultez [À propos des exigences de chiffrement et des passerelles VPN Azure](../articles/vpn-gateway/vpn-gateway-about-compliance-crypto.md#ipsecike-policy-faq). 

3. Pour charger le fichier de configuration de route VPN, sélectionnez **Charger**. 

    ![Configurer l’interface utilisateur locale 3](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/configure-vpn-local-ui-3.png)

    - Accédez au fichier *json* des étiquettes de service que vous avez téléchargé sur votre système local à l’étape précédente.
    - Sélectionnez la région Azure associée à l’appareil, au réseau virtuel et aux passerelles.
    - Sélectionnez **Appliquer**.

        ![Configurer l’interface utilisateur locale 4](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/configure-vpn-local-ui-4.png)
    
    Le chargement prend de 7 à 8 minutes sur l’appareil.

4. Pour ajouter des routes propres au client, configurez des plages d’adresses IP accessibles uniquement à l’aide du VPN. 

    - Sous **Plages d’adresses IP accessibles uniquement à l’aide du VPN**, sélectionnez **Configurer**.
    - Indiquez une plage IPv4 valide et sélectionnez **Ajouter**. Répétez les étapes pour ajouter d’autres plages.
    - Sélectionnez **Appliquer**.

        ![Configurer l’interface utilisateur locale 5](../articles/databox-online/media/azure-stack-edge-pro-r-configure-vpn-powershell/configure-vpn-local-ui-5.png)

