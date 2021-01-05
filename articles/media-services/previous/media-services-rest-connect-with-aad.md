---
title: Utiliser l’authentification Azure AD pour accéder à l’API Azure Media Services avec REST | Microsoft Docs
description: Découvrez comment accéder à une API Azure Media Services avec l’authentification Azure Active Directory par le biais de REST.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2019
ms.author: juliako
ms.reviewer: willzhan; johndeu
ms.openlocfilehash: 8bea4c049c3d7ea17e173f069a3e99cbcca1fe48
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93041986"
---
# <a name="use-azure-ad-authentication-to-access-the-media-services-api-with-rest"></a>Utiliser l’authentification Azure AD pour accéder à l’API Media Services avec REST

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Lorsque vous utilisez l’authentification Azure AD avec Azure Media Services, vous pouvez vous authentifier de deux manières :

- L’ **authentification utilisateur** authentifie une personne qui utilise l’application pour interagir avec les ressources Azure Media Services. L’application interactive invite tout d’abord l’utilisateur à entrer ses informations d’identification. Par exemple, une application de console de gestion peut être utilisée par les utilisateurs autorisés pour contrôler les travaux d’encodage ou de streaming en direct. 
- L’ **authentification de principal de service** authentifie un service. Les applications qui utilisent généralement cette méthode d’authentification sont des applications qui exécutent des services démon, des services de niveau intermédiaire ou des travaux planifiés (par exemple, applications web, applications de fonction, applications logiques, API ou microservices).

    Ce didacticiel explique comment utiliser l’authentification de **principal de service** Azure AD pour accéder aux API AMS avec REST. 

    > [!NOTE]
    > Le **principal de service** est la meilleure pratique recommandée pour la plupart des applications qui se connectent à Azure Media Services. 

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Obtenir les informations d’authentification à partir du portail Azure
> * Obtenir le jeton d’accès à l’aide de Postman
> * Tester l’API **Assets** à l’aide du jeton d’accès


> [!IMPORTANT]
> À l’heure actuelle, Media Services prend en charge le modèle d’authentification des services Azure Access Control. Toutefois, l’authentification Access Control sera déconseillée à compter du 1er juin 2018. Nous vous recommandons de migrer vers le modèle d’authentification Azure AD dès que possible.

## <a name="prerequisites"></a>Conditions préalables requises

- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) avant de commencer.
- [Créez un compte Azure Media Services avec le portail Azure](media-services-portal-create-account.md).
- Consultez l’article [Accéder à l’API Azure Media Services avec l’authentification Azure AD](media-services-use-aad-auth-to-access-ams-api.md).
- Installez le client REST [Postman](https://www.getpostman.com/) pour exécuter les API REST indiquées dans cet article. 

    Dans ce tutoriel, nous utilisons **Postman** , mais tout autre outil REST serait approprié. Les autres solutions sont : **Visual Studio Code** avec le plug-in REST ou **Telerik Fiddler**. 

## <a name="get-the-authentication-information-from-the-azure-portal"></a>Obtenir les informations d’authentification à partir du portail Azure

### <a name="overview"></a>Vue d’ensemble

Pour accéder aux API Media Services, vous devez collecter des points de données suivants.

|Paramètre|Exemple|Description|
|---|-------|-----|
|Domaine du locataire Azure Active Directory|microsoft.onmicrosoft.com|Azure AD en tant que point de terminaison Secure Token Service (STS) est créé au format suivant : <https://login.microsoftonline.com/{your-ad-tenant-name.onmicrosoft.com}/oauth2/token>. Azure AD émet un JWT pour accéder aux ressources (jeton d’accès).|
|Point de terminaison d'API REST|<https://amshelloworld.restv2.westus.media.azure.net/api/>|Il s’agit du point de terminaison vis-à-vis duquel tous les appels d’API REST Media Services dans votre application sont effectués.|
|ID client (ID d’application)|f7fbbb29-a02d-4d91-bbc6-59a2579259d2|ID (client) d’application Azure AD. L’ID client est nécessaire pour obtenir le jeton d’accès. |
|Clé secrète client|+mUERiNzVMoJGggD6aV1etzFGa1n6KeSlLjIq+Dbim0=|Clés d’application Azure AD (clé secrète client). La clé secrète client est nécessaire pour obtenir le jeton d’accès.|

### <a name="get-azure-active-directory-auth-info-from-the-azure-portal"></a>Récupérer les informations d’authentification Azure Active Directory depuis le portail Azure

Pour obtenir les informations, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Accédez à votre instance AMS.
3. Sélectionnez **Accès d’API**.
4. Cliquez sur **Se connecter à l'API Azure Media Services avec le principal de service**.

    ![Capture d’écran montrant l’élément « Accès d’API » sélectionné dans le menu « Media Services », et l’option « Se connecter à l’API Azure Media Services avec le principal de service » sélectionnée dans le volet droit.](./media/connect-with-rest/connect-with-rest01.png)

5. Sélectionnez une **application Azure AD** existante ou créez-en une (voir ci-dessous).

    > [!NOTE]
    > Pour que la demande Azure Media REST réussisse, l’utilisateur appelant doit avoir un rôle **Collaborateur** ou **Propriétaire** pour le compte Media Services auquel il tente d’accéder. S’il obtient une exception avec un message du type « Le serveur distant a retourné une erreur : (401) Non autorisé », consultez [Contrôle d’accès](media-services-use-aad-auth-to-access-ams-api.md#access-control).

    Si vous devez créer une application AD, procédez comme suit :
    
   1. Cliquez sur **Créer**.
   2. Entrez un nom.
   3. Cliquez de nouveau sur **Créer**.
   4. Appuyez sur **Enregistrer**.

      ![Capture d’écran montrant la boîte de dialogue « Créer » avec la zone de texte « Créer une application » mise en évidence, et le bouton « Enregistrer » sélectionné.](./media/connect-with-rest/new-app.png)

      La nouvelle application s’affiche sur la page.

6. Obtenez l’ **ID client** (ID d’application).
    
   1. Sélectionnez l’application.
   2. Obtenez l’ **ID client** dans la fenêtre de droite. 

      ![Capture d’écran montrant les éléments « Application Azure AD » et « Gérer l’application » sélectionnés, ainsi que l’« ID client » mis en évidence dans le volet droit.](./media/connect-with-rest/existing-client-id.png)

7. Obtenez la **clé** de l’application (clé secrète client). 

   1. Cliquez sur le bouton **Gérer l'application** (notez que les informations sur l’ID client se trouvent sous **ID d’application** ). 
   2. Cliquez sur **Clés**.
    
       ![Capture d’écran montrant le bouton « Gérer l’application » sélectionné, la section « ID de l’application » mise en évidence dans le volet central et l’élément « Clés » sélectionné dans le volet droit.](./media/connect-with-rest/manage-app.png)
   3. Pour générer la clé d’application (clé secrète client), renseignez les champs **DESCRIPTION** et **DATE D’EXPIRATION** et cliquez sur **Enregistrer**.
    
       Après avoir cliqué sur le bouton **Enregistrer** , la valeur de la clé s’affiche. Copiez la valeur de la clé avant de quitter le panneau.

   ![Accès d’API](./media/connect-with-rest/connect-with-rest03.png)

Vous pouvez ajouter à votre fichier web.config ou app.config des valeurs pour les paramètres de connexion AD, en vue d’une utilisation ultérieure dans votre code.

> [!IMPORTANT]
> La **clé client** est une information confidentielle importante qui doit être correctement sécurisée dans un coffre de clés ou chiffrée en production.

## <a name="get-the-access-token-using-postman"></a>Obtenir le jeton d’accès à l’aide de Postman

Cette section explique comment utiliser **Postman** pour exécuter une API REST qui retourne un jeton de porteur JWT (jeton d’accès). Pour appeler une API REST Media Services, vous devez ajouter l’en-tête « d’autorisation » pour les appels, et ajouter la valeur de « Porteur *votre_jeton_d’accès* » pour chaque appel (comme indiqué dans la section suivante de ce didacticiel). 

1. Ouvrez **Postman**.
2. Sélectionnez **POST**.
3. Entrez l’URL qui inclut votre nom de locataire en utilisant le format suivant : le nom du locataire doit se terminer par **. onmicrosoft.com** et l’URL doit se terminer par **oauth2/token** : 

    `https://login.microsoftonline.com/{your-aad-tenant-name.onmicrosoft.com}/oauth2/token`

4. Sélectionnez l’onglet **En-têtes**.
5. Entrez les informations sur les **en-têtes** à l’aide de la grille de données « Clé/Valeur ». 

    ![Capture d’écran montrant l’onglet « Headers » et l’action « Bulk Edit » sélectionnés.](./media/connect-with-rest/headers-data-grid.png)

    Vous pouvez également cliquer sur le lien **Modification en bloc** situé à droite de la fenêtre Postman et coller le code suivant.

    ```javascript
    Content-Type:application/x-www-form-urlencoded
    Keep-Alive:true
    ```

6. Cliquez sur l’onglet **Corps**.
7. Entrez les informations sur le corps à l’aide de la grille de données « Clé/valeur » (remplacez l’ID client et les valeurs des secrets). 

    ![Grille de données](./media/connect-with-rest/data-grid.png)

    Vous pouvez également cliquer sur le lien **Modification en bloc** situé à droite de la fenêtre Postman et coller le corps suivant (remplacez l’ID client et les valeurs des secrets) :

    ```javascript
    grant_type:client_credentials
    client_id:{Your Client ID that you got from your Azure AD Application}
    client_secret:{Your client secret that you got from your Azure AD Application's Keys}
    resource:https://rest.media.azure.net
    ```

8. Appuyez sur **Envoyer**.

    ![Capture d’écran montrant la zone de texte « Post », les onglets « Headers » et « Body », et l’élément « access_token » mis en évidence, ainsi que le bouton « Send » détecté.](./media/connect-with-rest/connect-with-rest04.png)

La réponse retournée contient le **jeton d’accès** que vous devez utiliser pour accéder à toute API AMS.

## <a name="test-the-assets-api-using-the-access-token"></a>Tester l’API **Assets** à l’aide du jeton d’accès

Cette section explique comment accéder à l’API **Assets** à l’aide de **Postman**.

1. Ouvrez **Postman**.
2. Sélectionnez **GET**.
3. Collez le point de terminaison de l’API REST (par exemple, https://amshelloworld.restv2.westus.media.azure.net/api/Assets)
4. Sélectionnez l’onglet **Autorisation**. 
5. Sélectionnez le type **jeton du porteur**.
6. Collez le jeton créé dans la section précédente.

    ![Obtention d’un jeton](./media/connect-with-rest/connect-with-rest05.png)

    > [!NOTE]
    > L’expérience utilisateur Postman peut varier entre un Mac et un PC. Si la version Mac ne propose pas d’option de « jeton du porteur » dans liste déroulante d’ **authentification** , vous devez ajouter l’en-tête d’ **autorisation** manuellement sur le client Mac.

   ![En-tête d’autorisation](./media/connect-with-rest/auth-header.png)

7. Sélectionnez **En-têtes**.
5. Cliquez sur le lien **Modification en bloc** situé à droite de la fenêtre Postman.
6. Collez les en-têtes suivants :

    ```javascript
    x-ms-version:2.19
    Accept:application/json
    Content-Type:application/json
    DataServiceVersion:3.0
    MaxDataServiceVersion:3.0
    ```

7. Appuyez sur **Envoyer**.

La réponse retournée contient les ressources qui se trouvent dans votre compte.

## <a name="next-steps"></a>Étapes suivantes

* Essayez l’exemple de code qui se trouve dans [Azure AD Authentication for Azure Media Services Access: Both via REST API](https://github.com/willzhan/WAMSRESTSoln) (Authentification Azure AD pour l’accès à Azure Media Services : par le biais d’une API REST)
* [Charger des fichiers à l’aide de .NET](media-services-dotnet-upload-files.md)
