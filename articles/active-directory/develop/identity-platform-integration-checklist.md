---
title: Meilleures pratiques pour la plateforme d’identité Microsoft | Azure
description: Découvrez les bonnes pratiques, recommandations et les oublis courants lors de l’intégration à la plateforme d’identités Microsoft.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/08/2020
ms.author: ryanwi
ms.reviewer: lenalepa, sureshja, jesakowi
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started
ms.openlocfilehash: 47c3c091e767cac63bb814469397ef1a0aeca3a4
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516504"
---
# <a name="microsoft-identity-platform-best-practices-and-recommendations"></a>Meilleures pratiques et recommandations relatives à la plateforme d’identité Microsoft

Cet article présente les bonnes pratiques, recommandations et les oublis courants lors de l’intégration à la plateforme d’identités Microsoft.  Cette check-list vous guide pour réussir une intégration sécurisée et de haute qualité. Consultez cette liste régulièrement pour vous assurer de maintenir la qualité et la sécurité de l’intégration de votre application à la plateforme d’identité. La check-list n’est pas prévue pour passer en revue l’intégralité de votre application. Le contenu de la check-list est susceptible d’être modifié au fur et à mesure que nous apportons des améliorations à la plateforme.

Si vous venez tout juste de commencer, consultez la [documentation de la plateforme d’identité Microsoft](index.yml) pour en savoir plus sur les principes fondamentaux de l’authentification, les scénarios d’application dans la plateforme d’identités Microsoft, et bien plus encore.

Utilisez la check-list suivante pour vous assurer que votre application est intégrée convenablement à la [plateforme d’identités Microsoft](./index.yml).

> [!TIP]
> L’*Assistant Intégration* dans le Portail Azure peut vous aider à appliquer un grand nombre de ces meilleures pratiques et suggestions. Sélectionnez l’une des [inscriptions d’applications](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade) dans le portail Azure, puis sélectionnez l’élément de menu **Assistant Intégration** pour commencer à utiliser l’Assistant.

## <a name="basics"></a>Concepts de base

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Lisez et comprenez les [Stratégies de la plateforme Microsoft](/legal/microsoft-identity-platform/terms-of-use). Assurez-vous que votre application respecte les conditions décrites, car celles-ci sont conçues pour protéger les utilisateurs et la plateforme.

## <a name="ownership"></a>Propriété

![case à cocher](./media/active-directory-integration-checklist/checkbox-two.svg) Assurez-vous que les informations associées au compte utilisé pour vous inscrire et gérer des applications sont à jour.

## <a name="branding"></a>Personnalisation

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Conformez-vous aux [Directives de personnalisation des applications](howto-add-branding-in-azure-ad-apps.md).

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Fournissez un nom et un logo clairs pour votre application. Ces informations s’affichent dans [l’invite de consentement de votre application](application-consent-experience.md). Assurez-vous que votre nom et le logo associé sont représentatifs de votre société/produit, afin que les utilisateurs puissent se décider en toute connaissance de cause. Assurez-vous que vous ne violez aucune marque commerciale.

## <a name="privacy"></a>Confidentialité

![case à cocher](./media/active-directory-integration-checklist/checkbox-two.svg) Fournissez les liens vers les conditions d’utilisation du service et la déclaration de confidentialité de votre application.

## <a name="security"></a>Sécurité

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Gérez vos URI de redirection : <ul><li>Maintenez la propriété de tous vos URI de redirection et tenez à jour les enregistrements DNS connexes.</li><li>N’utilisez pas les caractères génériques (*) dans vos URI.</li><li>Pour les applications web, assurez-vous que tous les URI sont sécurisés et chiffrés (au moyen, par exemple, de schémas https).</li><li>Pour les clients publics, utilisez les URI de redirection spécifiques à la plateforme, si nécessaire (principalement pour iOS et Android). Sinon, utilisez les URI de redirection avec une grande quantité de caractère aléatoire pour empêcher les collisions lors du rappel à votre application.</li><li>Si votre application est utilisée à partir d’un agent web isolé, vous pouvez utiliser `https://login.microsoftonline.com/common/oauth2/nativeclient`.</li><li>Passez en revue régulièrement tous les URI de redirection et supprimez ceux qui sont inutiles ou inutilisés.</li></ul>

![case à cocher](./media/active-directory-integration-checklist/checkbox-two.svg) Si votre application est inscrite dans un annuaire, réduisez et supervisez manuellement la liste des propriétaires d’inscription d’application.

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) N’activez pas la prise en charge du [flux d’octroi implicite OAuth2](v2-oauth2-implicit-grant-flow.md), sauf si elle est explicitement demandée. Apprenez-en davantage sur le scénario valide [ici](v2-oauth2-implicit-grant-flow.md#suitable-scenarios-for-the-oauth2-implicit-grant).

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Allez au-delà de la combinaison nom d’utilisateur/mot de passe. N’utilisez pas le [flux d’informations d’identification du mot de passe du propriétaire de la ressource (ROPC)](v2-oauth-ropc.md), qui gère directement les mots de passe des utilisateurs. Ce flux nécessite un degré élevé de confiance et d’exposition des utilisateurs ; il devrait être utilisé seulement lorsqu’il est impossible d’utiliser d’autres flux plus sécurisés. Ce flux reste nécessaire dans certains scénarios (comme DevOps), mais il faut savoir que son utilisation peut imposer des contraintes à votre application.  Pour approches plus modernes, consultez [Flux d’authentification et scénarios d’applications](authentication-flows-app-scenarios.md).

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Protégez et gérez vos informations d’identification d’application confidentielles pour les applications web, les API web et les applications de démon. Utilisez les [informations d’identification du certificat](active-directory-certificate-credentials.md), et non pas celles du mot de passe (clés secrètes clientes). Si vous devez utiliser une information d’identification de mot de passe, ne la définissez pas manuellement. Ne stockez pas les informations d’identification dans le code ou la configuration, et n’autorisez jamais leur gestion par des personnes. Si possible, utilisez les [identités managées des ressources Azure](../managed-identities-azure-resources/overview.md) ou [Azure Key Vault](../../key-vault/general/basic-concepts.md) pour stocker et faire tourner régulièrement vos informations d’identification.

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Assurez-vous que votre application demande des autorisations avec des privilèges minimum. Demandez uniquement les autorisations dont votre application a absolument besoin, et seulement lorsque vous en avez besoin. Connaissez les différents [types d’autorisations](v2-permissions-and-consent.md#permission-types). Utilisez uniquement les autorisations d’application si nécessaire ; servez-vous des autorisations déléguées lorsque cela est possible. Pour obtenir la liste complète des autorisations Microsoft Graph, consultez [Informations de référence sur les autorisations](/graph/permissions-reference).

![case à cocher](./media/active-directory-integration-checklist/checkbox-two.svg) Si vous sécurisez une API à l’aide de la plateforme des identités Microsoft, réfléchissez longuement aux autorisations qui devront être exposées. Déterminez le bon niveau de granularité pour votre solution, et la ou les autorisations nécessitant le consentement de l’administrateur. Vérifiez les autorisations attendues dans les jetons entrants avant de prendre une décision en matière d’autorisation.

## <a name="implementation"></a>Implémentation

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Utilisez des solutions d’authentification moderne (OAuth 2.0, [OpenID Connect](v2-protocols-oidc.md)) pour connecter les utilisateurs en toute sécurité.

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Ne programmez pas directement pour des protocoles comme OAuth 2.0 et Open ID. Au lieu de cela, tirez parti de [Microsoft Authentication Library (MSAL)](msal-overview.md). Les bibliothèques MSAL encapsulent de façon sûre les protocoles de sécurité dans une bibliothèque facile à utiliser, et vous bénéficiez d’une prise en charge intégrée des scénarios [d’accès conditionnel](../conditional-access/overview.md), de [l'authentification unique](../manage-apps/what-is-single-sign-on.md) à l’échelle de l’appareil et de la mise en cache des jetons. Pour plus d’informations, consultez la liste des [bibliothèques clientes](reference-v2-libraries.md#microsoft-supported-client-libraries) et [bibliothèques intergicielles](reference-v2-libraries.md#microsoft-supported-server-middleware-libraries) prises en charge par Microsoft, ainsi que la liste des [bibliothèques clientes tierces compatibles](reference-v2-libraries.md#compatible-client-libraries).<br/><br/>Si vous devez coder de façon manuelle pour les protocoles d’authentification, vous devez suivre une méthodologie comme [Microsoft SDL](https://www.microsoft.com/sdl/default.aspx). Prêtez une attention toute particulière aux considérations de sécurité dans les spécifications de normes pour chaque protocole.

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Migrez des applications existantes de la [Bibliothèque d'authentification Active Directory (ADAL)](../azuread-dev/active-directory-authentication-libraries.md) vers [Microsoft Authentication Library](msal-overview.md). MSAL est la dernière solution de plateforme d’identité de Microsoft et est préférable à ADAL. Elle est disponible pour .NET, JavaScript, Android, iOS, macOS ainsi qu’en préversion publique pour Python et Java. En savoir plus sur la migration d’applications [ADAL.NET](msal-net-migration.md), [ADAL.js](msal-compare-msal-js-and-adal-js.md) et [ADAL.NET et répartiteur iOS](msal-net-migration-ios-broker.md).

![case à cocher](./media/active-directory-integration-checklist/checkbox-two.svg) Pour les applications mobiles, configurez chaque plateforme à l’aide de l’expérience d’inscription d’application. Pour que votre application puisse tirer parti de Microsoft Authenticator ou du portail d’entreprise Microsoft pour l’authentification unique, votre application a besoin d’une « URI de redirection de répartiteur » configurée. Cela permet à Microsoft de rendre le contrôle à votre application après l’authentification. Lors de la configuration de chaque plateforme, l’expérience d’inscription de l’application vous guide tout au long du processus. Utilisez le démarrage rapide pour télécharger un exemple fonctionnel. Sur iOS, utilisez les répartiteurs et la webview système dans la mesure du possible.

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Dans les applications web et les API web, conservez un cache de jeton par compte.  Pour les applications web, le cache de jeton doit être indexé par ID de compte.  Pour les API web, le compte doit être indexé avec le hachage du jeton utilisé pour appeler l’API. MSAL.NET fournit la sérialisation personnalisée du cache de jeton dans les sous-plateformes .NET Framework et .NET Core. Pour des raisons de sécurité et de performances, nous vous recommandons de sérialiser un cache par utilisateur. Pour plus d’informations, consultez [Sérialisation du cache de jeton](msal-net-token-cache-serialization.md#token-cache-for-a-web-app-confidential-client-application).

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Si les données exigées par votre application sont disponibles par le biais de [Microsoft Graph](https://developer.microsoft.com/graph), demandez les autorisations pour ces données avec le point de terminaison Microsoft Graph, plutôt qu’avec l’API individuelle.

![case à cocher](./media/active-directory-integration-checklist/checkbox-two.svg) N’examinez pas les valeurs des jetons d’accès et n’essayez pas de les analyser en tant que client.  Elles peuvent changer de valeur, de format ou même être chiffrées sans avertissement. Utilisez toujours id_token si votre client souhaite en savoir plus sur l’utilisateur, ou appelez Microsoft Graph.  Seules les API web doivent analyser les jetons d’accès (puisque ce sont elles qui définissent le format et les clés de chiffrement).

## <a name="end-user-experience"></a>Expérience de l’utilisateur final

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) [Comprenez l’expérience de consentement](application-consent-experience.md) et configurez les éléments de l’invite de consentement de votre application, afin que les utilisateurs finaux et les administrateurs aient suffisamment d’informations pour déterminer s’ils font confiance à votre application.

![case à cocher](./media/active-directory-integration-checklist/checkbox-two.svg) Réduisez le nombre de fois où un utilisateur doit entrer des informations d’identification de connexion lorsqu’il utilise votre application ; essayez pour cela l’authentification silencieuse (acquisition de jeton en mode silencieux) avant les flux interactifs.

![case à cocher](./media/active-directory-integration-checklist/checkbox-two.svg) N’utilisez pas « prompt=consent » pour chaque connexion. Utilisez uniquement prompt=consent si vous avez déterminé que vous deviez demander le consentement pour des autorisations supplémentaires (par exemple, si vous avez modifié les autorisations exigées de votre application).

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Le cas échéant, enrichissez votre application avec des données utilisateur. L’utilisation de l’[API Microsoft Graph](https://developer.microsoft.com/graph) est un moyen simple de le faire. L’outil [Afficheur Graph](https://developer.microsoft.com/graph/graph-explorer) peut vous aider à démarrer.

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Inscrivez le jeu complet d’autorisations dont votre application a besoin, afin que les administrateurs puissent donner leur consentement facilement à leur locataire. Utilisez le [consentement incrémentiel](../azuread-dev/azure-ad-endpoint-comparison.md#incremental-and-dynamic-consent) au moment de l’exécution pour aider les utilisateurs à comprendre pourquoi votre application demande des autorisations qui, sinon, peuvent inquiéter ou désorienter les utilisateurs lorsqu’elles sont demandées au premier démarrage.

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Implémentez une [expérience de déconnexion unique, normale](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-6-SignOut). Il s’agit d’une exigence de sécurité et de confidentialité, et elle contribue à une bonne expérience utilisateur.

## <a name="testing"></a>Test

![checkbox](./media/active-directory-integration-checklist/checkbox-two.svg) Testez les [stratégies d’accès conditionnel](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-6-SignOut) qui peuvent avoir une incidence sur la capacité de vos utilisateurs à utiliser votre application.

![case à cocher](./media/active-directory-integration-checklist/checkbox-two.svg) Testez votre application avec tous les comptes possibles que vous prévoyez de prendre en charge (par exemple, les comptes professionnels ou scolaires, les comptes Microsoft personnels, les comptes enfants et les comptes souverains).

## <a name="additional-resources"></a>Ressources supplémentaires

Informations approfondies sur v2.0 :

* [Plateforme d’identités Microsoft (vue d’ensemble de v2.0)](v2-overview.md)
* [Informations de référence sur les protocoles de la plateforme d’identités Microsoft](active-directory-v2-protocols.md)
* [Référence sur les jetons d’accès](access-tokens.md)
* [Référence sur les jetons d’ID](id-tokens.md)
* [Informations de référence sur les bibliothèques d’authentification](reference-v2-libraries.md)
* [Autorisations et consentement dans la plateforme d’identités Microsoft](v2-permissions-and-consent.md)
* [API Microsoft Graph](https://developer.microsoft.com/graph)
