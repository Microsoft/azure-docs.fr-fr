---
title: FAQ sur la collaboration B2B - Azure Active Directory | Microsoft Docs
description: Trouvez les réponses aux questions les plus fréquentes sur Azure Active Directory B2B Collaboration.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: reference
ms.date: 09/23/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 90cc15dc3944efd82d025df1bb5072c57346055e
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97355628"
---
# <a name="azure-active-directory-b2b-collaboration-faqs"></a>FAQ sur Azure Active Directory B2B Collaboration

Cette foire aux questions (FAQ) sur la collaboration interentreprises (B2B) Azure Active Directory (Azure AD) est régulièrement complétée par de nouvelles rubriques.

> [!IMPORTANT]
> - **À compter du 4 janvier 2021**, Google [déconseille la prise en charge de la connexion WebView](https://developers.googleblog.com/2020/08/guidance-for-our-effort-to-block-less-secure-browser-and-apps.html). Si vous utilisez la fédération Google ou l’inscription en libre-service avec Gmail, [testez la compatibilité de vos applications métier natives](google-federation.md#deprecation-of-webview-sign-in-support).
> - **À compter du 31 mars 2021**, Microsoft ne prendra plus en charge l’échange d’invitations en créant des locataires et des comptes Azure AD non managés pour les scénarios de collaboration B2B. Dans cette optique, nous encourageons les clients à choisir l’[authentification au moyen d’un code secret à usage unique envoyé par e-mail](one-time-passcode.md). Nous serions heureux de recevoir vos commentaires sur cette fonctionnalité de préversion publique, et sommes ravis de vous proposer encore plus de moyens de collaborer.

### <a name="can-we-customize-our-sign-in-page-so-its-more-intuitive-for-our-b2b-collaboration-guest-users"></a>Peut-on personnaliser sa page de connexion afin qu’elle soit plus intuitive pour les utilisateurs invités B2B Collaboration ?
Absolument ! Consultez notre [billet de blog sur cette fonctionnalité](https://blogs.technet.microsoft.com/enterprisemobility/2017/04/07/improving-the-branding-logic-of-azure-ad-login-pages/). Pour plus d’informations sur la personnalisation de la page de connexion d’une organisation, consultez la page [Ajouter la personnalisation de l’entreprise aux pages de connexion et du volet d’accès](../fundamentals/customize-branding.md).

### <a name="can-b2b-collaboration-users-access-sharepoint-online-and-onedrive"></a>Les utilisateurs de B2B Collaboration peuvent-ils accéder à SharePoint Online et OneDrive ?
Oui. La fonctionnalité de recherche d’utilisateurs invités existants avec le sélecteur de personnes dans SharePoint Online est cependant **désactivée** par défaut. Pour activer l’option de recherche d’utilisateurs invités, définissez **ShowPeoplePickerSuggestionsForGuestUsers** sur **Activé**. Vous pouvez activer ce paramètre au niveau du client ou bien de la collection de sites. Vous pouvez modifier ce paramètre à l’aide des applets de commande Set-SPOTenant et Set-SPOSite. Grâce à ces applets de commande, les membres peuvent rechercher parmi tous les utilisateurs invités existants du répertoire. Les modifications apportées à l’étendue du client n’affectent pas les sites SharePoint Online déjà configurés.

### <a name="is-the-csv-upload-feature-still-supported"></a>La fonctionnalité de chargement CSV est-elle toujours prise en charge ?
Oui. Pour plus d’informations sur la fonctionnalité de chargement de fichiers .csv, consultez [cet exemple PowerShell](code-samples.md).

### <a name="how-can-i-customize-my-invitation-emails"></a>Comment puis-je personnaliser mes e-mails d’invitation ?
Vous pouvez personnaliser pratiquement tous les éléments du processus de l’inviteur à l’aide des [API d’invitation B2B](customize-invitation-api.md).

### <a name="can-guest-users-reset-their-multi-factor-authentication-method"></a>Les utilisateurs invités peuvent-ils réinitialiser leur méthode d’authentification multifacteur ?
Oui. Les utilisateurs invités peuvent réinitialiser leur méthode d’authentification multifacteur de la même manière que les utilisateurs normaux.

### <a name="which-organization-is-responsible-for-multi-factor-authentication-licenses"></a>Quelle est l’organisation responsable des licences de l’authentification multifacteur ?
L’organisation à l’origine de l’invitation effectue l’authentification multifacteur. Elle doit s’assurer que l’organisation dispose d’un nombre suffisant de licences pour ses utilisateurs B2B qui utilisent l’authentification multifacteur.

### <a name="what-if-a-partner-organization-already-has-multi-factor-authentication-set-up-can-we-trust-their-multi-factor-authentication-and-not-use-our-own-multi-factor-authentication"></a>Que se passe-t-il si une organisation partenaire a déjà configuré l’authentification multifacteur ? Pouvons-nous faire confiance à son authentification multifacteur et ne pas utiliser notre propre authentification multifacteur ?
Cette fonctionnalité n’est pas prise en charge pour l’instant. Si l’accès aux ressources de votre organisation nécessite une authentification multifacteur, l’organisation partenaire doit s’inscrire pour l’authentification multifacteur dans votre organisation (celle qui invite).

### <a name="how-can-i-use-delayed-invitations"></a>Comment utiliser les invitations différées ?
Une organisation peut souhaiter ajouter des utilisateurs B2B Collaboration et les approvisionner pour les applications au fil des besoins, avant d'envoyer des invitations. Vous pouvez utiliser l’API d’invitation B2B Collaboration pour personnaliser le workflow d’intégration.

### <a name="can-i-make-guest-users-visible-in-the-exchange-global-address-list"></a>Puis-je rendre les utilisateurs invités visibles dans la liste d’adresses globale Exchange ?
Oui. Par défaut, les objets invités ne sont pas visibles dans la liste d’adresses globale (GAL) de votre organisation, mais vous pouvez utiliser Azure Active Directory PowerShell pour les rendre visibles. Consultez [Puis-je rendre les objets invités visibles dans la liste d’adresses globale ?](/office365/admin/create-groups/manage-guest-access-in-groups#add-guests-to-the-global-address-list)

### <a name="can-i-make-a-guest-user-a-limited-administrator"></a>Peut-on faire d’un utilisateur invité un administrateur limité ?
Absolument. Pour plus d’informations, consultez [Ajouter des utilisateurs invités à un rôle](add-guest-to-role.md).

### <a name="does-azure-ad-b2b-collaboration-allow-b2b-users-to-access-the-azure-portal"></a>Azure AD B2B Collaboration permet-il aux utilisateurs B2B d’accéder au Portail Azure ?
À moins que le rôle d’administrateur limité ne leur soit assigné, les utilisateurs de collaboration B2B n’ont pas besoin d’accéder au portail Azure. Toutefois, les utilisateurs de collaboration B2B qui disposent du rôle d’administrateur limité peuvent accéder au portail. En outre, si un utilisateur invité qui ne possède aucun de ces rôles Administrateur accède au portail, il peut être en mesure d’accéder à certaines parties de l’expérience. Le rôle utilisateur invité possède quelques autorisations dans le répertoire.

### <a name="can-i-block-access-to-the-azure-portal-for-guest-users"></a>Puis-je bloquer l’accès au portail Azure pour les utilisateurs invités ?

Oui. Vous pouvez créer une stratégie d’accès conditionnel qui empêche tous les utilisateurs invités et externes d’accéder au Portail Azure. Lorsque vous configurez cette stratégie, faites attention à ne pas bloquer accidentellement l’accès aux membres et aux administrateurs.

1. Connectez-vous au [portail Azure](https://portal.azure.com/) en tant qu’administrateur de sécurité ou administrateur de l’accès conditionnel.
2. Dans le portail Azure, sélectionnez **Azure Active Directory**. 
3. Dans **Gérer**, sélectionnez **Sécurité**.
4. Sous **Sécurité**, sélectionnez **Accès conditionnel**. Sélectionnez **Nouvelle stratégie**.
5. Sur la page **Nouveau**, donnez un nom à la stratégie (par exemple, « Empêcher les invités d’accéder au portail ») dans la zone de texte **Nom**.
6. Sous **Affectations**, sélectionnez **Utilisateurs et groupes**.
7. Dans l’onglet **Inclure**, choisissez **Sélectionner des utilisateurs et groupes**, puis **Tous les utilisateurs invités et externes (préversion)** .
9. Sélectionnez **Terminé**.
10. Sur la page **Nouveau**, sélectionnez **Applications cloud ou actions** dans la section **Affectations**.
11. Sur la page **Applications cloud ou actions**, choisissez **Sélectionner des applications**, puis **Sélectionner**.
12. Dans la page **Sélectionner**, choisissez **Gestion Microsoft Azure**, puis **Sélectionner**.
13. Sur la page **Applications cloud ou actions**, sélectionnez **Terminé**.

### <a name="does-azure-ad-b2b-collaboration-support-multi-factor-authentication-and-consumer-email-accounts"></a>Azure AD B2B Collaboration prend-il en charge l’authentification multifacteur et les comptes de messagerie grand public ?
Oui. Les comptes de messagerie grand public et l’authentification multifacteur sont tous deux pris en charge par Azure AD B2B Collaboration.

### <a name="do-you-support-password-reset-for-azure-ad-b2b-collaboration-users"></a>Prenez-vous en charge la réinitialisation de mot de passe pour les utilisateurs Azure AD B2B Collaboration ?
Si votre locataire Azure AD est le répertoire de base d’un utilisateur, vous pouvez [réinitialiser le mot de passe de l’utilisateur](../fundamentals/active-directory-users-reset-password-azure-portal.md) à partir du portail Azure. Mais vous ne pouvez pas directement réinitialiser le mot de passe d’un utilisateur invité qui se connecte avec un compte géré par un autre répertoire Azure AD ou fournisseur d'identité externe. Seul l’utilisateur invité ou un administrateur dans le répertoire de base de l’utilisateur peut réinitialiser le mot de passe. Voici quelques exemples de la façon dont le mot de passe réinitialisé fonctionne pour les utilisateurs invités :
 
* Les utilisateurs invités qui se connectent avec un compte Microsoft (par exemple guestuser@live.com) peuvent réinitialiser leur mot de passe à l’aide de la méthode Self-service Password Reset (SSPR) du compte Microsoft. Consultez [Réinitialisation du mot de passe de votre compte Microsoft](https://support.microsoft.com/help/4026971/microsoft-account-how-to-reset-your-password).
* Les utilisateurs invités qui se connectent avec un compte Google ou un autre fournisseur d’identité externe peuvent réinitialiser leur mot de passe à l’aide de la méthode SSPR de leur fournisseur d’identité. Par exemple, un utilisateur invité avec le compte Google guestuser@gmail.com peut réinitialiser son mot de passe en suivant les instructions de la section [Modifier ou réinitialiser votre mot de passe](https://support.google.com/accounts/answer/41078).
* Si le locataire de l’identité est de locataire de type juste-à-temps (JIT) ou « viral » (un client Azure distinct et non managé), seul l’utilisateur invité peut réinitialiser son mot de passe. Parfois, une organisation [reprend la gestion des locataires viraux](../enterprise-users/domains-admin-takeover.md) qui sont créés lorsque des employés utilisent leur adresse e-mail professionnelle pour s'inscrire à des services. Dès lors que l'organisation adopte un locataire viral, seul un administrateur de cette organisation peut réinitialiser le mot de passe de l'utilisateur ou activer la réinitialisation de mot de passe en libre-service. Le cas échéant, en tant qu'organisation à l'origine de l'invitation, vous pouvez supprimer le compte d'utilisateur invité de votre annuaire et renvoyer une invitation.

* Si le répertoire de base de l’utilisateur invité est votre client Azure AD, vous pouvez réinitialiser le mot de passe. Par exemple, vous avez peut-être créé un utilisateur ou synchronisé un utilisateur à partir de votre annuaire Active Directory local et défini sa valeur UserType sur invité. Comme cet utilisateur est hébergé dans votre annuaire, vous pouvez réinitialiser son mot de passe à partir du portail Azure.

### <a name="does-microsoft-dynamics-365-provide-online-support-for-azure-ad-b2b-collaboration"></a>Microsoft Dynamics 365 fournit-il un support en ligne pour Azure AD B2B Collaboration ?
Oui, Dynamics 365 (en ligne) prend en charge Azure AD B2B Collaboration. Pour plus d’informations, consultez l’article Dynamics 365 [Inviter des utilisateurs avec Azure Active Directory B2B Collaboration](/dynamics365/customer-engagement/admin/invite-users-azure-active-directory-b2b-collaboration).

### <a name="what-is-the-lifetime-of-an-initial-password-for-a-newly-created-b2b-collaboration-user"></a>Quelle est la durée de vie d’un mot de passe initial pour un utilisateur B2B Collaboration nouvellement créé ?
Azure AD dispose d’un ensemble fixe d'exigences en matière de nombre de caractères, de force du mot de passe et de verrouillage de compte qui s’appliquent à tous les comptes d’utilisateur cloud Azure AD. Les comptes d’utilisateur cloud sont des comptes qui ne sont pas fédérés avec un autre fournisseur d’identité, comme 
* Compte Microsoft
* Facebook
* Services de fédération Active Directory (AD FS)
* Un autre locataire cloud (pour B2B Collaboration)

Pour les comptes fédérés, la stratégie de mot de passe dépend de la stratégie qui s’applique dans la location locale et des paramètres de compte Microsoft de l’utilisateur.

### <a name="an-organization-might-want-to-have-different-experiences-in-their-applications-for-tenant-users-and-guest-users-is-there-standard-guidance-for-this-is-the-presence-of-the-identity-provider-claim-the-correct-model-to-use"></a>Une organisation peut avoir différentes expériences dans ses applications pour les utilisateurs clients et les utilisateurs invités. Une assistance standard est-elle disponible pour cela ? La présence de la revendication de fournisseur d’identité représente-t-elle le bon modèle ?
Un utilisateur invité peut utiliser n’importe quel fournisseur d’identité pour s’authentifier. Pour plus d’informations, consultez la page [Propriétés d’un utilisateur B2B Collaboration](user-properties.md). Utilisez la propriété **UserType** pour déterminer l’expérience utilisateur. La revendication **UserType** n’est pour le moment pas incluse dans le jeton. Les applications doivent utiliser l’API Microsoft Graph pour interroger le répertoire de l’utilisateur et obtenir le UserType.

### <a name="where-can-i-find-a-b2b-collaboration-community-to-share-solutions-and-to-submit-ideas"></a>Où puis-je trouver une communauté B2B Collaboration pour partager des solutions et envoyer des idées ?
Nous sommes constamment à l’écoute de vos commentaires afin d’améliorer B2B Collaboration. Veuillez partager vos scénarios utilisateur, vos meilleures pratiques et ce que vous appréciez par rapport à Azure AD B2B Collaboration. Participez à la discussion dans la [Communauté Microsoft Tech](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-B2B/bd-p/AzureAD_B2b).
 
Nous vous invitons également à soumettre vos idées et à voter pour les prochaines fonctionnalités sur la page [Idées B2B Collaboration](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-B2B-Ideas/idb-p/AzureAD_B2B_Ideas).

### <a name="can-we-send-an-invitation-that-is-automatically-redeemed-so-that-the-user-is-just-ready-to-go-or-does-the-user-always-have-to-click-through-to-the-redemption-url"></a>Peut-on envoyer une invitation utilisée automatiquement, afin que l’utilisateur soit simplement « prêt » ? L’utilisateur doit-il toujours cliquer sur l’URL d’utilisation ?
Vous pouvez inviter d’autres utilisateurs dans l’organisation partenaire à l’aide de l’interface utilisateur, de scripts PowerShell ou d’API. Vous pouvez ensuite envoyer à l’utilisateur invité un lien direct vers une application partagée. Dans la plupart des cas, il n’est plus nécessaire d’ouvrir l’invitation par e-mail et de cliquer sur une URL d’échange. Voir [Utilisation d’invitations Azure Active Directory B2B Collaboration](redemption-experience.md).

### <a name="how-does-b2b-collaboration-work-when-the-invited-partner-is-using-federation-to-add-their-own-on-premises-authentication"></a>Comment fonctionne la collaboration B2B lorsque le partenaire invité utilise la fédération pour ajouter sa propre authentification locale ?
Si le partenaire a un client Azure AD qui est fédéré à l’infrastructure d’authentification locale, l’authentification unique (SSO) locale s’effectue automatiquement. Si le partenaire n’a pas de client Azure AD, un compte Azure AD est créé pour les nouveaux utilisateurs. 

### <a name="i-thought-azure-ad-b2b-didnt-accept-gmailcom-and-outlookcom-email-addresses-and-that-b2c-was-used-for-those-kinds-of-accounts"></a>Je pensais qu’Azure AD B2B n’acceptait pas les adresses de messagerie gmail.com et outlook.com, et que B2C était utilisé pour ces types de comptes ?
Nous supprimons les différences entre B2B et B2C (des entreprises aux particuliers) Collaboration en ce qui concerne la prise en charge des identités. L’identité utilisée n’est pas un bon critère de choix entre B2B et B2C. Pour plus d’informations sur le choix de l’option de collaboration, consultez la page [Comparer B2C et B2B Collaboration dans Azure Active Directory](compare-with-b2c.md).

### <a name="can-an-azure-ad-b2c-local-account-be-invited-to-an-azure-ad-tenant-for-b2b-collaboration"></a>Un compte local Azure AD B2C peut-il être invité sur un locataire Azure AD pour B2B Collaboration ?
Non. Un compte local Azure AD B2C ne peut être utilisé que pour se connecter au locataire Azure AD B2C. Le compte ne peut pas être utilisé pour se connecter à un locataire Azure AD. L’invitation d’un compte local Azure AD B2C sur un locataire Azure AD pour B2B Collaboration n’est pas prise en charge.

### <a name="what-applications-and-services-support-azure-b2b-guest-users"></a>Quelles applications et services prennent en charge les utilisateurs invités Azure B2B ?
Toutes les applications intégrées Azure AD peuvent prendre en charge les utilisateurs invités Azure B2B, mais elles doivent utiliser un point de terminaison configuré comme locataire pour authentifier les utilisateurs invités. Vous devrez peut-être également [personnaliser les revendications](claims-mapping.md) dans le jeton SAML émis lorsqu’un utilisateur invité s’authentifie auprès de l’application. 

### <a name="can-we-force-multi-factor-authentication-for-b2b-guest-users-if-our-partners-dont-have-multi-factor-authentication"></a>Pouvons-nous forcer l’authentification multifacteur pour les utilisateurs invités B2B si nos partenaires n’ont pas l’authentification multifacteur ?
Oui. Pour plus d’informations, voir [Accès conditionnel pour les utilisateurs de collaboration B2B](conditional-access.md).

### <a name="in-sharepoint-you-can-define-an-allow-or-deny-list-for-external-users-can-we-do-this-in-azure"></a>Dans SharePoint, il est possible de définir une liste « allow (autoriser) » ou « deny (refuser) » pour les utilisateurs externes. Peut-on faire cela dans Azure ?
Oui. Azure AD B2B Collaboration prend en charge les listes autorisées et refusées. 

### <a name="what-licenses-do-we-need-to-use-azure-ad-b2b"></a>Quelles licences faut-il avoir pour utiliser Azure AD B2B ?
Pour plus d’informations sur les licences dont votre organisation a besoin pour utiliser Azure AD B2B, consultez [Prix des identités externes](external-identities-pricing.md).

### <a name="next-steps"></a>Étapes suivantes

- [Qu'est-ce que la collaboration B2B d'Azure AD ?](what-is-b2b.md)