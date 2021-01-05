---
title: Tutoriel d’application web – Configuration de l’application cliente
description: Ce tutoriel vous accompagne tout au long des étapes d’inscription d’une application publique en vue de préparer le déploiement d’une application web
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: matjazl
ms.author: cavoeg
author: caitlinv39
ms.date: 01/03/2020
ms.openlocfilehash: bf357dcc9c4790632a1131278edc41c13ac810b1
ms.sourcegitcommit: 7e97ae405c1c6c8ac63850e1b88cf9c9c82372da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97802607"
---
# <a name="client-application-registration"></a>Inscription d’une application cliente
Dans le tutoriel précédent, vous avez déployé et configuré votre API Azure pour FHIR. Maintenant que votre API Azure pour FHIR est configurée, nous allons inscrire une application cliente publique. Vous pouvez parcourir entièrement le guide pratique qui explique comment [inscrire une application cliente publique](register-public-azure-ad-client-app.md) ou comment résoudre les problèmes, mais nous en avons extrait les principales étapes dans le tutoriel ci-dessous.

1. Accéder à Azure Active Directory
1. Sélectionnez **Inscription des applications** --> **Nouvelle inscription**.
1. Nommer votre application
1. Sélectionnez **Client public/natif (Bureau et mobile)** et définissez l’URI de redirection sur `https://www.getpostman.com/oauth2/callback`.

   :::image type="content" source="media/tutorial-web-app/register-public-app.png" alt-text="Capture d’écran du volet Inscrire une application avec des exemples de nom d’application et d’URL de redirection":::

## <a name="client-application-settings"></a>Paramètres de l’application cliente

Une fois l’application cliente inscrite, copiez l’ID d’application (client) et l’ID de locataire à partir de la page Vue d’ensemble. Vous aurez besoin de ces deux valeurs par la suite, quand vous accéderez au client.

:::image type="content" source="media/tutorial-web-app/client-id-tenant-id.png" alt-text="Capture d’écran du volet de paramètres de l’application cliente avec les ID d’application et d’annuaire mis en évidence":::

### <a name="connect-with-web-app"></a>Se connecter à une application web

Si vous avez [écrit votre application web](tutorial-web-app-write-web-app.md) pour qu’elle se connecte à l’API Azure pour FHIR, vous devez également définir les options d’authentification correctes. 

1. Dans le menu de gauche, sous **Gérer**, sélectionnez **Authentification**. 

1. Pour ajouter une nouvelle configuration de plateforme, sélectionnez **Web**.

1. Configurez l’URI de redirection en préparation de la création de votre application web dans la quatrième partie de ce tutoriel. Pour cela, ajoutez `https://\<WEB-APP-NAME>.azurewebsites.net` à la liste des URI de redirection. Si vous avez choisi un nom différent au moment d’[écrire votre application web](tutorial-web-app-write-web-app.md), vous devez revenir en arrière et le mettre à jour.

1. Activez les cases à cocher **Jeton d’accès** et **Jeton d’ID**.

   :::image type="content" source="media/tutorial-web-app/web-app-authentication.png" alt-text="Capture d’écran du panneau de paramètres d’authentification d’application avec les étapes d’ajout de plateforme mises en évidence":::

## <a name="add-api-permissions"></a>Ajouter des autorisations d’API

Vous avez configuré l’authentification appropriée. Vous allez à présent définir les autorisations de l’API :

1. Sélectionnez **Autorisations de l’API**, puis cliquez sur **Ajouter une autorisation**.
1. Sous **API utilisées par mon organisation**, effectuez une recherche sur Azure Healthcare APIs.
1. Sélectionnez **user_impersonation**, puis cliquez sur **Ajouter des autorisations**.

:::image type="content" source="media/tutorial-web-app/api-permissions.png" alt-text="Capture d’écran du panneau d’ajout d’autorisations d’API avec les étapes d’ajout d’autorisations d’API mises en évidence":::

## <a name="next-steps"></a>Étapes suivantes
Vous disposez à présent d’une application cliente publique. Dans le prochain tutoriel, nous allons voir en détail comment tester et accéder à cette application via Postman.

>[!div class="nextstepaction"]
>[Tester l’application cliente dans Postman](tutorial-web-app-test-postman.md)
