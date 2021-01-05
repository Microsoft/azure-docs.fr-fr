---
title: Forum Aux Questions (FAQ) sur Azure Active Directory B2C
description: Réponses aux questions fréquentes sur Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 10/14/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 30b7788419695db7ee5d4b81714bf55248a41b4a
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97108720"
---
# <a name="azure-ad-b2c-frequently-asked-questions-faq"></a>Azure AD B2C : Forum Aux Questions (FAQ)

Cette page répond aux questions fréquemment posées sur Azure Active Directory B2C (Azure AD B2C). N'hésitez pas à la consulter pour vous tenir au courant des mises à jour.

### <a name="why-cant-i-access-the-azure-ad-b2c-extension-in-the-azure-portal"></a>Pourquoi ne puis-je pas accéder à l’extension Azure AD B2C sur le Portail Azure ?

Si l’extension Azure AD ne fonctionne pas pour vous, il peut y avoir deux raisons courantes à cela. Votre rôle utilisateur dans le répertoire doit être administrateur général pour Azure AD B2C. Veuillez contacter votre administrateur si vous pensez que vous devriez y avoir accès. Si vous disposez de privilèges Administrateur général, vérifiez que vous vous trouvez dans un annuaire Azure AD B2C et non un répertoire Azure Active Directory. Vous trouverez ici des instructions pour [créer un locataire Azure AD B2C](tutorial-create-tenant.md).

### <a name="can-i-use-azure-ad-b2c-features-in-my-existing-employee-based-azure-ad-tenant"></a>Puis-je utiliser les fonctionnalités d’Azure AD B2C dans mon client Azure AD existant, basé sur les employés ?

Azure AD et Azure AD B2C sont deux offres de produits distinctes qui ne peuvent pas coexister dans le même locataire. Un locataire Azure AD représente une organisation. Un locataire Azure AD B2C représente une collection d’identités à utiliser avec des applications par partie de confiance. En ajoutant un **Nouveau fournisseur OpenID Connect** sous **Azure AD B2C > Fournisseurs d’identité** ou avec des stratégies personnalisées, Azure AD B2C peut fédérer pour Azure AD en autorisant l’authentification des employés dans une organisation.

### <a name="can-i-use-azure-ad-b2c-to-provide-social-login-facebook-and-google-into-microsoft-365"></a>Puis-je utiliser Azure AD B2C pour activer la connexion à des réseaux sociaux (Facebook et Google+) dans Microsoft 365 ?

Vous ne pouvez pas utiliser Azure AD B2C pour authentifier les utilisateurs pour Microsoft 365. Azure AD est la solution Microsoft permettant de gérer l’accès des employés aux applications SaaS. Elle présente des fonctionnalités conçues à cet effet, telles que l’accès conditionnel et la gestion des licences. Azure AD B2C fournit une plateforme de gestion des identités et des accès pour la création d’applications web et mobiles. Quand Azure AD B2C est configuré pour établir la fédération avec un locataire Azure AD, le locataire Azure AD gère l’accès des employés aux applications qui s’appuient sur Azure AD B2C.

### <a name="what-are-local-accounts-in-azure-ad-b2c-how-are-they-different-from-work-or-school-accounts-in-azure-ad"></a>Que sont les comptes locaux dans Azure AD B2C ? En quoi sont-ils différents des comptes professionnels ou scolaires dans Azure AD ?

Dans un locataire Azure AD, les utilisateurs qui appartiennent au locataire se connectent avec une adresse e-mail au format `<xyz>@<tenant domain>`. Le `<tenant domain>` est l’un des domaines vérifiés dans le locataire ou le domaine `<...>.onmicrosoft.com` initial. Ce type de compte est un compte professionnel ou scolaire.

Dans un locataire Azure AD B2C, la plupart des applications demandent que l’utilisateur se connecte avec une adresse e-mail arbitraire (par exemple, joe@comcast.net, bob@gmail.com, sarah@contoso.com ou jim@live.com). Ce type de compte est un compte local. Nous prenons également en charge les noms d’utilisateur arbitraires en tant que comptes locaux (par exemple joe, bob, sarah ou jim). Vous pourrez choisir entre ces deux types de comptes locaux pendant la configuration des fournisseurs d’identité pour Azure AD B2C sur le Portail Azure. Dans votre client Azure AD B2C, sélectionnez **Fournisseurs d’identité**, **Compte local**, puis **Nom d’utilisateur**.

Les comptes d’utilisateur des applications peuvent être créés par le biais d’un flux d’utilisateur d’inscription ou de connexion, de l’API Microsoft Graph ou dans le Portail Azure.

### <a name="which-social-identity-providers-do-you-support-now-which-ones-do-you-plan-to-support-in-the-future"></a>Quels fournisseurs d’identité sociaux prenez-vous en charge maintenant ? Lesquels envisagez-vous de prendre en charge à l'avenir ?

Nous prenons actuellement en charge plusieurs fournisseurs d’identité sociale, notamment Amazon, Facebook, GitHub (préversion), Google, LinkedIn, compte Microsoft (MSA), QQ (préversion), Twitter, WeChat (préversion) et Weibo (préversion). Nous envisageons d’ajouter la prise en charge d’autres fournisseurs d’identité sociale populaires en fonction de la demande des clients.

Azure AD B2C prend également en charge les [stratégies personnalisées](custom-policy-overview.md). Les stratégies personnalisées vous permettent de créer votre propre stratégie pour tous les fournisseurs d’identité qui prennent en charge [OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) ou SAML. Bien démarrer avec les stratégies personnalisées en consultant notre [pack de démarrage des stratégies personnalisées](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack).

### <a name="can-i-configure-scopes-to-gather-more-information-about-consumers-from-various-social-identity-providers"></a>Puis-je configurer des étendues pour collecter plus d’informations sur les clients depuis différents fournisseurs d’identité sociaux ?

Non. Les étendues par défaut utilisées pour notre jeu de fournisseurs d'identité sociaux pris en charge sont :

* Facebook : e-mail
* Google+ : e-mail
* Compte Microsoft : profil de messagerie openid
* Amazon : profil
* LinkedIn : r_emailaddress, r_basicprofile

### <a name="does-my-application-have-to-be-run-on-azure-for-it-work-with-azure-ad-b2c"></a>Mon application doit-elle être exécutée sous Azure pour fonctionner avec Azure AD B2C ?

Non, vous pouvez héberger votre application n'importe où (dans le cloud ou sur site). Pour interagir avec Azure AD B2C, il suffit qu’elle puisse envoyer et recevoir des requêtes HTTP sur les points de terminaison accessibles publiquement.

### <a name="i-have-multiple-azure-ad-b2c-tenants-how-can-i-manage-them-on-the-azure-portal"></a>Je dispose de plusieurs locataires Azure AD B2C. Comment puis-je les gérer sur le portail Azure ?

Avant d’ouvrir « Azure AD B2C » dans le menu à gauche du portail Azure, vous devez basculer dans le répertoire que vous souhaitez gérer. Changez de répertoire en cliquant sur votre identité dans le coin supérieur droit du portail Azure, puis sélectionnez un répertoire dans la liste déroulante qui s’affiche.

### <a name="how-do-i-customize-verification-emails-the-content-and-the-from-field-sent-by-azure-ad-b2c"></a>Comment puis-je personnaliser les e-mails de vérification (le contenu et le champ « De: ») envoyés par Azure AD B2C ?

Vous pouvez utiliser la [fonctionnalité de personnalisation de la société](../active-directory/fundamentals/customize-branding.md) pour personnaliser le contenu des e-mails de vérification. Plus précisément, ces deux éléments du message peuvent être personnalisés :

* **Logo de bannière** : affiché en bas à droite.
* **Couleur d’arrière-plan** : affiché en haut.

    ![Capture d’écran d’un e-mail de vérification personnalisée](./media/faq/company-branded-verification-email.png)

La signature d’e-mail contient le nom du client Azure AD B2C que vous avez fourni lors de la création du client Azure AD B2C. Vous pouvez modifier le nom à l’aide de ces instructions :

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en tant qu’administrateur général.
1. Ouvrez le panneau **Azure Active Directory**.
1. Cliquez sur l’onglet **Propriétés**.
1. Changez le champ **Nom**.
1. Cliquez sur **Enregistrer** dans la partie supérieure de la page.

Il n’existe actuellement aucun moyen de modifier le champ « De : » de l’e-mail.

### <a name="how-can-i-migrate-my-existing-user-names-passwords-and-profiles-from-my-database-to-azure-ad-b2c"></a>Comment puis-je migrer mes noms d’utilisateur, mots de passe et profils existants à partir de ma base de données vers Azure AD B2C ?

Vous pouvez utiliser l’API Microsoft Graph pour écrire l’outil de migration. Consultez le [guide de migration des utilisateurs](user-migration.md) pour plus d’informations.

### <a name="what-password-user-flow-is-used-for-local-accounts-in-azure-ad-b2c"></a>Quel flux d’utilisateur de mot de passe est utilisé pour les comptes locaux dans Azure AD B2C ?

Le flux d’utilisateur de mot de passe Azure AD B2C pour les comptes locaux est basé sur la stratégie pour Azure AD. Les flux d’utilisateur de réinitialisation du mot de passe, d’inscription ou de connexion et d’inscription Azure AD B2C utilisent des mots de passe « forts » et qui n’expirent pas. Pour plus d’informations, consultez [Stratégies et restrictions de mot de passe dans Azure Active Directory](../active-directory/authentication/concept-sspr-policy.md).

Pour plus d’informations sur les verrouillages de compte et mots de passe, voir l’article sur la [gestion les menaces pesant sur les ressources et données dans Azure Active Directory B2C](threat-management.md).

### <a name="can-i-use-azure-ad-connect-to-migrate-consumer-identities-that-are-stored-on-my-on-premises-active-directory-to-azure-ad-b2c"></a>Puis-je utiliser Azure AD Connect pour migrer les identités de clients stockées dans mon répertoire Active Directory local vers Azure AD B2C ?

Non, Azure AD Connect n'est pas conçu pour fonctionner avec Azure AD B2C. Vous pouvez envisager d’utiliser l’[API Microsoft Graph](manage-user-accounts-graph-api.md) pour la migration des utilisateurs. Consultez le [guide de migration des utilisateurs](user-migration.md) pour plus d’informations.

### <a name="can-my-app-open-up-azure-ad-b2c-pages-within-an-iframe"></a>Mon application peut-elle ouvrir des pages Azure Active Directory B2C dans un iFrame ?

Non, pour des raisons de sécurité, les pages Azure AD B2C ne peuvent pas être ouvertes dans un iFrame. Notre service communique avec le navigateur pour interdire les iFrames. La communauté de sécurité en général et la spécification OAUTH2 déconseillent d’utiliser des iFrames pour les expériences d’identité en raison du risque d’attaques par détournement de clics.

### <a name="does-azure-ad-b2c-work-with-crm-systems-such-as-microsoft-dynamics"></a>Azure AD B2C fonctionne-t-il avec les systèmes CRM, tels que Microsoft Dynamics ?

L’intégration avec le portail Microsoft Dynamics 365 est disponible. Consultez [Configuration du portail Dynamics 365 pour utiliser Azure AD B2C pour l’authentification ](/dynamics365/customer-engagement/portals/azure-ad-b2c).

### <a name="does-azure-ad-b2c-work-with-sharepoint-on-premises-2016-or-earlier"></a>Azure AD B2C fonctionne-t-il avec SharePoint localement 2016 ou version antérieure ?

Azure AD B2C n’est pas conçu pour le scénario de partage partenaire externe SharePoint ; consultez plutôt cet article sur [Azure AD B2B](https://cloudblogs.microsoft.com/enterprisemobility/2015/09/15/learn-all-about-the-azure-ad-b2b-collaboration-preview/).

### <a name="should-i-use-azure-ad-b2c-or-b2b-to-manage-external-identities"></a>Dois-je utiliser Azure AD B2C ou B2B pour gérer les identités externes ?

Lisez [Comparer B2B Collaboration et B2C dans Azure AD](../active-directory/external-identities/compare-with-b2c.md) pour en savoir plus sur l’application des fonctionnalités appropriées à vos scénarios d’identités externes.

### <a name="what-reporting-and-auditing-features-does-azure-ad-b2c-provide-are-they-the-same-as-in-azure-ad-premium"></a>Quelles sont les fonctionnalités de création de rapports et d’audit proposées par Azure AD B2C ? Sont-elles identiques à celles d’Azure AD Premium ?

Non, Azure AD B2C ne prend pas en charge le même ensemble de rapports qu'Azure AD Premium. Toutefois, de nombreux points communs existent :

* Les **rapports de connexion** donnent un enregistrement de chaque connexion avec peu de détails.
* Les **rapports d’audit** comportent les activités d’administration et d’application.
* Les **rapports d’utilisation** comprennent le nombre d’utilisateurs, le nombre de connexions et le volume de MFA.

### <a name="can-i-localize-the-ui-of-pages-served-by-azure-ad-b2c-what-languages-are-supported"></a>Puis-je localiser l'interface utilisateur des pages présentées par Azure AD B2C ? Quelles sont les langues prises en charge ?

Oui, la localisation est possible. Consultez [Personnalisation de la langue](language-customization.md). Nous fournissons des traductions en 36 langues, et vous pouvez remplacer n’importe quelle chaîne pour l’adapter à vos besoins.

### <a name="can-i-use-my-own-urls-on-my-sign-up-and-sign-in-pages-that-are-served-by-azure-ad-b2c-for-instance-can-i-change-the-url-from-contosob2clogincom-to-logincontosocom"></a>Puis-je utiliser mes propres URL dans les pages d’inscription et de connexion présentées par Azure AD B2C ? Par exemple, puis-je remplacer l’URL contoso.b2clogin.com par login.contoso.com ?

Pas actuellement. Cette fonctionnalité est sur notre feuille de route. La vérification de votre domaine sous l’onglet **Domaines** sur le portail Azure ne remplit pas cet objectif. Cependant, avec b2clogin.com, nous offrons un [domaine de premier niveau neutre](b2clogin.md), et l’apparence externe peut donc être implémentée sans la mention Microsoft.

### <a name="how-do-i-delete-my-azure-ad-b2c-tenant"></a>Comment supprimer mon client Azure AD B2C ?

Procédez comme suit pour supprimer votre locataire Azure AD B2C.

Vous pouvez utiliser notre nouvelle expérience unifiée **Inscriptions d’applications** ou notre expérience héritée **Applications (héritées)** . [En savoir plus sur la nouvelle expérience](./app-registrations-training-guide.md).

#### <a name="app-registrations"></a>[Inscriptions des applications](#tab/app-reg-ga/)

1. Connectez-vous au [Portail Azure](https://portal.azure.com/) en tant qu’*administrateur d’abonnements*. Utilisez le compte professionnel ou scolaire ou le compte Microsoft que vous avez utilisé pour vous inscrire à Azure.
1. Sélectionnez le filtre **Annuaire et abonnement** dans le menu supérieur, puis l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans le menu de gauche, sélectionnez **Azure AD B2C**. Ou sélectionnez **Tous les services**, puis recherchez et sélectionnez **Azure AD B2C**.
1. Supprimez tous les **flux d’utilisateurs (stratégies)** dans votre locataire Azure AD B2C.
1. Sélectionnez **Inscriptions d’applications**, puis sélectionnez l’onglet **Toutes les applications**.
1. Supprimez toutes les applications que vous avez inscrites.
1. Supprimez l’application **b2c-extensions-app**.
1. Sous **Gérer**, sélectionnez **Utilisateurs**.
1. Sélectionnez chaque utilisateur tour à tour (excluez l’utilisateur *Administrateur d’abonnements* sous lequel vous êtes connecté actuellement). Sélectionnez **Supprimer** en bas de la page, puis **Oui** lorsque vous y êtes invité.
1. Sélectionnez **Azure Active Directory** dans le menu à gauche.
1. Sous **Gérer**, sélectionnez **Paramètres utilisateur**.
1. Sous **Gérer**, sélectionnez **Propriétés**.
1. Sous **Gestion de l’accès pour les ressources Azure**, sélectionnez **Oui**, puis **Enregistrer**.
1. Déconnectez-vous du portail Microsoft Azure, puis reconnectez-vous pour actualiser votre accès.
1. Sélectionnez **Azure Active Directory** dans le menu à gauche.
1. Dans la page **Vue d’ensemble**, sélectionnez **Supprimer le locataire**. Suivez les instructions à l’écran pour achever le processus.

#### <a name="applications-legacy"></a>[Applications (héritées)](#tab/applications-legacy/)

1. Connectez-vous au [Portail Azure](https://portal.azure.com/) en tant qu’*administrateur d’abonnements*. Utilisez le compte professionnel ou scolaire ou le compte Microsoft que vous avez utilisé pour vous inscrire à Azure.
1. Sélectionnez le filtre **Annuaire et abonnement** dans le menu supérieur, puis l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans le menu de gauche, sélectionnez **Azure AD B2C**. Ou sélectionnez **Tous les services**, puis recherchez et sélectionnez **Azure AD B2C**.
1. Supprimez tous les **flux d’utilisateurs (stratégies)** dans votre locataire Azure AD B2C.
1. Supprimez toutes les **applications (héritées)** inscrites dans votre locataire Azure AD B2C.
1. Sélectionnez **Azure Active Directory** dans le menu à gauche.
1. Sous **Gérer**, sélectionnez **Utilisateurs**.
1. Sélectionnez chaque utilisateur tour à tour (excluez l’utilisateur *Administrateur d’abonnements* sous lequel vous êtes connecté actuellement). Sélectionnez **Supprimer** en bas de la page, puis **OUI** lorsque vous y êtes invité.
1. Sous **Gérer**, sélectionnez **Inscriptions des applications**.
1. Sélectionnez **Afficher toutes les applications**.
1. Sélectionnez l’application nommée **b2c-extensions-app**, **Supprimer**, puis **Oui** lorsque vous y êtes invité.
1. Sous **Gérer**, sélectionnez **Paramètres utilisateur**.
1. Le cas échéant, sous **Connexions au compte LinkedIn**, sélectionnez **Non**, puis **Enregistrer**.
1. Sous **Gérer**, sélectionnez **Propriétés**.
1. Sous **Gestion de l’accès pour les ressources Azure**, sélectionnez **Oui**, puis **Enregistrer**.
1. Déconnectez-vous du portail Microsoft Azure, puis reconnectez-vous pour actualiser votre accès.
1. Sélectionnez **Azure Active Directory** dans le menu à gauche.
1. Dans la page **Vue d’ensemble**, sélectionnez **Supprimer le répertoire**. Suivez les instructions à l’écran pour achever le processus.

* * *

### <a name="can-i-get-azure-ad-b2c-as-part-of-enterprise-mobility-suite"></a>Puis-je obtenir Azure AD B2C dans le cadre d’Enterprise Mobility Suite ?

Non, Azure AD B2C est un service Azure avec paiement à l’utilisation et ne fait pas partie d’Enterprise Mobility Suite.

### <a name="how-do-i-report-issues-with-azure-ad-b2c"></a>Comment signaler des problèmes avec Azure AD B2C ?

Consultez [Azure Active Directory B2C : dépôt de demandes de support](support-options.md).