---
title: Créez des applications permettant de connecter des utilisateurs d’Azure AD
titleSuffix: Microsoft identity platform
description: Cet article montre comment créer une application mutualisée qui peut connecter un utilisateur à partir de tout client Azure Active Directory.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 10/27/2020
ms.author: ryanwi
ms.reviewer: marsma, jmprieur, lenalepa, sureshja, kkrishna
ms.custom: aaddev
ms.openlocfilehash: 22d6cacc36363b17f9bc32d354982eb71974b31d
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96779751"
---
# <a name="how-to-sign-in-any-azure-active-directory-user-using-the-multi-tenant-application-pattern"></a>Procédure : Connecter un utilisateur Azure Active Directory à l’aide du modèle d’application multilocataire

Si vous proposez une application SaaS (Software as a Service) à de nombreuses organisations, vous pouvez configurer votre application pour accepter des connexions à partir de tout client Azure Active Directory (Azure AD). Cette configuration est appelée *quand vous rendez votre application mutualisée*. Les utilisateurs de n’importe quel client Azure AD pourront se connecter à votre application après votre consentement afin d’utiliser leur compte avec votre application.

Si vous avez une application existante qui possède son propre système de compte, ou prend en charge d’autres types de connexion auprès d’autres fournisseurs cloud, l’ajout d’une connexion Azure AD à partir de tout client est simple. Inscrivez simplement votre application, ajoutez le code de connexion via OAuth2, OpenID Connect ou SAML, et placez un [bouton « Se connecter avec Microsoft »][AAD-App-Branding] dans votre application.

> [!NOTE]
> Cet article suppose que vous êtes déjà familiarisé avec la création d’une application à locataire unique pour Azure AD. Si ce n’est pas le cas, commencez par l’un des démarrages rapides décrits dans la [page d’accueil du guide de développement][AAD-Dev-Guide].

Il existe quatre étapes pour convertir votre application en une application mutualisée Azure AD :

1. [Convertir votre application d’inscription en application mutualisée](#update-registration-to-be-multi-tenant)
2. [Mettre à jour votre code pour envoyer des demandes au point de terminaison /common](#update-your-code-to-send-requests-to-common)
3. [Mettre à jour votre code pour gérer plusieurs valeurs issuer](#update-your-code-to-handle-multiple-issuer-values)
4. [Comprendre le consentement de l’utilisateur et de l’administrateur et apporter des modifications de code appropriées](#understand-user-and-admin-consent)

Examinons chaque étape en détail. Vous pouvez également accéder directement à l’exemple [Créer une application web SaaS mutualisée qui appelle Microsoft Graph à l’aide d’Azure AD et d’OpenID Connect](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/blob/master/2-WebApp-graph-user/2-3-Multi-Tenant/README.md).

## <a name="update-registration-to-be-multi-tenant"></a>Conversion d’une inscription en inscription mutualisée

Par défaut, les inscriptions d’application web ou d’API dans Azure AD sont à locataire unique. Vous pouvez rendre votre inscription mutualisée en recherchant le commutateur **Types de comptes pris en charge** dans le volet **Authentification** de l’inscription de votre application dans le [portail Azure][AZURE-portal], et en le définissant sur **Comptes dans un annuaire organisationnel**.

Avant qu’une application soit mutualisée, Azure AD nécessite que l’URI ID d’application soit globalement unique. L’URI ID d’application est l’une des méthodes d’identification d'une application dans les messages de protocole. Pour une application à client unique, il suffit que l’URI ID d’application soit unique au sein de ce locataire. Pour une application mutualisée, l’URI doit être globalement unique afin qu’Azure AD puisse trouver l’application sur tous les clients. L’unicité globale est appliquée en obligeant l’URI ID d’application à avoir un nom d’hôte correspondant à un domaine vérifié du client Azure AD.

Par défaut, les applications créées via le portail Azure disposent d’un URI d’ID d’application unique au monde dès leur création, mais vous pouvez modifier cette valeur. Par exemple, si le nom de votre client était contoso.onmicrosoft.com, un URI ID d’application valide serait `https://contoso.onmicrosoft.com/myapp`. Si votre client possède le domaine vérifié `contoso.com`, un URI ID d’application valide serait alors `https://contoso.com/myapp`. Si l’URI ID d’application ne suit pas ce modèle, une application ne peut pas être définie comme multi-locataire.

## <a name="update-your-code-to-send-requests-to-common"></a>Mise à jour de votre code pour envoyer des demandes à /common

Dans une application à locataire unique, les demandes de connexion sont envoyées au point de terminaison de connexion du locataire. Par exemple, pour contoso.onmicrosoft.com, le point de terminaison serait : `https://login.microsoftonline.com/contoso.onmicrosoft.com`. Les demandes envoyées au point de terminaison d’un client permettent aux utilisateurs (ou invités) de ce client de se connecter aux applications de ce client.

Avec une application mutualisée, l’application ne sait pas à l’avance de quel client provient l’utilisateur et vous ne pouvez donc pas envoyer des demandes au point de terminaison d’un client. Au lieu de cela, les demandes sont envoyées à un point de terminaison qui est multiplexé entre tous les clients Azure AD : `https://login.microsoftonline.com/common`

Quand la plateforme d’identités Microsoft reçoit une demande sur le point de terminaison /common, il connecte l’utilisateur et, par conséquent, détecte le client dont il provient. Le point de terminaison /common fonctionne avec tous les protocoles d’authentification pris en charge par Azure AD :  OpenID Connect, OAuth 2.0, SAML 2.0 et WS-Federation.

La réponse de connexion envoyée à l’application contient un jeton représentant l’utilisateur. La valeur issuer du jeton indique à une application de quel client provient l’utilisateur. Quand une réponse revient du point de terminaison /common, la valeur issuer du jeton correspond au locataire de l’utilisateur.

> [!IMPORTANT]
> Le point de terminaison /common n’est ni client, ni un émetteur, mais simplement un multiplexeur. Lorsque vous utilisez /common, la logique de votre application permettant de valider les jetons doit être mise à jour en conséquence.

## <a name="update-your-code-to-handle-multiple-issuer-values"></a>Mise à jour de votre code pour gérer plusieurs valeurs issuer

Les applications web et les API web reçoivent et valident les jetons de la plateforme d’identités Microsoft.

> [!NOTE]
> Bien que les applications clientes natives demandent et reçoivent les jetons de la plateforme d’identités Microsoft, elles le font pour les envoyer aux API, où ils sont validés. Les applications natives ne valident pas les jetons d’accès et doivent les traiter comme des valeurs opaques.

Examinons la manière dont une application valide les jetons qu’elle reçoit de la plateforme d’identités Microsoft. Une application à locataire unique a généralement une valeur de point de terminaison de type :

```http
https://login.microsoftonline.com/contoso.onmicrosoft.com
```

…et l’utilise pour construire une URL de métadonnées (dans ce cas, OpenID Connect) comme :

```http
https://login.microsoftonline.com/contoso.onmicrosoft.com/.well-known/openid-configuration
```

pour télécharger deux informations essentielles utilisées pour valider les jetons : les clés de connexion du client et la valeur issuer.

Chaque client Azure AD possède une valeur issuer unique de type :

```http
https://sts.windows.net/31537af4-6d77-4bb9-a681-d2394888ea26/
```

…où la valeur GUID est la version « rename-safe » de l’ID du locataire. Si vous sélectionnez le lien de métadonnées précédent pour `contoso.onmicrosoft.com`, vous pouvez afficher cette valeur issuer dans le document.

Lorsqu’une application à locataire unique valide un jeton, elle vérifie la signature du jeton par rapport aux clés de signature du document de métadonnées. Ce test permet de vous assurer que la valeur de l’émetteur du jeton correspond à celui qui a été trouvé dans le document de métadonnées.

Comme le point de terminaison /common ne correspond pas à un client et n’a pas la valeur issuer, lorsque vous examinez la valeur issuer dans les métadonnées pour /common, elle comporte une URL basée sur un modèle au lieu d’une valeur réelle :

```http
https://sts.windows.net/{tenantid}/
```

Par conséquent, une application mutualisée ne peut pas valider les jetons simplement en faisant correspondre la valeur issuer dans les métadonnées avec la valeur `issuer` dans le jeton. Une application mutualisée a besoin d’une logique pour déterminer les valeurs issuer valides et celles qui ne le sont pas, en fonction de la partie ID client de la valeur issuer.

Par exemple, si une application mutualisée permet uniquement la connexion à partir de clients spécifiques qui se sont inscrits à leur service, elle doit vérifier la valeur issuer ou la valeur de revendication `tid` dans le jeton pour s’assurer que ce client figure dans sa liste d’abonnés. Si une application mutualisée ne gère que des personnes et ne prend aucune décision concernant l’accès en fonction des clients, elle peut donc totalement ignorer la valeur issuer.

Dans les [exemples mutualisés][AAD-Samples-MT], la validation de la valeur issuer est désactivée pour permettre à tout client Azure AD de se connecter.

## <a name="understand-user-and-admin-consent"></a>Comprendre le consentement de l’utilisateur et de l’administrateur

Pour qu’un utilisateur puisse se connecter à une application dans Azure AD, cette application doit être représentée dans le client de l’utilisateur. Cela permet à l’organisation d’effectuer différentes tâches, par exemple appliquer des stratégies uniques lorsque les utilisateurs de leurs clients se connectent à l’application. Pour une application à locataire unique, l’inscription est plus facile ; il s’agit de l’inscription de l’application dans le [portail Azure][AZURE-portal].

Pour une application mutualisée, l’inscription initiale de l’application s’effectue dans le client Azure AD utilisé par le développeur. Lorsqu’un utilisateur d’un autre client se connecte à l’application pour la première fois, Azure AD l’invite à donner son consentement pour les autorisations demandées par l’application. S’il donne son consentement, une représentation de l’application appelée *principal du service* est créée dans le client de l’utilisateur, et la connexion peut alors continuer. Une délégation est également créée dans le répertoire qui enregistre le consentement de l’utilisateur pour l’application. Pour plus d’informations sur les objets ServicePrincipal et Application de l’application et sur les liens qui les unissent, voir [Objets principal de service et application][AAD-App-SP-Objects].

![Illustre le consentement pour une application à niveau unique][Consent-Single-Tier]

Ce processus de consentement dépend des autorisations demandées par l’application. La plateforme d’identités Microsoft prend en charge deux types d’autorisations : application seule et application déléguée.

* une autorisation déléguée accorde à une application la possibilité d’agir comme un utilisateur connecté pour un sous-ensemble d’actions que l’utilisateur peut effectuer. Par exemple, vous pouvez accorder à une application l’autorisation déléguée pour lire le calendrier de l’utilisateur connecté.
* Une autorisation d’application seule est directement accordée à l’identité de l’application. Par exemple, vous pouvez accorder à une application une autorisation application seule pour lire la liste des utilisateurs d’un client, quels que soient les clients connectés à l’application.

Certaines autorisations peuvent être accordées par un utilisateur standard, tandis que d’autres nécessitent le consentement de l’administrateur d’un client.

Pour en savoir plus sur le consentement de l’utilisateur et de l’administrateur, consultez [Configurer le workflow du consentement administrateur](../manage-apps/configure-admin-consent-workflow.md).

### <a name="admin-consent"></a>Consentement de l’administrateur

Les autorisations application seule nécessitent toujours le consentement de l’administrateur d’un client. Si votre application demande une autorisation application seule et qu’un utilisateur tente de se connecter à l’application, un message d’erreur indiquant que l’utilisateur n’est pas en mesure de donner son consentement s’affiche.

Certaines autorisations déléguées nécessitent également le consentement de l’administrateur d’un client. Par exemple, la possibilité de réécrire dans Azure AD en tant que l’utilisateur connecté requiert le consentement de l’administrateur d’un client. Comme pour les autorisations application seule, si un utilisateur standard tente de se connecter à une application qui demande une autorisation déléguée nécessitant le consentement de l’administrateur, votre application reçoit une erreur. Le fait qu’une autorisation nécessite le consentement d’un administrateur est déterminé par le développeur qui a publié la ressource, et ces informations sont disponibles dans la documentation de cette ressource. La documentation sur les autorisations pour l’[API Microsoft Graph][MSFT-Graph-permission-scopes] indique les autorisations qui nécessitent le consentement de l’administrateur.

Si votre application utilise des autorisations qui nécessitent le consentement de l’administrateur, vous devez y intégrer une option comme un bouton ou un lien afin que l’administrateur puisse initier l’action. La requête que votre application envoie pour cette action est une demande d’autorisation OAuth2/OpenID Connect ordinaire, mais qui inclut également le paramètre de chaîne de requête `prompt=admin_consent`. Une fois que l’administrateur a donné son consentement et que le principal de service est créé dans le client, les connexions ultérieures n’ont plus besoin du paramètre `prompt=admin_consent`. Comme l’administrateur a décidé que les autorisations demandées sont acceptables, les autres utilisateurs n’ont plus à donner leur consentement par la suite.

L’administrateur d’un client peut empêcher les utilisateurs standard de donner son consentement aux applications. Si cette fonctionnalité est désactivée, le consentement de l’administrateur est toujours requis pour que l’application soit utilisée dans le client. Si vous souhaitez tester votre application avec le consentement de l’utilisateur final désactivé, vous trouverez le commutateur de configuration sur le [Portail Azure][AZURE-portal], dans la section **[Paramètres utilisateur](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/UserSettings/menuId/)** sous **Applications d’entreprise**.

Le paramètre `prompt=admin_consent` peut également être utilisé par les applications qui demandent des autorisations qui ne nécessitent pas d’autorisation de l’administrateur. Un exemple d’utilisation de cela est quand l’application requiert une expérience où l’administrateur du client « s’inscrit » une fois, et qu’aucun autre utilisateur n’est invité à donner son consentement à partir de ce moment.

Si une application requiert le consentement de l’administrateur, et qu’un administrateur se connecte sans que le paramètre `prompt=admin_consent` soit envoyé, le consentement de l’administrateur s’applique **uniquement pour son compte d’utilisateur**. Les utilisateurs standard ne pourront toujours pas se connecter ou donner leur consentement à l’application. Cette fonctionnalité s’avère utile si vous souhaitez donner à l’administrateur du locataire la possibilité d’explorer votre application avant d’autoriser l’accès à d’autres utilisateurs.

### <a name="consent-and-multi-tier-applications"></a>Consentement et applications multiniveau

Votre application peut comporter plusieurs niveaux, chacun représenté par sa propre inscription dans Azure AD. Par exemple, une application native qui appelle une API web ou une application web qui appelle une autre API web. Dans ces deux cas, le client (application native ou application web) demande des autorisations pour appeler la ressource (API web). Pour que le client puisse donner son consentement pour un client, toutes les ressources nécessitant des autorisations doivent déjà exister dans ce client. Si cette condition n’est pas remplie, Azure AD renvoie une erreur indiquant que la ressource doit d’abord être ajoutée.

#### <a name="multiple-tiers-in-a-single-tenant"></a>Plusieurs niveaux dans un seul client

Cela peut poser problème si votre application logique implique deux ou plusieurs inscriptions d’application, par exemple un client et une ressource distincts. Comment ajouter d’abord la ressource au client ? Azure AD traite ce cas en permettant au client et aux ressources d’être consentis en une seule étape. L’utilisateur voit l’ensemble des autorisations demandées par le client et les ressources sur la page de consentement. Pour activer ce comportement, l’inscription d’application de la ressource doit inclure l’ID d’application du client en tant que `knownClientApplications` dans son [manifeste d’application][AAD-App-Manifest]. Par exemple :

```json
"knownClientApplications": ["94da0930-763f-45c7-8d26-04d5938baab2"]
```

Cela est illustré par un client natif multiniveau appelant un exemple d’API web dans la section [Contenu connexe](#related-content) à la fin de cet article. Le diagramme ci-dessous décrit le processus de consentement pour une application multiniveau enregistrée dans un seul client.

![Illustre le consentement pour une application cliente multiniveau connue][Consent-Multi-Tier-Known-Client]

#### <a name="multiple-tiers-in-multiple-tenants"></a>Plusieurs niveaux dans plusieurs clients

Un cas similaire se produit si les différents niveaux d’une application sont enregistrés dans différents clients. Prenons par exemple le cas de la création d’une application cliente native qui appelle l’API Exchange Online. Pour développer l’application native, et pour que l’application native s’exécute ensuite sur un client, le principal du service Exchange Online doit être présent. Dans ce cas, le développeur et l’utilisateur doivent acheter Exchange Online afin de créer le principal du service sur leurs clients.

Dans le cas d’une API générée par une organisation autre que Microsoft, le développeur de l’API doit fournir un moyen à ses clients de donner leur consentement à l’application sur leurs clients. La conception recommandée est que le développeur tiers génère l’API de façon à pouvoir également fonctionner comme un client web pour implémenter l’inscription. Pour ce faire :

1. Suivez les sections précédentes pour vous assurer que l’API implémente les exigences de code/d’inscription d’application mutualisée.
2. Outre l’exposition des rôles/étendues de l’API, vérifiez que l’inscription inclut l’autorisation « Se connecter et lire le profil utilisateur » (fournie par défaut).
3. Implémentez une page de connexion/inscription dans le client web, et suivez le guide [Consentement de l’administrateur](#admin-consent).
4. Une fois que l’utilisateur donne son consentement à l’application, les liens du principal de service et de la délégation de consentement sont créés dans son client, et l’application native peut obtenir des jetons pour l’API.

Le diagramme suivant décrit le processus de consentement pour une application multiniveau enregistrée dans différents clients.

![Illustre le consentement pour une application multiniveau avec différentes tierces parties][Consent-Multi-Tier-Multi-Party]

### <a name="revoking-consent"></a>Révocation d’un consentement

Les utilisateurs et les administrateurs peuvent à tout moment révoquer leur consentement pour votre application :

* Les utilisateurs révoquent l’accès à des applications individuelles en les supprimant de leur liste [Applications du panneau d’accès][AAD-Access-Panel].
* Les administrateurs révoquent l’accès à des applications en les supprimant d’Azure AD à l’aide de la section [Applications d’entreprise](https://portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps) du [portail Azure][AZURE-portal].

Si un administrateur donne son consentement à une application pour tous les utilisateurs d’un client, ces utilisateurs ne peuvent pas révoquer l’accès individuellement. Seul l’administrateur peut révoquer l’accès et uniquement pour l’application entière.

## <a name="multi-tenant-applications-and-caching-access-tokens"></a>Applications mutualisées et mise en cache des jetons d’accès

Les applications mutualisées peuvent également obtenir des jetons d’accès pour appeler des API protégées par Azure AD. Une erreur courante lors de l’utilisation de la bibliothèque d’authentification Microsoft (MSAL) avec une application mutualisée consiste à demander initialement un jeton pour un utilisateur en utilisant le point de terminaison /common, à recevoir une réponse, puis à demander un nouveau jeton pour ce même utilisateur également en utilisant le point de terminaison /common. Comme la réponse d’Azure AD provient d’un locataire et non du point de terminaison /common, MSAL met en cache le jeton comme provenant du locataire. L’appel suivant à /common pour obtenir un jeton d’accès pour l’utilisateur manque l’entrée du cache, et l’utilisateur est invité à se reconnecter. Pour éviter de manquer le cache, assurez-vous que les appels suivants pour un utilisateur déjà connecté sont effectués vers le point de terminaison du client.

## <a name="related-content"></a>Contenu connexe

* [Exemple d’application mutualisée](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/blob/master/2-WebApp-graph-user/2-3-Multi-Tenant/README.md)
* [Directives de personnalisation des applications][AAD-App-Branding]
* [Objets principal de service et application][AAD-App-SP-Objects]
* [Intégration d’applications dans Azure Active Directory][AAD-Integrating-Apps]
* [Vue d’ensemble de l’infrastructure de consentement][AAD-Consent-Overview]
* [Étendues des autorisations de l’API Microsoft Graph][MSFT-Graph-permission-scopes]

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a montré comment créer une application pouvant connecter un utilisateur à partir de tout client Azure AD. Après activation de l’authentification unique (SSO) entre votre application et Azure AD, vous pouvez également mettre à jour votre application pour accéder aux API exposées par des ressources Microsoft telles que Microsoft 365. Cela vous permet de proposer une expérience personnalisée dans votre application, par exemple en affichant des informations contextuelles aux utilisateurs, comme leur photo de profil ou leur prochain rendez-vous de calendrier.

Pour en savoir plus sur les appels d’API à des services Azure AD et Microsoft 365 comme Exchange, SharePoint, OneDrive, OneNote, et bien plus, voir [API Microsoft Graph][MSFT-Graph-overview].

<!--Reference style links IN USE -->
[AAD-Access-Panel]:  https://myapps.microsoft.com
[AAD-App-Branding]:howto-add-branding-in-azure-ad-apps.md
[AAD-App-Manifest]:reference-azure-ad-app-manifest.md
[AAD-App-SP-Objects]:app-objects-and-service-principals.md
[AAD-Auth-Scenarios]:authentication-scenarios.md
[AAD-Consent-Overview]:consent-framework.md
[AAD-Dev-Guide]:azure-ad-developers-guide.md
[AAD-Integrating-Apps]:quickstart-v1-integrate-apps-with-azure-ad.md
[AAD-Samples-MT]: /samples/browse/?products=azure-active-directory
[AAD-Why-To-Integrate]: ./active-directory-how-to-integrate.md
[AZURE-portal]: https://portal.azure.com
[MSFT-Graph-overview]: /graph/
[MSFT-Graph-permission-scopes]: /graph/permissions-reference

<!--Image references-->
[AAD-Sign-In]: ./media/active-directory-devhowto-multi-tenant-overview/sign-in-with-microsoft-light.png
[Consent-Single-Tier]: ./media/howto-convert-app-to-be-multi-tenant/consent-flow-single-tier.svg
[Consent-Multi-Tier-Known-Client]: ./media/howto-convert-app-to-be-multi-tenant/consent-flow-multi-tier-known-clients.svg
[Consent-Multi-Tier-Multi-Party]: ./media/howto-convert-app-to-be-multi-tenant/consent-flow-multi-tier-multi-party.svg

<!--Reference style links -->
[AAD-App-Manifest]:reference-azure-ad-app-manifest.md
[AAD-App-SP-Objects]:app-objects-and-service-principals.md
[AAD-Auth-Scenarios]:authentication-scenarios.md
[AAD-Integrating-Apps]:quickstart-v1-integrate-apps-with-azure-ad.md
[AAD-Dev-Guide]:azure-ad-developers-guide.md
[AAD-How-To-Integrate]: ./active-directory-how-to-integrate.md
[AAD-Security-Token-Claims]: ./active-directory-authentication-scenarios/#claims-in-azure-ad-security-tokens
[AAD-Tokens-Claims]:access-tokens.md
[AAD-V2-Dev-Guide]: v2-overview.md
[AZURE-portal]: https://portal.azure.com
[Duyshant-Role-Blog]: http://www.dushyantgill.com/blog/2014/12/10/roles-based-access-control-in-cloud-applications-using-azure-ad/
[JWT]: https://tools.ietf.org/html/draft-ietf-oauth-json-web-token-32
[O365-Perm-Ref]: /graph/permissions-reference
[OAuth2-Access-Token-Scopes]: https://tools.ietf.org/html/rfc6749#section-3.3
[OAuth2-AuthZ-Grant-Types]: https://tools.ietf.org/html/rfc6749#section-1.3
[OAuth2-Client-Types]: https://tools.ietf.org/html/rfc6749#section-2.1
[OAuth2-Role-Def]: https://tools.ietf.org/html/rfc6749#page-6
[OpenIDConnect]: https://openid.net/specs/openid-connect-core-1_0.html
[OpenIDConnect-ID-Token]: https://openid.net/specs/openid-connect-core-1_0.html#IDToken