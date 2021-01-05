---
title: 'Locataire Azure AD pour les connexions VPN utilisateur : Authentification Azure AD'
description: Vous pouvez utiliser un VPN utilisateur Azure Virtual WAN (point-to-site) pour vous connecter à votre réseau virtuel à l’aide de l'authentification Azure AD
titleSuffix: Azure Virtual WAN
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 09/22/2020
ms.author: alzam
ms.openlocfilehash: a54397fe7cfecef6813105645b7f2b218894875e
ms.sourcegitcommit: 5dbea4631b46d9dde345f14a9b601d980df84897
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91367850"
---
# <a name="prepare-azure-active-directory-tenant-for-user-vpn-openvpn-protocol-connections"></a>Préparer un locataire Azure Active Directory pour les connexions de VPN utilisateur de protocole OpenVPN

Pour vous connecter à votre hub virtuel via le protocole IKEV2, vous pouvez utiliser l’authentification par certificat ou l’authentification par protocole RADIUS (Remote Authentication Dial-In User Service). Toutefois, lorsque vous utilisez le protocole OpenVPN, vous pouvez également utiliser l’authentification Azure Active Directory. Cet article vous aide à configurer un locataire Azure AD pour utiliser un VPN utilisateur Virtual WAN (point à site) à l’aide de l’authentification OpenVPN.

> [!NOTE]
> L’authentification Azure AD n’est prise en charge que pour les connexions de protocole OpenVPN&reg;.
>

## <a name="1-create-the-azure-ad-tenant"></a><a name="tenant"></a>1. Créer le locataire Azure AD

Vérifiez que vous disposez d'un locataire Azure AD. Si vous n'avez pas de locataire Azure AD, créez-en un en suivant la procédure décrite dans l'article [Créer un locataire](../active-directory/fundamentals/active-directory-access-create-new-tenant.md) :

* Nom de l’organisation
* Nom de domaine initial

Exemple :

   ![Nouveau locataire Azure AD](./media/openvpn-create-azure-ad-tenant/newtenant.png)

## <a name="2-create-azure-ad-tenant-users"></a><a name="users"></a>2. Créer des utilisateurs de locataire Azure AD

Ensuite, créez deux comptes d’utilisateur dans le locataire Azure AD nouvellement créé, un compte Administrateur général et un compte d’utilisateur. Le compte d’utilisateur peut être utilisé pour tester l’authentification OpenVPN et le compte Administrateur général sera utilisé pour accorder le consentement à l’inscription de l’application VPN Azure. Une fois que vous avez créé un compte d’utilisateur Azure AD, attribuez un **rôle d’annuaire** à l’utilisateur afin de lui déléguer des autorisations d’administration.

Utilisez les étapes décrites dans [cet article](../active-directory/fundamentals/add-users-azure-active-directory.md) pour créer les deux utilisateurs de votre locataire Azure AD. Veillez à changer le **rôle d’annuaire** sur l’un des comptes créés en **Administrateur général**.

## <a name="3-grant-consent-to-the-azure-vpn-app-registration"></a><a name="enable-authentication"></a>3. Accorder le consentement à l’inscription de l’application VPN Azure

1. Connectez-vous au portail Azure en tant qu’utilisateur doté du rôle **Administrateur général**.

2. Ensuite, accordez le consentement administrateur de votre organisation, ce qui permet à l’application VPN Azure de se connecter et de lire les profils utilisateur. Copiez et collez l'URL correspondant à votre emplacement de déploiement sur la barre d'adresse de votre navigateur :

    Public

    ```
    https://login.microsoftonline.com/common/oauth2/authorize?client_id=41b23e61-6c1e-4545-b367-cd054e0ed4b4&response_type=code&redirect_uri=https://portal.azure.com&nonce=1234&prompt=admin_consent
    ````

    Azure Government

    ```
    https://login-us.microsoftonline.com/common/oauth2/authorize?client_id=51bb15d4-3a4f-4ebf-9dca-40096fe32426&response_type=code&redirect_uri=https://portal.azure.us&nonce=1234&prompt=admin_consent
    ````

    Microsoft Cloud Allemagne

    ```
    https://login-us.microsoftonline.de/common/oauth2/authorize?client_id=538ee9e6-310a-468d-afef-ea97365856a9&response_type=code&redirect_uri=https://portal.microsoftazure.de&nonce=1234&prompt=admin_consent
    ````

    Azure China 21Vianet

    ```
    https://https://login.chinacloudapi.cn/common/oauth2/authorize?client_id=49f817b6-84ae-4cc0-928c-73f27289b3aa&response_type=code&redirect_uri=https://portal.azure.cn&nonce=1234&prompt=admin_consent
    ```

3. Si vous y êtes invité, sélectionnez le compte **Administrateur général**.

    ![ID du répertoire](./media/openvpn-create-azure-ad-tenant/pick.png)

4. Lorsque vous y êtes invité, sélectionnez **Accepter**.

    ![Capture d’écran montrant une boîte de dialogue avec le message Autorisations demandées – Accepter pour votre organisation et des informations supplémentaires.](./media/openvpn-create-azure-ad-tenant/accept.jpg)

5. Sous votre instance Azure AD, dans **Applications d’entreprise**, vous devriez maintenant voir **VPN Azure** dans la liste.

    ![VPN Azure](./media/openvpn-create-azure-ad-tenant/azurevpn.png)

## <a name="next-steps"></a>Étapes suivantes

Pour vous connecter à vos réseaux virtuels à l’aide de l’authentification Azure AD, vous devez créer une configuration VPN utilisateur et l’associer à un hub virtuel. Consultez [Configurer l’authentification Azure AD pour une connexion point à site à Azure](virtual-wan-point-to-site-azure-ad.md).
