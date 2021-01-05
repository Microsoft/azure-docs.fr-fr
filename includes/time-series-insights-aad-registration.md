---
title: Fichier include
description: Fichier include
ms.topic: include
ms.custom: include file
services: time-series-insights
ms.service: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: cshankar
ms.date: 10/02/2020
ms.openlocfilehash: 50f203357d29d450f5b34593952f611f025b14ff
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95559890"
---
1. Dans le [portail Azure](https://ms.portal.azure.com/), sélectionnez **Azure Active Directory** > **Inscriptions des applications** > **Nouvelle inscription**.

   [![Nouvelle inscription d’application dans Azure Active Directory](media/time-series-insights-aad-registration/active-directory-new-application-registration.png)](media/time-series-insights-aad-registration/active-directory-new-application-registration.png#lightbox)

    Votre application est répertoriée ici une fois que vous l’enregistrez.

1. Donnez à l’application un nom et sélectionnez **Comptes dans cet annuaire organisationnel uniquement** pour spécifier les **Types de comptes pris en charge** qui peuvent accéder à l’API. Si vous créez une [application cliente publique](../articles/active-directory/develop/msal-client-application-configuration.md#redirect-uri), ajoutez un URI de redirection valide, puis sélectionnez **Inscrire**.

   [![Créer l’application dans Azure Active Directory](media/time-series-insights-aad-registration/active-directory-registration.png)](media/time-series-insights-aad-registration/active-directory-registration.png#lightbox)

1. Des informations importantes sur l’application Azure Active Directory s’affichent dans le panneau **Vue d’ensemble** de votre application répertoriée. Sélectionnez votre application sous **Applications possédées**, puis **Vue d’ensemble**.

   [![Copier l’ID de l’application](media/time-series-insights-aad-registration/active-directory-copy-application-id.png)](media/time-series-insights-aad-registration/active-directory-copy-application-id.png#lightbox)

   Copie votre **ID d’application (client)** à utiliser dans votre application cliente.

1. Le panneau **Authentification** spécifie les paramètres importants de configuration d’authentification.

    1. Ajoutez des **URI de redirection** et configurez des **Jetons d’accès** en sélectionnant **+ Ajouter une plateforme**.

    1. Spécifiez si l’application est un **client public** en sélectionnant **Oui** ou **Non**.

    1. Vérifiez quels comptes et locataire sont pris en charge.

    [![Configurer l’octroi implicite](media/time-series-insights-aad-registration/active-directory-auth-blade.png)](media/time-series-insights-aad-registration/active-directory-auth-blade.png#lightbox)

1. Après avoir sélectionné la plateforme appropriée, configurez vos **URI de redirection** et **Jetons d’accès** dans le volet latéral situé à droite de l’interface utilisateur.

    1. **L’URI de redirection** doit correspondre à l’adresse fournie par la requête d’authentification :

        * Pour les applications hébergées dans un environnement de développement local, sélectionnez **Client public (mobile et bureau)** . Veillez à affecter la valeur **Oui** à **Client public**.
        * Pour les applications monopages hébergées sur Azure App Service, sélectionnez **Web**.

    1. Déterminez si une **URL de déconnexion** est appropriée.

    1. Activez le flux d’octroi implicite en cochant les **Jeton d'accès** ou **Jetons d'ID**.

    [![Créer des URI de redirection](media/time-series-insights-aad-registration/active-directory-auth-redirect-uri.png)](media/time-series-insights-aad-registration/active-directory-auth-redirect-uri.png#lightbox)

    Cliquez sur **Configurer**, puis sur **Enregistrer**.

1. Sélectionnez **Certificats et secrets**, puis **Nouveau secret client** pour créer un mot de passe d’application que votre application cliente peut utiliser pour prouver son identité.

   [![Créer une nouvelle clé secrète client](media/time-series-insights-aad-registration/active-directory-application-keys-save.png)](media/time-series-insights-aad-registration/active-directory-application-keys-save.png#lightbox)

   Le mot de passe de votre secret client est alors affiché. Copiez la clé dans votre éditeur de texte.

   > [!NOTE]
   > Vous avez la possibilité d’importer un certificat à la place. Pour une sécurité renforcée, un certificat est recommandé. Pour utiliser un certificat, sélectionnez **Charger un certificat**.

1. Associez votre application Azure Active Directory à Azure Time Series Insights. Sélectionnez **Autorisations de l’API** > **Ajouter une autorisation** > **API que mon organisation utilise**.

    [![Associer une API à votre application Azure Active Directory](media/time-series-insights-aad-registration/active-directory-app-api-permission.png)](media/time-series-insights-aad-registration/active-directory-app-api-permission.png#lightbox)

   Saisissez `Azure Time Series Insights` dans la barre de recherche, puis sélectionnez `Azure Time Series Insights`.

1. Ensuite, spécifiez le genre d’autorisation d’API exigé par votre application. Par défaut, **Autorisations déléguées** est en surbrillance. Choisissez un type d’autorisation puis, sélectionnez **Ajouter les autorisations**.

    [![Spécifier le type d’autorisation d’API que votre application exige](media/time-series-insights-aad-registration/active-directory-app-permission-grant.png)](media/time-series-insights-aad-registration/active-directory-app-permission-grant.png#lightbox)