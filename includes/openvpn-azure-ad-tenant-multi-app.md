---
title: Fichier include
description: Fichier include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 10/15/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: c3236f9c60cb359349d96e93f674c3e278e44f1e
ms.sourcegitcommit: 6a902230296a78da21fbc68c365698709c579093
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93375856"
---
## <a name="1-create-the-azure-ad-tenant"></a><a name="tenant"></a>1. Créer le locataire Azure AD

Pour créer un locataire Azure AD, suivez la procédure décrite dans l'article [Créer un locataire](../articles/active-directory/fundamentals/active-directory-access-create-new-tenant.md) :

* Nom de l'organisation
* Nom de domaine initial

  Exemple :

   ![Nouveau locataire Azure AD](./media/openvpn-azure-ad-tenant-multi-app/new-tenant.png)

## <a name="2-create-tenant-users"></a><a name="users"></a>2. Créer des utilisateurs du locataire

Au cours de cette étape, vous allez créer deux utilisateurs Azure du locataire Azure AD : un compte d’administrateur général et un compte d’utilisateur maître. Le compte d'utilisateur principal est utilisé comme compte d'intégration principal (compte de service). Lorsque vous créez un compte d'utilisateur de locataire Azure AD, vous définissez le rôle d'annuaire en fonction du type d'utilisateur que vous souhaitez créer. Utilisez les étapes décrites dans [cet article](../articles/active-directory/fundamentals/add-users-azure-active-directory.md) pour créer au moins deux utilisateurs pour votre locataire Azure AD. Veillez à modifier le **rôle d'annuaire** pour créer les types de compte :

* Administrateur général
* Utilisateur

## <a name="3-register-the-vpn-client"></a><a name="register-client"></a>3. Inscrire le client VPN

Inscrivez le client VPN dans le locataire Azure AD.

1. Recherchez l'ID du répertoire que vous souhaitez utiliser pour l'authentification. Celui-ci est répertorié dans la section Propriétés de la page Active Directory.

    ![ID du répertoire](./media/openvpn-azure-ad-tenant-multi-app/directory-id.png)

2. Copiez l'ID du répertoire.

3. Connectez-vous au portail Azure en tant qu'utilisateur doté du rôle d'**Administrateur général**.

4. Octroyez ensuite le consentement administrateur. Copiez et collez l'URL correspondant à votre emplacement de déploiement sur la barre d'adresse de votre navigateur :

    Public

    ```
    https://login.microsoftonline.com/common/oauth2/authorize?client_id=41b23e61-6c1e-4545-b367-cd054e0ed4b4&response_type=code&redirect_uri=https://portal.azure.com&nonce=1234&prompt=admin_consent
    ````

    Azure Government

    ```
    https://login.microsoftonline.us/common/oauth2/authorize?client_id=51bb15d4-3a4f-4ebf-9dca-40096fe32426&response_type=code&redirect_uri=https://portal.azure.us&nonce=1234&prompt=admin_consent
    ````

    Microsoft Cloud Allemagne

    ```
    https://login-us.microsoftonline.de/common/oauth2/authorize?client_id=538ee9e6-310a-468d-afef-ea97365856a9&response_type=code&redirect_uri=https://portal.microsoftazure.de&nonce=1234&prompt=admin_consent
    ````

    Azure China 21Vianet

    ```
    https://https://login.chinacloudapi.cn/common/oauth2/authorize?client_id=49f817b6-84ae-4cc0-928c-73f27289b3aa&response_type=code&redirect_uri=https://portal.azure.cn&nonce=1234&prompt=admin_consent
    ```

> [!NOTE]
> Si vous utilisez un compte d’administrateur général qui n’est pas natif pour le locataire Azure AD pour fournir un consentement, remplacez « Common » par l’ID d’annuaire Azure AD dans l’URL. Dans d’autres cas, vous devrez peut-être également remplacer « Common » par votre ID d’annuaire.
>

5. Si vous y êtes invité, sélectionnez le compte d'**Administrateur général**.

    ![ID du répertoire 2](./media/openvpn-azure-ad-tenant-multi-app/pick.png)

6. Lorsque vous y êtes invité, sélectionnez **Accepter**.

    ![La capture d’écran montre une fenêtre avec le message Autorisations demandées Accepter pour votre organisation, et des informations sur la demande.](./media/openvpn-azure-ad-tenant-multi-app/accept.jpg)

7. Sous votre Azure AD, dans **Applications d’entreprise**, vous verrez **VPN Azure**.

     ![VPN Azure](./media/openvpn-azure-ad-tenant-multi-app/azure-vpn.png)

## <a name="4-register-additional-applications"></a><a name="register-apps"></a>4. Inscrire des applications supplémentaires

Au cours de cette étape, vous allez inscrire des applications supplémentaires pour différents utilisateurs et groupes.

1. Sous votre Azure Active Directory, cliquez sur **Inscriptions d’applications**, puis **+ Nouvelle inscription**.

    ![VPN Azure 2](./media/openvpn-azure-ad-tenant-multi-app/app1.png)

2. Dans la page **Inscrire une application**, entrez le **Nom**. Sélectionnez les **Types de comptes pris en charge** souhaités, puis cliquez sur **Inscrire**.

    ![VPN Azure 3](./media/openvpn-azure-ad-tenant-multi-app/app2.png)

3. Une fois la nouvelle application inscrite, cliquez sur **Exposer une API** sous le panneau de l’application.

4. Cliquez sur **+ Ajouter une étendue**.

5. Laissez la valeur par défaut **URI d’ID d’application**. Cliquez sur **Enregistrer et continuer**.

    ![VPN Azure 4](./media/openvpn-azure-ad-tenant-multi-app/app3.png)

6. Renseignez les champs requis et assurez-vous que **État** est **Activé**. Cliquez sur **Ajouter une étendue**.

    ![VPN Azure 5](./media/openvpn-azure-ad-tenant-multi-app/app4.png)

7. Cliquez sur **Exposer une API**, puis sur **+ Ajouter une application cliente**.  Pour **ID client**, saisissez les valeurs suivantes en fonction du Cloud :

    - Entrez **41b23e61-6c1e-4545-b367-cd054e0ed4b4** pour Azure **Public**
    - Entrez **51bb15d4-3a4f-4EBF-9dca-40096fe32426** pour Azure **Government**
    - Entrez **538ee9e6-310a-468d-afef-ea97365856a9** pour Azure **German**
    - Entrez **49f817b6-84AE-4cc0-928c-73f27289b3aa** pour Azure **China21Vianet**

8. Cliquez sur **Ajouter une application**.

    ![VPN Azure 6](./media/openvpn-azure-ad-tenant-multi-app/app5.png)

9. Copiez l’**ID d’application (client)** à partir de la page **Vue d’ensemble**. Vous aurez besoin de ces informations pour configurer vos passerelles VPN.

    ![VPN Azure 7](./media/openvpn-azure-ad-tenant-multi-app/app6.png)

10. Répétez les étapes de cette section [Inscrire des applications supplémentaires](#register-apps) pour créer le nombre d’applications nécessaires pour vos besoins de sécurité. Chaque application est associée à une passerelle VPN et peut avoir un ensemble différent d’utilisateurs. Une seule application peut être associée à une passerelle.

## <a name="5-assign-users-to-applications"></a><a name="assign-users"></a>5. Affecter des utilisateurs à des applications

Affectez des utilisateurs à vos applications.

1. Sous **Azure AD -> Applications d’entreprise**, sélectionnez l’application que vous venez d’inscrire, puis cliquez sur **Propriétés**. Assurez-vous que **Affectation d’utilisateur requise ?** est défini sur **Oui**. Cliquez sur **Enregistrer**.

    ![VPN Azure 8](./media/openvpn-azure-ad-tenant-multi-app/user2.png)

2. Sur la page de l’application, cliquez sur **Utilisateurs et groupes**, puis sur **+ Ajouter un utilisateur**.

    ![VPN Azure 9](./media/openvpn-azure-ad-tenant-multi-app/user3.png)

3. Sous **Ajouter une attribution**, cliquez sur **Utilisateurs et groupes**. Sélectionnez les utilisateurs auxquels vous souhaitez autoriser l’accès à cette application VPN. Cliquez sur **Sélectionner**.

    ![VPN Azure 10](./media/openvpn-azure-ad-tenant-multi-app/user4.png)
