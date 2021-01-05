---
title: Planifier les modifications de nom d’utilisateur principal Azure et résoudre les problèmes associés
description: Comprendre les problèmes connus et les solutions d’atténuation pour les modifications d’UPN
services: active-directory
ms.service: active-directory
ms.subservice: hybrid
ms.topic: how-to
ms.date: 03/13/2020
ms.author: baselden
author: barbaraselden
manager: daveba
ms.reviewer: jsimmons
ms.collection: M365-identity-device-management
ms.openlocfilehash: 19e40d135316c1c7cd270d2804fff1f487937685
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96858533"
---
# <a name="plan-and-troubleshoot-user-principal-name-changes-in-azure-active-directory"></a>Planifier les modifications de nom d’utilisateur principal et résoudre les problèmes associés dans Azure Active Directory

Un nom d’utilisateur principal (UPN) est un attribut constituant une norme de communication Internet pour les comptes d’utilisateur. Un UPN se compose d’un préfixe UPN (nom de compte d’utilisateur) et d’un suffixe UPN (nom de domaine DNS). Le préfixe et le suffixe sont accolés par le symbole « \@ ». Par exemple : someone@example.com. Un UPN doit être unique parmi tous les objets principaux de sécurité d’une forêt de répertoires. 

**Cet article suppose que vous utilisez l’UPN comme identificateur d’utilisateur. Il traite de la planification des modifications d’UPN, et de la résolution de problèmes pouvant en découler.**

> [!NOTE]
> Nous recommandons aux développeurs d’utiliser l’ID d’objet utilisateur comme identificateur immuable, plutôt que l’UPN ou les adresse e-mail, puisque leurs veleurs peuvent changer.


## <a name="learn-about-upns-and-upn-changes"></a>Découvrir les UPN et les modifications d’UPN
Les pages de connexion invitent souvent les utilisateurs à entrer leur adresse e-mail alors que la valeur demandée est en réalité leur UPN. Ainsi, vous devez veiller à modifier l’UPN de l’utilisateur chaque fois que son adresse e-mail principale change.

Les adresses e-mail principales des utilisateurs peuvent changer pour de nombreuses raisons :

* changement de nom de la société ;

* employés changeant de division au sein de l’entreprise ; 

* fusions et acquisitions ;

* modifications de nom d’employé.

### <a name="types-of-upn-changes"></a>Types de modifications de l’UPN

Vous pouvez modifier un UPN en modifiant le préfixe, le suffixe ou les deux à la fois.

* **Modification du préfixe**.

   *  Par exemple, si le nom d’une personne a changé, vous pouvez modifier son nom de compte et changer :  
‎BSimon@contoso.com pour BJohnson@contoso.com

   * Vous pouvez également modifier la norme d’entreprise pour les préfixes et changer :  
‎Bsimon@contoso.com pour Britta.Simon@contoso.com

* **Modification du suffixe**. <br>

    Par exemple, si une personne a changé de division, vous pouvez modifier son domaine : 

   * Il lance Britta.Simon@contoso.com sur Britta.Simon@contosolabs.com. <br>
     ou<br>
    * Il lance Britta.Simon@corp.contoso.com sur Britta.Simon@labs.contoso.com. 

Nous vous recommandons de modifier l’UPN des utilisateurs chaque fois que leur adresse de messagerie principale est mise à jour.

Pendant la synchronisation initiale entre Active Directory et Azure AD, assurez-vous que les e-mails des utilisateurs sont identiques à leurs UPN.

### <a name="upns-in-active-directory"></a>Les UPN dans Active Directory

Dans Active Directory, le suffixe UPN par défaut est le nom DNS du domaine dans lequel vous avez créé le compte d’utilisateur. Dans la plupart des cas, il s’agit du nom de domaine que vous enregistrez en tant que domaine d’entreprise sur Internet. Si vous créez le compte d’utilisateur dans le domaine contoso.com, l’UPN par défaut est

username@contoso.com

 Toutefois, vous pouvez [ajouter d’autres suffixes UPN](../fundamentals/add-custom-domain.md) en utilisant les domaines et approbations Active Directory. 

Ainsi, vous avez peut-être envie d’ajouter labs.contoso.com et faire en sorte que les UPN et e-mails des utilisateurs reflètent cette information. Ils deviennent alors

username@labs.contoso.com.

>[!IMPORTANT]
> Si vous [modifiez le suffixe dans Active Directory](../fundamentals/add-custom-domain.md), vous devez vous assurer qu’un nom de domaine personnalisé correspondant a été [ajouté et vérifié sur Azure AD](../fundamentals/add-custom-domain.md). 

![Capture d’écran des domaines vérifiés](./media/howto-troubleshoot-upn-changes/custom-domains.png)

### <a name="upns-in-azure-active-directory"></a>UPN dans Azure Active Directory

Les utilisateurs se connectent à Azure AD avec la valeur de leur attribut userPrincipalName. 

Lorsque vous utilisez Azure AD conjointement avec votre instance Active Directory locale, les comptes d’utilisateur sont synchronisés à l’aide du service Azure AD Connect. Par défaut, l’Assistant Azure AD Connect utilise l’attribut userPrincipalName de l’annuaire Active Directory local comme étant l’UPN dans Azure AD. Vous pouvez le remplacer par un autre attribut dans une installation personnalisée.

Il est important de disposer d’un processus bien défini lorsque vous mettez à jour le nom d’utilisateur principal (UPN) d’un seul utilisateur, ou pour l’ensemble de votre organisation. 

Consultez la section sur les problèmes connus et les solutions de contournement dans ce document.

Lorsque vous synchronisez des comptes d’utilisateur depuis Active Directory vers Azure AD, assurez-vous que les UPN d’Active Directory mappent vers les domaines vérifiés d’Azure AD.

![Capture d’écran montrant des exemples d’UPN mappés à des domaines Azure AD vérifiés.](./media/howto-troubleshoot-upn-changes/verified-domains.png)

Si la valeur de l’attribut userPrincipalName ne correspond pas à un domaine vérifié dans Azure AD, le processus de synchronisation remplace le suffixe par une valeur .onmicrosoft.com par défaut.


### <a name="roll-out-bulk-upn-changes"></a>Déployer les modifications d’UPN en bloc

Pour les modifications d'UPN en bloc, suivez les [meilleures pratiques qui s'appliquent à un pilote](../fundamentals/active-directory-deployment-plans.md). Vous devez également disposer d’un plan de restauration testé pour rétablir les UPN si vous rencontrez des problèmes ne pouvant pas être résolus rapidement. Dès lors que le pilote est en cours d’exécution, vous pouvez commencer à cibler de petits nombres d’utilisateurs, avec divers rôles de l’organisation et leurs ensembles spécifiques d’applications ou d’appareils.

En procédant avec ce premier sous-ensemble d’utilisateurs, vous obtenez une bonne idée de ce à quoi les utilisateurs doivent s’attendre avec cette modification. Intégrez ces informations à vos communications destinées aux utilisateurs.

Créez une procédure définie pour modifier les UPN sur des utilisateurs individuels dans le cadre d’opérations normales. Nous vous recommandons d’adopter une procédure testée, comportant la documentation relative aux problèmes connus et aux solutions de contournement associées.

Les sections suivantes détaillent les problèmes potentiels connus liés aux modifications d’UPN et leurs solutions de contournement.

## <a name="apps-known-issues-and-workarounds"></a>Problèmes connus liés aux applications et solutions de contournement associées

Les applications [SaaS (Software as a service)](https://azure.microsoft.com/overview/what-is-saas/) et métier s’appuient souvent sur les UPN pour rechercher des utilisateurs et stocker des informations de profil utilisateur, dont les rôles. Les applications qui utilisent le [provisionnement juste-à-temps](../app-provisioning/user-provisioning.md) pour créer un profil utilisateur lors de la première connexion d’un utilisateur à l’application peuvent être affectées par les modifications d’UPN.

**Problème connu**<br>
La modification de l’UPN d’un utilisateur peut rompre la relation entre l’utilisateur Azure AD et le profil utilisateur créé dans l’application. Si l’application utilise le [provisionnement juste-à-temps](../app-provisioning/user-provisioning.md), elle peut créer un nouveau profil utilisateur. L’administrateur de l’application devra apporter manuellement des modifications pour rétablir cette relation.

**Solution de contournement**<br>
L’[attribution automatique d’utilisateurs Azure AD](../app-provisioning/user-provisioning.md) vous permet de créer, gérer et supprimer automatiquement vos identités utilisateur dans les applications cloud prises en charge. La configuration de l’attribution automatique d’utilisateurs dans vos applications met automatiquement à jour les UPN dans les applications. Testez les applications dans le cadre du déploiement progressif pour bien vérifier qu’elles ne sont pas affectées par des modifications d’UPN.
Si vous êtes développeur, vous pouvez [ajouter la prise en charge de SCIM à votre application](../app-provisioning/use-scim-to-provision-users-and-groups.md) pour activer l'approvisionnement automatique des utilisateurs à partir d'Azure Active Directory. 

## <a name="managed-devices-known-issues-and-workarounds"></a>Problèmes connus liés aux appareils managés et solutions de contournement associées

En [intégrant vos appareils à Azure AD](../devices/overview.md), vous optimisez la productivité de vos utilisateurs avec l’authentification unique (SSO) sur vos ressources cloud et locales.

### <a name="azure-ad-joined-devices"></a>Appareils joints Azure AD

Les appareils [joints à Azure AD](../devices/concept-azure-ad-join.md) sont directement rattachés à Azure AD et permettent aux utilisateurs de se connecter à l’appareil au moyen de l’identité de leur organisation.

**Problèmes connus** <br>
Les utilisateurs peuvent rencontrer des problèmes d’authentification unique avec les applications qui dépendent d’Azure AD pour l’authentification.

**Résolution :** <br>
Les problèmes mentionnés dans cette section ont été résolus sur la mise à jour de Windows 10 mai 2020 (2004).

**Solution de contournement** <br>
Laissez suffisamment de temps pour que le changement d’UPN soit synchronisé sur Azure AD. Une fois que vous avez vérifié la prise en compte du nouvel UPN sur le portail Azure AD, demandez à l’utilisateur de sélectionner la vignette « Autre utilisateur » afin qu’il se connecte avec son nouvel UPN. Vous pouvez également procéder à la vérification à l'aide de [PowerShell](/powershell/module/azuread/get-azureaduser). Une fois l'utilisateur connecté avec son nouvel UPN, des références à l'ancien UPN peuvent encore apparaître sur le paramètre Windows « Accès professionnel ou scolaire ».

![Capture d’écran des domaines vérifiés](./media/howto-troubleshoot-upn-changes/other-user.png)


### <a name="hybrid-azure-ad-joined-devices"></a>Appareils joints Azure AD hybrides

Les appareils [joints à Azure AD Hybride](../devices/concept-azure-ad-join-hybrid.md) sont rattachés à Active Directory et Azure AD. Vous pouvez implémenter la jonction Azure AD Hybride si votre environnement comporte une empreinte locale Active Directory et que vous souhaitez également profiter des fonctionnalités proposées par Azure AD.

**Problèmes connus** 

Les appareils Windows 10 joints à Azure AD Hybride sont susceptibles d’être confrontés à des redémarrages inattendus et des problèmes d’accès.

Si l’accès conditionnel a été configuré afin d’imposer l’utilisation d’appareils joints Hybride pour accéder aux ressources, les utilisateurs qui se connectent à Windows avant que la synchronisation du nouvel UPN à Azure AD a été effectuée, ou qui continuent d’utiliser une session Windows existante, peuvent rencontrer des problèmes d’authentification unique avec les applications se servant d’Azure AD pour l’authentification. 

De plus, le message suivant s’affiche, obligeant un redémarrage au bout d’une minute. 

« Votre PC va redémarrer automatiquement dans une minute. Windows a rencontré un problème et doit redémarrer. Fermez ce message maintenant et enregistrez votre travail. »

**Résolution :** <br>
Les problèmes mentionnés dans cette section ont été résolus sur la mise à jour de Windows 10 mai 2020 (2004).

**Solution de contournement** 

L’appareil doit être disjoint depuis Azure AD, puis redémarré. Après le redémarrage, l’appareil est de nouveau automatiquement joint à Azure AD, et l’utilisateur doit se connecter à l’aide du nouvel UPN en sélectionnant la vignette « Autre utilisateur ». Pour disjoindre un appareil depuis Azure AD, exécutez la commande suivante à l’invite de commandes :

**dsregcmd /leave**

L’utilisateur doit se [réinscrire](/windows/security/identity-protection/hello-for-business/hello-hybrid-cert-whfb-provision) pour Windows Hello Entreprise si cette fonctionnalité est utilisée. Les appareils Windows 7 et 8.1 ne sont pas affectés par ce problème après des modifications d’UPN.


## <a name="microsoft-authenticator-known-issues-and-workarounds"></a>Problèmes connus liés à Microsoft Authenticator et solutions de contournement associées

Votre organisation peut demander l’utilisation de l’[application Microsoft Authenticator](../user-help/user-help-auth-app-overview.md) pour se connecter et accéder aux données et applications de l’organisation. Même si un nom d’utilisateur peut apparaître dans l’application, le compte n’est pas configuré pour fonctionner comme méthode de vérification tant que l’utilisateur n’a pas terminé le processus d’inscription.

L’[application Microsoft Authenticator](../user-help/user-help-auth-app-overview.md) comporte quatre fonctions principales :

* Authentification multifacteur par le biais d’une notification Push ou d’un code de vérification

* Action en tant que broker d’authentification sur les appareils iOS et Android afin de fournir une authentification unique pour les applications utilisant [l’authentification répartie](../develop/msal-android-single-sign-on.md)

* Inscription d'appareils (également appelée Workplace Join) auprès d'Azure AD, qui constitue une exigence pour d'autres fonctionnalités telles que Intune App Protection et Gestion/Inscription des appareils

* Connexion par téléphone, qui exige MFA et l’inscription d’appareils

### <a name="multi-factor-authentication-with-android-devices"></a>Multi-Factor Authentication avec les appareils Android

L’application Microsoft Authenticator offre une option de vérification hors bande. Au lieu de passer un appel téléphonique ou d’envoyer un SMS automatiquement à l’utilisateur au cours de la connexion, [Multi-Factor Authentication (MFA)](../authentication/concept-mfa-howitworks.md) envoie une notification à l’application Microsoft Authenticator sur le smartphone ou la tablette de l’utilisateur. L’utilisateur appuie simplement sur Approuver (ou entre un code confidentiel ou des informations biométriques, puis appuie sur « Authentifier ») dans l’application pour terminer sa connexion.

**Problèmes connus** 

Lorsque vous modifiez l'UPN d'un utilisateur, l'ancien UPN s'affiche toujours sur le compte de l'utilisateur et il arrive qu'aucune notification ne soit reçue. [Les codes de vérification](../user-help/user-help-auth-app-faq.md) continuent de fonctionner.

**Solution de contournement**

Si une notification est reçue, demandez à l’utilisateur d’ignorer la notification, d’ouvrir l’application Authenticator, d’appuyer sur l’option « Vérifier les notifications » et d’approuver l’invite MFA. À la suite de cela, l’UPN affiché dans le compte sera mis à jour. Notez que l’UPN mis à jour peut s’afficher sous la forme d’un nouveau compte en raison de l’utilisation d’une autre fonctionnalité d’authentification en cours. Pour plus d'informations, reportez-vous aux autres problèmes connus décrits dans cet article.

### <a name="brokered-authentication"></a>Authentification répartie

Sur Android et iOS, les brokers comme Microsoft Authenticator permettent les fonctionnalités suivantes :

* Authentification unique (SSO) - Vos utilisateurs n’ont pas besoin de se connecter à chaque application.

* Identification des appareils - Le broker accède au certificat de l’appareil, créé sur cet appareil lors de son rattachement à un espace de travail.

* Vérification de l’identification de l’application - Lorsqu’une application appelle le broker, elle transmet son URL de redirection et le broker la vérifie.

Par ailleurs, les applications peuvent participer à des fonctionnalités plus avancées, telles que l’[Accès conditionnel](../conditional-access/index.yml), et la prise en charge des [Scénarios Microsoft Intune](../develop/msal-net-use-brokers-with-xamarin-apps.md) est rendue possible.

**Problèmes connus**<br>
L’utilisateur reçoit plus d’invites d’authentification interactive sur les nouvelles applications utilisant la connexion avec assistance broker, en raison d’une incompatibilité entre le login_hint transmis par l’application et l’UPN stocké sur le broker.

**Solution de contournement** <br> L’utilisateur doit supprimer manuellement le compte depuis Microsoft Authenticator, et démarrer une nouvelle connexion à partir d’une application avec assistance répartiteur. Le compte sera automatiquement ajouté à l’issue de l’authentification initiale.

### <a name="device-registration"></a>Enregistrement de l’appareil

L’application Microsoft Authenticator est responsable de l’inscription de l’appareil auprès d’Azure AD. L’inscription de l’appareil permet à un appareil de s’authentifier auprès d’Azure AD, et constitue une condition exigée pour les scénarios suivants :

* Intune App Protection

* Inscription des appareils Intune

* Connexion par téléphone

**Problèmes connus**<br>
Lorsque vous modifiez l’UPN, un nouveau compte doté du nouvel UPN apparaît dans la liste de l’application Microsoft Authenticator alors que le compte avec l’ancien UPN est toujours listé. De plus, l’ancien UPN s’affiche dans la section Inscription de l’appareil, dans les paramètres de l’application. Il n’y a aucun changement noté dans le fonctionnement normal de l’option Inscription de l’appareil ni dans les scénarios dépendants.

**Solution de contournement** <br> Pour supprimer toutes les références à l’ancien UPN dans l’application Microsoft Authenticator, demandez à l’utilisateur de supprimer manuellement l’ancien et le nouveau compte à partir de Microsoft Authenticator, puis de se réinscrire auprès de MFA et de joindre de nouveau l’appareil.

### <a name="phone-sign-in"></a>Connexion par téléphone

La connexion par téléphone permet aux utilisateurs de se connecter à Azure AD sans mot de passe. Pour activer la connexion par téléphone, l’utilisateur doit s’inscrire à MFA à l’aide de l’application Authenticator, puis activer la connexion par téléphone directement sur Authenticator. Dans le cadre de la configuration, l’appareil s’inscrit auprès d’Azure AD.

**Problèmes connus** <br>
Les utilisateurs ne sont pas en mesure d’utiliser l’option connexion par téléphone, car ils ne reçoivent aucune notification. Si l’utilisateur appuie sur Vérifier les notifications, il obtient une erreur.

**Solution de contournement**<br>
L’utilisateur doit sélectionner le menu déroulant du compte activé pour la Connexion par téléphone, puis sélectionner Désactiver la connexion par téléphone. Si elle est souhaitée, il est possible d’activer de nouveau la Connexion par téléphone.

## <a name="security-key-fido2-known-issues-and-workarounds"></a>Problèmes connus liés à la clé de sécurité (FIDO2) et solutions de contournement associées

**Problèmes connus** <br>
Lorsque plusieurs utilisateurs sont inscrits sur la même clé, l’écran de connexion affiche une page de sélection de comptes dans laquelle l’ancien UPN s’affiche. Les connexions utilisant des clés de sécurité ne sont pas affectées par les modifications d’UPN.  

**Solution de contournement**<br>
Pour supprimer les références à d’anciens UPN, les utilisateurs doivent [réinitialiser la clé de sécurité et se réinscrire](../authentication/howto-authentication-passwordless-security-key.md#known-issues).

## <a name="onedrive-known-issues-and-workarounds"></a>Problèmes connus liés à OneDrive et solutions de contournement associées

Les utilisateurs OneDrive rencontrent des problèmes suite à des modifications d’UPN. Pour plus d’informations, consultez [Comment les modifications d’UPN affectent l’URL OneDrive et les fonctionnalités OneDrive](/onedrive/upn-changes).

## <a name="next-steps"></a>Étapes suivantes

Consultez ces ressources :
* [Azure AD Connect : Principes de conception](./plan-connect-design-concepts.md)

* [Remplissage de UserPrincipalName dans Azure AD](./plan-connect-userprincipalname.md)

* [Jetons d’ID de la plateforme d’identités Microsoft](../develop/id-tokens.md)
