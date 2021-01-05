---
title: Notes pour les développeurs relatives aux stratégies personnalisées
titleSuffix: Azure AD B2C
description: Notes à destination des développeurs pour configurer et maintenir Azure AD B2C avec des stratégies personnalisées.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 05/19/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 68e282f192b87b9f2217e0727753e7d37ff1aeb1
ms.sourcegitcommit: 66479d7e55449b78ee587df14babb6321f7d1757
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97516100"
---
# <a name="developer-notes-for-custom-policies-in-azure-active-directory-b2c"></a>Notes développeur pour les stratégies personnalisées dans Azure Active Directory B2C

La configuration de stratégie personnalisée dans Azure Active Directory B2C est désormais mise à la disposition générale. Cette méthode de configuration s’adresse aux développeurs d’identité d’un niveau avancé en matière de création de solutions d’identité complexes. Les stratégies personnalisées mettent la puissance d’Identity Experience Framework au service des locataires Azure AD B2C.
Les développeurs d’identité avancés qui utilisent des stratégies personnalisées doivent prévoir du temps pour suivre des procédures pas à pas et lire des documents de référence.

Si la plupart des options de stratégie personnalisée disponibles sont désormais à la disposition générale, certaines fonctionnalités sous-jacentes comme les types de profil technique et les API de définition de contenu se trouvent à différentes phases du cycle de vie logiciel. Beaucoup d’autres seront bientôt disponibles. Le tableau ci-dessous fournit des informations plus précises sur leur niveau de disponibilité.

## <a name="features-that-are-generally-available"></a>Fonctionnalités mises à la disposition générale

- Créer et charger des parcours utilisateur d’authentification personnalisés à l’aide de stratégies personnalisées.
    - Décrire des parcours utilisateur étape par étape comme des échanges entre des fournisseurs de revendications.
    - Définir le branchement conditionnel dans des parcours utilisateur.
- Interagir avec des services compatibles avec l’API REST dans vos parcours utilisateur d’authentification personnalisés.
- Se fédérer avec des fournisseurs d’identité conformes au protocole OpenIDConnect.
- Se fédérer avec des fournisseurs d’identité qui adhèrent au protocole SAML 2.0.

## <a name="responsibilities-of-custom-policy-feature-set-developers"></a>Responsabilités des développeurs de l’ensemble de fonctionnalités de stratégie personnalisée

La configuration manuelle de stratégie accorde le niveau d’accès minimal à la plateforme sous-jacente d’Azure AD B2C et se traduit par la création d’un framework de confiance unique. Les nombreuses permutations possibles de fournisseurs d’identité personnalisés, les relations de confiance, les intégrations avec des services externes et les workflows étape par étape exigent une approche de conception et de configuration méthodique.

Les développeurs qui utilisent les fonctionnalités de stratégie personnalisée doivent respecter les recommandations suivantes :

- Se familiariser avec le langage de configuration des stratégies personnalisées et la gestion des clés/secrets. Pour plus d’informations, consultez [TrustFrameworkPolicy](trustframeworkpolicy.md).
- S’approprier les scénarios et les intégrations personnalisées. Documenter votre travail et informer l’organisation de votre site actif.
- Effectuer un test de scénario méthodique.
- Suivre les meilleures pratiques de test et de développement de logiciels avec au minimum un environnement de développement et de test et un environnement de production.
- Suivre les nouveaux développements des services et fournisseurs d’identité que vous intégrez. Par exemple, suivre les modifications apportées aux secrets, ainsi que les modifications planifiées et non planifiées du service.
- Configurer la surveillance active et surveiller la réactivité des environnements de production. Pour plus d’informations sur l’intégration avec Application Insights, consultez [Azure Active Directory B2C : collecte des journaux](analytics-with-application-insights.md).
- Garder les adresses e-mail de contact à jour dans l’abonnement Azure et rester attentif aux e-mails de l’équipe de site en ligne Microsoft.
- Prendre les mesures adéquates lorsque cela est recommandé par l’équipe de site en ligne Microsoft.

## <a name="terms-for-features-in-public-preview"></a>Conditions d’utilisation des fonctionnalités de la préversion publique

- Nous vous encourageons à utiliser les fonctionnalités de la préversion publique à des fins d’évaluation uniquement.
- Les contrats de niveau de service (SLA) ne s’appliquent pas aux fonctionnalités de la préversion publique.
- Les demandes de support pour les fonctionnalités de la préversion publique peuvent être soumises par le biais des canaux de support habituels.

## <a name="features-by-stage-and-known-issues"></a>Fonctionnalités par étape et problèmes connus

Les fonctionnalités de stratégie personnalisée/Identity Experience Framework font l’objet d’un développement rapide et constant. Le tableau suivant indique la disponibilité des fonctionnalités et des composants.


### <a name="protocols-and-authorization-flows"></a>Protocoles et flux d’autorisation

| Fonctionnalité | Développement | PRÉVERSION | GA | Notes |
|-------- | :-----------: | :-------: | :--: | ----- |
| [Code d’autorisation OAuth2](authorization-code-flow.md) |  |  | X |  |
| Code d’autorisation OAuth2 avec PKCE |  |  | X | [Clients publics et applications monopages](authorization-code-flow.md)  |
| [Flux implicite OAuth2](implicit-flow-single-page-application.md) |  |  | X |  |
| [Informations d’identification de mot de passe du propriétaire de ressource OAuth2](ropc-custom.md) |  | X |  |  |
| [OIDC Connect](openid-connect.md) |  |  | X |  |
| [SAML2](connect-with-saml-service-providers.md)  |  |  |X  | Liaisons POST et de redirection. |
| OAuth1 |  |  |  | Non pris en charge. |
| WSFED | X |  |  |  |

### <a name="identify-providers-federation"></a>Identifier la fédération des fournisseurs 

| Fonctionnalité | Développement | PRÉVERSION | GA | Notes |
|-------- | :-----------: | :-------: | :--: | ----- |
| [OpenID Connect](openid-connect-technical-profile.md) |  |  | X | Par exemple, Google+.  |
| [OAuth2](oauth2-technical-profile.md) |  |  | X | Par exemple, Facebook.  |
| [OAuth1](oauth1-technical-profile.md) |  | X |  | Par exemple, Twitter. |
| [SAML2](saml-identity-provider-technical-profile.md) |  |   | X | Par exemple, Salesforce, ADFS. |
| WSFED| X |  |  |  |


### <a name="rest-api-integration"></a>Intégration de l’API REST

| Fonctionnalité | Développement | PRÉVERSION | GA | Notes |
|-------- | :-----------: | :-------: | :--: | ----- |
| [API REST avec authentification de base](secure-rest-api.md#http-basic-authentication) |  |  | X |  |
| [API REST avec authentification par certificat client](secure-rest-api.md#https-client-certificate-authentication) |  |  | X |  |
| [API REST avec authentification par jeton du porteur OAuth2](secure-rest-api.md#oauth2-bearer-authentication) |  | X |  |  |

### <a name="component-support"></a>Prise en charge des composants

| Fonctionnalité | Développement | PRÉVERSION | GA | Notes |
| ------- | :-----------: | :-------: | :--: | ----- |
| [Authentification par facteur téléphonique](phone-factor-technical-profile.md) |  |  | X |  |
| [Authentification Azure AD MFA](multi-factor-auth-technical-profile.md) |  | X |  |  |
| [Mot de passe à usage unique](one-time-password-technical-profile.md) |  | X |  |  |
| [Azure Active Directory](active-directory-technical-profile.md) comme répertoire local |  |  | X |  |
| Sous-système de messagerie Azure pour la vérification par e-mail |  |  | X |  |
| [Fournisseurs de services de messagerie tiers](custom-email-mailjet.md) |  |X  |  |  |
| [Prise en charge multilingue](localization.md)|  |  | X |  |
| [Validations de prédicats](predicates.md) |  |  | X | Par exemple, la complexité des mots de passe. |
| [Contrôles d’affichage](display-controls.md) |  |X  |  |  |


### <a name="app-ief-integration"></a>Intégration App-IEF

| Fonctionnalité | Développement | PRÉVERSION | GA | Notes |
| ------- | :-----------: | :-------: | :--: | ----- |
| Paramètre de chaîne de requête `domain_hint` |  |  | X | Disponible sous forme de revendication, peut être transmis au fournisseur d’identité. |
| Paramètre de chaîne de requête `login_hint` |  |  | X | Disponible sous forme de revendication, peut être transmis au fournisseur d’identité. |
| Insérer du code JSON dans le parcours utilisateur via `client_assertion` | X |  |  | Sera déconseillé. |
| Insérer du code JSON dans le parcours utilisateur en tant que `id_token_hint` |  | X |  | Approche à adopter pour transmettre JSON. |
| [Transmettre le jeton du fournisseur d’identité à l’application](idp-pass-through-user-flow.md) |  | X |  | Par exemple, de Facebook à l’application. |


### <a name="session-management"></a>Gestion des sessions

| Fonctionnalité | Développement | PRÉVERSION | GA | Notes |
| ------- | :-----------: | :-------: | :--: | ----- |
| [Fournisseur de session SSO par défaut](custom-policy-reference-sso.md#defaultssosessionprovider) |  |  | X |  |
| [Fournisseur de session de connexion externe](custom-policy-reference-sso.md#externalloginssosessionprovider) |  |  | X |  |
| [Fournisseur de session par authentification unique SAML](custom-policy-reference-sso.md#samlssosessionprovider) |  |  | X |  |
| [OAuthSSOSessionProvider](custom-policy-reference-sso.md#oauthssosessionprovider)  |  | X |  |  |
| [Authentification unique](session-behavior.md#sign-out)  |  | X |  |  |

### <a name="security"></a>Sécurité

| Fonctionnalité | Développement | PRÉVERSION | GA | Notes |
|-------- | :-----------: | :-------: | :--: | ----- |
| Clés de stratégie : générer, manuel, charger |  |  | X |  |
| Clés de stratégie : RSA/certificat, secrets |  |  | X |  |


### <a name="developer-interface"></a>Interface de développeur

| Fonctionnalité | Développement | PRÉVERSION | GA | Notes |
| ------- | :-----------: | :-------: | :--: | ----- |
| Expérience utilisateur de l’infrastructure d’expérience d’identité dans le Portail Azure |  |  | X |  |
| Chargement de stratégie |  |  | X |  |
| [Journaux de parcours utilisateur Application Insights](troubleshoot-with-application-insights.md) |  | X |  | Utilisés pour la résolution des problèmes pendant le développement.  |
| [Journaux d’événements Application Insights](application-insights-technical-profile.md) |  | X |  | Utilisés pour superviser les flux utilisateur en production. |


## <a name="next-steps"></a>Étapes suivantes

- Consultez [Opérations Microsoft Graph disponibles pour Azure AD B2C](microsoft-graph-operations.md).
- [Approfondissez vos connaissances sur les stratégies personnalisées et découvrez ce qui les distingue des flux utilisateur](custom-policy-overview.md).
