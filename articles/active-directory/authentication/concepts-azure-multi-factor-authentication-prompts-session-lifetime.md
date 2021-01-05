---
title: Invites et durée de vie des sessions Azure AD Multi-Factor Authentication
description: Familiarisez-vous avec la configuration recommandée pour les invites de réauthentification auprès d'Azure AD Multi-Factor Authentication et sur la façon dont s'applique la durée de vie des sessions.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: inbarc
ms.collection: M365-identity-device-management
ms.openlocfilehash: 14a65b83195bff48934668ff029f925e13884bad
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743698"
---
# <a name="optimize-reauthentication-prompts-and-understand-session-lifetime-for-azure-ad-multi-factor-authentication"></a>Optimiser les invites de réauthentification et comprendre le fonctionnement de la durée de vie des sessions Azure AD Multi-Factor Authentication

Azure AD (Azure Active Directory) utilise plusieurs paramètres qui déterminent la fréquence à laquelle les utilisateurs doivent se réauthentifier. Cette réauthentification peut se faire avec un premier facteur de type mot de passe, FIDO ou Microsoft Authenticator sans mot de passe, ou en utilisant l’authentification multifacteur (MFA). Vous pouvez configurer ces paramètres de réauthentification en fonction des exigences de votre environnement et de l’expérience utilisateur que vous souhaitez offrir.

La configuration par défaut d’Azure AD pour la fréquence de connexion utilisateur est une fenêtre dynamique de 90 jours. Demander aux utilisateurs d’entrer des informations d’identification semble souvent être une chose à faire, mais cela peut avoir l’effet inverse. Si les utilisateurs sont formés pour entrer leurs informations d’identification sans les penser, ils peuvent les fournir involontairement à une invite d’informations d’identification malveillante.

Il peut paraître alarmant de ne pas demander à un utilisateur de se reconnecter. Cependant, toute violation des stratégies informatiques révoque la session. Certains exemples incluent une modification de mot de passe, un appareil non conforme ou une opération de désactivation de compte. Vous pouvez aussi explicitement [révoquer les sessions des utilisateurs avec PowerShell](/powershell/module/azuread/revoke-azureaduserallrefreshtoken).

Cet article explique en détail les configurations recommandées ainsi que la manière dont les différents paramètres fonctionnent et interagissent les uns avec les autres.

## <a name="recommended-settings"></a>Paramètres recommandés

Pour offrir à vos utilisateurs le meilleur compromis entre sécurité et simplicité d’utilisation en leur demandant de s’authentifier à une fréquence adaptée, nous vous recommandons les configurations suivantes :

* Si vous avez Azure AD Premium :
    * Activez l’authentification unique (SSO) entre les applications en utilisant des [appareils managés](../devices/overview.md) ou l’[authentification unique transparente](../hybrid/how-to-connect-sso.md).
    * Si la réauthentification est requise, utilisez une [stratégie d’accès conditionnel pour la fréquence de connexion](../conditional-access/howto-conditional-access-session-lifetime.md).
    * Pour les utilisateurs qui se connectent à partir d’appareils non managés ou d’appareils mobiles, utilisez l’accès conditionnel pour appliquer des stratégies de fréquence de connexion et de session de navigateur persistante.
* Si vous avez des licences Applications Microsoft 365 ou Azure AD Free :
    * Activez l’authentification unique (SSO) entre les applications en utilisant des [appareils managés](../devices/overview.md) ou l’[authentification unique transparente](../hybrid/how-to-connect-sso.md).
    * Gardez l’option *Rester connecté* activée et recommandez à vos utilisateurs de l’accepter.
* Pour les scénarios d’appareils mobiles, assurez-vous que vos utilisateurs utilisent l’application Microsoft Authenticator. Cette application est utilisée en tant que répartiteur pour d’autres applications Azure AD fédérées, et réduit les invites d’authentification sur l’appareil.

Les études que nous avons menées montrent que ces paramètres conviennent pour la plupart des locataires. Certaines combinaisons de ces paramètres, telles que *Mémoriser Multi-Factor Authentication* associé à *Rester connecté*, peuvent rendre les invites d’authentification des utilisateurs trop fréquentes. Les invites de réauthentification continuelles impactent la productivité des utilisateurs et risquent de les rendre plus vulnérables aux attaques.

## <a name="azure-ad-session-lifetime-configuration-settings"></a>Paramètres de configuration de la durée de vie des sessions Azure AD

Pour optimiser la fréquence des invites d’authentification envoyées aux utilisateurs, vous pouvez configurer les paramètres de la durée de vie des sessions Azure AD. Déterminez les besoins de votre entreprise et de vos utilisateurs, et configurez les paramètres qui offrent le meilleur compromis pour votre environnement.

### <a name="evaluate-session-lifetime-policies"></a>Évaluer les stratégies de durée de vie des sessions

Sans paramètres définis pour la durée de vie des sessions, il n’y a pas de cookies persistants dans une session de navigateur. Chaque fois qu’un utilisateur ferme et rouvre le navigateur, il reçoit une invite de réauthentification. Dans les clients Office, la période par défaut est de 90 jours consécutifs. Avec cette configuration Office par défaut, si l’utilisateur a réinitialisé son mot de passe ou s’il n’y a pas eu d’activité depuis plus de 90 jours, l’utilisateur est invité à se réauthentifier avec tous les facteurs requis (premier et deuxième facteurs).

Un utilisateur peut voir plusieurs invites d’authentification multifacteur sur un appareil qui n’a pas d’identité dans Azure AD. Des invites multiples se produisent quand chaque application a son propre jeton d’actualisation OAuth qui n’est pas partagé avec d’autres applications clientes. Dans ce scénario, l’authentification multifacteur plusieurs invites, car chaque application demande un jeton d’actualisation OAuth à valider avec l’authentification multifacteur.

Dans Azure AD, la stratégie la plus restrictive pour la durée de vie des sessions détermine à quel moment l’utilisateur doit se réauthentifier. Examinez le cas suivant :

* Vous activez l’option *Rester connecté*, qui utilise un cookie de navigateur persistant, et
* Vous activez également *Mémoriser Multi-Factor Authentication pendant 14 jours*

Dans cet exemple de scénario, l’utilisateur doit se réauthentifier tous les 14 jours. Ce comportement suit la stratégie la plus restrictive, même si l’option *Rester connecté* n’exige pas en principe que l’utilisateur se réauthentifie dans le navigateur.

### <a name="managed-devices"></a>Appareils gérés

Les appareils joints à Azure AD à l’aide d’une jonction Azure AD ou d’une jonction Azure AD Hybride reçoivent des [jetons d’actualisation principaux (PRT)](../devices/concept-primary-refresh-token.md) pour utiliser l’authentification unique (SSO) entre les applications. Ce jeton d’actualisation principal permet à un utilisateur de s’authentifier une seule fois sur l’appareil, tout en donnant la garantie à l’équipe informatique que les exigences de sécurité et de conformité sont respectées. Si un utilisateur doit être invité à s’authentifier plus fréquemment sur un appareil joint dans certaines applications ou certains scénarios, cela est possible en configurant la [fréquence de connexion de l’accès conditionnel](../conditional-access/howto-conditional-access-session-lifetime.md).

### <a name="show-option-to-remain-signed-in"></a>Afficher l’option permettant de rester connecté

Quand un utilisateur sélectionne **Oui** dans l’option *Rester connecté ?* au moment de la connexion, un cookie persistant est défini dans le navigateur. Ce cookie persistant mémorise le premier et le deuxième facteurs, et il sert uniquement aux demandes d’authentification dans le navigateur.

![Capture d’écran d’un exemple d’invite pour rester connecté](./media/concepts-azure-multi-factor-authentication-prompts-session-lifetime/stay-signed-in-prompt.png)

Si vous disposez d’une licence Azure AD Premium 1, nous vous recommandons d’utiliser la stratégie d’accès conditionnel configurée pour la *session de navigateur persistante*. Cette stratégie prévaut sur le paramètre *Rester connecté ?* et améliore l’expérience utilisateur. Si vous n’avez pas de licence Azure AD Premium 1, nous vous recommandons d’activer le paramètre Rester connecté pour vos utilisateurs.

Pour plus d’informations sur la configuration de l’option permettant aux utilisateurs de rester connectés, consultez [Personnaliser votre page de connexion Azure AD](../fundamentals/customize-branding.md#customize-your-azure-ad-sign-in-page).

### <a name="remember-multi-factor-authentication"></a>Mémoriser Multi-Factor Authentication  

Ce paramètre vous permet de configurer des valeurs comprises entre 1 et 365 jours, et définit un cookie persistant dans le navigateur quand un utilisateur sélectionne l’option **Ne plus me le demander pendant X jours** au moment de la connexion.

![Capture d’écran d’un exemple d’invite pour accepter une demande de connexion](./media/concepts-azure-multi-factor-authentication-prompts-session-lifetime/approve-sign-in-request.png)

Ce paramètre réduit le nombre d’authentifications dans les applications web, mais il l’augmente dans les clients d’authentification modernes, tels que les clients Office. Normalement, ces clients envoient une invite uniquement après la réinitialisation du mot de passe ou après 90 jours d’inactivité. Toutefois, la définition de cette valeur sur une valeur inférieure à 90 jours raccourcit les invites MFA par défaut pour les clients Office, et augmente la fréquence de réauthentification. Quand ce paramètre est combiné avec le paramètre **Rester connecté** ou des stratégies d’accès conditionnel, les invites d’authentification peuvent devenir plus fréquentes.

Si vous utilisez *Mémoriser Multi-Factor Authentication* et que vous avez des licences Azure AD Premium 1, prévoyez de migrer ces paramètres vers un accès conditionnel pour la fréquence de connexion. Sinon, choisissez l’option *Rester connecté ?* à la place.

Pour plus d’informations, consultez [Mémoriser Multi-Factor Authentication](howto-mfa-mfasettings.md#remember-multi-factor-authentication).

### <a name="authentication-session-management-with-conditional-access"></a>Gestion de session d’authentification avec l’accès conditionnel

Avec la stratégie **Fréquence de connexion**, l’administrateur peut choisir la fréquence de connexion à appliquer pour le premier et le deuxième facteurs à la fois dans le client et le navigateur. Nous vous recommandons d’utiliser ces paramètres, conjointement à l’utilisation d’appareils managés, dans les scénarios où vous devez restreindre la session d’authentification, par exemple pour les applications métier critiques.

Avec la stratégie **Session de navigateur persistante**, les utilisateurs restent connectés quand ils ferment et rouvrent la fenêtre de leur navigateur. Comme avec le paramètre *Rester connecté*, un cookie persistant est défini dans le navigateur. Toutefois, étant donné qu’il est configuré par l’administrateur, il n’exige pas que l’utilisateur sélectionne **Oui** dans l’option *Rester connecté ?* , ce qui améliore l’expérience utilisateur. Si vous utilisez l’option *Rester connecté ?* , nous vous recommandons d’activer à la place la stratégie **Session de navigateur persistante**.

Pour plus d’informations, consultez [Configurer la gestion de session d’authentification avec l’accès conditionnel](../conditional-access/howto-conditional-access-session-lifetime.md).

### <a name="configurable-token-lifetimes"></a>Durées de vie des jetons configurables

Ce paramètre permet de configurer la durée de vie des jetons émis par Azure Active Directory. Cette stratégie est remplacée par la *gestion de session d’authentification avec l’accès conditionnel*. Si vous utilisez l’option *Durées de vie des jetons configurables*, nous vous recommandons de commencer la migration vers des stratégies d’accès conditionnel.

## <a name="review-your-tenant-configuration"></a>Vérifier la configuration de vos locataires  

Maintenant que vous savez comment fonctionnent les différents paramètres et que vous connaissez la configuration recommandée, il est temps de vérifier la configuration de vos locataires et d’apporter les modifications nécessaires :

Pour configurer ou vérifier l’option *Rester connecté*, effectuez les étapes suivantes :

1. Dans le portail Azure AD, recherchez et sélectionnez *Azure Active Directory*.
1. Sélectionnez **Marque de société** et, pour chaque paramètre régional, choisissez **Afficher l’option permettant de rester connecté**.
1. Choisissez *Oui*, puis sélectionnez **Enregistrer**.

Pour mémoriser les paramètres Multi-Factor Authentication, effectuez les étapes suivantes :

1. Dans le portail Azure AD, recherchez et sélectionnez *Azure Active Directory*.
1. Sélectionnez **Sécurité**, puis **MFA**.
1. Sous **Configurer**, sélectionnez **Paramètres de MFA basé dans le cloud supplémentaires**.
1. Dans la page des *paramètres du service Azure Multi-Factor Authentication*, faites défiler jusqu’aux **paramètres de mémorisation Multi-Factor Authentication**. Désactivez le paramètre en décochant la case.

Pour configurer des stratégies d’accès conditionnel pour la fréquence de connexion et la session de navigateur persistante, effectuez les étapes suivantes :

1. Dans le portail Azure AD, recherchez et sélectionnez *Azure Active Directory*.
1. Sélectionnez **Sécurité**, puis **Accès conditionnel**.
1. Configurez une stratégie avec les options de gestion de session recommandées qui sont décrites dans cet article.

Pour vérifier les durées de vie des jetons, [utilisez Azure AD PowerShell pour rechercher les stratégies Azure AD existantes](../develop/configure-token-lifetimes.md#prerequisites). Désactivez les stratégies qui sont appliquées.

Si plusieurs paramètres sont activés dans votre locataire, nous vous recommandons d’effectuer leur mise à jour en fonction des licences que vous avez. Par exemple, si vous avez des licences Azure AD Premium, vous devez utiliser uniquement les stratégies d’accès conditionnel *Fréquence de connexion* et *Session de navigateur persistante*. Si vous avez des licences Applications Microsoft 365 ou Azure AD Free, vous devez choisir la configuration *Rester connecté ?* .

Si vous avez activé les durées de vie des jetons configurables, cette fonctionnalité sera bientôt supprimée. Prévoyez une migration vers une stratégie d’accès conditionnel.

Le tableau suivant récapitule les recommandations par licence :

|              | Applications Azure AD Free et Microsoft 365 | Azure AD Premium |
|------------------------------|-----------------------------------|------------------|
| **SSO**                      | [Jonction Azure AD](../devices/concept-azure-ad-join.md) ou [Jonction Azure AD Hybride](../devices/concept-azure-ad-join-hybrid.md), ou [Authentification unique transparente](../hybrid/how-to-connect-sso.md) pour les appareils non managés. | Jonction Azure AD<br />Jonction Azure AD hybride |
| **Paramètres de réauthentification** | Rester connecté                  | Utiliser des stratégies d’accès conditionnel pour la fréquence de connexion et la session de navigateur persistante |

## <a name="next-steps"></a>Étapes suivantes

Pour commencer, suivez le tutoriel [Sécuriser les événements de connexion utilisateur avec Azure AD Multi-Factor Authentication](tutorial-enable-azure-mfa.md) ou [Utiliser les détections de risques liés aux connexions utilisateur pour déclencher Azure AD Multi-Factor Authentication](tutorial-risk-based-sspr-mfa.md).
