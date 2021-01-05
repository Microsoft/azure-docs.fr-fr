---
title: Inscrire une application cliente publique dans Azure AD - API Azure pour FHIR
description: Cet article explique comment inscrire une application cliente publique dans Azure Active Directory, en préparation du déploiement de l’API FHIR dans Azure.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/07/2019
ms.author: matjazl
ms.openlocfilehash: ff7bb6084839af56b5f6e874b39929021f23f8a1
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93398077"
---
# <a name="register-a-public-client-application-in-azure-active-directory"></a>Inscrire une application cliente publique dans Azure Active Directory

Dans cet article, vous découvrez comment inscrire une application publique dans Azure Active Directory.  

Les inscriptions d’applications clientes sont des représentations Azure Active Directory des applications qui peuvent être utilisées pour l’authentification et la demande d’autorisations d’API au nom d’un utilisateur. Les clients publics sont des applications telles que des applications mobiles et des applications JavaScript monopages qui ne peuvent pas garder les secrets confidentiels. La procédure est similaire à [l’inscription d’un client confidentiel](register-confidential-azure-ad-client-app.md), mais étant donné que les clients publics ne peut pas être garder un secret d’application, il est inutile d’en ajouter un.

Le guide de démarrage rapide fournit des informations générales sur la façon d’[inscrire une application auprès de la plateforme d’identités Microsoft](../active-directory/develop/quickstart-register-app.md).

## <a name="app-registrations-in-azure-portal"></a>Inscriptions d’applications dans le portail Azure

1. Dans le volet de navigation gauche du [portail Azure](https://portal.azure.com), cliquez sur **Azure Active Directory**.

2. Dans le panneau **Azure Active Directory** , cliquez sur **Inscriptions d’applications**  :

    ![Portail Azure. Nouvelle inscription d’application.](media/how-to-aad/portal-aad-new-app-registration.png)

3. Cliquez sur **Nouvelle inscription**.

## <a name="application-registration-overview"></a>Vue d’ensemble de l’inscription d’application

1. Donnez un nom d’affichage à l’application.

2. Fournissez une URL de réponse. L’URL de réponse est utilisée pour retourner les codes d’authentification à l’application cliente. Vous pouvez ajouter plusieurs URL de réponse et les modifier ultérieurement.

    ![Portail Azure. Nouvelle inscription d’application publique.](media/how-to-aad/portal-aad-register-new-app-registration-PUB-CLIENT-NAME.png)


Pour configurer votre application [de bureau](../active-directory/develop/scenario-desktop-app-registration.md), [mobile](../active-directory/develop/scenario-mobile-app-registration.md) ou [monopage](../active-directory/develop/scenario-spa-app-registration.md) en tant qu’application publique :

1. Dans le [portail Azure](https://portal.azure.com), dans **Inscriptions d’applications** , sélectionnez votre application, puis sélectionnez **Authentification**.

2. Sélectionnez **Paramètres avancés** > **Type de client par défaut**. Pour **Considérer l’application comme un client public** , sélectionnez **Oui**.

3. Pour une application monopage, sélectionnez **Jetons d’accès** et **Jetons d’ID** afin d’activer le flux implicite.

   - Si votre application connecte des utilisateurs, sélectionnez **Jetons d’ID**.
   - Si votre application doit également appeler une API web protégée, sélectionnez **Jetons d’accès**.

## <a name="api-permissions"></a>Autorisations des API

Comme pour [l’application cliente confidentielle](register-confidential-azure-ad-client-app.md), vous devez sélectionner quelles autorisations d’API cette application peut demander au nom des utilisateurs :

1. Ouvrez les **Autorisations de l’API**.

    Si vous utilisez l’API Azure pour FHIR, vous devez ajouter une autorisation aux API Azure Healthcare en recherchant les API Azure Healthcare sous **API utilisées par mon organisation**. Vous ne pouvez le trouver que si vous avez déjà [déployé l’API Azure pour FHIR](fhir-paas-powershell-quickstart.md).

    
    Si vous référencez une autre application de ressource, sélectionnez l’ [inscription d’application de ressource d’API FHIR](register-resource-azure-ad-client-app.md) que vous avez créée sous **Mes API**  :

    ![Portail Azure. Nouvelles autorisations d’API publiques - API Azure pour FHIR (valeur par défaut)](media/public-client-app/api-permissions.png)


2. Sélectionnez les autorisations que l’application doit pouvoir demander : ![Portail Azure. Autorisations d’application](media/public-client-app/app-permissions.png)

## <a name="validate-fhir-server-authority"></a>Valider l’autorité du serveur FHIR
Si l’application que vous avez inscrite dans cet article et votre serveur FHIR se trouvent dans le même locataire Azure AD, vous pouvez passer aux étapes suivantes.

Si vous configurez votre application cliente dans un autre locataire Azure AD que celui de votre serveur FHIR, vous devez mettre à jour l’ **l’autorité**. Dans l’API Azure pour FHIR, vous définissez l’autorité sous Paramètres --> Authentification. Affectez la valeur **https://login.microsoftonline.com/\<TENANT-ID>** à votre autorité.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à inscrire une application cliente publique dans Azure Active Directory. Testez ensuite l’accès à votre serveur FHIR à l’aide de Postman.
 
>[!div class="nextstepaction"]
>[Accéder à l’API Azure pour FHIR avec Postman](access-fhir-postman-tutorial.md)