---
title: Questions fréquentes (FAQ) - Azure Active Directory | Microsoft Docs
description: Réponses aux questions courantes sur Azure et Azure Active Directory, la gestion des mots de passe et l’accès aux applications.
services: active-directory
author: ajburnle
manager: daveba
ms.assetid: b8207760-9714-4871-93d5-f9893de31c8f
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: troubleshooting
ms.date: 11/12/2018
ms.author: ajburnle
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4a8fcb14ac397791822c3a9065275dbfb2b3b7dd
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96860420"
---
# <a name="frequently-asked-questions-about-azure-active-directory"></a>Questions fréquentes (FAQ) sur Azure Active Directory
Azure Active Directory (Azure AD) est une solution IDaaS (Identity as a Service) complète qui couvre tous les aspects de l’identité, la gestion des accès et la sécurité.

Pour plus d’informations, consultez [Qu’est-ce qu’Azure Active Directory ?](active-directory-whatis.md).


## <a name="access-azure-and-azure-active-directory"></a>Accédez à Azure et à Azure Active Directory
**Q : Pourquoi le message « Aucun abonnement trouvé » s’affiche quand j’essaie d’accéder à Azure AD dans le portail Azure ?**

**R :** Pour accéder au portail Azure, chaque utilisateur doit y être autorisé dans le cadre d’un abonnement Azure. Si vous n’avez pas d’abonnement Microsoft 365 ou Azure AD payant, vous devez activer un [compte Azure](https://azure.microsoft.com/free/
) gratuit ou un abonnement payant.

Pour plus d'informations, consultez les pages suivantes :

* [Association des abonnements Azure avec Azure Active Directory](active-directory-how-subscriptions-associated-directory.md)

---
**Q : Quelle est la relation entre Azure AD, Microsoft 365 et Azure ?**

**R :** Azure AD vous fournit une identité et des fonctionnalités d’accès communes à tous les services web. Que vous utilisiez Microsoft 365, Microsoft Azure, Intune ou d’autres outils, vous utilisez déjà Azure AD afin d’activer l’authentification et la gestion des accès pour l’ensemble de ces services.

Tous les utilisateurs autorisés à utiliser les services web sont définis en tant que comptes d’utilisateurs dans une ou plusieurs instances d’Azure AD. Vous pouvez configurer ces comptes pour des fonctionnalités Azure AD gratuites, par exemple l’accès aux applications cloud.

Les services Azure AD payants comme Enterprise Mobility + Security complètent d’autres services web comme Microsoft 365 et Microsoft Azure avec des solutions avancées de gestion et de sécurité à l’échelle de l’entreprise.

---

**Q :  Quelles sont les différences entre Propriétaire et Administrateur général ?**

**R :** Par défaut, le rôle Propriétaire de ressources Azure est affecté à la personne qui souscrit un abonnement Azure. Un Propriétaire peut utiliser un compte Microsoft ou un compte professionnel ou scolaire provenant du répertoire auquel l’abonnement Azure est associé.  Ce rôle est autorisé à gérer les services sur le portail Azure.

Si d’autres personnes doivent se connecter et accéder aux services à l’aide de l’abonnement, vous pouvez leur assigner le [rôle intégré](../../role-based-access-control/built-in-roles.md) approprié. Pour obtenir des informations supplémentaires, consultez [Ajouter ou supprimer des attributions de rôles Azure à l’aide du portail Azure](../../role-based-access-control/role-assignments-portal.md).

Par défaut, le rôle d’administrateur général de l’annuaire est affecté à la personne qui souscrit un abonnement Azure. L’Administrateur général a accès à toutes les fonctionnalités de répertoire d’Azure AD. Azure AD dispose d’un autre ensemble de rôles d’administrateur qui permet de gérer le répertoire et les fonctionnalités liées à l’identité. Ces administrateurs ont accès à diverses fonctionnalités dans le portail Azure. Le rôle de l’administrateur détermine les opérations qu’il peut effectuer : créer ou modifier des utilisateurs, assigner des rôles d’administrateur à d’autres personnes, réinitialiser les mots de passe utilisateur, gérer les licences utilisateur et les domaines, etc.  Pour plus d’informations sur les administrateurs de répertoires Azure AD et leurs rôles, consultez [Attribuer des rôles d’administrateur à un utilisateur dans Azure Active Directory](active-directory-users-assign-role-azure-portal.md) et [Attribution de rôles d’administrateur dans Azure Active Directory](../roles/permissions-reference.md).

Par ailleurs, les services Azure AD payants comme Enterprise Mobility + Security complètent d’autres services web, tels que Microsoft 365 et Microsoft Azure, avec des solutions avancées de gestion et de sécurité à l’échelle de l’entreprise.

---
**Q : Existe-t-il un rapport qui indique la date d’expiration de mes licences utilisateur Azure AD ?**

**R :** Non.  Ce n’est pas le cas pour l’instant.

---

## <a name="get-started-with-hybrid-azure-ad"></a>Prise en main d’Azure AD hybride


**Q : Comment quitter un locataire quand je suis ajouté comme collaborateur ?**

**R :** Quand vous êtes ajouté comme collaborateur au locataire d’une autre organisation, vous pouvez utiliser le « sélecteur de locataire » en haut à droite pour basculer entre les locataires.  Il n’existe actuellement aucun moyen de quitter l’organisation à l’origine de l’invitation, mais Microsoft travaille à la mise en place de cette fonctionnalité.  En attendant que cette fonctionnalité soit disponible, vous pouvez demander à l’organisation à l’origine de l’invitation de vous retirer de son client.

---
**Q : Comment connecter mon annuaire local à Azure AD ?**

**R :** Vous pouvez connecter votre annuaire local à Azure AD à l’aide d’Azure AD Connect.

Pour plus d’informations, consultez [Intégration des identités locales avec Azure Active Directory](../hybrid/whatis-hybrid-identity.md).

---
**Q : Comment configurer l’authentification unique entre mon annuaire local et mes applications cloud ?**

**R :** Vous devez uniquement configurer l’authentification unique (SSO) entre votre annuaire local et Azure AD. Tant que vous accédez à vos applications cloud via Azure AD, le service conduit automatiquement vos utilisateurs à s’authentifier correctement avec leurs informations d’identification locales.

L’implémentation du SSO à partir de l’emplacement local peut être facilement mise en œuvre à l’aide de solutions de fédération telles que les services de fédération Active Directory (AD FS) ou en configurant la synchronisation du hachage de mot de passe. Vous pouvez facilement déployer les deux options à l’aide de l’Assistant de configuration Azure AD Connect.

Pour plus d’informations, consultez [Intégration des identités locales avec Azure Active Directory](../hybrid/whatis-hybrid-identity.md).

---
**Q : Azure AD fournit-il un portail en libre-service aux utilisateurs de mon organisation ?**

**R :** Oui, Azure AD fournit le [Panneau d’accès Azure AD](https://myapps.microsoft.com) pour l’accès aux applications en libre-service. Si vous êtes un client Microsoft 365, vous trouverez la plupart des mêmes fonctionnalités dans le [portail Office 365](https://portal.office.com).

Pour plus d’informations, consultez [Présentation du volet d’accès](../user-help/my-apps-portal-end-user-access.md).

---
**Q : Azure AD m’aide-t-il à gérer mon infrastructure locale ?**

**R :** Oui. L’édition Azure AD Premium offre l’outil Azure AD Connect Health. Azure AD Connect Health vous permet de surveiller et d’analyser votre infrastructure d’identité locale et les services de synchronisation.  

Pour plus d’informations, consultez [Surveillez votre infrastructure d’identité locale et vos services de synchronisation dans le cloud](../hybrid/whatis-azure-ad-connect.md).  

---
## <a name="password-management"></a>Gestion des mots de passe
**Q : Puis-je utiliser la réécriture de mot de passe Azure AD sans synchronisation de mot de passe ? (Dans ce scénario, est-il possible d’utiliser la réinitialisation de mot de passe en libre-service d’Azure AD avec l’écriture différée de mot de passe et de ne pas stocker les mots de passe dans le cloud ?)**

**R :** Vous n’avez pas besoin de synchroniser vos mots de passe Active Directory sur Azure AD pour activer la réécriture. Dans un environnement fédéré, l’authentification unique (SSO) Azure AD s’appuie sur l’annuaire local pour authentifier l’utilisateur. Ce scénario ne nécessite pas le suivi du mot de passe local dans Azure AD.

---
**Q : Combien de temps faut-il pour réécrire un mot de passe sur Active Directory en local ?**

**R :** La réécriture de mot de passe fonctionne en temps réel.

Pour en savoir plus, voir [Prise en main de la gestion de mot de passe](../authentication/tutorial-enable-sspr.md).

---
**Q : Puis-je utiliser la réécriture de mot de passe avec des mots de passe gérés par un administrateur ?**

**R :** Oui, si cette fonction est activée, les opérations de mot de passe effectuées par un administrateur sont réécrites dans votre environnement local.  

<a name="for-more-answers-to-password-related-questions-see-password-management-frequently-asked-questions"></a>Si vous avez d’autres questions concernant les mots de passe, consultez [Forum aux questions - Gestion des mots de passe](../authentication/active-directory-passwords-faq.md).
---
**Q :  Que faire si je ne me souviens pas de mon mot de passe Microsoft 365/Azure AD actuel quand je tente de changer mon mot de passe ?**

**R :** Pour ce cas de figure, plusieurs options s’offrent à vous.  Utilisez la réinitialisation de mot de passe en libre-service si elle est disponible.  Le fonctionnement de la réinitialisation de mot de passe en libre-service dépend de la façon dont elle est configurée.  Pour plus d’informations, consultez [Fonctionnement du portail de réinitialisation de mot de passe](../authentication/howto-sspr-deployment.md).

Pour les utilisateurs Microsoft 365, l’administrateur peut réinitialiser le mot de passe en appliquant la procédure décrite dans [Réinitialiser les mots de passe utilisateur](https://support.office.com/article/Admins-Reset-user-passwords-7A5D073B-7FAE-4AA5-8F96-9ECD041ABA9C?ui=en-US&rs=en-US&ad=US).

Pour les comptes Azure AD, les administrateurs peuvent réinitialiser les mots de passe à l’aide de l’une des opérations suivantes :

- [Réinitialiser les comptes dans le portail Azure](active-directory-users-reset-password-azure-portal.md)
- [Utiliser PowerShell](/powershell/module/msonline/set-msoluserpassword)


---
## <a name="security"></a>Sécurité
**Q : Les comptes sont-ils verrouillés après un nombre spécifique de tentatives infructueuses, ou une stratégie plus élaborée est-elle utilisée ?**

Nous appliquons une stratégie plus étoffée pour verrouiller les comptes.  Cette dernière repose sur l’adresse IP de la demande et sur les mots de passe entrés. La durée du verrouillage augmente également en fonction de la probabilité de l’existence d’une attaque.  

**Q :  Certains mots de passe (courants) sont rejetés avec un message du type : « Ce mot de passe a été utilisé trop souvent ». Ce message fait-il référence aux mots de passe utilisés dans l’instance Active Directory actuelle ?**

Ce message s’applique aux mots de passe couramment utilisés au niveau mondial, tels que toutes les variantes de « Password » et de « 123456 ».

**Q : Une demande de connexion émanant de sources douteuses (botnets, point de terminaison Tor) sera-t-elle bloquée dans un client B2C ou nécessite-t-elle un client d’une édition De base ou Premium ?**

Nous disposons d’une passerelle qui filtre les demandes et offre une certaine protection contre les botnets. Elle s’applique à tous les clients B2C.

## <a name="application-access"></a>Accès aux applications

**Q : Où puis-je trouver une liste des applications qui sont déjà intégrées à Azure AD et leurs fonctionnalités ?**

**R :** Azure AD offre plus de 2 600 applications pré-intégrées développées par Microsoft, des fournisseurs de services d’application et des partenaires. Toutes les applications pré-intégrées prennent en charge l’authentification unique (SSO). L’authentification unique vous permet d’utiliser vos informations d’identification professionnelles pour accéder à vos applications. Certaines applications prennent également en charge l’approvisionnement et l’annulation d’approvisionnement automatisés.

Pour obtenir une liste complète des applications déjà intégrées, voir [Marketplace Active Directory](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.AzureActiveDirectory).

---
**Q : Que se passe-t-il si l’application dont j’ai besoin ne figure pas dans la Place de marché Azure AD ?**

**R :** Azure AD Premium vous permet d’ajouter et de configurer n’importe quelle application. Vous pouvez configurer, selon les fonctionnalités de votre application et vos préférences, l’authentification unique et l’approvisionnement automatisé.  

Pour plus d'informations, consultez les pages suivantes :

* [Configuration de l'authentification unique pour les applications ne faisant pas partie de la galerie d'applications Azure Active Directory.](../manage-apps/configure-saml-single-sign-on.md)
* [Utilisation de SCIM pour activer la configuration automatique des utilisateurs et des groupes d’Azure Active Directory sur des applications](../app-provisioning/use-scim-to-provision-users-and-groups.md)

---
**Q : Comment les utilisateurs peuvent-ils se connecter aux applications à l’aide d’Azure AD ?**

**R :** Azure AD offre aux utilisateurs plusieurs moyens de voir leurs applications et d’y accéder, par exemple :

* Panneau d’accès Azure AD
* Lanceur d’applications Microsoft 365
* Authentification directe pour les applications fédérées
* Liens ciblés vers des applications fédérées, avec mot de passe ou des applications existantes

Pour plus d’informations, consultez [Expériences d’utilisateurs finaux auprès des applications](../manage-apps/end-user-experiences.md).

---
**Q : Par quels moyens Azure AD active-t-il l’authentification et l’authentification unique pour la connexion aux applications ?**

**R :** Azure AD prend en charge de nombreux protocoles standardisés pour l’authentification et l’autorisation, tels que SAML 2.0, OpenID Connect, OAuth 2.0 et WS-Federation. Azure AD prend également en charge la mise en coffre du mot de passe et les fonctionnalités d’authentification automatisées pour les applications qui prennent uniquement en charge l’authentification basée sur les formulaires.  

Pour plus d'informations, consultez les pages suivantes :

* [Scénarios d’authentification pour Azure AD](../develop/authentication-vs-authorization.md)
* [Protocoles d’authentification Active Directory](/previous-versions/azure/dn151124(v=azure.100))
* [Authentification unique pour les applications dans Azure AD](../manage-apps/what-is-single-sign-on.md)

---
**Q : Puis-je ajouter des applications si l’exécution s’effectue en local ?**

**R :** Le proxy d’application Azure AD vous offre un accès facile et sécurisé aux applications web en local de votre choix. Vous pouvez accéder à ces applications de la même façon que vous accédez à vos applications SaaS (Software as a Service) dans Azure AD. Vous n’avez pas besoin de recourir à un VPN ou de modifier votre infrastructure réseau.  

Pour plus d’informations, consultez [Offrir un accès à distance sécurisé aux applications locales](../manage-apps/application-proxy.md).

---
**Q : Comment imposer l’authentification multifacteur aux utilisateurs qui accèdent à une application spécifique ?**

**R :** L’accès conditionnel Azure AD vous permet d’affecter une stratégie d’accès unique à chaque application. Dans votre stratégie, vous pouvez exiger l’authentification multificateur en permanence ou lorsque les utilisateurs ne sont pas connectés au réseau local.  

Pour plus d’informations, consultez [Sécurisation de l’accès à Microsoft 365 et à d’autres applications connectées à Azure Active Directory](../conditional-access/overview.md).

---
**Q : Qu’est-ce que l’attribution automatique des utilisateurs dans les applications SaaS ?**

**R :** Azure AD vous permet d’automatiser la création, la maintenance et la suppression d’identités utilisateur dans de nombreuses applications SaaS cloud populaires.

Pour plus d’informations, consultez [Automatisation de l’approvisionnement et de l’annulation de l’approvisionnement des utilisateurs pour les applications SaaS avec Azure Active Directory](../app-provisioning/user-provisioning.md).

---
**Q :  Puis-je configurer une connexion LDAP sécurisée avec Azure AD ?**

**R :**  Non. Azure AD ne prend pas directement en charge le protocole LDAP (Lightweight Directory Access Protocol) (LDAP) ou LDAP sécurisé. Il est cependant possible d’activer l’instance Azure AD Domain Services (Azure AD DS) sur votre locataire Azure AD avec des groupes de sécurité réseau correctement configurés via la mise en réseau Azure afin d'obtenir une connectivité LDAP. Pour plus d’informations, consultez [Configurer le protocole LDAP pour un domaine géré par les services de domaine Azure Active Directory](../../active-directory-domain-services/tutorial-configure-ldaps.md)
