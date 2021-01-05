---
title: Inscrire une application de service dans Azure AD - API Azure pour FHIR
description: Découvrez comment inscrire une application cliente de service dans Azure Active Directory.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/07/2019
ms.author: matjazl
ms.openlocfilehash: 6f7bf122b292ca144eac406957f19a13c7ba6662
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91975822"
---
# <a name="register-a-service-client-application-in-azure-active-directory"></a>Inscrire une application cliente de service dans Azure Active Directory

Dans cet article, vous découvrez comment inscrire une application cliente de service dans Azure Active Directory. Les inscriptions d’applications clientes sont des représentations Azure Active Directory des applications qui peuvent être utilisées pour l’authentification et l’obtention de jetons. Un client de service est destiné à être utilisé par une application pour obtenir un jeton d’accès sans authentification interactive d’un utilisateur. Il disposera de certaines permissions d’application et utilisera un secret d’application (mot de passe) pour l’obtention de jetons d’accès.

Suivez cette procédure pour créer un client de service.

## <a name="app-registrations-in-azure-portal"></a>Inscriptions d’applications dans le portail Azure

1. Dans le [portail Azure](https://portal.azure.com), accédez à **Azure Active Directory**.

2. Sélectionnez **Inscriptions d’applications**.

    ![Portail Azure. Nouvelle inscription d’application.](media/how-to-aad/portal-aad-new-app-registration.png)

3. Sélectionnez **Nouvelle inscription**.

4. Attribuez un nom d’affichage au client de service. En général, les applications clientes de service n’utilisent pas d’URL de réponse.

    :::image type="content" source="media/service-client-app/service-client-registration.png" alt-text="Portail Azure. Nouvelle inscription d’application cliente de service.":::

5. Sélectionnez **Inscription**.

## <a name="api-permissions"></a>Autorisations des API

Maintenant que vous avez enregistré votre application, vous devez sélectionner quelles autorisations des API cette application peut demander au nom des utilisateurs :

1. Sélectionnez **Autorisations de l’API**.
1. Sélectionnez **Ajouter une autorisation**.

    Si vous utilisez l’API Azure pour FHIR, vous devez ajouter une autorisation aux API Azure Healthcare en recherchant les **API Azure Healthcare** sous **API utilisées par mon organisation**. 

    Si vous référencez une autre application de ressource, sélectionnez l’[inscription d’application de ressource d’API FHIR](register-resource-azure-ad-client-app.md) que vous avez créée sous **Mes API**.

    :::image type="content" source="media/service-client-app/service-client-org-api.png" alt-text="Portail Azure. Nouvelle inscription d’application cliente de service." lightbox="media/service-client-app/service-client-org-api-expanded.png":::

1. Sélectionnez les étendues (autorisations) que l’application confidentielle doit pouvoir demander au nom d’un utilisateur :

    :::image type="content" source="media/service-client-app/service-client-add-permission.png" alt-text="Portail Azure. Nouvelle inscription d’application cliente de service.":::

1. Accordez les autorisations de l’application. Si vous ne disposez pas des autorisations nécessaires, contactez votre administrateur Active Directory Azure :

    :::image type="content" source="media/service-client-app/service-client-grant-permission.png" alt-text="Portail Azure. Nouvelle inscription d’application cliente de service.":::

## <a name="application-secret"></a>Secret d’application

Le client de service a besoin d’un secret (mot de passe) pour obtenir un jeton.

1. Cliquez sur **Certificats et secrets**.
2. Sélectionnez **Nouveau secret client**.

    ![Portail Azure. Clé secrète client de service](media/how-to-aad/portal-aad-register-new-app-registration-SERVICE-CLIENT-SECRET.png)

3. Fournissez une description et la durée du secret (soit 1 an, soit 2 ans, soit jamais).

4. Après avoir été généré, le secret sera affiché une seule fois dans le portail. Notez-le et stockez-le en toute sécurité.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à inscrire une application cliente de service dans Azure Active Directory. Testez ensuite l’accès à votre serveur FHIR à l’aide de Postman.
 
>[!div class="nextstepaction"]
>[Accéder à l’API Azure pour FHIR avec Postman](access-fhir-postman-tutorial.md)
