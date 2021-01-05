---
title: Connexion utilisateur par SMS pour Azure Active Directory
description: Découvrir comment configurer la fonctionnalité qui permet aux utilisateurs de se connecter à Azure Active Directory par SMS (préversion)
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 10/05/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: rateller
ms.collection: M365-identity-device-management
ms.openlocfilehash: 10bac65fa8b1ed192e2ece1682f22e7feb528431
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743341"
---
# <a name="configure-and-enable-users-for-sms-based-authentication-using-azure-active-directory-preview"></a>Configurer l’authentification utilisateur par SMS via Azure Active Directory (préversion)

Pour réduire la complexité et les risques de sécurité auxquels sont confrontés les utilisateurs qui se connectent à des applications et des services, Azure AD (Azure Active Directory) fournit plusieurs options d’authentification. L’authentification par SMS, en préversion, permet aux utilisateurs de se connecter sans avoir à fournir, ni même connaître, leur nom d’utilisateur et leur mot de passe. Une fois qu’un administrateur d’identité a créé son compte, il peut entrer son numéro de téléphone à l’invite de connexion et fournir un code d’authentification qui lui est envoyé par SMS. Cette méthode d’authentification simplifie l’accès aux applications et aux services, en particulier pour les travailleurs qui sont en première ligne.

Cet article vous montre comment activer l’authentification par SMS pour certains utilisateurs ou groupes dans Azure AD.

> [!NOTE]
> L’authentification par SMS des utilisateurs est une fonctionnalité en préversion publique d’Azure Active Directory. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="before-you-begin"></a>Avant de commencer

Pour faire ce qui est décrit dans cet article, vous avez besoin des ressources et des privilèges suivants :

* Un abonnement Azure actif.
    * Si vous n’avez pas d’abonnement Azure, [créez un compte](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un locataire Azure Active Directory associé à votre abonnement.
    * Si nécessaire, [créez un locataire Azure Active Directory][create-azure-ad-tenant] ou [associez un abonnement Azure à votre compte][associate-azure-ad-tenant].
* Vous devez disposer de privilèges d’*administrateur général* dans votre locataire Azure AD pour activer l’authentification par SMS.
* Chaque utilisateur activé dans la stratégie de méthode d’authentification par SMS doit disposer d’une licence, même s’il ne l’utilise pas. Chaque utilisateur activé doit avoir l’une des licences Azure AD, EMS ou Microsoft 365 suivantes :
    * [Azure AD Premium P1 ou P2][azuread-licensing]
    * [Microsoft 365 (M365) F1 ou F3][m365-firstline-workers-licensing]
    * [Enterprise Mobility + Security (EMS) E3 ou E5][ems-licensing] ou [Microsoft 365 (M365) E3 ou E5][m365-licensing]

## <a name="limitations"></a>Limites

Pendant la phase de préversion publique de l’authentification par SMS, les limitations suivantes s’appliquent :

* L'authentification par SMS n'est pas compatible avec Azure AD Multi-Factor Authentication.
* À l’exception de Teams, l’authentification par SMS n’est pas compatible avec les applications Office natives.
* L’authentification par SMS n’est pas recommandée pour les comptes B2B.
* Les utilisateurs fédérés ne s’authentifient pas dans le locataire de base. Ils s’authentifient uniquement dans le cloud.

## <a name="enable-the-sms-based-authentication-method"></a>Activer la méthode d’authentification par SMS

Il existe trois étapes principales pour activer et utiliser l’authentification par SMS dans votre organisation :

* Activez la stratégie de méthode d’authentification.
* Sélectionnez les utilisateurs ou les groupes qui peuvent utiliser la méthode d’authentification basée sur SMS.
* Affectez un numéro de téléphone à chaque compte d’utilisateur.
    * Ce numéro de téléphone peut être affecté dans le portail Azure (illustré dans cet article) ainsi que dans *Mon personnel* ou *Mon profil*.

Tout d’abord, activons l’authentification par SMS pour votre locataire Azure AD.

1. Connectez-vous au [portail Azure][azure-portal] en tant qu’*administrateur général*.
1. Recherchez et sélectionnez **Azure Active Directory**.
1. Dans le menu de navigation sur le côté gauche de la fenêtre Azure Active Directory, sélectionnez **Sécurité > Méthodes d’authentification > Stratégie de méthode d’authentification (préversion)** .

    [![Browse to and select the Authentication method policy (preview) window in the Azure portal.](media/howto-authentication-sms-signin/authentication-method-policy-cropped.png)](media/howto-authentication-sms-signin/authentication-method-policy.png#lightbox)

1. Dans la liste des méthodes d’authentification disponibles, sélectionnez **SMS**.
1. Affectez à **Activer** la valeur *Oui*.

    ![Activer l’authentification par SMS dans la fenêtre de stratégie de méthode d’authentification](./media/howto-authentication-sms-signin/enable-text-authentication-method.png)

    Vous avez le choix entre activer l’authentification par SMS pour *Tous les utilisateurs* ou *Sélectionner des utilisateurs et des groupes*. Dans la section suivante, vous activez l’authentification par SMS pour un utilisateur de test.

## <a name="assign-the-authentication-method-to-users-and-groups"></a>Affecter la méthode d’authentification aux utilisateurs et aux groupes

Une fois l’authentification par SMS activée dans votre locataire Azure AD, sélectionnez les utilisateurs ou les groupes autorisés à utiliser cette méthode d’authentification.

1. Dans la fenêtre de stratégie d’authentification par SMS, affectez à **Cible** la valeur *Sélectionner les utilisateurs*.
1. Choisissez d’**ajouter des utilisateurs ou des groupes**, puis sélectionnez un utilisateur ou un groupe de test, par exemple *Contoso User* (Utilisateur Contoso) ou *Contoso SMS Users* (Utilisateurs Contoso - SMS).

    [![Choose users or groups to enable for SMS-based authentication in the Azure portal.](media/howto-authentication-sms-signin/add-users-or-groups-cropped.png)](media/howto-authentication-sms-signin/add-users-or-groups.png#lightbox)

1. Une fois que vous avez sélectionné vos utilisateurs ou vos groupes, vous devez choisir **Sélectionner**, puis **Enregistrer** la stratégie de méthode d’authentification mise à jour.

Chaque utilisateur activé dans la stratégie de méthode d’authentification par SMS doit disposer d’une licence, même s’il ne l’utilise pas. Vérifiez que vous disposez des licences appropriées pour les utilisateurs que vous activez dans la stratégie de méthode d’authentification, en particulier quand vous activez la fonctionnalité dans le cadre de grands groupes d’utilisateurs.

## <a name="set-a-phone-number-for-user-accounts"></a>Définir un numéro de téléphone pour les comptes d’utilisateur

Désormais, l’authentification par SMS est activée pour les utilisateurs. Toutefois, vous devez associer leur numéro de téléphone au profil utilisateur dans Azure AD pour leur permettre de se connecter. Vous pouvez soit laisser l’utilisateur [définir lui-même ce numéro de téléphone](../user-help/sms-sign-in-explainer.md) dans *Mon profil*, soit affecter le numéro de téléphone via le portail Azure. Les numéros de téléphone peuvent être définis par les *administrateurs généraux*, les *administrateurs d’authentification* ou les *administrateurs d’authentification privilégiés*.

Lorsqu'un numéro de téléphone est défini pour l'authentification par SMS, il peut également être utilisé avec [Azure AD Multi-Factor Authentication][tutorial-azure-mfa] et la [réinitialisation de mot de passe en libre-service][tutorial-sspr].

1. Recherchez et sélectionnez **Azure Active Directory**.
1. Dans le menu de navigation sur le côté gauche de la fenêtre Azure Active Directory, sélectionnez **Utilisateurs**.
1. Sélectionnez l’utilisateur pour lequel vous avez activé l’authentification par SMS dans la section précédente, par exemple *Contoso User*, puis sélectionnez **Méthodes d’authentification**.
1. Sélectionnez **+ Ajouter une méthode d’authentification**, puis, dans le menu déroulant *Choisir une méthode*, sélectionnez **Numéro de téléphone**.

    Entrez le numéro de téléphone de l’utilisateur, notamment l’indicatif téléphonique international, par exemple *+1 xxxxxxxxx*. Le portail Azure vérifie que le numéro de téléphone est au format approprié.

    Ensuite, dans le menu déroulant *Type de téléphone*, sélectionnez *Mobile*, *Autre mobile* ou *Autre* selon les besoins.

    :::image type="content" source="media/howto-authentication-sms-signin/set-user-phone-number.png" alt-text="Définir le numéro de téléphone d’un utilisateur dans le portail Azure pour l’utiliser avec l’authentification par SMS":::

    Le numéro de téléphone doit être unique dans votre locataire. Si vous essayez d’utiliser le même numéro de téléphone pour plusieurs utilisateurs, un message d’erreur s’affiche.

1. Pour appliquer le numéro de téléphone au compte d’un utilisateur, sélectionnez **Ajouter**.

Une fois le provisionnement réussi, une coche apparaît pour *Connexion par SMS activée*.

## <a name="test-sms-based-sign-in"></a>Tester la connexion par SMS

Si vous souhaitez tester le compte d’utilisateur pour lequel la connexion par SMS est désormais activée, effectuez les étapes suivantes :

1. Ouvrez une nouvelle fenêtre de navigateur web InPrivate ou en mode privé pour [https://www.office.com][office]
1. Dans le coin supérieur droit, sélectionnez **Connexion**.
1. À l’invite de connexion, entrez le numéro de téléphone associé à l’utilisateur dans la section précédente, puis sélectionnez **Suivant**.

    ![Entrez un numéro de téléphone à l’invite de connexion pour l’utilisateur de test](./media/howto-authentication-sms-signin/sign-in-with-phone-number.png)

1. Un SMS est envoyé au numéro de téléphone fourni. Pour effectuer le processus de connexion, entrez le code à 6 chiffres fourni dans le SMS à l’invite de connexion.

    ![Entrez le code de confirmation envoyé par SMS au numéro de téléphone de l’utilisateur](./media/howto-authentication-sms-signin/sign-in-with-phone-number-confirmation-code.png)

1. L’utilisateur est désormais connecté sans avoir à fournir un nom d’utilisateur ou un mot de passe.

## <a name="troubleshoot-sms-based-sign-in"></a>Résoudre les problèmes de connexion par SMS

Vous pouvez utiliser les scénarios et étapes de résolution des problèmes suivants si vous rencontrez des difficultés pour activer et utiliser la connexion par SMS.

### <a name="phone-number-already-set-for-a-user-account"></a>Numéro de téléphone déjà défini pour un compte d’utilisateur

Si un utilisateur s'est déjà inscrit à Azure AD Multi-Factor Authentication et/ou à la réinitialisation de mot de passe en libre-service (SSPR), un numéro de téléphone est déjà associé à son compte. Ce numéro de téléphone n’est pas automatiquement disponible pour une connexion par SMS.

Un utilisateur dont le numéro de téléphone est déjà défini pour son compte voit s’afficher un bouton permettant d’*activer la connexion par SMS* dans la page **Mon profil**. Sélectionnez ce bouton afin d'activer la connexion par SMS ainsi que l'inscription à Azure AD Multi-Factor Authentication ou SSPR pour le compte.

Pour plus d’informations du point de vue de l’utilisateur final, consultez l’article sur l’[expérience utilisateur de connexion par SMS à l’aide d’un numéro de téléphone (préversion)](../user-help/sms-sign-in-explainer.md).

### <a name="error-when-trying-to-set-a-phone-number-on-a-users-account"></a>Erreur durant la tentative de définition d’un numéro de téléphone pour le compte d’un utilisateur

Si vous recevez une erreur quand vous essayez de définir un numéro de téléphone pour un compte d’utilisateur dans le portail Azure, passez en revue les étapes de résolution des problèmes suivantes :

1. Vérifiez que la préversion de la fonctionnalité de connexion par SMS est activée pour votre compte.
1. Vérifiez que la stratégie de méthode d’authentification *SMS* est activée pour le compte d’utilisateur.
1. Veillez à définir le numéro de téléphone au format approprié, tel que validé dans le portail Azure (par exemple *+1 4251234567*).
1. Vérifiez que le numéro de téléphone n’est pas utilisé ailleurs dans votre locataire.
1. Vérifiez qu’aucun numéro vocal n’est défini pour le compte. Si un numéro vocal est défini, supprimez-le, puis réessayez d’utiliser le numéro de téléphone.

## <a name="next-steps"></a>Étapes suivantes

Pour connaître d’autres façons de se connecter à Azure AD sans mot de passe, par exemple à l’aide de l’application Microsoft Authenticator ou des clés de sécurité FIDO2, consultez [Options d’authentification sans mot de passe pour Azure AD][concepts-passwordless].

Vous pouvez également utiliser la version bêta de l’API REST Microsoft Graph pour [activer][rest-enable] ou [désactiver][rest-disable] la connexion par SMS.

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../fundamentals/active-directory-how-subscriptions-associated-directory.md
[concepts-passwordless]: concept-authentication-passwordless.md
[tutorial-azure-mfa]: tutorial-enable-azure-mfa.md
[tutorial-sspr]: tutorial-enable-sspr.md
[rest-enable]: /graph/api/phoneauthenticationmethod-enablesmssignin?view=graph-rest-beta&tabs=http
[rest-disable]: /graph/api/phoneauthenticationmethod-disablesmssignin?view=graph-rest-beta&tabs=http

<!-- EXTERNAL LINKS -->
[azure-portal]: https://portal.azure.com
[office]: https://www.office.com
[m365-firstline-workers-licensing]: https://www.microsoft.com/licensing/news/m365-firstline-workers
[azuread-licensing]: https://azure.microsoft.com/pricing/details/active-directory/
[ems-licensing]: https://www.microsoft.com/microsoft-365/enterprise-mobility-security/compare-plans-and-pricing
[m365-licensing]: https://www.microsoft.com/microsoft-365/compare-microsoft-365-enterprise-plans
[o365-f1]: https://www.microsoft.com/microsoft-365/business/office-365-f1?market=af
[o365-f3]: https://www.microsoft.com/microsoft-365/business/office-365-f3?activetab=pivot%3aoverviewtab
