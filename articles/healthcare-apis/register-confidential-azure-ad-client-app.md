---
title: Inscrire une application cliente confidentielle dans Azure AD - API Azure pour FHIR
description: Inscrire une application cliente confidentielle dans Azure Active Directory qui s’authentifie au nom d’un utilisateur et demande l’accès aux applications de ressources.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/07/2019
ms.author: matjazl
ms.openlocfilehash: 8021fb3fa9f11ef895569f48a2ae21b3f7adcd36
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91826225"
---
# <a name="register-a-confidential-client-application-in-azure-active-directory"></a>Inscrire une application cliente confidentielle dans Azure Active Directory

Dans ce tutoriel, vous découvrirez comment inscrire une application cliente confidentielle dans Azure Active Directory. 

L’inscription d’une application cliente est une représentation Azure Active Directory d’une application qui peut être utilisée pour authentifier au nom d’un utilisateur et demander l’accès aux [applications de ressources](register-resource-azure-ad-client-app.md). Une application cliente confidentielle est une application qui peut être approuvée pour garder un secret et présenter ce secret lors de la demande de jetons d’accès. Ces exemples d’applications confidentielles sont des applications côté serveur.

Pour inscrire une nouvelle application confidentielle dans le portail, suivez ces étapes.

## <a name="register-a-new-application"></a>Inscrire une nouvelle application

1. Dans le [portail Azure](https://portal.azure.com), accédez à **Azure Active Directory**.

1. Sélectionnez **Inscriptions d’applications**.

    ![Portail Azure. Nouvelle inscription d’application.](media/how-to-aad/portal-aad-new-app-registration.png)

1. Sélectionnez **Nouvelle inscription**.

1. Donnez un nom d’affichage à l’application.

1. Fournissez une URL de réponse. Ces détails peuvent être modifiés ultérieurement, mais si vous connaissez l’URL de réponse de votre application, saisissez-la maintenant.

    ![Nouvelle inscription d’application cliente confidentielle.](media/how-to-aad/portal-aad-register-new-app-registration-CONF-CLIENT.png)
1. Sélectionnez **Inscription**.

## <a name="api-permissions"></a>Autorisations des API

Maintenant que vous avez enregistré votre application, vous devez sélectionner quelles autorisations des API cette application peut demander au nom des utilisateurs :

1. Sélectionnez **Autorisations de l’API**.

    ![Client confidentiel. Autorisations des API](media/how-to-aad/portal-aad-register-new-app-registration-CONF-CLIENT-API-Permissions.png)

1. Sélectionnez **Ajouter une autorisation**.

    Si vous utilisez l’API Azure pour FHIR, vous devez ajouter une autorisation aux API Azure Healthcare en recherchant les **API Azure Healthcare** sous **API utilisées par mon organisation**. Vous ne pouvez le trouver que si vous avez déjà [déployé l’API Azure pour FHIR](fhir-paas-powershell-quickstart.md).

    Si vous référencez une autre application de ressource, sélectionnez l’[inscription d’application de ressource d’API FHIR](register-resource-azure-ad-client-app.md) que vous avez créée sous **Mes API**.


    :::image type="content" source="media/conf-client-app/confidential-client-org-api.png" alt-text="Client confidentiel. API Mon organisation" lightbox="media/conf-client-app/confidential-app-org-api-expanded.png":::
    

3. Sélectionnez les étendues (autorisations) que l’application confidentielle doit pouvoir demander au nom d’un utilisateur :

    :::image type="content" source="media/conf-client-app/confidential-client-add-permission.png" alt-text="Client confidentiel. API Mon organisation":::

## <a name="application-secret"></a>Secret d’application

1. Cliquez sur **Certificats et secrets**.
1. Sélectionnez **Nouveau secret client**. 

    ![Client confidentiel. Secret d’application](media/how-to-aad/portal-aad-register-new-app-registration-CONF-CLIENT-SECRET.png)

2. Fournissez une description et la durée du secret (soit 1 an, soit 2 ans, soit jamais).

3. Une fois généré, il ne s’affichera qu’une seule fois dans le portail. Notez-le et stockez-le en toute sécurité.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert comment inscrire une application cliente confidentielle dans Azure Active Directory. Ensuite vous pouvez accéder à votre serveur FHIR à l’aide de Postman
 
>[!div class="nextstepaction"]
>[Accéder à l’API Azure pour FHIR avec Postman](access-fhir-postman-tutorial.md)
