---
title: Guide pratique pour sécuriser une application monopage avec connexion de l’utilisateur
titleSuffix: Azure Maps
description: Explique comment configurer une application monopage qui prend en charge l’authentification unique Azure AD avec le SDK web Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/12/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: devx-track-js
ms.openlocfilehash: 60d8dc45fb26ea210b1827a6938716474faa0304
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895611"
---
# <a name="secure-a-single-page-application-with-user-sign-in"></a>Sécuriser une application monopage avec connexion de l’utilisateur

Le guide suivant concerne une application hébergée sur un serveur de contenu ou ayant des dépendances de serveur web minimales. L’application fournit des ressources protégées et sécurisées uniquement aux utilisateurs Azure AD. L’objectif du scénario est de permettre à l’application web de s’authentifier auprès d’Azure AD et d’appeler les API REST Azure Maps pour le compte de l’utilisateur.

[!INCLUDE [authentication details](./includes/view-authentication-details.md)]

## <a name="create-an-application-registration-in-azure-ad"></a>Créer une inscription d’application dans Azure AD

Créez l’application web dans Azure AD pour que les utilisateurs se connectent. L’application web délègue l’accès des utilisateurs aux API REST Azure Maps.

1. Dans le portail Azure, dans la liste des services Azure, sélectionnez **Azure Active Directory** > **Inscriptions d’applications** > **Nouvelle inscription** .  

    > [!div class="mx-imgBorder"]
    > ![Inscription d’application](./media/how-to-manage-authentication/app-registration.png)

2. Entrez un **Nom** , choisissez un **Type de compte de support** , fournissez un URI de redirection qui représente l’URL à laquelle Azure AD émettra le jeton et est l’URL où le contrôle de carte est hébergé. Pour obtenir un exemple détaillé, consultez [Exemples Azure AD Azure Maps](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples/tree/master/src/ImplicitGrant). Sélectionnez ensuite **Inscription** .  

3. Pour affecter des autorisations d’API déléguées à Azure Maps, accédez à l’application. Ensuite, sous **Inscriptions d’applications** , sélectionnez **Autorisations d’API** > **Ajouter une autorisation** . Sous **API utilisées par que mon organisation** , recherchez et sélectionnez **Azure Maps** .

    > [!div class="mx-imgBorder"]
    > ![Ajouter des autorisations d’API d’application](./media/how-to-manage-authentication/app-permissions.png)

4. Cochez la case en regard de **Accéder à Azure Maps** , puis sélectionnez **Ajouter des autorisations** .

    > [!div class="mx-imgBorder"]
    > ![Sélectionner des autorisations d’API d’application](./media/how-to-manage-authentication/select-app-permissions.png)

5. Activez `oauth2AllowImplicitFlow`. Pour l’activer, dans la section **Manifeste** de l’inscription de votre application, affectez la valeur `true` à `oauth2AllowImplicitFlow`.

6. Copiez l’ID d’application Azure AD et l’ID de locataire Azure AD à partir de l’inscription d’application afin de les utiliser dans le SDK web. Ajoutez les détails d’inscription d’application Azure AD et le `x-ms-client-id` du compte Azure Maps au SDK web.

    ```javascript
        <link rel="stylesheet" href="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.css" type="text/css" />
        <script src="https://atlas.microsoft.com/sdk/javascript/mapcontrol/2/atlas.min.js" />
        <script>
            var map = new atlas.Map("map", {
                center: [-122.33, 47.64],
                zoom: 12,
                language: "en-US",
                authOptions: {
                    authType: "aad",
                    clientId: "<insert>",  // azure map account client id
                    aadAppId: "<insert>",  // azure ad app registration id
                    aadTenant: "<insert>", // azure ad tenant id
                    aadInstance: "https://login.microsoftonline.com/"
                }
            });
        </script>   
    ```

7. Configurez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour les utilisateurs ou les groupes. Consultez les [sections suivantes pour activer Azure RBAC](#grant-role-based-access-for-users-to-azure-maps).
   
[!INCLUDE [grant role access to users](./includes/grant-rbac-users.md)]

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur le scénario d’application monopage :
> [!div class="nextstepaction"]
> [Application monopage](../active-directory/develop/scenario-spa-overview.md)

Recherchez les métriques d’utilisation de l’API pour votre compte Azure Maps :
> [!div class="nextstepaction"]
> [Afficher les métriques d’utilisation](how-to-view-api-usage.md)

Explorez des exemples qui montrent comment intégrer Azure AD à Azure Maps :
> [!div class="nextstepaction"]
> [Exemples Azure Maps](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples/tree/master/src/ImplicitGrant)