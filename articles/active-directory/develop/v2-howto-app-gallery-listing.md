---
title: Publication de votre application dans la galerie d'applications Azure Active Directory
description: Apprendre à lister une application qui prend en charge l’authentification unique dans la galerie d’applications Azure Active Directory.
services: active-directory
author: kenwith
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 12/14/2020
ms.author: kenwith
ms.reviewer: jeedes
ms.custom: aaddev
ms.openlocfilehash: 37d9c24d58e0dd9fb10db607ef1b96de0a44eb78
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97399364"
---
# <a name="publish-your-app-to-the-azure-ad-app-gallery"></a>Publier votre application dans la galerie d’applications Azure AD

Vous pouvez publier votre application dans la Galerie d’applications Azure AD. Quand votre application est publiée, elle s’affiche comme une option pour les clients quand ils ajoutent des applications à leur locataire. 

Voici quelques-uns des avantages de l’ajout de votre application à la Galerie Azure AD :

- Les clients bénéficient de l’expérience d’authentification unique la meilleure possible pour votre application.
- La configuration de l’application est simple et minime.
- Une recherche rapide permet de trouver votre application dans la galerie.
- Les clients Azure AD des abonnements Gratuit, De base et Premium peuvent tous utiliser cette intégration.
- Les clients mutuels obtiennent un tutoriel de configuration pas à pas.

En outre, il existe de nombreux avantages lorsque vos clients utilisent Azure AD comme fournisseur d’identité pour votre application. entres autres :

- Proposez l'authentification unique pour vos utilisateurs. Avec l’authentification unique, vous réduisez les coûts de support en facilitant l’authentification unique pour vos clients. Si le SSO en un seul clic est activé, les administrateurs informatiques de vos clients n’ont pas besoin d’apprendre à configurer votre application pour l’utiliser dans leur organisation. Pour en savoir plus sur l’authentification unique, consultez [Qu’est-ce que l’authentification unique ?](../manage-apps/what-is-single-sign-on.md).
- Votre application peut être détectable dans la galerie d’applications Microsoft 365, dans le lanceur d’applications Microsoft 365 et au sein de la Recherche Microsoft sur Office.com. 
- Gestion intégrée des applications. Pour en savoir plus sur la gestion des applications dans Azure AD, consultez l’article [Gestion des applications](../manage-apps/what-is-application-management.md).
- Votre application peut utiliser l’[API Graph](/graph/) pour accéder aux données qui engendrent la productivité des utilisateurs dans l’écosystème Microsoft.
- La documentation spécifique à l’application coproduite avec l’équipe d’Azure AD pour nos clients communs facilite l’adoption.
- Vous offrez à vos clients la possibilité de gérer complètement l’authentification et l’autorisation des identités de leurs employés et de leurs invités.
- Toute la responsabilité de la gestion des comptes et de la conformité incombe alors au client propriétaire de ces identités.
- En offrant la possibilité d’activer ou de désactiver le SSO pour des fournisseurs d’identité, des groupes ou des utilisateurs spécifiques, vous êtes en mesure de répondre à leurs besoins d’affaires.
- Vous augmentez vos possibilités de commercialisation et d’adoption. De nombreuses grandes entreprises exigent (ou souhaitent) que leurs employés bénéficient d’une expérience SSO transparente dans toutes les applications. Il est important de faciliter l’authentification unique.
- Vous réduisez les frustrations des utilisateurs finaux, ce qui peut augmenter l’utilisation par les utilisateurs finaux et vos revenus.
- Les clients qui utilisent [SCIM](https://techcommunity.microsoft.com/t5/Identity-Standards-Blog/Provisioning-with-SCIM-getting-started/ba-p/880010) (System for Cross-domain Identity Management) peuvent utiliser le provisionnement pour la même application.
- Ajoutez de la sécurité et de la commodité quand les utilisateurs se connectent à des applications à l’aide de l’authentification unique Azure AD et supprimez le besoin d’informations d’identification distinctes.

> [!TIP]
> Lorsque vous offrez votre application à d’autres entreprises par le biais d’un achat ou d’un abonnement, vous la mettez à disposition des clients au sein de leurs propres locataires Azure. Ceci permet de créer une application multilocataires. Pour un aperçu de ce concept, consultez [Applications multilocataires dans Azure](../../dotnet-develop-multitenant-applications.md) et [Location dans Azure Active Directory](single-and-multi-tenant-apps.md).

> [!IMPORTANT]
> Pour publier votre application dans la Galerie Azure AD, vous devez accepter les conditions générales spécifiques. Avant de commencer, veillez à lire et à accepter les [conditions générales](https://azure.microsoft.com/support/legal/active-directory-app-gallery-terms/).

Les étapes de publication de votre application dans la Galerie d’applications Azure AD sont les suivantes :
1. Choisissez la norme d’authentification unique appropriée pour votre application.
2. Implémentez l’authentification unique dans votre application.
3. Créez votre locataire Azure et testez votre application.
4. Créez et publiez la documentation.
5. Envoyez votre application.
6. Rejoignez le Microsoft Partner Network.

## <a name="what-is-the-azure-ad-application-gallery"></a>Qu’est-ce que la galerie d’applications Azure AD ?

- Les clients bénéficient de l’expérience d’authentification unique la meilleure possible.
- La configuration de l’application est simple et minime.
- Une recherche rapide permet de trouver votre application dans la galerie.
- Les clients Azure AD des abonnements Gratuit, De base et Premium peuvent tous utiliser cette intégration.
- Les clients mutuels obtiennent un tutoriel de configuration pas à pas.
- Les clients qui utilisent [SCIM](https://techcommunity.microsoft.com/t5/Identity-Standards-Blog/Provisioning-with-SCIM-getting-started/ba-p/880010) (System for Cross-domain Identity Management) peuvent utiliser le provisionnement pour la même application.

## <a name="prerequisites"></a>Prérequis

Vous avez besoin d’un compte permanent pour les tests avec au moins deux utilisateurs inscrits.

- Pour des applications fédérées (Open ID et SAML/WS-Fed), l’application doit prendre en charge le modèle SaaS (Software as a Service, logiciel en tant que service) pour être répertoriée dans la galerie d’applications Azure AD. Les applications de galerie d’entreprise doivent prendre en charge plusieurs configurations client et non pas un client spécifique.
- Pour Open ID Connect, l’application doit être mutualisée et l’[infrastructure de consentement Azure AD](../develop/consent-framework.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json) doit être correctement implémentée pour l’application. L’utilisateur peut envoyer la demande de connexion à un point de terminaison commun de façon à ce que tout client puisse donner son consentement à l’application. Vous pouvez contrôler l’accès utilisateur en fonction de l’ID de locataire et de l’UPN de l’utilisateur, reçus dans le jeton.
- Pour SAML 2.0/WS-Fed, votre application doit pouvoir effectuer l’intégration de l’authentification unique SAML/WS-Fed en mode SP ou IDP. Assurez-vous que cette fonctionnalité fonctionne correctement avant d’envoyer la demande.
- Pour une authentification unique par mot de passe, assurez-vous que votre application prend en charge l’authentification par formulaire de telle sorte que le mot de passe puisse être stocké dans le coffre pour que le travail d’authentification unique fonctionne comme prévu.
- Vous avez besoin d’un compte permanent pour les tests avec au moins deux utilisateurs inscrits.

**Comment obtenir Azure AD pour développeurs ?**

Vous pouvez obtenir un compte de test gratuit avec toutes les fonctionnalités Azure AD Premium : 90 jours gratuits qui peuvent être étendus tant que vous l’utilisez pour effectuer des tâches de développement : https://docs.microsoft.com/office/developer-program/office-365-developer-program

## <a name="step-1---choose-the-right-single-sign-on-standard-for-your-app"></a>Étape 1 : Choisissez la norme d’authentification unique appropriée pour votre application

Pour répertorier une application dans la Galerie d’applications Azure AD, vous devez implémenter au moins une des options d’authentification unique prises en charge. Pour comprendre les options d’authentification unique et la façon dont les clients les configurent dans Azure AD, consultez [Options d’authentification unique](../manage-apps/sso-options.md).

Le tableau suivant compare les normes principales : Open Authentication 2.0 (OAuth 2.0) avec OpenID Connect (OIDC), Security Assertion Markup Language (SAML) et Web Services Federation (WS-Fed).

| Fonctionnalité| OAuth / OIDC| SAML/WS-Fed |
| - |-|-|
| Authentification unique basée sur le Web| √| √ |
| Déconnexion unique basée sur le Web| √| √ |
| Authentification unique basée sur mobile| √| √* |
| Déconnexion unique basée sur mobile| √| √* |
| Stratégies d’accès conditionnel pour les applications mobiles| √| √* |
| Expérience d’authentification multifacteur intégrée pour les applications mobiles| √| √* |
| Approvisionnement SCIM| √| √ |
| Accès à Microsoft Graph| √| X |

*Possible, mais Microsoft ne fournit pas d’exemples ou de conseils.

### <a name="oauth-20-and-openid-connect"></a>OAuth 2.0 et OpenID Connect
OAuth 2.0 est un protocole [standard de l’industrie](https://oauth.net/2/) pour l’autorisation. OpenID Connect (OIDC) est une couche d’authentification [standard de l’industrie](https://openid.net/connect/) basée sur le protocole OAuth 2.0. 

**Raisons de choisir OAuth/OIDC**
- L’autorisation intégrée à ces protocoles permet à votre application d’accéder à de nombreuses données utilisateur et organisationnelles et de les intégrer via l’API Microsoft Graph.
- Simplifie l’expérience utilisateur de vos clients lors de l’adoption du SSO pour votre application. Vous pouvez facilement définir les ensembles d’autorisations nécessaires, qui sont ensuite automatiquement reproduits pour l’administrateur ou pour l’utilisateur final avec son consentement.
- L’utilisation de ces protocoles permet à vos clients d’utiliser des politiques d’accès conditionnel et d’authentification multifacteur pour contrôler l’accès aux applications. 
- Microsoft fournit des bibliothèques et des [exemples de code sur plusieurs plateformes technologiques](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Samples) pour faciliter votre développement.  

**Quelques points à prendre en compte**
- Si vous avez déjà implémenté l’authentification unique basée sur SAML pour votre application, vous ne voudrez peut-être pas implémenter une nouvelle norme pour publier votre application dans la Galerie.

### <a name="saml-20-or-ws-fed"></a>SAML 2.0 ou WS-Fed

SAML est une [norme d'authentification unique](https://www.oasis-open.org/standards#samlv2.0) mature et largement adoptée pour les applications web. Pour en savoir plus sur la façon dont Azure utilise SAML, consultez [Comment Azure utilise le protocole SAML](active-directory-saml-protocol-reference.md). 

Web Services Federation (WS-Fed) est un [standard de l’industrie](https://docs.oasis-open.org/wsfed/federation/v1.2/ws-federation.html) généralement utilisé pour les applications Web qui sont développées à l’aide de la plateforme .NET.

**Raisons de choisir SAML**
- SAML 2.0 est un standard éprouvé et la plupart des plates-formes technologiques prennent en charge les bibliothèques open source pour SAML 2.0. 
- Vous pouvez fournir à vos clients une interface d’administration pour configurer SSO SAML. Ils peuvent configurer SSO SAML pour Microsoft Azure AD et tout autre fournisseur d’identité qui prend en charge SAML.

**Quelques points à prendre en compte**
- Si vous utilisez les protocoles SAML 2.0 ou WSFed pour les applications mobiles, certaines stratégies d’accès conditionnel, notamment l’authentification multifacteur, présenteront une expérience détériorée.
- Si vous souhaitez accéder à Microsoft Graph, vous devez implémenter l’autorisation via OAuth 2.0 pour générer les jetons nécessaires. 

### <a name="password-based"></a>Par mot de passe
L'authentification unique basée sur un mot de passe, également appelée  archivage de mot de passe, vous permet de gérer l'accès utilisateur et les mots de passe pour les applications Web qui ne prennent pas en charge la fédération d'identité. Elle est également utile pour les scénarios où plusieurs utilisateurs doivent partager un seul compte, par exemple les comptes d’applications de médias sociaux de votre organisation.


## <a name="step-2---implement-single-sign-on-in-your-app"></a>Étape 2 : Implémentez l’authentification unique dans votre application
Chaque application de la Galerie doit implémenter une des options d’authentification unique prises en charge. Pour en savoir plus sur les options prises en charge, consultez [Options d’authentification unique](../manage-apps/sso-options.md).

Pour OAuth et OIDC, consultez [conseils sur les modèles d’authentification](v2-app-types.md) et les [exemples de code Azure Active Directory](sample-v2-code.md).

Pour SAML et WS-Fed, votre application doit pouvoir effectuer l’intégration de l’authentification unique en mode SP ou IDP. Assurez-vous que cette fonctionnalité fonctionne correctement avant d’envoyer la demande.

Pour en savoir plus sur l’authentification, consultez [Qu’est-ce que l’authentification ?](../azuread-dev/v1-authentication-scenarios.md).

> [!IMPORTANT]
> Pour les applications fédérées (OpenID et SAML/WS-Fed), l’application doit prendre en charge le modèle SaaS (Software as a service). Les applications de la Galerie Azure AD doivent prendre en charge plusieurs configurations clientes et ne doivent pas être spécifiques à un seul client.

### <a name="implement-oauth-20-and-openid-connect"></a>Implémenter OAuth 2.0 et OpenID Connect

Pour OpenID Connect, l’application doit être mutualisée et l’[infrastructure de consentement Azure AD](consent-framework.md) doit être correctement implémentée pour l’application. L’utilisateur peut envoyer la demande de connexion à un point de terminaison commun de façon à ce que tout client puisse donner son consentement à l’application. Vous pouvez contrôler l’accès utilisateur en fonction de l’ID de locataire et de l’UPN de l’utilisateur, reçus dans le jeton.

Pour consulter des exemples spécifiques, consultez [exemples de code de plateforme d’identité Microsoft](sample-v2-code.md). 

Pour consulter des exemples spécifiques aux appareils mobiles, consultez : 
* [Android](quickstart-v2-android.md)
* [iOS](quickstart-v2-ios.md)
* [Plateforme Windows universelle](quickstart-v2-uwp.md)

### <a name="implement-saml-20"></a>Implémenter SAML 2.0

Si votre application prend en charge SAML 2.0, vous pouvez l’intégrer directement à un locataire Azure AD. Pour en savoir plus sur la configuration SAML avec Azure AD, consultez [Configurer l’authentification unique SAML](../manage-apps/configure-saml-single-sign-on.md).

Microsoft ne fournit pas et ne recommande pas de bibliothèques pour les implémentations SAML. Il existe de nombreuses bibliothèques open source disponibles.

### <a name="implement-ws-fed"></a>Implémenter WS-Fed
Pour en savoir plus sur WS-Fed dans ASP.NET Core, consultez [Authentifier les utilisateurs avec WS-Federation dans ASP.NET Core](/aspnet/core/security/authentication/ws-federation).

### <a name="implement-password-vaulting"></a>Implémenter un coffre de mots de passe

Créez une application web avec une page de connexion HTML. Assurez-vous que votre application prend en charge l’authentification par formulaire de telle sorte que le mot de passe puisse être stocké dans le coffre pour que le travail d’authentification unique fonctionne comme prévu.

## <a name="step-3---implement-scim-user-provisioning-in-your-app"></a>Étape 3 : Implémenter l’approvisionnement d’utilisateurs SCIM dans votre application
La prise en charge de l’approvisionnement [SCIM](https://aka.ms/scimoverview) est une étape facultative, mais fortement recommandée, dans la création de votre application. La prise en charge de la norme SCIM est simple et permet aux clients de créer et de mettre à jour automatiquement les comptes d’utilisateur dans votre application, sans recourir à des processus manuels tels que le chargement de fichiers CSV. En outre, les clients peuvent automatiser la suppression des utilisateurs et le maintien de la synchronisation des appartenances aux groupes, ce qui n’est pas possible avec une solution comme SAML JIT. 

### <a name="learn-about-scim"></a>En savoir plus sur SCIM
Pour en savoir plus sur les normes SCIM et les avantages pour vos clients, consultez [Approvisionnement avec SCIM - prise en main](https://aka.ms/scimoverview).

### <a name="understand-the-azure-ad-scim-implementation"></a>Comprendre l'implémentation de SCIM Azure AD
Pour en savoir plus sur l’implémentation d’Azure AD SCIM, consultez [Créer un point de terminaison SCIM et configurer l'attribution des utilisateurs à l'aide d'Azure AD](https://docs.microsoft.com/azure/active-directory/app-provisioning/use-scim-to-provision-users-and-groups).

### <a name="implement-scim"></a>Implémenter SCIM
Azure AD fournit un [code de référence](https://aka.ms/scimoverview) pour vous aider à créer un point de terminaison SCIM. De nombreuses bibliothèques/références tierces sont également disponibles sur GitHub.  

## <a name="step-4---create-your-azure-tenant-and-test-your-app"></a>Étape 4 : Créez votre locataire Azure et testez votre application

Vous aurez besoin d’un locataire Azure AD pour tester votre application. Pour configurer votre environnement de développement, consultez [Démarrage rapide : Configurer un locataire](quickstart-create-new-tenant.md).

En guise d’alternative, un locataire Azure AD est fourni avec chaque abonnement Microsoft 365. Pour configurer un environnement de développement Microsoft 365 gratuit, consultez [Rejoindre le programme pour développeurs Microsoft 365](/office/developer-program/microsoft-365-developer-program).

Une fois que vous avez un locataire, vous devez tester l’authentification unique et l’[approvisionnement](https://docs.microsoft.com/azure/active-directory/app-provisioning/use-scim-to-provision-users-and-groups#step-4-integrate-your-scim-endpoint-with-the-azure-ad-scim-client). 

**Pour les applications OIDC ou Oath**, [inscrivez votre application](quickstart-register-app.md) comme une application mutualisée. Sous Types de comptes pris en charge, sélectionnez l’option Comptes dans un annuaire organisationnel et comptes personnels Microsoft.

**Pour les applications basées sur SAML et WS-Fed**, vous [Configurez des applications avec authentification unique basée sur SAML](../manage-apps/configure-saml-single-sign-on.md) à l’aide d’un modèle SAML générique dans Azure AD.

Vous pouvez également [convertir une application à locataire unique en application mutualisée](howto-convert-app-to-be-multi-tenant.md) si nécessaire.


## <a name="step-5---create-and-publish-documentation"></a>Étape 5 : Créez et publiez la documentation

### <a name="documentation-on-your-site"></a>Documentation sur votre site

La facilité d’adoption est un facteur important dans les décisions liées aux logiciels d’entreprise. Une documentation claire et intuitive soutient vos clients dans leur processus d’adoption et réduit les coûts de support. Pour avoir travaillé avec des milliers d’éditeurs de logiciels, Microsoft sait ce qui fonctionne.

Nous recommandons que la documentation sur votre site contienne au minimum les éléments suivants.

* Présentation de vos fonctionnalités d’authentification unique
  * Protocoles pris en charge
  * Version et référence SKU
  * Liste de fournisseurs d’identité pris en charge avec des liens vers leurs documentations respectives
* Informations de licence pour votre application
* Contrôle d’accès en fonction du rôle pour configurer l’authentification unique
* Étapes de configuration de l’authentification unique
  * Éléments de configuration d’interface utilisateur pour SAML avec les valeurs attendues pour le fournisseur
  * Informations du fournisseur de service à transmettre aux fournisseurs d’identité
* Si OIDC/OAuth
  * Liste d’autorisations nécessaires pour le consentement avec les justifications métier
* Étapes de tests pour les utilisateurs pilotes
* Informations de dépannage, notamment messages et codes d’erreur
* Mécanismes de support pour les clients
* Détails sur votre point de terminaison SCIM, y compris les ressources et les attributs pris en charge

### <a name="documentation-on-the-microsoft-site"></a>Documentation sur le site Microsoft

Lorsque vous répertoriez votre application dans la galerie d’applications Azure Active Directory, qui publie également votre application sur la Place de marché Azure, Microsoft génère la documentation pour nos clients mutuels expliquant le processus pas à pas. Vous pouvez consulter un exemple [ici](../saas-apps/tutorial-list.md). Cette documentation est créée en fonction de ce que vous envoyez à la galerie, et vous pouvez facilement la mettre à jour si vous apportez des modifications à votre application à l’aide de votre compte GitHub.


## <a name="step-6---submit-your-app"></a>Étape 6 : Soumettez votre application

Après avoir testé le bon fonctionnement de l’intégration de votre application avec Azure AD, soumettez votre demande d’application dans le [portail Application Network Microsoft](https://microsoft.sharepoint.com/teams/apponboarding/Apps).

La première fois que vous essayez de vous connecter au portail, vous verrez un écran parmi deux options. 

Si vous recevez le message « Cela n’a pas fonctionné », vous devez contacter l’[équipe d’intégration de l’authentification unique Azure AD](<mailto:SaaSApplicationIntegrations@service.microsoft.com>). Indiquez le compte de messagerie que vous souhaitez utiliser pour envoyer la demande. Une adresse de messagerie professionnelle telle que `name@yourbusiness.com` est préférable. L’équipe Azure AD ajoutera le compte dans le portail Application Network Microsoft.

Si vous voyez une page « Demander l’accès », indiquez la justification métier, puis sélectionnez **Demander l’accès**.

Une fois le compte ajouté, vous pouvez vous connecter au portail Application Network Microsoft et envoyer la demande en sélectionnant la vignette **Submit Request (ISV)** (Envoyer la demande (ISV)) dans la page d’accueil.

![Vignette Submit Request (ISV) (Envoyer une demande (ISV)) dans la page d’accueil](./media/howto-app-gallery-listing/homepage.png)

### <a name="issues-on-logging-into-portal"></a>Problèmes de connexion au portail

Si cette erreur s’affiche lors de la connexion, voici les détails du problème et la façon de le corriger.

* Si votre connexion a été bloquée comme indiqué ci-dessous :

  ![problèmes lors de la résolution d’une application dans la galerie](./media/howto-app-gallery-listing/blocked.png)

**Ce qui se passe :**

L’utilisateur invité est fédéré en locataire de base qui est également un annuaire Azure AD. L’utilisateur invité présente un risque élevé. Microsoft n’autorise pas les utilisateurs à risque élevé à accéder à ses ressources. Tous les utilisateurs à risque élevé (employés ou invités/fournisseurs) doivent corriger/fermer leur risque pour accéder aux ressources Microsoft. Pour les utilisateurs invités, ce risque utilisateur provient du locataire de base et la stratégie provient du locataire de ressource (Microsoft, dans le cas présent).
 
**Solutions sécurisées :**

* Les utilisateurs invités inscrits à l’authentification MFA corrigent leur propre risque utilisateur. L’utilisateur invité peut effectuer cette opération en procédant à un changement ou une réinitialisation sécurisés du mot de passe (https://aka.ms/sspr) sur son locataire de base (ce qui nécessite MFA et SSPR sur le locataire de base). Le changement ou la réinitialisation sécurisés du mot de passe doivent être lancés sur Azure AD, et non pas en local.

* Le risque lié aux utilisateurs invités peut être résolu par leurs administrateurs. Dans ce cas, l’administrateur effectue une réinitialisation du mot de passe (génération d’un mot de passe temporaire). Cette opération ne nécessite pas Identity Protection. L’administrateur de l’utilisateur invité peut accéder à https://aka.ms/RiskyUsers et cliquer sur « Réinitialiser le mot de passe ».

* Le risque lié aux utilisateurs invités peut être fermé/ignoré par leurs administrateurs. Une fois encore, cette opération ne nécessite pas Identity Protection. L’administrateur peut accéder à https://aka.ms/RiskyUsers et cliquer sur « Ignorer le risque lié à l’utilisateur ». Toutefois, l’administrateur doit faire preuve de diligence pour garantir qu’il s’agissait d’un faux positif d’évaluation des risques avant de fermer le risque utilisateur. Sinon, il fait courir un risque à ses ressources et à celles de Microsoft en supprimant une évaluation des risques sans investigation.

> [!NOTE]
> En cas de problème d’accès, contactez l’[équipe d’intégration Azure AD SSO](<mailto:SaaSApplicationIntegrations@service.microsoft.com>).

### <a name="implementation-specific-options"></a>Options spécifiques d’implémentation
Si vous souhaitez ajouter votre application à la liste dans la galerie à l’aide d’OpenID Connect, sélectionnez **OpenID Connect & OAuth 2.0** comme illustré ici.

![Listing d’une application OpenID Connect dans la galerie](./media/howto-app-gallery-listing/openid.png)

Si vous souhaitez ajouter votre application à la liste dans la galerie à l’aide de **SAML 2.0** ou **WS-Fed**, sélectionnez **SAML 2.0/WS-Fed** comme illustré ici.

![Listing d’une application SAML 2.0 ou WS-Fed dans la galerie](./media/howto-app-gallery-listing/saml.png)

Si vous souhaitez ajouter votre application à la liste dans la galerie à l’aide de l’authentification unique par mot de passe, sélectionnez **Authentification unique par mot de passe** comme illustré ici.

![Listing d’une application avec authentification unique par mot de passe dans la galerie](./media/howto-app-gallery-listing/passwordsso.png)

Si vous implémentez un point de terminaison [SCIM](../app-provisioning/use-scim-to-provision-users-and-groups.md) 2.0 pour l’Attribution d’utilisateurs, sélectionnez l’option comme indiqué. Lorsque vous fournissez le schéma dans la demande d’intégration, suivez les instructions spécifiées [ici](https://docs.microsoft.com/azure/active-directory/app-provisioning/export-import-provisioning-configuration) pour télécharger votre schéma. Nous utiliserons le schéma que vous avez configuré lors du test de l’application ne figurant pas dans la galerie pour créer l’application de la galerie. 

   ![Demande de provisionnement d’utilisateurs](./media/howto-app-gallery-listing/user-provisioning.png)

### <a name="update-or-remove-an-existing-listing"></a>Mettre à jour ou supprimer un listing existant

Vous pouvez mettre à jour ou supprimer une application de galerie existante dans le [portail Application Network Microsoft](https://microsoft.sharepoint.com/teams/apponboarding/Apps).

![Listing d’une application SAML dans la galerie](./media/howto-app-gallery-listing/updateorremove.png)

> [!NOTE]
> Si vous rencontrez des problèmes d’accès, consultez la section précédente sur la création de votre compte. Si cela ne fonctionne pas, contactez l’[équipe d’intégration de l’authentification unique Azure AD](<mailto:SaaSApplicationIntegrations@service.microsoft.com>).

### <a name="list-requests-by-customers"></a>Demandes de listing par les clients

Les clients peuvent envoyer une demande pour lister une application en sélectionnant **Demandes d’application par les clients** > **Soumettre une nouvelle demande**.

![Montre la vignette des applications demandées par les clients](./media/howto-app-gallery-listing/customer-submit-request.png)

Voici le flux des applications demandées par les clients.

![Montre le flux des applications demandées par les clients](./media/howto-app-gallery-listing/customer-request-2.png)


### <a name="timelines"></a>Chronologies

La durée du processus de listing d’une application SAML 2.0 ou WS-Fed dans la galerie est de 7 à 10 jours ouvrables.

![Chronologie de listing d’une application SAML dans la galerie](./media/howto-app-gallery-listing/timeline.png)

La durée du processus de listing d’une application OpenID Connect dans la galerie est de 2 à 5 jours ouvrables.

![Chronologie de listing d’une application OpenID Connect dans la galerie](./media/howto-app-gallery-listing/timeline2.png)

### <a name="escalations"></a>Escalades

Pour faire remonter un problème, envoyez un e-mail à l’[équipe d’intégration de l’authentification unique Azure AD](mailto:SaaSApplicationIntegrations@service.microsoft.com) et nous vous répondrons dès que possible.


## <a name="step-7---join-the-microsoft-partner-network"></a>Étape 7 : Rejoignez le Microsoft Partner Network
Le Microsoft Partner Network fournit un accès instantané et exclusif à des ressources, des programmes, des outils et des connexions. Pour rejoindre le réseau et créer votre plan Go to Market, consultez [Atteindre des clients commerciaux](https://partner.microsoft.com/explore/commercial#gtm).


## <a name="next-steps"></a>Étapes suivantes
* [Créer un point de terminaison SCIM et configurer l’attribution des utilisateurs](../app-provisioning/use-scim-to-provision-users-and-groups.md)
* [Scénarios d’authentification pour Azure AD](authentication-flows-app-scenarios.md)
