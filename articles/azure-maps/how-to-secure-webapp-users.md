---
title: Guide pratique pour sécuriser une application web avec authentification unique interactive
titleSuffix: Azure Maps
description: Explique comment configurer une application web qui prend en charge l’authentification unique Azure AD avec le SDK web Azure Maps à l’aide du protocole OpenID Connect.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/12/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: devx-track-js
ms.openlocfilehash: ebdc4b219e0840c18e6bef8ebfe9b8eefa8faf3b
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895566"
---
# <a name="secure-a-web-application-with-user-sign-in"></a>Sécuriser une application web avec connexion de l’utilisateur

Le guide suivant se rapporte à une application hébergée sur des serveurs web, gère plusieurs scénarios d’entreprise et procède au déploiement sur des serveurs web. L’application doit impérativement fournir des ressources protégées et sécurisées uniquement aux utilisateurs Azure AD. L’objectif du scénario est de permettre à l’application web de s’authentifier auprès d’Azure AD et d’appeler les API REST Azure Maps pour le compte de l’utilisateur.

[!INCLUDE [authentication details](./includes/view-authentication-details.md)]

## <a name="create-an-application-registration-in-azure-ad"></a>Créer une inscription d’application dans Azure AD

Vous devez créer l’application web dans Azure AD pour que les utilisateurs se connectent. Cette application web déléguera ensuite l’accès des utilisateurs aux API REST Azure Maps.

1. Dans le portail Azure, dans la liste des services Azure, sélectionnez **Azure Active Directory** > **Inscriptions d’applications** > **Nouvelle inscription** .  

    > [!div class="mx-imgBorder"]
    > ![Inscription d’application](./media/how-to-manage-authentication/app-registration.png)

2. Entrez un **Nom** , choisissez un **Type de compte de support** , fournissez un URI de redirection qui représente l’URL à laquelle Azure AD émettra le jeton et est l’URL où le contrôle de carte est hébergé. Pour plus d’informations, consultez [Scénario Azure AD : application web qui connecte les utilisateurs](../active-directory/develop/scenario-web-app-sign-user-overview.md). Effectuez les étapes indiquées dans le scénario Azure AD.  

3. Une fois l’inscription de l’application terminée, vérifiez que la connexion à l’application fonctionne pour les utilisateurs. Une fois que la connexion réussit, l’application peut bénéficier d’un accès délégué aux API REST Azure Maps.
    
4.  Pour affecter des autorisations d’API déléguées à Azure Maps, accédez à l’application. Sélectionnez **Autorisations des API** > **Ajouter une autorisation** . Sous **API utilisées par que mon organisation** , recherchez et sélectionnez **Azure Maps** .

    > [!div class="mx-imgBorder"]
    > ![Ajouter des autorisations d’API d’application](./media/how-to-manage-authentication/app-permissions.png)

5. Cochez la case en regard de **Accéder à Azure Maps** , puis sélectionnez **Ajouter des autorisations** .

    > [!div class="mx-imgBorder"]
    > ![Sélectionner des autorisations d’API d’application](./media/how-to-manage-authentication/select-app-permissions.png)

6. Autorisez l’application web à appeler les API REST Azure Maps en configurant l’inscription d’application avec un secret d’application. Pour obtenir des instructions détaillées, consultez [Application web qui appelle des API web : Inscription d’application](../active-directory/develop/scenario-web-app-call-api-app-registration.md). Un secret est nécessaire afin de s’authentifier auprès d’Azure AD pour le compte de l’utilisateur. Le certificat ou le secret d’inscription d’application doit être stocké dans un magasin sécurisé afin que l’application web puisse le récupérer pour s’authentifier auprès d’Azure AD. 
   
   * Si l’application a déjà configuré un secret et une inscription d’application Azure AD, cette étape peut être ignorée.

> [!Tip]
> Si l’application est hébergée dans un environnement Azure, nous vous recommandons d’utiliser des [identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md) et une instance d’Azure Key Vault afin d’accéder aux secrets en [acquérant un jeton d’accès](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md) pour accéder aux secrets ou certificats Azure Key Vault. Pour vous connecter à Azure Key Vault afin de récupérer des secrets, consultez le [tutoriel sur la connexion par le biais d’une identité managée](../key-vault/general/tutorial-net-create-vault-azure-web-app.md).
   
7. Implémentez un point de terminaison de jeton sécurisé pour le SDK web Azure Maps pour accéder à un jeton. 
   
   * Pour obtenir un exemple de contrôleur de jetons, consultez [Exemples Azure AD Azure Maps](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples/blob/master/src/OpenIdConnect/AzureMapsOpenIdConnectv1/AzureMapsOpenIdConnect/Controllers/TokenController.cs). 
   * Pour une implémentation non-AspNetCore ou autre, consultez [Acquérir un jeton pour l’application](../active-directory/develop/scenario-web-app-call-api-acquire-token.md) dans la documentation d’Azure AD.
   * Le point de terminaison de jeton sécurisé est chargé de retourner un jeton d’accès pour l’utilisateur authentifié et autorisé afin d’appeler les API REST Azure Maps.

8. Configurez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour les utilisateurs ou les groupes. Consultez la section [Accorder l’accès en fonction du rôle pour des utilisateurs](#grant-role-based-access-for-users-to-azure-maps).

9. Configurez la page d’application web avec le SDK web Azure Maps pour accéder au point de terminaison de jeton sécurisé. 

```javascript
var map = new atlas.Map("map", {
        center: [-122.33, 47.64],
        zoom: 12,
        language: "en-US",
        authOptions: {
            authType: "anonymous",
            clientId: "<insert>",  // azure map account client id
            getToken: function (resolve, reject, map) {
                var xhttp = new XMLHttpRequest();
                xhttp.open("GET", "/api/token", true); // the url path maps to the token endpoint.
                xhttp.onreadystatechange = function () {
                    if (this.readyState === 4 && this.status === 200) {
                        resolve(this.responseText);
                    } else if (this.status !== 200) {
                        reject(this.responseText);
                    }
                };

                xhttp.send();
            }
        }
    });
    map.events.add("tokenacquired", function () {
        console.log("token acquired");
    });
    map.events.add("error", function (err) {
        console.log(JSON.stringify(err.error));
    });
```

[!INCLUDE [grant role-based access to users](./includes/grant-rbac-users.md)]

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur le scénario d’application web :
> [!div class="nextstepaction"]
> [Scénario : application web qui connecte les utilisateurs](../active-directory/develop/scenario-web-app-sign-user-overview.md)

Recherchez les métriques d’utilisation de l’API pour votre compte Azure Maps :
> [!div class="nextstepaction"]
> [Afficher les métriques d’utilisation](how-to-view-api-usage.md)

Explorez des exemples qui montrent comment intégrer Azure AD à Azure Maps :
> [!div class="nextstepaction"]
> [Exemples d’applications web Azure AD Azure Maps](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples/tree/master/src/OpenIdConnect)