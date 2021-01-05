---
title: Configurer les mots de passe d'application pour Azure AD Multi-Factor Authentication - Azure Active Directory
description: Apprenez à configurer et utiliser les mots de passe des applications héritées dans Azure AD Multi-Factor Authentication
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 06/05/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: dfb38f9fcdba6898b690d0af68b715fea07e80bb
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743103"
---
# <a name="enable-and-use-azure-ad-multi-factor-authentication-with-legacy-applications-using-app-passwords"></a>Activer et utiliser Azure AD Multi-Factor Authentication avec des applications héritées à l'aide de mots de passe d'application

Certaines applications plus anciennes qui n’utilisent pas de navigateur, comme Office 2010 ou une version antérieure et Apple Mail avant iOS 11, ne comprennent pas les pauses ou les interruptions dans le processus d’authentification. Si un utilisateur a activé Azure AD Multi-Factor Authentication et qu'il tente d'utiliser l'une de ces anciennes applications sans navigateur, il ne peut pas s'authentifier. Pour utiliser ces applications de façon sécurisée lorsqu'Azure AD Multi-Factor Authentication est activé pour les comptes d'utilisateur, vous pouvez utiliser des mots de passe d'application. Ces mots de passe d’application ont remplacé votre mot de passe traditionnel pour permettre à une application de contourner l’authentification multifacteur et de fonctionner correctement.

L’authentification moderne est prise en charge pour les clients Microsoft Office 2013 et versions ultérieures. Les clients Office 2013, y compris Outlook, prennent en charge des protocoles d’authentification moderne et peuvent être activés pour fonctionner dans le cadre de la vérification en deux étapes. Une fois le client activé, les mots de passe d’application ne sont pas requis par le client.

Cet article explique comment activer et utiliser les mots de passe d’application pour les applications héritées qui ne prennent pas en charge les invites d’authentification multifacteur.

>[!NOTE]
> Les mots de passe d’application ne fonctionnent pas avec les stratégies d’authentification multifacteur basées sur l’accès conditionnel ni avec l’authentification moderne.

## <a name="overview-and-considerations"></a>Vue d’ensemble et considérations

Lorsqu'Azure AD Multi-Factor Authentication est activé pour un compte d'utilisateur, l'invite de connexion standard est interrompue par une requête de vérification supplémentaire. Certaines applications plus anciennes ne comprennent pas cette interruption dans le processus de connexion, donc l’authentification échoue. Pour assurer la sécurité du compte d'utilisateur tout en maintenant Azure AD Multi-Factor Authentication activé, des mots de passe d'application peuvent être utilisés à la place du nom d'utilisateur et du mot de passe habituels. Lorsqu’un mot de passe d’application est utilisé lors de la connexion, il n’y a aucune invite de vérification supplémentaire pour que l’authentification soit réussie.

Les mots de passe d’application sont générés automatiquement, il ne sont pas spécifiés par l’utilisateur. Le mot de passe généré automatiquement est beaucoup plus difficile à pirater, il est donc bien mieux sécurisé. Les utilisateurs n’ont pas besoin d’effectuer le suivi des mots de passe ou de les entrer à chaque fois, car les mots de passe d’application ne sont entrés qu’une seule fois par application.

Lorsque vous utilisez des mots de passe d’application, tenez compte des considérations suivantes :

* Un utilisateur peut posséder jusqu’à 40 mots de passe d’application.
* Les applications qui mettent en cache les mots de passe et les utilisent dans les scénarios locaux peuvent se bloquer, car le mot de passe d’application n’est pas connu à l’extérieur du compte professionnel ou scolaire. Par exemple, les courriers électroniques Exchange sont, par exemple, stockés localement, mais la messagerie archivée se trouve dans le cloud. Ainsi, le même mot de passe ne peut pas fonctionner.
* Dès qu'Azure AD Multi-Factor Authentication est activé sur le compte d'un utilisateur, des mots de passe d'application peuvent être utilisés avec la plupart des clients sans navigateur, comme Outlook et Microsoft Skype Entreprise. Cependant, les actions d’administration ne peuvent pas être effectuées à l’aide de mots de passe d’application, dans des applications sans navigateur comme Windows PowerShell. Les actions sont impossibles, même quand l’utilisateur a un compte d’administrateur.
    * Pour exécuter des scripts PowerShell, créez un compte de service avec un mot de passe fort et n’activez pas la vérification en deux étapes sur ce compte.
* Si vous soupçonnez qu’un compte d’utilisateur est compromis et que vous révoquez/réinitialisez le mot de passe du compte, les mots de passe d’application doivent également être mis à jour. Les mots de passe d’application ne sont pas révoqués automatiquement lorsqu’un mot de passe de compte d’utilisateur est révoqué/réinitialisé. L’utilisateur doit supprimer les mots de passe d’application existants et en créer de nouveaux.
   * Pour plus d’informations, consultez [Créer et supprimer des mots de passe d’application à partir de la page Vérification de sécurité supplémentaire](../user-help/multi-factor-authentication-end-user-app-passwords.md#create-and-delete-app-passwords-from-the-additional-security-verification-page).

>[!WARNING]
> Les mots de passe d’application ne fonctionnent pas dans les environnements hybrides où les clients communiquent avec les points de terminaison locaux et les points de terminaison à découverte automatique cloud. Les mots de passe de domaine sont requis pour l’authentification locale. Les mots de passe d’application sont requis pour s’authentifier auprès du cloud.

### <a name="app-password-names"></a>Noms des mots de passe d’application

Les noms des mots de passe d’application doivent correspondre à l’appareil sur lequel ils sont utilisés. Si vous disposez d’un ordinateur portable qui contient des applications sans navigateur, comme Outlook, Word et Excel, créez un mot de passe d’application nommé **Ordinateur portable** pour ces applications. Créez un autre mot de passe d’application nommé **Bureau** pour les mêmes applications qui s’exécutent sur votre ordinateur de bureau.

Nous vous recommandons de créer un mot de passe d’application par appareil, au lieu d’un mot de passe d’application par application.

## <a name="federated-or-single-sign-on-app-passwords"></a>Mots de passe d’application pour authentification unique ou fédérée

Azure AD prend en charge la fédération, ou l’authentification unique (SSO) avec les services locaux Active Directory Domain Services (AD DS). Si votre organisation est fédérée à Azure AD et que vous utilisez Azure AD Multi-Factor Authentication, gardez en tête les points suivants sur les mots de passe d'application :

>[!NOTE]
> Les points suivants ne concernent que les clients fédérés (SSO).

* Les mots de passe d’application sont vérifiés par Azure AD et contournent ainsi la fédération. La fédération n’est utilisée activement que lorsque vous configurez des mots de passe d’application.
* Le fournisseur d’identité (IdP) n’est jamais contacté pour les utilisateurs fédérés (SSO), contrairement au flux passif. Les mots de passe d’application sont stockés dans le compte professionnel ou scolaire. Si un utilisateur quitte l’entreprise, ses informations sont transférées au compte professionnel ou scolaire à l’aide de **DirSync** en temps réel. La désactivation/suppression du compte peut prendre jusqu’à trois heures de synchronisation, retardant la désactivation/suppression du mot de passe d’application dans Azure AD.
* Les paramètres Microsoft Azure Access Control Service des clients locaux ne sont pas honorés par la fonctionnalité des mots de passe d’application.
* Aucune capacité locale de journalisation/d’audit de l’authentification n’est disponible pour les fonctionnalités de mots de passe d’application.

Certaines architectures avancées nécessitent une combinaison d’informations d’identification pour l’authentification multifacteur des clients. Ces informations d’identification peuvent inclure le nom d’utilisateur et les mots de passe d’un compte professionnel ou scolaire, ainsi que des mots de passe d’application. La configuration requise dépend de la façon dont l’authentification est effectuée. Pour les clients qui s’authentifient auprès d’une infrastructure locale, vous devez utiliser le nom d’utilisateur et le mot de passe d’un compte professionnel ou scolaire. Pour les clients qui s’authentifient auprès d’Azure AD, un mot de passe d’application est indispensable.

Par exemple, supposons que vous ayez l’architecture suivante :

* Votre instance locale d’Active Directory est fédérée à Azure AD.
* Vous utilisez Exchange Online.
* Vous utilisez Skype Entreprise localement.
* Vous utilisez Azure AD Multi-Factor Authentication.

Dans ce scénario, vous utilisez les informations d’identification suivantes :

* Pour vous connecter à Skype Entreprise, utilisez le nom d’utilisateur et le mot de passe de votre compte professionnel ou scolaire.
* Pour accéder au carnet d’adresses depuis un client Outlook qui se connecte à Exchange Online, utilisez un mot de passe d’application.

## <a name="allow-users-to-create-app-passwords"></a>Permettre aux utilisateurs de créer des mots de passe d’application

Par défaut, les utilisateurs ne peuvent pas créer des mots de passe d’application. La fonctionnalité des mots de passe d’application doit être activée avant qu’un utilisateur ne puisse les utiliser. Pour permettre aux utilisateurs de créer des mots de passe d’application, suivez ces étapes :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
2. Recherchez et sélectionnez **Azure Active Directory**, puis choisissez **Utilisateurs**.
3. Sélectionnez **Multi-Factor Authentication** depuis la barre de navigation qui se trouve en haut de la fenêtre des *Utilisateurs*.
4. Sous Multi-Factor Authentication, sélectionnez **Paramètres du service**.
5. Sur la page **Paramètres du service**, sélectionnez l’option **Autoriser les utilisateurs à créer des mots de passe d’application pour se connecter à des applications sans navigateur**.

    ![Capture d’écran du Portail Azure qui affiche les paramètres de service pour l’authentification multifacteur afin d’autoriser l’utilisateur des mots de passe d’application](media/concept-authentication-methods/app-password-authentication-method.png)
    
> [!NOTE]
>
> Lorsque vous désactivez la possibilité pour les utilisateurs de créer des mots de passe d’application, les mots de passe d’application existants continuent de fonctionner. Toutefois, les utilisateurs ne peuvent pas gérer ou supprimer ces mots de passe lorsque vous désactivez cette fonctionnalité.
>
> Lorsque vous désactivez la possibilité de créer des mots de passe d’application, il est également recommandé de [créer une stratégie d’accès conditionnel pour désactiver l’utilisation de l’authentification héritée](../conditional-access/block-legacy-authentication.md). Cette approche empêche le fonctionnement des mots de passe d’application existants et force l’utilisation de méthodes d’authentification modernes.

## <a name="create-an-app-password"></a>Créer un mot de passe d’application

Lorsque les utilisateurs procèdent à leur inscription initiale à Azure AD Multi-Factor Authentication, vous avez la possibilité de créer des mots de passe d'application à la fin du processus d'inscription.

Les utilisateurs peuvent aussi créer des mots de passe d’application après l’inscription. Pour plus d'informations et pour connaître les étapes détaillées pour vos utilisateurs, consultez [Que sont les mots de passe d'application dans Azure AD Multi-Factor Authentication ?](../user-help/multi-factor-authentication-end-user-app-passwords.md)

## <a name="next-steps"></a>Étapes suivantes

Pour permettre aux utilisateurs de s'inscrire rapidement à Azure AD Multi-Factor Authentication, consultez [Présentation de l'inscription combinée des informations de sécurité](concept-registration-mfa-sspr-combined.md).
