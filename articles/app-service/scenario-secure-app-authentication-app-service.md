---
title: 'Tutoriel : Ajouter l’authentification à une application web sur Azure App Service | Azure'
description: Dans ce tutoriel, vous allez apprendre à activer l’authentification et l’autorisation pour une application web s’exécutant sur Azure App Service. Limiter l’accès à l’application web aux utilisateurs de votre organisation.
services: active-directory, app-service-web
author: rwike77
manager: CelesteDG
ms.service: app-service-web
ms.topic: tutorial
ms.workload: identity
ms.date: 11/09/2020
ms.author: ryanwi
ms.reviewer: stsoneff
ms.custom: azureday1
ms.openlocfilehash: a8bd2ef1348692bf57f7e5cb7b6606cfcfd324fe
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905568"
---
# <a name="tutorial-add-authentication-to-your-web-app-running-on-azure-app-service"></a>Tutoriel : Ajouter l’authentification à votre application web s’exécutant sur Azure App Service

Découvrez comment activer l’authentification pour votre application web s’exécutant sur Azure App Service et limiter l’accès aux utilisateurs de votre organisation.

:::image type="content" source="./media/scenario-secure-app-authentication-app-service/web-app-sign-in.svg" alt-text="Diagramme illustrant la connexion de l’utilisateur." border="false":::

App Service offre une prise en charge intégrée de l’authentification et de l’autorisation, qui vous permet de connecter les utilisateurs et d’accéder aux données sans avoir à écrire beaucoup de code dans votre application web. L’utilisation du module d’authentification/autorisation App Service n’est pas obligatoire, mais elle permet de simplifier l’authentification et l’autorisation de votre application. Cet article montre comment sécuriser votre application web avec le module d’authentification/autorisation App Service, en utilisant Azure Active Directory (Azure AD) comme fournisseur d’identité.

Le module d’authentification/autorisation est activé et configuré par le biais des paramètres d’application et du portail Azure. Aucun kit SDK, aucun langage spécifique ni aucune modification du code de l’application ne sont nécessaires. De nombreux fournisseurs d’identité sont pris en charge, notamment Azure AD, compte Microsoft, Facebook, Google et Twitter. Lorsque le module d’authentification/autorisation est activé, chaque requête HTTP entrante le traverse avant d’être gérée par le code de l’application. Pour plus d’informations, consultez [Authentification et autorisation dans Azure App Service](overview-authentication-authorization.md).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
>
> * Configurer l’authentification pour l’application web.
> * Limiter l’accès à l’application web aux utilisateurs de votre organisation.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-and-publish-a-web-app-on-app-service"></a>Créer et publier une application web sur App Service.

Pour ce tutoriel, vous avez besoin d’une application web déployée sur App Service. Vous pouvez utiliser une application web existante, ou vous pouvez suivre le [guide de démarrage rapide ASP.NET Core](quickstart-dotnetcore.md) pour créer et publier une nouvelle application web sur App Service.

Que vous utilisiez une application web existante ou que vous en créiez une nouvelle, prenez note du nom de l’application web et du nom du groupe de ressources sur lequel l’application web est déployée. Vous aurez besoin de ces noms tout au long de ce tutoriel. Dans ce tutoriel, les exemples de noms dans les procédures et les captures d’écran contiennent *SecureWebApp*.

## <a name="configure-authentication-and-authorization"></a>Configurer l’authentification et l’autorisation

Vous disposez maintenant d’une application web s’exécutant sur App Service. pour l’application web s’exécutant sur App Service. Vous utilisez Azure AD comme fournisseur d’identité. Pour plus d’informations, consultez [Configurer l’authentification Azure AD pour votre application App Service](configure-authentication-provider-aad.md).

Dans le menu du [portail Azure](https://portal.azure.com), sélectionnez **Groupes de ressources** ou recherchez et sélectionnez **Groupes de ressources** dans n’importe quelle page.

Dans **Groupes de ressources**, recherchez et sélectionnez votre groupe de ressources. Dans **Vue d’ensemble**, sélectionnez la page de gestion de l’application.

:::image type="content" alt-text="Capture d’écran montrant la sélection de la page de gestion de votre application." source="./media/scenario-secure-app-authentication-app-service/select-app-service.png":::

Dans le menu de gauche de l’application, sélectionnez **Authentification/Autorisation**, puis activez l’authentification App Service en sélectionnant **Activé**.

Dans **Mesure à prendre quand une demande n’est pas authentifiée**, sélectionnez **Se connecter avec Azure Active Directory**.

Sous **Fournisseurs d’authentification**, cliquez sur **Azure Active Directory**. Sélectionnez **Express**, puis acceptez les paramètres par défaut pour créer une application Active Directory. Sélectionnez **OK**.

:::image type="content" alt-text="Capture d’écran montrant l’authentification Express." source="./media/scenario-secure-app-authentication-app-service/configure-authentication.png":::

Dans la page **Authentification/Autorisation**, sélectionnez **Enregistrer**.

Lorsque vous voyez la notification avec le message `Successfully saved the Auth Settings for <app-name> App`, actualisez la page du portail.

Vous disposez maintenant d’une application sécurisée par l’authentification et l’autorisation App Service.

## <a name="verify-limited-access-to-the-web-app"></a>Vérifier l’accès limité à l’application web

Lorsque vous avez activé le module d’authentification/autorisation App Service, une inscription d’application a été créée dans votre locataire Azure AD. L’inscription d’application a le même nom d’affichage que votre application web. Pour vérifier les paramètres, sélectionnez **Azure Active Directory** dans le menu du portail, puis **Inscriptions d’applications**. Sélectionnez l’inscription d’application qui a été créée. Dans la vue d’ensemble, vérifiez que **Types de comptes pris en charge** a la valeur **Mon organisation uniquement**.

:::image type="content" alt-text="Capture d’écran montrant la vérification de l’accès." source="./media/scenario-secure-app-authentication-app-service/verify-access.png":::

Pour vérifier que l’accès à votre application est limité aux utilisateurs de votre organisation, démarrez un navigateur en mode Incognito ou privé, puis accédez à `https://<app-name>.azurewebsites.net`. Vous devez être dirigé vers une page de connexion sécurisée, qui permet de s’assurer que les utilisateurs non authentifiés ne sont pas autorisés à accéder au site. Connectez-vous en tant qu’utilisateur de votre organisation pour accéder au site. Vous pouvez également démarrer un nouveau navigateur et essayer de vous connecter à l’aide d’un compte personnel afin de vérifier que les utilisateurs en dehors de l’organisation n’ont pas accès.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez terminé ce tutoriel et que vous n’avez plus besoin de l’application web ni des ressources associées, [nettoyez les ressources que vous avez créées](scenario-secure-app-clean-up-resources.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
>
> * Configurer l’authentification pour l’application web.
> * Limiter l’accès à l’application web aux utilisateurs de votre organisation.

> [!div class="nextstepaction"]
> [Accès d’une application de service au stockage](scenario-secure-app-access-storage.md)
