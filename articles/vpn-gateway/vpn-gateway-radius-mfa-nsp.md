---
title: Intégrer NPS à l’authentification RADIUS de passerelle VPN pour MFA
description: Décrit comment intégrer l’authentification RADIUS de la passerelle Azure avec un serveur NPS pour l’authentification multifacteur.
services: vpn-gateway
documentationcenter: na
author: ahmadnyasin
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: vpn-gateway
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/16/2019
ms.author: genli
ms.openlocfilehash: 208e99f61694f5a81a98dbc649e2a6035f57891b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018271"
---
# <a name="integrate-azure-vpn-gateway-radius-authentication-with-nps-server-for-multi-factor-authentication"></a>Intégrer l’authentification RADIUS de la passerelle VPN Azure avec un serveur NPS pour l’authentification multifacteur 

L’article décrit comment intégrer un serveur NPS (Network Policy Server) avec l’authentification RADIUS de passerelle VPN Azure pour fournir une authentification multifacteur (MFA) pour les connexions VPN point à site. 

## <a name="prerequisite"></a>Configuration requise

Pour activer l’authentification multifacteur, les utilisateurs doivent être dans Azure Active Directory (Azure AD), qui doit être synchronisé à partir de l’environnement local ou cloud. L’utilisateur doit également avoir terminé le processus d’inscription automatique pour l’authentification multifacteur.  Pour plus d’informations, voir [Configurer mon compte pour la vérification en deux étapes](../active-directory/user-help/multi-factor-authentication-end-user-first-time.md)

## <a name="detailed-steps"></a>Procédure détaillée

### <a name="step-1-create-a-virtual-network-gateway"></a>Étape 1 : créer une passerelle de réseau virtuel

1. Connectez-vous au [Portail Azure](https://portal.azure.com).
2. Dans le réseau virtuel qui hébergera la passerelle de réseau virtuel, sélectionnez **Sous-réseaux**, puis **Sous-réseau de passerelle** pour créer un sous-réseau. 

    ![Image relative à l’ajout d’une passerelle de sous-réseau](./media/vpn-gateway-radiuis-mfa-nsp/gateway-subnet.png)
3. Créez une passerelle de réseau virtuel en spécifiant les paramètres suivants :

    - **Type de passerelle** : sélectionnez **VPN**.
    - **Type de VPN** : sélectionnez **Route-based**.
    - **Référence SKU** : sélectionnez un type de référence SKU en fonction de vos besoins.
    - **Réseau virtuel** : sélectionnez le réseau virtuel dans lequel vous avez créé le sous-réseau de passerelle.

        ![Image relative aux paramètres de passerelle de réseau virtuel](./media/vpn-gateway-radiuis-mfa-nsp/create-vpn-gateway.png)


 
### <a name="step-2-configure-the-nps-for-azure-ad-mfa"></a>Étape 2 : configurer le serveur NPS pour Azure AD MFA

1. Sur le serveur NPS, [installez l’extension NPS pour Azure AD MFA](../active-directory/authentication/howto-mfa-nps-extension.md#install-the-nps-extension).
2. Ouvrez la console NPS, cliquez avec le bouton droit sur **Clients RADIUS**, puis sélectionnez **Nouveau**. Créez le client RADIUS en spécifiant les paramètres suivants :

    - **Nom convivial** : saisissez n’importe quel nom.
    - **Adresse (IP ou DNS)** : indiquez le sous-réseau de passerelle que vous avez créé à l’étape 1.
    - **Secret partagé** : saisissez une clé secrète et gardez-la en mémoire pour plus tard.

      ![Image relative aux paramètres du client RADIUS](./media/vpn-gateway-radiuis-mfa-nsp/create-radius-client1.png)

 
3.  Dans l’onglet **Avancé**, définissez le nom du fournisseur sur **RADIUS Standard** et assurez-vous que la case **Options supplémentaires** n’est pas cochée.

    ![Image relative aux paramètres avancés du client RADIUS](./media/vpn-gateway-radiuis-mfa-nsp/create-radius-client2.png)

4. Accédez à **Stratégies** > **Stratégies réseau**, double-cliquez sur la stratégie relative aux **connexions au serveur de routage et d’accès à distance Microsoft**, sélectionnez  **Accorder l’accès**, puis cliquez sur **OK**.

### <a name="step-3-configure-the-virtual-network-gateway"></a>Étape 3 : configurer la passerelle de réseau virtuel

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Ouvrez la passerelle de réseau virtuel que vous avez créée. Assurez-vous que le type de passerelle est défini sur **VPN** et que le type VPN est **Route-based**.
3. Cliquez sur **Configuration point à site** > **Configurer maintenant**, puis spécifiez les paramètres suivants :

    - **Pool d’adresses** : indiquez le sous-réseau de passerelle que vous avez créé à l’étape 1.
    - **Type d’authentification** : sélectionnez **Authentification RADIUS**.
    - **Adresse IP du serveur** : tapez l’adresse IP du serveur NPS.

      ![Image relative aux paramètres point à site](./media/vpn-gateway-radiuis-mfa-nsp/configure-p2s.png)

## <a name="next-steps"></a>Étapes suivantes

- [Azure AD Multi-Factor Authentication](../active-directory/authentication/concept-mfa-howitworks.md)
- [Intégration de votre infrastructure NPS existante à Azure AD Multi-Factor Authentication](../active-directory/authentication/howto-mfa-nps-extension.md)