---
title: Guide pratique pour sécuriser des appareils d’entrée contraints avec des API REST Azure Maps et Azure AD
titleSuffix: Azure Maps
description: Explique comment configurer une application sans navigateur qui prend en charge la connexion à Azure AD et appelle des API REST Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/12/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: 3833cbfd0802f334e482203d269984eb0e299797
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895628"
---
# <a name="secure-an-input-constrained-device-with-azure-ad-and-azure-maps-rest-apis"></a>Sécuriser un appareil d’entrée contraint avec des API REST Azure Maps et Azure AD

Ce guide explique comment sécuriser des applications ou des appareils publics qui ne peuvent pas des secrets stocker de manière sécurisée ou accepter une entrée de navigateur. Ces types d’applications relèvent de la catégorie IoT (Internet des objets). Il s’agit entre autres d’applications telles que des TV intelligentes ou des applications émettant des données de capteur. 

[!INCLUDE [authentication details](./includes/view-authentication-details.md)]

## <a name="create-an-application-registration-in-azure-ad"></a>Créer une inscription d’application dans Azure AD

> [!NOTE]
> * **Lecture requise :** [Scénario : Application de bureau qui appelle des API web](../active-directory/develop/scenario-desktop-overview.md)
> * Le scénario suivant utilise le flux de code d’appareil, qui n’implique pas de navigateur web pour obtenir un jeton.

Créez l’application basée sur un appareil dans Azure AD pour activer la connexion Azure AD. Cette application sera autorisée à accéder aux API REST Azure Maps.

1. Dans le portail Azure, dans la liste des services Azure, sélectionnez **Azure Active Directory** > **Inscriptions d’applications** > **Nouvelle inscription** .  

    > [!div class="mx-imgBorder"]
    > ![Inscription d’application](./media/how-to-manage-authentication/app-registration.png)

2. Entrez un **Nom** et choisissez **Comptes dans cet annuaire d’organisation uniquement** comme **Type de compte pris en charge** . Dans **URI de redirection** , spécifiez **Client public/natif (mobile bureau)** , puis ajoutez `https://login.microsoftonline.com/common/oauth2/nativeclient` à la valeur. Pour plus d’informations, consultez [Azure AD : application de bureau qui appelle des API web : Inscription d’application](../active-directory/develop/scenario-desktop-app-registration.md). Ensuite, **inscrivez** l’application.

    > [!div class="mx-imgBorder"]
    > ![Ajouter les détails de l’inscription d’application pour le nom et l’URI de redirection](./media/azure-maps-authentication/devicecode-app-registration.png)

3. Accédez à **Authentification** et activez **Considérer l’application comme un client public** . Cela active l’authentification du code d’appareil auprès d’Azure AD.
    
    > [!div class="mx-imgBorder"]
    > ![Activer l’inscription d’application en tant que client public](./media/azure-maps-authentication/devicecode-public-client.png)

4.  Pour affecter des autorisations d’API déléguées à Azure Maps, accédez à l’application. Sélectionnez **Autorisations des API** > **Ajouter une autorisation** . Sous **API utilisées par que mon organisation** , recherchez et sélectionnez **Azure Maps** .

    > [!div class="mx-imgBorder"]
    > ![Ajouter des autorisations d’API d’application](./media/how-to-manage-authentication/app-permissions.png)

5. Cochez la case en regard de **Accéder à Azure Maps** , puis sélectionnez **Ajouter des autorisations** .

    > [!div class="mx-imgBorder"]
    > ![Sélectionner des autorisations d’API d’application](./media/how-to-manage-authentication/select-app-permissions.png)

6. Configurez le contrôle d’accès en fonction du rôle Azure (Azure RBAC) pour les utilisateurs ou les groupes. Consultez [Accorder l’accès en fonction du rôle pour des utilisateurs à Azure Maps](#grant-role-based-access-for-users-to-azure-maps).

7. Ajoutez du code pour l’acquisition du flux de jeton dans l’application. Pour plus d’informations sur l’implémentation, consultez [Flux de code d’appareil](../active-directory/develop/scenario-desktop-acquire-token.md#device-code-flow). Lors de l’acquisition des jetons, référencez l’étendue : `user_impersonation` qui a été sélectionnée lors des étapes précédentes.

> [!Tip]
> Utilisez la bibliothèque d’authentification Microsoft (MSAL) pour acquérir des jetons d’accès. Consultez les recommandations fournies dans [Applications de bureau qui appelle des API web : Configuration de code](../active-directory/develop/scenario-desktop-app-configuration.md)

8. Composez la requête HTTP avec le jeton obtenu auprès d’Azure AD et envoyez la requête avec un client HTTP valide.

### <a name="sample-request"></a>Exemple de requête
Voici un exemple de corps de requête pour le chargement d’une limite géographique simple représentée sous la forme d’un cercle avec un point central et un rayon.

```http
POST /mapData/upload?api-version=1.0&dataFormat=geojson
Host: atlas.microsoft.com
x-ms-client-id: 30d7cc….9f55
Authorization: Bearer eyJ0e….HNIVN
```

 L’exemple de corps de requête ci-dessous est au format GeoJSON :
```json
{
    "type": "FeatureCollection",
    "features": [{
        "type": "Feature",
        "geometry": {
            "type": "Point",
            "coordinates": [-122.126986, 47.639754]
        },
        "properties": {
            "geometryId": "001",
            "radius": 500
        }
    }]
}
```

### <a name="sample-response"></a>Exemple de réponse :

Headers:
```http
Location: https://atlas.microsoft.com/mapData/metadata/{udid}?api-version=1.0
Access-Control-Expose-Headers: Location
```

Corps :
```json
{
  "operationId": "{operationId}",
  "status": "Succeeded",
  "created": "2020-01-02 1:02:03 AM +00:00",
  "resourceLocation": "https://atlas.microsoft.com/mapData/metadata/{resourceId}?api-version=1.0"
}
```

[!INCLUDE [grant role-based access to users](./includes/grant-rbac-users.md)]

## <a name="next-steps"></a>Étapes suivantes

Recherchez les métriques d’utilisation de l’API pour votre compte Azure Maps :
> [!div class="nextstepaction"]
> [Afficher les métriques d’utilisation](how-to-view-api-usage.md)